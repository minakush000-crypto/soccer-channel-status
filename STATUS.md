# STATUS.md — verified current state

Verified against code on 2026-09-08. This rebuild supersedes the 2026-09-05
version: the tactical_overlay wiring it described (`produce_v2.py:232`) no
longer exists, and the "no upload" claim is reversed (2 uploads happened
2026-09-06). No plans, no hopes, no hand-typed quality scores. Every claim has
the command that proved it.

## What runs today

### produce_v2.py (the authoritative 9-step pipeline, ARCHITECTURE.md)

Latest produce_v2.py output (liverpool-forest):

```
$ ffprobe -v error -show_entries stream=width,height -of csv=p=0:s=x renders/2026-08-30_liverpool-forest/shorts/final_video_shorts.mp4
720x1280
$ ffprobe -v error -show_entries format=duration -of csv=p=0 renders/2026-08-30_liverpool-forest/shorts/final_video_shorts.mp4
62.276009
$ stat -c '%s %n' renders/2026-08-30_liverpool-forest/shorts/final_video_shorts.mp4
24811062 ... Sep  7 00:07
```

### assemble_words_match.py (standalone, words-match-pictures path)

Arsenal-Chelsea, built 2026-09-08 (NOT a produce_v2 run):

```
$ ffprobe -v error -show_entries stream=width,height -of csv=p=0:s=x renders/2026-09-06_arsenal-chelsea/final_video.mp4
1280x720
$ ffprobe -v error -show_entries format=duration -of csv=p=0 renders/2026-09-06_arsenal-chelsea/final_video.mp4
45.200000
$ stat -c '%s %n' renders/2026-09-06_arsenal-chelsea/final_video.mp4
15034830 ... Sep  8 19:33
```
PRIVATE upload: https://www.youtube.com/watch?v=WFi2LBwXINU (and a second
PsEz5ITTIpM). `artifacts/publish-log/` has 2 entries:

```
$ ls artifacts/publish-log/
2026-09-06_arsenal-chelsea_PsEz5ITTIpM.json
2026-09-06_arsenal-chelsea_WFi2LBwXINU.json
```

### 200MB local-video guard (new, 2026-09-08)

`ffmpeg_utils.py` now enforces `LOCAL_VIDEO_LIMIT_MB = 200` on every local
video download (produce_v2 yt-dlp, runpod_fulltrack annotated-MP4 return,
cloud_produce download_url). See ARCHITECTURE.md "200MB guard".

## What is broken, worst first

### 1. Footage cuts ignore narration (broken logic)
`produce_v2.py:433`:
```
start = (seg.get("clip_idx", 0) * 5) % max(1, int(clip_total) - 5)
```
Fixed 5-second offsets indexed by `clip_idx`. The script's `[VISUAL: footage=]`
tags only select board-vs-footage (line 361 `has_footage = "footage=" in
tag.lower()`), not which part of the clip. `assemble_words_match.py` holds the
content-matched fix (parses `footage=START-END` windows) but is NOT folded in
— blocked by missing timestamp-window data + schema mismatch, see
RECONCILIATION 1.2.

### 2. Script is not validated against match data (broken correctness)
No call to `validate_script.py` in produce_v2.py (`grep -c validate_script
tools/produce_v2.py` → 0). CONTEXT example: script said "Gravenberch receives"
but match data has him as a 71st-minute sub.

### 3. Output is 720x1280, not 1080x1920 (broken format)
`shorts_crop.py:64` `OUT_W, OUT_H = 720, 1280`. Deliberate downscale to avoid a
soft 1.78x upscale from 1080p. A 2160p source can crop+downscale to 1080x1920
(comment, line 78) but that path is not the default and no 4K source has been
tested through produce_v2.

### 4. OVERLAYS ARE GONE (was broken, now resolved)
The 2026-09-05 STATUS said `tactical_overlay.py` guesses coords via
`produce_v2.py:232`. That wiring is GONE (`grep -n tactical_overlay
tools/produce_v2.py` → exit 1). The step is now `step4b_tactical_render`
(line 202) → `runpod_fulltrack.py` + `tactical_render.py` (real tracking data,
top-down renderer, Opus 8/8.5). `tactical_overlay.py` is DEAD.

