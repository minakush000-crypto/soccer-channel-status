# Brief 09 — measurements (job 4)

Every number below is computed by the command printed next to it.
Counts come from reports/brief09/frames_{A,B,C}.csv (job 3) and
the pod scene-cut lists (cuts.json, job 2). No number here is
estimated by eye or recalled from a transcript.

## 1. Share of frames per shot type per video

Command: this script (make_measurements.py) —
`count(shot_type) / count(all rows)` per video, from frames_<L>.csv.

### Video A (253 classified frames)

- 2D diagram: 74 (29.2%)
- 3D pitch: 45 (17.8%)
- footage+overlay: 44 (17.4%)
- game simulation: 20 (7.9%)
- live footage: 23 (9.1%)
- other: 26 (10.3%)
- stats card: 4 (1.6%)
- talking head or coach clip: 15 (5.9%)
- text card: 2 (0.8%)

### Video B (250 classified frames)

- 2D diagram: 78 (31.2%)
- footage+overlay: 23 (9.2%)
- frozen footage: 10 (4.0%)
- live footage: 40 (16.0%)
- other: 60 (24.0%)
- stats card: 14 (5.6%)
- talking head or coach clip: 14 (5.6%)
- text card: 11 (4.4%)

### Video C (142 classified frames)

- footage+overlay: 55 (38.7%)
- live footage: 70 (49.3%)
- other: 17 (12.0%)

## 2. Average shot length from the scene-cut list

Command: `python - <<` (below) — diffs between consecutive scene
cuts from reports/brief09/cut_<L>.json (threshold 0.3), plus the
tail-to-end span. Shorter-than-2s shots are kept (they are real
cuts); the mean and median are both printed.

- A: cuts=74 mean_shot=6.73s median=4.42s min=0.29s max=30.88s
- B: cuts=7 mean_shot=70.59s median=27.48s min=6.95s max=202.85s
- C: cuts=32 mean_shot=8.33s median=6.10s min=0.03s max=55.18s

Caveat (from the job-2 score sweep, reports/brief09/scores_*.json):
B is edited with gradual animated transitions - only 8 candidate
transitions exist across all 30048 scored frames even at a very low
threshold, so B's detected cut list understates its scene changes and
its mean_shot is not like-for-like with A and C. A and C cut hard;
B morphs.
## 3. Overlay frames per minute

An overlay frame = a row with a non-empty elements column (drawn
tactical element, on footage or on a diagram). The footage+overlay
shot type is the on-footage subset. Per-minute rate uses the
studied region duration (durations.json).

- A: overlay_frames=133 (52.6%) over 506.9s = 15.7/min (on-footage: 44 = 5.2/min)
- B: overlay_frames=99 (39.6%) over 500.8s = 11.9/min (on-footage: 23 = 2.8/min)
- C: overlay_frames=55 (38.7%) over 284.0s = 11.6/min (on-footage: 55 = 11.6/min)

## 4. The 10 most common overlay element combinations

Command: `python - <<` (below) — `Counter(tuple(sorted(elements)))`
over rows with non-empty elements, all three videos pooled.

1. highlight ring — 53 frames
2. text — 47 frames
3. highlight ring + line between players — 22 frames
4. player label — 18 frames
5. player label + text — 18 frames
6. zone — 17 frames
7. arrow — 17 frames
8. arrow + player label — 14 frames
9. shape outline — 10 frames
10. line between players — 9 frames
