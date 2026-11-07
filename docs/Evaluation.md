# ChefOST evaluation

Use frame-averaged precision, recall (detail), cosine similarity and state-change timing F1. These definitions govern shared comparisons; a paper's metrics or an implementation's outputs do not redefine them.

## Framewise precision and recall

For each object, let `P_t` be its predicted state set and `T_t` its ground-truth state set at frame `t`. Multiple states may apply simultaneously.

- Precision: `|P_t ∩ T_t| / |P_t|`, averaged over frames with predictions.
- Recall (detail): `|P_t ∩ T_t| / |T_t|`, averaged over frames where the object is present. Missing predictions give zero recall when GT states exist.

Average per-frame ratios equally. Agree on sampling, frame alignment, label normalization, empty-GT/no-eligible-frame handling and aggregation across objects/videos before comparing models; these choices are not fixed by the formulas. Some state labels are phrases. Keep training pseudo-labels separate from evaluation ground truth.

## Cosine similarity and speed

Specify the embedding model, comparison unit and aggregation with the experiment in W&B. Measure wall-clock latency and throughput separately from state accuracy. Distinguish offline temporal context from causal online inference.

## State-change timing F1

MOSCATO's published segment F1 is a different metric.

1. Events: for each object, record the timestamp whenever its normalized state-label set changes. Several labels added/removed at the same timestamp count as one event. Extract ground-truth and predicted events separately; matching ignores which labels changed and whether they were added or removed.
2. Eligible intervals: use the same ground-truth-defined intervals for every model, where the object is visible and its state annotation is known. Reset at each interval: its first frame establishes the initial state and is not an event. Do not count or match across visibility/unknown-annotation gaps, or exclude frames because a prediction is missing. Empty GT is unknown unless the dataset explicitly defines it as a valid state set.
3. Tolerance: within each eligible object/video interval, calculate the gaps in seconds between consecutive GT events. Pool all such gaps `g_1, …, g_N` across the fixed evaluation split and set `δ = sqrt(sum((g_i − mean(g))²) / N)` (population standard deviation). Freeze this one value for all models on that split before inspecting their results. If fewer than two gaps exist, use `δ = 0` as an exact-match fallback; zero standard deviation also means exact matching.
4. Matching: pair predicted and GT events for the same video, object and eligible interval when `|predicted time − GT time| ≤ δ`. Each event can be used once. Choose the pairing with the most matches; break ties by the smallest total absolute timing error. Keep extra nearby events; do not merge them to improve the score.
5. Score: pool counts across the split. With `M` matched pairs, `G` GT events and `P` predicted events, `TP = M`, `FN = G − M`, `FP = P − M`, and `F1 = 2M / (G + P)`. A missed event is an FN; an unmatched prediction is an FP. A prediction outside every eligible window leaves both an FP and, if still unmatched, an FN. If `G + P = 0`, report N/A.

Example: GT changes at 10, 20 and 34 seconds have gaps 10 and 14 seconds, so `δ = 2 s`. Predictions at 11, 22 and 40 seconds yield two matches, one miss and one false alarm: `F1 = 4/6 ≈ 0.667`.

Store with the W&B run the dataset/split and annotation revision, object mapping, label normalization, timestamp/alignment rule, model reporting frequency, `N`, `δ` in seconds, and TP/FP/FN. Use the declared annotation timeline (MOSCATO: 10 fps), not raw-video frame indices. Fix these choices across compared models.

This tolerance measures variation in GT change spacing, not annotation uncertainty or a confidence interval. Long or irregular gaps can widen it substantially; scores from different splits may use different tolerances. Correctly timed but semantically wrong changes can earn timing credit; precision, detail and cosine similarity assess state content separately.