## Option C Stage 1: per-frame player positions — DONE (2026-09-05)

`cv_annotate.py` exports `player_positions` in its tracking JSON: one entry per
processed frame, each with `{frame, players: [{id, bbox, team}]}`. Existing
exports (team_assignment, ball_positions) intact.

Verified on RunPod (L4, 300 frames = 10s of 1920x1080 footage):
- 300 frames, 129 unique tracker IDs, 38 team_assignment entries
- Pod inference: 23s (0.077s/frame on L4 vs 1.30s/frame local)
- Pod cost: ~$0.05 (720s uptime at $0.25/hr L4)
- Tracker fragmentation: 129 IDs span 3 camera shots (cuts at frames 130 and
  245). Within shots, tracking is usable: median consecutive run 32 frames
  (1.07s), 72 IDs survive >25 frames. Shot 2 (frames 131-244) cleanest: 35
  IDs, median run 41 frames, 2 survive the full 3.8s shot.

No path from tracker ID to player name exists (DECISIONS 2026-09-05, question A).

## Option C+E: segment selection + top-down tactical renderer (2026-09-05 session 3)

- **C (segment selection)**: DONE. `segment_scorer.py` scores 1-second windows
  by detection, persistence, stability, team classification. Full 146s: 15/146
  segments score >=65 (15s usable). Best sec 1 (85.6), worst sec 45 (7.5).
- **E (top-down tactical renderer)**: `tactical_render.py` renders dark pitch
  with mowing stripes, player dots in team colors, movement trails, Bezier
  arrows. Outputs PNG or MP4. Screen-space projection (no homography). Opus
  assessment: 8/10 then 8.5/10 across tasks 2-4 (authoritative). All elements
  visible.
- Known gaps for E (from Opus): no context layer (title, team names, ball
  marker, attacking direction); pitch layout off-centre, missing 6-yard
  boxes/penalty spots/arcs/corner arcs/goals, stripe contrast too high;
  movement encoding has trail/dot color conflict, no hierarchy, raw polylines,
  no min arrow length filter.

Full-clip tracking: DONE. 146s on RunPod L4, 156s, $0.011.
`clip_nxPNT4TU5_Q_full.tracking.json` (8MB, 4380 frames).

## Gemini video inventory assessment (2026-09-08)

Tested whether Gemini can produce a timestamped footage inventory. Half-works;
the half that's wrong is the half that matters for cutting.
- GEMINI_API_KEY valid (prepaid AI Studio). ~$0.11/clip on gemini-3.1-pro-preview,
  ~$0.04 on gemini-3.6-flash. 91 video tokens/sec @720p.
- Content classification accurate (goals/celebration/replay/crowd/subs).
- Timestamps NOT cut-accurate: boundaries 1-2s early, goal windows bloated,
  wrong team in open play. Found 2/3 real goals, missed the winner, invented 1
  phantom.
- Player names work via jersey+lineup lookup (all 7 tested correct).
- ~5-6 usable action passages (~50s) in 482s; reel ~90% non-action.
- DECISION: match_data.json is ground truth for events; Gemini is only a
  segment-finder. Never trust Gemini timestamps as cut points without
  match_data cross-check.

## Scoreboard scanner — goal-finding by scoreline change (2026-09-08)

