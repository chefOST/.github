# ChefOST evaluation

Use frame-averaged precision, recall (detail), cosine similarity and state-change timing F1. These definitions govern shared comparisons; a paper's metrics or an implementation's outputs do not redefine them.

All evaluation code, adapters, metric tests and scoring workflows live in [chefOST-data](https://github.com/chefOST/chefOST-data). Other repositories and their branches export raw predictions and link to this evaluator; they must not maintain or launch separate scoring implementations.

## Scope and timelines

Evaluate the videos available in the [shared R2 release](https://github.com/chefOST/chefOST-data#videos), using manually annotated evaluation ground truth and vocabularies from `chefOST-data/main`. Evaluate ingredients only, with ingredient identities and representations taken from the ground-truth annotations. Keep training pseudo-labels separate from evaluation ground truth. Record the R2 manifest, evaluated video/ingredient list, and annotation/vocabulary revisions with the run.

For each ingredient, the outer evaluation bounds are the first and last source frames in which ground truth identifies it. Score every ground-truth frame within those bounds. Frames with unknown or empty ground-truth state annotations are irrelevant: predictions there receive neither reward nor penalty. Do not infer valid negative labels from an annotation gap.

Each model window's output is a complete replacement state set, not an incremental update to the preceding set. Project that set onto the ground-truth frames covered by its window using verified source-frame timestamps. Record window boundaries and alignment; differing input/reporting rates do not change the every-frame scoring grid.

For VidOSC, broadcast the same native prediction stream to every annotated ingredient in the video and report all ingredient scores. Do not select a best-scoring ingredient or use GT to assign the stream. This measures the broadcast adaptation's limitations; a high score for one ingredient does not establish which ingredient the model intended to track.

Unavailable initial state estimates are excluded neutrally. Declare and report the initial unavailable interval from the native output/initialization support before thresholding or mapping. An empty set produced by a threshold, an unmapped label, or a later abstention does not extend that initial exclusion. Missing predictions after the initial exclusion receive zero recall on frames with known ground truth. Report excluded support and undefined scores rather than silently treating them as correct predictions.

## Probability threshold sweeps

Evaluate both MOSP (the MOSCATO model) and VidOSC at `tau = 0.25, 0.5, 0.75`. At each threshold, select every native state candidate whose score satisfies `p(s) >= tau`. Threshold before vocabulary mapping, then deduplicate mapped canonical states. Reuse the same selected candidates for both precision/recall mapping variants below. Do not force a top-one prediction when no state passes.

MOSP supplies independent sigmoid state scores; VidOSC supplies softmax scores over competing labels. The sweep applies to both, but their probabilities are not assumed to be equally calibrated, and multiple selected VidOSC labels can reflect competing alternatives. A common numerical threshold does not imply equivalent confidence.

Report precision/recall at every threshold. Do not choose a different threshold per test video or present independently maximized precision and recall as one operating point. Claude and TubeletGraph semantic outputs do not supply comparable state probabilities; detector/mask confidence is not a substitute.

## Framewise precision and recall

For each object, let `P_t` be its predicted state set and `T_t` its ground-truth state set at frame `t`. Multiple states may apply simultaneously.

- Precision: `|P_t ∩ T_t| / |P_t|`, averaged over eligible frames with predictions.
- Recall (detail): `|P_t ∩ T_t| / |T_t|`, averaged over eligible frames with known GT states, including zero for missing predictions after the initial exclusion.

Average per-frame ratios equally within each ingredient, then average ingredient scores equally within each video, then average video scores equally across the evaluation set. Do not pool label counts or weight ingredients/videos by duration. An undefined denominator or no eligible frames gives N/A; omit undefined values from the corresponding mean and report the number of contributing ingredients/videos. This hierarchy applies to framewise precision/recall; timing F1 retains its pooled event-count definition below.

### Vocabulary-only and model-assisted mapping

Report two versions of discrete precision and recall, under the same frames, threshold, and aggregation:

- **Vocabulary-only precision/recall:** use canonical labels and only the supplied `all2one` mappings in the [vocabulary files](https://github.com/chefOST/chefOST-data#vocabularies). Preserve multiword labels as units. Do not introduce new semantic equivalences in this version.
- **Model-assisted precision/recall:** use `gpt-6.2-sol` to map saved predicted state labels/descriptions into the same canonical vocabulary. The mapper receives predicted text and the allowed vocabulary, not ground-truth states/timelines or additional video evidence. It normalizes what the model said; it must not invent omitted states or correct predictions from knowledge of the task.

Keep ground truth fixed between versions. Use the same model-assisted mapper and input format across methods; cache its mapping outputs so a threshold sweep does not repeatedly resample mappings. Record raw predictions, mappings, mapper model/version, prompt, generation settings, and vocabulary revision with the scores. Label the assisted results explicitly because the mapper contributes to the evaluated system.

Retain unmatched predicted labels as incorrect predictions in the precision denominator; they receive no intersection credit. Preserve an unmatched remainder when a description is only partially mapped rather than silently deleting that content. Deduplicate identical predictions and canonical matches. Empty native output is an empty set, not a positive state inferred from an empty-string vocabulary entry.

## Cosine similarity and speed

Use `Qwen/Qwen3-Embedding-8B`. For each eligible ingredient/frame, embed the complete GT label set into one vector and the complete native output label set into another. Serialize each set as a JSON array of sorted unique labels with Unicode preserved, retaining multiword labels. Use the same text encoding for both sides and record the model revision, embedding instructions and normalization. Do not replace native output text with GPT-corrected labels for this metric.

Compute `cos_t = dot(e_GT, e_output) / (norm(e_GT) * norm(e_output))`, sum over eligible frames and divide by their count. Apply the same bounds, initial exclusions and unknown-GT exclusions as the discrete metrics. An empty prediction after the initial exclusion contributes zero; a zero-norm model embedding is an error, not a valid score. Then average ingredients equally within each video and videos equally overall, as for precision/recall. Cache embeddings for repeated label sets. This is set-description similarity, not an average of individual label-pair similarities.

Measure wall-clock latency and throughput separately from state accuracy. Distinguish offline temporal context from causal online inference.

## State-change timing F1

MOSCATO's published segment F1 is a different metric.

1. Events: for each object, record the timestamp whenever its normalized state-label set changes. Several labels added/removed at the same timestamp count as one event. Extract ground-truth and predicted events separately; matching ignores which labels changed and whether they were added or removed.
2. Eligible intervals: start from the same ground-truth-defined intervals for every model, within the ingredient's outer bounds and where it is visible and its state annotation is known. Apply only the declared initial unavailable interval described above as a model-specific exclusion. Reset at each remaining interval: its first frame establishes the initial state and is not an event. Do not count or match across visibility/unknown-annotation gaps, or exclude later frames because a prediction is missing. Empty GT is unknown.
3. Tolerance: within each full ground-truth-defined eligible object/video interval, before any model-specific initial exclusion, calculate the gaps in seconds between consecutive GT events. Pool all such gaps `g_1, …, g_N` across the fixed evaluation split and set `δ = sqrt(sum((g_i − mean(g))²) / N)` (population standard deviation). Freeze this one value for all models on that split before inspecting their results. If fewer than two gaps exist, use `δ = 0` as an exact-match fallback; zero standard deviation also means exact matching.
4. Matching: pair predicted and GT events for the same video, object and eligible interval when `|predicted time − GT time| ≤ δ`. Each event can be used once. Choose the pairing with the most matches; break ties by the smallest total absolute timing error. Keep extra nearby events; do not merge them to improve the score.
5. Score: pool counts across the split. With `M` matched pairs, `G` GT events and `P` predicted events, `TP = M`, `FN = G − M`, `FP = P − M`, and `F1 = 2M / (G + P)`. A missed event is an FN; an unmatched prediction is an FP. A prediction outside every eligible window leaves both an FP and, if still unmatched, an FN. If `G + P = 0`, report N/A.

Example: GT changes at 10, 20 and 34 seconds have gaps 10 and 14 seconds, so `δ = 2 s`. Predictions at 11, 22 and 40 seconds yield two matches, one miss and one false alarm: `F1 = 4/6 ≈ 0.667`.

Store with the W&B run the dataset/split and annotation revision, object mapping, label normalization, timestamp/alignment rule, model reporting frequency, `N`, `δ` in seconds, and TP/FP/FN. Fix these choices across compared models.

If thresholded state sets are used to extract events, report timing scores per threshold using this same tolerance. The two mapping versions above are specified for discrete precision/recall; record the state-normalization variant used for timing rather than silently changing it between comparisons.

This tolerance measures variation in GT change spacing, not annotation uncertainty or a confidence interval. Long or irregular gaps can widen it substantially; scores from different splits may use different tolerances. Correctly timed but semantically wrong changes can earn timing credit; precision, detail and cosine similarity assess state content separately.
