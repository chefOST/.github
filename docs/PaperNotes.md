# ChefOST: A review on object state tracking

The canonical reference for relevant papers and project goals. Converted from the supplied nine-page `ChefOST - A Review on Object State Tracking.pdf` and reconciled with project knowledge on October 3, 2026. The six original reviews are retained, with targeted source corrections and additions below.

## Project goals

Track individual object states throughout long procedural cooking videos. Recognizing the current state is in scope; anticipating future states is not. Prioritize consistency, then accurate detail without hallucination or partial correctness, then inference speed.

Current work evaluates TubeletGraph and adapted OSCAR on MOSCATO, explores VidOSC + VLM/Claude descriptions beyond fixed initial/transition/end phases, and develops lightweight Ego4D pseudo-labels per frame or at a documented interval. Surface multiple applicable states per object/timepoint; evaluate frame-averaged precision and recall (detail). Long-term, preserve identity across multiple objects and transformations. Video-duration thresholds and remaining evaluation conventions are undecided; current baselines do not yet establish the full long-video goal.

## Where to look

- [VidOSC](#1-vidosc): temporal state phases and open-world recognition.
- [SPOC](#2-spoc): spatially actionable/transformed regions.
- [OSCA](#3-osca): state memory for future anticipation; context only.
- [OSCAR](#4-oscar): object-status descriptions for recipe progress.
- [Jelodar et al.](#5-image-based-object-states): image-based state taxonomy/classification.
- [TubeletGraph](#6-tubeletgraph): object lineage and transformation graphs.
- [MOSCATO](#7-moscato): current multi-object state benchmark and data context.
- [FluxGraph](#newer-candidate-fluxgraph): a newer efficiency follow-up, not yet fully reviewed here.

Agents: read the goals and relevant paper entries, then follow primary sources for exact claims. These are literature results, not ChefOST reproduction results. The original review's details are preserved except where corrections are identified; this is a targeted reconciliation, not a complete independent replication or exhaustive literature search. Use the [organization guide](https://github.com/chefOST/.github/blob/main/profile/README.md) to choose a repository, then read its README and applicable `AGENTS.md` for implementation instructions.

## Reconciliation with current work

- MOSCATO ground truth can contain multiple simultaneous states. For each object/frame, let `P_t` be the predicted state set and `T_t` the ground-truth state set. ChefOST precision is `|P_t ∩ T_t| / |P_t|`, averaged over frames with predictions; recall (detail) is `|P_t ∩ T_t| / |T_t|`, averaged over frames where the object is present. Average per-frame ratios equally; missing predictions give zero recall when GT states exist. These are multilabel precision/recall, distinct from the paper's metrics. Sampling, frame alignment, label normalization, empty-GT/no-eligible-frame handling and aggregation across objects/videos remain unspecified; some state labels are phrases.
- [chefOST-data](https://github.com/chefOST/chefOST-data) currently holds MOSCATO annotations/vocabularies, including CMU, EGTEA and EgoPER. EgoPER is distinct from Ego4D; shared Ego4D definitions/evaluation remain planned.
- Frame-averaged precision measures correctness of predicted states; recall (detail) measures coverage of ground-truth states. Offline temporal context must not be presented as causal online tracking.

## Literature selection

Historical seed order from the PDF, not the current implementation priority:

1. [Xue et al. — Learning Object State Changes in Videos: An Open-World Perspective](https://arxiv.org/html/2312.11782) (2023 preprint; CVPR 2024).
2. [Mandikal et al. — SPOC](https://arxiv.org/abs/2503.11953v2) (2025 preprint; accepted at WACV 2026).
3. [Manousaki et al. — Anticipating Object State Changes in Long Procedural Videos](https://arxiv.org/html/2405.12789v2) (2024; contextual evidence only, outside tracking scope).
4. [Li et al. — OSCAR](https://arxiv.org/pdf/2507.03330) (2025).

Additional candidates:

- [Jelodar et al. — Identifying Object States in Cooking-Related Images](https://arxiv.org/abs/1805.06956) (2018).
- [Li et al. — OSCAR, ASSETS 2025](https://doi.org/10.1145/3663547.3746318); the proceedings version of the same work as the preprint, not a second method. DOI corrected from the PDF.
- [Sun et al. — Tracking and Understanding Object Transformations](https://proceedings.neurips.cc/paper_files/paper/2025/file/7482e8ce4139df1a2d8195a0746fa713-Paper-Conference.pdf) (NeurIPS 2025).

Original search criteria: nonduplicate methods and papers not cited by more relevant/successful work. Google Scholar/arXiv queries: object state tracking; object state; ingredient tracking; object procedural videos; ingredient state; ingredient state tracking.

## Cross-paper findings

The supplied document marks its notes human reviewed and links a [paper comparison](https://opposite-utensil-214.notion.site/Paper-Comparison-Table-39d4e041d9ac8085a75ac6cc8e1d441c).

- Tasks differ: VidOSC labels temporal phases; SPOC segments state regions; TubeletGraph preserves lineage; OSCAR tracks recipe steps; Jelodar classifies images; OSCA anticipates future states.
- Metrics are not directly comparable: classification F1/P@1/top-k, segmentation mIoU, VOS J and step accuracy do not form a fair leaderboard. Of the six original reviews, only TubeletGraph includes a numerical runtime; the newer FluxGraph candidate also addresses efficiency.
- Common assumptions: known object/change/recipe or initial mask, one dominant transformation, short or pre-segmented clips, and mostly ordered/irreversible state evolution.
- Research gap: boundary-free, multi-object, identity-preserving tracking over long cooking videos, evaluated for consistency, detail, hallucination/error type and throughput.

## Paper reviews

### 1. VidOSC

Xue, Ashutosh, and Grauman. [Learning Object State Changes in Videos: An Open-World Perspective](https://arxiv.org/html/2312.11782), CVPR 2024 (2023 preprint).

- Research problem: Temporally localize initial, transition, and end states for novel object–state-change combinations without manual training labels.
- Input: Video sampled at 1 fps; known OSC/object category at inference; no test-time narration.
- Output: Per-timepoint initial / transitioning / end / background labels.
- Task formulation: Framewise four-class sequence labeling; open-world split holds out object–transition combinations while sharing constituent states.
- Dataset: HowToChange: 36,075 train and 5,424 evaluation videos; 122/134 objects, 20 transitions, 318/409 OSC combinations; average train clip 41.2 s. Also ChangeIt.
- Supervision type: Weak/pseudo-supervision: Llama 2 mines ASR, an LLM supplies state descriptions, and VLM similarity plus temporal constraints supplies frame labels; humans label evaluation only.
- Model architecture: Frozen InternVideo features → 3-layer, 4-head temporal Transformer (512 hidden) → one-layer MLP; shared state vocabulary and object-centric features.
- Training objective: Cross-entropy on pseudo-labeled frames; ambiguous frames ignored.
- Pretraining: InternVideo; CLIP/VLM; Llama 2 used for data mining/label construction.
- Baselines: CLIP, VideoCLIP, InternVideo, LookForTheChange, MultiTaskChange.
- Metrics: State/action P@1 on ChangeIt; frame F1, precision, and P@1 on HowToChange, split known/novel.
- Main results: HowToChange known/novel F1 46.4/43.1, precision 46.6/43.7, P@1 60.7/58.2; ChangeIt open-world state P@1 .56/.48, action P@1 .89/.82.
- Ablation findings: Full known/novel P@1 60.7/58.2; no temporal modeling 52.9/48.2; no shared state vocabulary 58.5/53.3; no object-centric features 59.8/56.7. Temporal modeling gives the largest absolute gain; shared states reduce the novelty gap most.
- Strengths: Open-world generalization; no manual train labels; explicit temporal consistency; broad cooking coverage.
- Limitations: Known OSC/query; one process per clip; no concurrent changes, spatial masks, identity lineage, or speed report; clips are not long-form. The temporal model uses succeeding as well as preceding frames (§3.2); it is not evidence of causal online inference.
- Assumptions: State change is visually detectable, mostly ordered/irreversible, and adequately represented as initial→transition→end.
- Relationship to other papers: SPOC spatially extends its state phases; TubeletGraph adds identity lineage; OSCAR reuses verb+noun status descriptions; unlike OSCA it recognizes present state rather than anticipating the future.
- Relevance to review question: High for temporal state recognition and open-world evaluation; incomplete for individual identity and long videos.
- Claim locations: Task/splits—p. 3, Fig. 1; pipeline/model—pp. 4–5, Fig. 2; dataset—pp. 5–6, Table 1; metrics/implementation—pp. 6–7; results—p. 7, Table 2; ablation/limitations—p. 8, Table 3 and conclusion. (PDF pages.)

### 2. SPOC

Mandikal et al. [SPOC: Spatially-Progressing Object State Change Segmentation in Video](https://arxiv.org/html/2503.11953v2), 2025 preprint; [accepted at WACV 2026](https://arxiv.org/abs/2503.11953v2).

- Research problem: Localize where an object is still actionable versus already transformed as a change progresses.
- Input: 1-fps video plus known object/OSC; training proposals from detector/tracker; final model uses global-frame and local-mask features.
- Output: Per-frame pixel masks for actionable / transformed / background regions; different parts of one object may have different states.
- Task formulation: Three-class, state-dependent video segmentation for one OSC per clip.
- Dataset: WhereToChange: Table 1 reports 116 objects, 10 transitions, 213 OSC combinations, 16,999 train and 1,162 evaluation videos, about 700k frames, average 41 s; derived from HowToChange and VOST. Counts differ elsewhere in v2: §5.1 lists 1,001 + 155 evaluation clips (1,156), and Appendix Table A lists 232 OSCs. Verify the released split before using these as exact dataset totals.
- Supervision type: VLM pseudo-labels plus causal/dynamics constraints; manual evaluation masks.
- Model architecture: GroundingDINO + SAM proposals every 10 frames; DeAOT propagation; CLIP labels proposals; final Transformer fuses global DINOv2 and local CLIP features.
- Training objective: Three-class cross-entropy; ambiguous proposals excluded; AdamW.
- Pretraining: GroundingDINO, SAM, DeAOT, CLIP ViT-B/32, DINOv2.
- Baselines: MaskCLIP, MaskCLIP+, GroundedSAM, SAM-Track, Random Label, DEVA; v2 also includes SAM+GPT-4o.
- Metrics: mIoU on full/transition and seen/novel splits; progress curves as downstream analysis.
- Main results: WhereToChange-HowTo mIoU .502; v2 Table 2's SAM+GPT-4o baseline reaches .427 (DEVA .282, the comparator in the supplied review). WhereToChange-VOST .267 versus DEVA .228; seen/novel .523/.492; transition-only .332.
- Ablation findings: Detecting every 10 frames beats first-frame-only (.411/.372); global+local features beat either alone for chop/grate/peel; trained model .502, pseudo-label prediction .455, oracle proposals .650.
- Strengths: Finest state detail here; captures intra-object progress; cooking-specific; produces a useful continuous progress signal.
- Limitations: Merged proposals may span both states; tracking loses rapidly changing objects; one known OSC and separate model per transition verb; no persistent ingredient lineage or speed report.
- Assumptions: One OSC per clip; known noun/verb; every not-final region is actionable; state regions are spatially separable and detector/tracker proposals are reliable.
- Relationship to other papers: Direct spatial extension of VidOSC; complementary to TubeletGraph’s identity lineage; could provide state evidence to OSCAR.
- Relevance to review question: High for detailed, localized states; medium for long-form individual tracking.
- Claim locations: Task/output—Fig. 1 and §3.1; pseudo-label pipeline—Fig. 2 and §§3.2–3.3; model/objective—Fig. 3 and §3.4; dataset—Table 1 and §5.1; main results—Table 2; transition analysis—Table 3; ablations—Appendix Tables D–F; failures—Appendix Fig. I. (Exact figure/table/section locations.)

### 3. OSCA

Manousaki et al. [Anticipating Object State Changes in Long Procedural Videos](https://arxiv.org/abs/2405.12789), 2024. The supplied review cites [v2](https://arxiv.org/html/2405.12789v2), whose displayed title is shortened to *Anticipating Object State Changes*; v3 is now available. The detailed notes below follow the cited v2.

- Research problem: Predict which OSC will occur during the next unseen action.
- Input: Current/recent SlowFast visual features plus histories of action verbs/nouns and object-state labels.
- Output: One of nine next-OSC classes, including No OSC.
- Task formulation: Multiclass anticipation at the onset of the next action, before its point-of-no-return.
- Dataset: Ego4D-OSCA: 1,610 videos, 9 state classes, 1,500 actions, 477 objects; 61,858 train and 31,846 test clips. The paper repurposes the Ego4D LTA validation split as its test set (§3).
- Supervision type: Fully supervised curated Ego4D annotations; oracle histories for training/analysis and recognizer outputs for realistic inference.
- Model architecture: Three streams: SlowFast visual features and two BiLSTM+MLP history encoders, fused for classification.
- Training objective: Categorical cross-entropy.
- Pretraining: SlowFast; external action and state recognizers.
- Baselines: Vision-only; vision + oracle/recognized action history; vision + oracle/recognized state history; both histories.
- Metrics: Top-1 and top-5 mean accuracy.
- Main results: Vision-only 23.93/89.10; oracle state history 34.35/85.92; oracle action+state 35.60/88.14; realistic recognized histories 28.97/92.25 (top-1/top-5).
- Ablation findings: State history helps more than action history. Injecting 25/50/75% history noise reduces oracle top-1 from 35.60 to 30.46/26.00/22.48.
- Strengths: Uses long multi-action context; directly demonstrates the value and fragility of state memory.
- Limitations: Anticipation, not current-state tracking/localization; assumes the decision boundary and object-in-use; modest top-1; not cooking-specific or open-world.
- Assumptions: The next-action onset is known; one next class suffices; histories and the interacted object are available.
- Relationship to other papers: A possible downstream consumer of VidOSC/SPOC/TubeletGraph state histories; its metrics are not comparable to tracking metrics.
- Relevance to review question: Low/directly out of scope; useful only as evidence that long-term state memory matters.
- Claim locations (v2): Task—Fig. 1 and §1; dataset—§3/Table 1; class/clip distributions—Fig. 5; architecture/objective—Fig. 6 and §4; main results—§5/Table 2; noise ablation—Table 3.

### 4. OSCAR

Li et al. [Exploring Object Status Recognition for Recipe Progress Tracking in Non-Visual Cooking](https://arxiv.org/html/2507.03330), [ASSETS 2025](https://doi.org/10.1145/3663547.3746318). One work with a preprint and proceedings version; the description below concerns the original method, not ChefOST's adaptation.

- Research problem: Use object status to infer cooking-recipe progress, especially for blind/low-vision cooks.
- Input: Recipe and ingredients, sampled video frames, generated verb+noun status phrases, and prior predictions.
- Output: Current, completed, remaining, and missing recipe steps—not per-object state trajectories.
- Task formulation: Frame/clip-to-step similarity followed by ordered, time-causal rule filtering.
- Dataset: 173 curated YouCook2 videos (average 7.7 steps) and 12 home cooking sessions by BLV participants (14–51 min, average 8.1 steps). The cited 5.27-minute average describes the original YouCook2 dataset, not explicitly this curated subset.
- Supervision type: Training-free/zero-shot pipeline; manually curated recipes/ingredients and ground-truth step timestamps for evaluation.
- Model architecture: GPT-4o reformats recipes/extracts statuses; CLIP or SigLIP aligns frames with steps/statuses; averaged similarity plus JSON history and deterministic causal rules selects steps.
- Training objective: None; no fine-tuning.
- Pretraining: Off-the-shelf GPT-4o, CLIP, SigLIP.
- Baselines: CLIP/SigLIP frame-to-step similarity without status augmentation or temporal logic.
- Metrics: Mean per-step classification accuracy with standard deviation.
- Main results: YouCook2: CLIP 41.7→68.0 (+26.3), SigLIP 62.2→82.8 (+20.6). Real-world: CLIP 33.7→58.4 (+24.7), SigLIP 41.9→66.7 (+24.8).
- Ablation findings: No factorized component ablation; baseline-to-full-pipeline comparisons combine status augmentation and temporal rules, so their effects cannot be separated.
- Strengths: Long cooking videos; real-world recordings; progress-focused outputs; no task-specific training.
- Limitations: Evaluation samples five frames inside annotated step segments and repeats sampling three times. Our interpretation: this protocol does not establish boundary-free online tracking. It tracks recipe steps, not ingredient identity; has a small real-world set and combined ablation confounds; and is sensitive to camera placement, lighting and occlusion.
- Assumptions: Recipe, ingredients, ordered steps, and evaluation step boundaries are known; progress is mostly linear; generated status text matches visual evidence.
- Relationship to other papers: Applies Xue-style verb+noun state descriptions downstream; longer/cooking-specific than VidOSC/SPOC but weaker as an object tracker; temporal history parallels OSCA.
- Relevance to review question: Medium for long cooking progress; low for direct individual-object state tracking.
- Claim locations (linked preprint): Pipeline—Fig. 1 and §3; YouCook2 sampling/evaluation—§§4.2.1–4.2.2/Table 1; BLV dataset/results—§§5.1–5.3/Table 2; repeated frame sampling—§§5.2.1–5.2.2; real-world limitations—§§6–7.

### 5. Image-based object states

Jelodar, Salekin, and Sun. [Identifying Object States in Cooking-Related Images](https://arxiv.org/html/1805.06956), 2018.

- Research problem: Recognize visually distinct cooking-object states from still images.
- Input: One image; object identity is effectively known for object-specific models.
- Output: One of 11 labels: whole, peeled, floured, sliced, diced, grated, julienne, juice, creamy, mixed, or other. The PDF review's `minced` is corrected to `mixed`.
- Task formulation: Closed-set single-image multiclass classification, generic or per object.
- Dataset: 9,309 web-crawled images of 17 common objects; 6,498 train, 1,398 validation, 1,413 test (§III-B). More than 250 cooking videos informed object/state selection. The PDF review transposed the training count to 6,489.
- Supervision type: Fully supervised manual class labels.
- Model architecture: ImageNet ResNet trunk plus added convolution/BN/ReLU/GAP/softmax head; generic model, object-specific fine-tuning, and three-model weighted voting.
- Training objective: Supervised softmax classification; the paper does not name the loss explicitly.
- Pretraining: ImageNet ResNet; new layers trained first, then the full network fine-tuned.
- Baselines: Generic ResNet; weighted-voting ensemble; generic versus object-specific fine-tuning; ImageNet-class subset sanity check.
- Metrics: Top-1/2/3 accuracy.
- Main results: Generic model 80.4 top-1/91.5 top-2; generic voting 82.0/92.0; per-object average 86.9 top-1 and 88.3 with voting over 16 evaluated objects (dough excluded, §V-A/Table II).
- Ablation findings: Object-specific fine-tuning gives the largest gain (80.4→86.9 average top-1); voting adds 1.6 points generically and 1.4 points per-object.
- Strengths: Foundational cooking-state taxonomy/dataset; simple transfer-learning reference; strong closed-set image accuracy.
- Limitations: No temporal tracking, identity persistence, transition localization, masks, open-world labels, or speed; one dominant label cannot represent mixed states; web/video-frame bias.
- Assumptions: A single dominant visible state exists and, for best results, object identity is already known.
- Relationship to other papers: Frame-level predecessor; VidOSC adds temporal/open-world phases, SPOC mixed-state masks, and TubeletGraph lineage.
- Relevance to review question: Low directly; useful taxonomy and static-classification baseline.
- Claim locations: Dataset scope—p. 1; taxonomy/statistics—p. 3, Figs. 1–2; architecture/training—p. 4, Figs. 3–4; generic results—p. 4, Table I; per-object/ImageNet results—p. 5, Tables II–III; ambiguity errors—p. 5, Figs. 5–6. (PDF pages.)

### 6. TubeletGraph

Sun, Yang, Sun, and Hariharan. [Tracking and Understanding Object Transformations](https://proceedings.neurips.cc/paper_files/paper/2025/file/7482e8ce4139df1a2d8195a0746fa713-Paper-Conference.pdf), NeurIPS 2025. [Author project](https://tubelet-graph.github.io/); [arXiv v2, January 2026](https://arxiv.org/html/2511.04678v2).

- Research problem: Preserve object identity through severe appearance/topology changes while detecting and describing each transformation.
- Input: Video plus a binary mask for the target object in the first frame.
- Output: All descendant mask tracks after splits plus state-change tuples: time, pre-tracks, post-tracks, and natural-language description; visualized as a state graph.
- Task formulation: Track Any State, jointly semi-supervised video-object segmentation and grounded transformation understanding.
- Dataset: VOST-TAS extends the 70-video VOST validation set with transformation boundaries, action descriptions, and resulting-object masks/text; tracking also evaluated on VOST, VSCOS (98 videos), M³-VOS (479), and DAVIS17. These videos remain short (VOST average 22.3 s; VSCOS 7.5 s; M³-VOS 14.3 s).
- Supervision type: Zero-shot/training-free system; manual benchmark annotations; thresholds tuned on VOST training data.
- Model architecture: CropFormer partitions entities; SAM2.1 tracks tubelets and initiates tracks in uncovered regions; spatial proximity and CLIP semantic consistency filter candidates; GPT-4.1 names changes/results and builds the graph.
- Training objective: None; modular inference.
- Pretraining: CropFormer, SAM2.1-L, FC-CLIP/CLIP, GPT-4.1.
- Baselines: XMem, Cutie, ReVOS, SAM2, SAM2Long, SAM2.1, DAM4SAM, SAMURAI, and SAM2.1 fine-tuned on VOST.
- Metrics: Jaccard J over the full video and last 25%; pixel precision/recall; temporal localization P/R; action/object semantic accuracy; spatiotemporal and overall recall.
- Main results in the supplied review: TubeletGraph J/J-last: VOST 51.0/36.9, VSCOS 75.9/72.2, M³-VOS 74.2/64.4, DAVIS17 85.6/82.5. Updated v2 Table 2 reports 50.9/36.7, 75.9/72.2, 74.1/64.1 and 85.6/82.6 respectively; do not mix versions. Full system: action/object accuracy 81.8/72.3, temporal P/R 43.1/20.4, spatiotemporal recall 12.0, overall recall 6.5. Semantic accuracies are conditional on successful temporal/spatial matches, not end-to-end state accuracy (v2 §4.2).
- Ablation findings in the supplied review: Spatiotemporal partition+filters improve base SAM2.1 VOST J/J-last 48.4/32.4→51.0/36.9 (v2's full result is 50.9/36.7). SAM auto-masks, Cutie tracking, DINOv2 filtering, or Qwen reasoning reduce overall recall to 0.9/2.8/5.6/1.9, versus 6.5 full.
- Strengths: Most direct match to individual-object tracking; handles one-to-many lineage; outputs spatial tracks and semantic transformation graph; zero-shot.
- Limitations: Spatiotemporal partition construction costs about 7 s/frame on VOST with one RTX A6000 (v2 §4.3); this is not a verified total including all VLM calls. Short benchmarks; transformations are inferred mainly from newly emerged/missed tracks; low temporal and overall recall; upstream false positives cascade; not cooking-specific.
- Assumptions: Initial mask supplied; short-term motion is smooth; descendants remain spatially close and semantically related; tracker false negatives expose transformations.
- Relationship to other papers: Complements VidOSC/SPOC: it supplies identity lineage while they supply phase/state labels and finer progress; OSCAR supplies longer cooking context.
- Relevance to review question: Highest conceptual fit, but present speed, recall, and video length prevent it from solving ChefOST directly.
- Claim locations: Task/output—§3.1 and Fig. 1; architecture—Fig. 2 and §§3.2–3.4; assumptions—§3.3; datasets/metrics—§4; component tracking ablation—Table 1; cross-dataset results—Table 2; semantic/system ablation—Table 3; speed—§4.3; dataset statistics—Appendix Table 4. (Exact figure/table/section locations.)

### 7. MOSCATO

Zameni, Shen, and Elhamifar. [MOSCATO: Predicting Multiple Object State Change Through Actions](https://openaccess.thecvf.com/content/ICCV2025/html/Zameni_MOSCATO_Predicting_Multiple_Object_State_Change_Through_Actions_ICCV_2025_paper.html), ICCV 2025, pp. 11600–11611. Added because it now anchors ChefOST evaluation.

- Task: Recognize framewise states of multiple objects throughout long cooking videos; multiple states may apply simultaneously. This concerns observed-video state recognition despite “predicting” in the title.
- Method/supervision: Training action annotations feed an LLM/VLM Pseudo-Label Acquisition pipeline; a Multiple Object State Prediction network learns from those labels, and State–Action Interaction refines state predictions using predicted actions. [Paper](https://openaccess.thecvf.com/content/ICCV2025/papers/Zameni_MOSCATO_Predicting_Multiple_Object_State_Change_Through_Actions_ICCV_2025_paper.pdf).
- Data: CMU, EGTEA and EgoPER; automatically generated training labels and manually annotated test ground truth. The release describes state arrays on a 10 fps annotation timeline, not the raw video's frame rate. [Dataset documentation](https://github.com/eelhami/iccv25_moscato).
- Metrics: Framewise F1-max/mAP and temporal segment F1 at overlap thresholds 0.1, 0.25 and 0.5. They are not interchangeable with ChefOST's frame-averaged multilabel precision and recall (detail). [Evaluation documentation](https://github.com/eelhami/iccv25_moscato).
- ChefOST relevance: The local `MOSCATO/` copy and `chefOST-data/MOSCATO/` provide annotations/vocabularies, not videos, features or the paper's model implementation. Check [current evaluation questions](#reconciliation-with-current-work) for alignment, normalization and scoring issues. This is a multi-state benchmark, not evidence of persistent instance lineage.

## Newer candidate: FluxGraph

Sun and Hariharan. [Efficient Tracking and Understanding Object Transformations](https://arxiv.org/abs/2607.19743), July 22, 2026 preprint. Abstract-level addition, not a full review: FluxGraph uses SAM2 multi-mask disagreement to trigger transformation handling and avoids tracking every scene entity. The authors report about 3.3× faster inference on VOST with preserved state-graph quality. This makes it relevant to ChefOST's efficiency priority; it has not been selected or evaluated by ChefOST. Its runtime protocol should be checked before comparing it with the original TubeletGraph timing.

## Provenance and corrections

- The supplied PDF covers the research definition (p. 1), literature selection (pp. 1–2), synthesis (p. 3), VidOSC (pp. 3–4), SPOC (pp. 4–5), OSCA (pp. 5–6), OSCAR (pp. 6–7), Jelodar (pp. 7–8) and TubeletGraph (pp. 8–9). The “claim locations” above refer to the underlying papers, not these review pages; they are inherited unless a version/correction is stated.
- OSCAR's correct DOI is `10.1145/3663547.3746318`, verified against the [institutional publication record](https://ink.library.smu.edu.sg/sis_research/10477/). The PDF's `10.1145/3746027.3755771` identifies a different recipe-dataset paper; it is not another OSCAR version.
- Targeted primary-source checks corrected Jelodar's taxonomy/training split and OSCAR's citation, duration attribution and claim locations; clarified VidOSC's future-frame context, OSCA's version/split, SPOC's current baseline/venue and inconsistent counts, and TubeletGraph's versioned scores, conditional metrics and timing scope. Unchanged details remain inherited review notes, not a claim of exhaustive verification.
- Project goals follow the supplied announcements through September 23, 2026 (latest technical update: September 13), the September 26 MOSCATO ground-truth correction, and subsequent user instructions. The local workspace README retains assignments and project context; it is not part of this published review. Newer explicit decisions take precedence. Update the short goals here when scope changes, rather than maintaining competing copies in entry-point READMEs.
