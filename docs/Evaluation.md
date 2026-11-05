# ChefOST evaluation

Current metrics are frame-averaged precision, recall (detail), cosine similarity, and the state-change timing F1 defined below. Precision/detail conventions are in [PaperNotes](PaperNotes.md#reconciliation-with-current-work); the cosine embedding, comparison unit and aggregation must be recorded per experiment.

## State-change timing F1

ChefOST metric decision, November 4, 2026. Shared evaluator implementation remains planned; MOSCATO's published segment F1 is a different metric.

1. Events: for each object, record the timestamp whenever its normalized state-label set changes. Several labels added/removed at the same timestamp count as one event. Extract ground-truth and predicted events separately; matching ignores which labels changed and whether they were added or removed.
2. Eligible intervals: use the same ground-truth-defined intervals for every model, where the object is visible and its state annotation is known. Reset at each interval: its first frame establishes the initial state and is not an event. Do not count or match across visibility/unknown-annotation gaps, or exclude frames because a prediction is missing. Empty GT is unknown unless the dataset explicitly defines it as a valid state set.
3. Tolerance: within each eligible object/video interval, calculate the gaps in seconds between consecutive GT events. Pool all such gaps `g_1, …, g_N` across the fixed evaluation split and set `δ = sqrt(sum((g_i − mean(g))²) / N)` (population standard deviation). Freeze this one value for all models on that split before inspecting their results. If fewer than two gaps exist, use `δ = 0` as an exact-match fallback; zero standard deviation also means exact matching.
4. Matching: pair predicted and GT events for the same video, object and eligible interval when `|predicted time − GT time| ≤ δ`. Each event can be used once. Choose the pairing with the most matches; break ties by the smallest total absolute timing error. Keep extra nearby events; do not merge them to improve the score.
5. Score: pool counts across the split. With `M` matched pairs, `G` GT events and `P` predicted events, `TP = M`, `FN = G − M`, `FP = P − M`, and `F1 = 2M / (G + P)`. A missed event is an FN; an unmatched prediction is an FP. A prediction outside every eligible window leaves both an FP and, if still unmatched, an FN. If `G + P = 0`, report N/A.

Example: GT changes at 10, 20 and 34 seconds have gaps 10 and 14 seconds, so `δ = 2 s`. Predictions at 11, 22 and 40 seconds yield two matches, one miss and one false alarm: `F1 = 4/6 ≈ 0.667`.

Report the dataset/split and annotation revision, object mapping, label normalization, timestamp/alignment rule, model reporting frequency, `N`, `δ` in seconds, and TP/FP/FN. Use the declared annotation timeline (MOSCATO: 10 fps), not raw-video frame indices. Fix these choices across compared models.

This tolerance measures variation in GT change spacing, not annotation uncertainty or a confidence interval. Long or irregular gaps can widen it substantially; scores from different splits may use different tolerances. Correctly timed but semantically wrong changes can earn timing credit; precision, detail and cosine similarity assess state content separately.