`scoreboard_scan.py`: samples every 3s, crops the top-left score bug (420x150),
reads the scoreline with gemma4:cloud (free), carries last-known score across
NONE frames, detects changes. On the 482s arsenal-chelsea clip: 161 frames,
76s, free. 3 real scoreline changes found (after run-based blip absorption,
MIN_PERSIST=9s):
- ~102s: 0-0 -> 0-1 Rogers (Chelsea)
- 171s: 0-1 -> 1-1 Havertz
- 261s: 1-1 -> 2-1 Ødegaard
Matches match_data.json exactly (Rogers 2', Havertz 25', Ødegaard 50').
Walk-back offset to the shot is variable (1-8s). A fixed 8s pre-roll captures
every goal's shot; 1s walk-back gives the exact shot frame. The 8s pre-roll
rule FAILED on the words-match build (offset varies in sign: +17/+10/-8s);
fix = search ±20s around each bug-update, relay-verified.

## Broadcast/filler signal (2026-09-08, Part 2 — verified)

The score bug is NOT a broadcast/filler signal (uploader burned it in across
ALL footage types, including fan-shot phone frames). NONE = "gemma4 failed to
OCR the bug", not "bug absent". Replacement signal: Gemini inventory
content classification. broadcast = {shot, build-up} with shot_type != replay;
filler = {non-action} or {replay}. On the 482s reel: 43.0s broadcast (9%) /
440.0s filler (21 segments). The old bug signal claimed ~210s (44%), inflated
~5x. Tool: `broadcast_filler.py`. Goal-finding is unaffected (scanner works).

## Build one video: words match pictures (2026-09-08)

End-to-end cut-list-first build on Arsenal-Chelsea via `assemble_words_match.py`
(no arrows, no renderer, boards + clean footage only). Cuts footage at exact
verified timestamps, replacing produce_v2's fixed-5s-offset bug. Output
final_video.mp4 45.2s, 1280x720. PRIVATE: WFi2LBwXINU.
Step 7 (the never-run test) PASSES: relay on the mid-frame of each of 7
sections — 6/7 clean match, 1 partial. All 3 goals match narration via
on-screen scorer captions. Possession board had a pre-existing data bug
(39.3/32.7 vs 54.6/45.4) — fixed (see below).

## Possession board bug (2026-09-08, Part 4a — fixed) + board ratings (4b)

Possession bug FIXED. Root cause: the ANIMATED possession.mp4 filled the bar
over 75% of frames, broadcasting intermediate values. Fix: fill in first ~6%
of frames, hold full ~94%, label only final values. Verified by Opus
(54.6/45.4, full width, sum 100).
Board appearance RATED by Opus 5 (authoritative), NOT fixed: formation 3/10,
possession 4/10, stat_card 5/10. Common fails: matplotlib defaults, flat, no
Bebas/Barlow, no narrative furniture, stat-card bars both grow the same way.

## Goal cut-window diagnosis (2026-09-08, Part 5)

- Rogers (Chelsea), 119-127 (8s): 100% celebration + ROGERS 1-0 caption, NO
  shot in the reel. Script fixed to narrate the celebration, no shot claimed.
- Havertz (Arsenal), 181-191 (10s): first second = ball-in-net aftermath +
  HAVERTZ 1-1 caption; the live STRIKE is NOT in the window (~163-166).
- Odegaard (Arsenal), 253-262 (9s): first second (253) = LIVE STRIKE (bug still
  1-1 pre-update); then aftermath/celebration, ØDEGAARD 1-2 caption @262.
Goals 2 and 3 read wrong because both windows are mostly aftermath, so
strike-implying narration plays over mostly-celebration.

## Audio (2026-09-08, Part 6)

Crowd ambience: `generate_ambience.py` was an orphan (produce_episode/cloud_produce
called it, produce_v2 did not). FIXED: produce_v2.py `step6b_ambience` (line
479) generates `crowd_ambience.mp3`; `merge_voice.py` mixes it under voice at
30% volume with fade. Verified on a temp slug.
Narrator voice: VOICE_ID is per-episode config. `generate_voice.py` accepts
`--voice-id <id>`. 3 candidate samples saved LOCAL at experiments/voice-test/audio/
for Mayo to pick. Current .env VOICE_ID = 1stSYyl7ZVPJk2ECrNlo.

## Numbers that were hand-typed, not measured

The old STATE.md's 7/10, 8/10, 9/10 vision quality scores were typed by hand.
No code produces a score. Do not cite them. `sharpness_check.py` exists and
could produce a real number but is not in produce_v2.py.