# STATUS.md — verified current state

Built 2026-09-05. Replaces the old STATE.md (which is gone). No plans, no
hopes, no hand-typed quality scores. Every claim has the command that proved it.

## What runs today

produce_v2.py end-to-end, exit 0, verified by the output on disk:

```
$ ffprobe renders/2026-08-30_liverpool-forest/shorts/final_video_shorts.mp4
width=720
height=1280
duration=64.200000
$ ls -la renders/2026-08-30_liverpool-forest/shorts/final_video_shorts.mp4
27.27MB  Sep 5 20:08
```

Runtime 6m31s and exit 0: UNVERIFIED for today's run (no log captured). The
CONTEXT.md brief states it; the output file's mtime (Sep 5 20:08) is
consistent with a run today but does not prove the runtime.

Steps that produced output today (by mtime in `renders/2026-08-30_liverpool-forest/`):
- match_data.json — Sep 5 20:02
- boards/ — Sep 5 19:17
- voice_elevenlabs.mp3 — Sep 5 19:18
- final_video.mp4 — Sep 5 20:07
- shorts/final_video_shorts.mp4 — Sep 5 20:08

## What is broken, worst first

### 1. Overlays are guessed, not tracked (broken quality)
`tactical_overlay.py` `generate_overlay_spec` (line ~120) sends the text
description to `glm-5.2:cloud` and asks for coordinates 0.0-1.0.
`draw_arrow` (line 58) stamps those fixed coords on every frame. No pixel
data is read. produce_v2.py calls this at line 232. The real tracker
(`cv_annotate.py`) is NOT called by produce_v2.py (grep confirmed).
UNVERIFIED today by a vision model; the CONTEXT.md brief carries a prior
vision-model verdict of "PowerPoint clipart, static markers."

### 2. Footage cuts ignore narration (broken logic)
`produce_v2.py:356`:
```
start = (seg.get("clip_idx", 0) * 5) % max(1, int(clip_total) - 5)
```
Fixed 5-second offsets indexed by `clip_idx`. The script's `[VISUAL: footage=]`
tags only select board-vs-footage (lines 300-301), not which part of the clip.

### 3. Script is not validated against match data (broken correctness)
No call to `validate_script.py` in produce_v2.py (grep confirmed). CONTEXT.md
example: script says "Gravenberch receives" but match data has him as a 71st-
minute sub. UNVERIFIED against today's match_data.json — not re-derived.

### 4. No upload has ever happened (broken delivery)
`publish-log/` is empty (`ls -la` confirmed, only `.` and `..`). Nothing calls
`youtube_upload.py` (grep confirmed, ORPHANED).

### 5. Output is 720x1280, not 1080x1920 (broken format)
ffprobe today: latest produce_v2.py output is 720x1280. shorts_crop.py
downscales deliberately (CONTEXT.md). One older output (iraola, Aug 29) is
1080x1920 but 493s and 135MB — a different/longer cut, not the v2 pipeline.

## Option C Stage 1: per-frame player positions — DONE (2026-09-05)

`cv_annotate.py` now exports `player_positions` in its tracking JSON: one
entry per processed frame, each with `{frame, players: [{id, bbox, team}]}`.
Existing exports (team_assignment, ball_positions) are intact.

Verified on RunPod (L4, 300 frames = 10s of 1920x1080 footage):
```
$ stat -c %s renders/2026-08-30_liverpool-forest/clips/clip_nxPNT4TU5_Q_annotated.tracking.json
1163475
$ ffprobe renders/2026-08-30_liverpool-forest/clips/clip_nxPNT4TU5_Q_annotated.mp4
width=1920 height=1080 duration=10.00
```
- 300 frames, 129 unique tracker IDs, 38 team_assignment entries
- Pod inference: 23s (0.077s/frame on L4 vs 1.30s/frame local)
- Pod cost: ~$0.05 (720s uptime at $0.25/hr L4)
- Tracker fragmentation is high: 129 IDs for 10s, ByteTrack loses and re-IDs
  constantly on the compressed wide-shot source. Tracker 37 survives 5 frames.
  CORRECTION (session 3): the 129 IDs span 3 camera shots (cuts at frames
  130 and 245). Within shots, tracking is usable: median consecutive run
  32 frames (1.07s), 72 IDs survive >25 frames. Shot 2 (frames 131-244)
  is cleanest: 35 IDs, median run 41 frames, 2 survive the full 3.8s shot.

No path from tracker ID to player name exists (see DECISIONS.md 2026-09-05,
question A). Stage 2 is arrows on unnamed tracked players, or hand-mapped.

Backup at `tools/cv_annotate.py.bak`. One-off runner at `tools/runpod_stage1.py`.

## Option C+E: segment selection + top-down tactical renderer (2026-09-05 session 3)

Direction picked after measuring the Stage 1 tracking data and analysing
options A-G against the RESUME_RESEARCH.md benchmark.

- **C (segment selection)**: `tools/segment_scorer.py` scores 1-second
  windows by detection, persistence, stability, and team classification.
  Validated on the 10s test data: 4/10 segments score >=65, correctly
  flags camera cuts (high new-ID rate) and close-ups (low detection).
  Needs full-clip tracking data (146s) to score all segments.
- **E (top-down tactical graphics)**: `pitch_radar.py` exists (195 lines)
  but is dead code, never imported by cv_annotate.py. Has screen-space
  fallback (no homography model needed). Needs expansion from 384x216 PIP
  to full-frame renderer with Bezier arrows and movement trails. Scores
  5/7 on the benchmark (layered depth, desaturated pitch, selective
  visibility, functional arrows, dark palette).
- Next: run cv_annotate on the full 146s clip on RunPod, then prototype E.

## C+E prototype status (2026-09-05 session 3 end)

**C (segment selection)**: DONE. `tools/segment_scorer.py` scores 1-second
windows by detection, persistence, stability, team classification. Run on
full 146s tracking data: 15/146 segments score >=65 (15s usable). Best
segment sec 1 (score 85.6), worst sec 45 (score 7.5).

**E (top-down tactical renderer)**: `tools/tactical_render.py` renders dark
pitch with mowing stripes, player dots in team colors, movement trails,
Bezier arrows. Outputs PNG or MP4. Screen-space projection (no homography).
Vision assessment: 8/10 then 8.5/10 across tasks 2-4 (Claude Opus 5,
authoritative — per coordinator). The earlier 5.5/10 was the session-3
PROTOTYPE before the context-layer / pitch-layout / movement-encoding fixes.
All elements visible (stripes, arrows, dots, trails). Local gemma4:cloud
rated higher but is not the judge of record.

**Known gaps for E (from Claude Opus 5 assessment)**:
1. No context layer: missing title, team names, ball marker, attacking
   direction, minute/phase caption.
2. Pitch layout: off-centre, missing 6-yard boxes, penalty spots, penalty
   arcs, corner arcs, goals. Stripe contrast too high.
3. Movement encoding: trail color conflicts with blue team dots, no visual
   hierarchy (all players equal weight), raw polylines instead of smoothed
   splines, no minimum arrow length filter for stubs.

**Full-clip tracking**: DONE. 146s on RunPod L4, 156s, $0.011.
`clip_nxPNT4TU5_Q_full.tracking.json` (8MB, 4380 frames).

- `cv_annotate.py` works and now exports per-frame positions. Called only by
  `produce_episode.py:288`, never by produce_v2.py. Verified by grep.
  RunPod end-to-end confirmed via `tools/runpod_stage1.py` (Sep 5).
- `runpod_annotate.py` exists but its end-to-end run is still UNVERIFIED
  (the one-off `runpod_stage1.py` proved the pod pattern works, not the
  multi-clip tool itself). See GAPS.md.

## Numbers that were hand-typed, not measured

The old STATE.md's 7/10, 8/10, 9/10 vision quality scores were typed by hand
(CONTEXT.md). No code produces a score. Do not cite them. `sharpness_check.py`
exists and could produce a real number but is not in produce_v2.py.

## Gemini video inventory assessment (2026-09-08) — footage-first idea

Tested whether Gemini can produce a timestamped footage inventory to drive a
script-first -> footage-first pipeline reversal. Result: half-works, and the
half that's wrong is the half that matters for cutting.

- GEMINI_API_KEY valid (prepaid AI Studio, $25 topped). 91 video tokens/sec
  @720p. ~$0.11/clip on gemini-3.1-pro-preview, ~$0.04 on gemini-3.6-flash.
  google-genai installed in ~/yt-digest/.venv (no breakage). 2.5 model family
  404-gone; only gemini-3.6-flash text is free, ALL video prepay-gated.
  Tool: tools/gemini_inventory_test.py (File API + inline modes).
- Content classification accurate (goals/celebration/replay/crowd/subs),
  confirmed by Claude on 18 frames.
- Timestamps NOT cut-accurate: boundaries 1-2s early, goal windows bloated
  (shot at front), wrong team in open play, 172 boundary wrong.
- WINNING GOAL mislocated ~60s: Gemini 250-254 is ARS 1-1 Chelsea attack;
  real Ødegaard 2-1 winner at ~305-315 (Gemini called it celebration+replay
  of a phantom). Found 2/3 real goals, missed the winner, invented 1 phantom.
- Player names work via jersey+lineup lookup (all 7 tested correct); no
  hallucination without lineup.
- ~5-6 usable action passages (~50s) in 482s; reel ~90% non-action. Video
  length capped by available action.
- DECISION: match_data.json is ground truth for events; Gemini is only a
  segment-finder. Never trust Gemini timestamps as cut points without
  match_data cross-check. Reel is partly fan-shot phone footage (Claude at
  250/330s) — a clean broadcast source may improve accuracy (unchecked).

## Scoreboard scanner — goal-finding by scoreline change (2026-09-08)

`tools/scoreboard_scan.py`: samples every 3s, crops the top-left score bug
(420x150), reads the scoreline with gemma4:cloud (vision_analyze.py, free),
carries last-known score across NONE (bug-absent) frames, detects changes.
Tesseract OCR rejected (0/3 frames readable). On the 482s arsenal-chelsea
clip: 161 frames, 76s, free. Saved clip_PrCW_geeRAU.scoreboard.json.

3 real scoreline changes found (after dropping single-frame blips that revert
<9s — "1-1 BUE"@213 and "2-4"@288 are gemma4 OCR blips):
- ~102s: 0-0 -> 0-1  Rogers (Chelsea)   [bug absent until ~99s; first 0-1 at 102]
- 171s:  0-1 -> 1-1  Havertz            [bug intermittent; value confirmed via
                                         HAVERTZ 1-1 caption @169 + bug @172.5]
- 261s:  1-1 -> 2-1  Ødegaard           [bug crop confirmed ARS 2-1 by Claude]
Matches match_data.json exactly (Rogers 2', Havertz 25', Ødegaard 50').

Walk-back from bug-update to the shot (Claude full frames):
- Chelsea:  bug 0-1 @102, shot ~99-101.   offset ~1-3s.
- Havertz:  bug 1-1 @171, shot ~163-166.   offset ~5-8s.
- Ødegaard: bug 2-1 @261, shot @253 ("Arsenal player shooting", still 1-1). offset 8s.
Offset is variable (1-8s). A fixed 8s pre-roll before each bug-update captures
every goal's shot; 1s walk-back gives the exact shot frame.

CORRECTION: the earlier "~305-315s ground truth" for the Ødegaard winner was
the CELEBRATION, not the goal. Scanner pins bug-update 261s, shot 253s — more
accurate than manual Claude-frame sampling (which missed 261 between the 252
and 315 samples).

Bug is intermittent in this reel (absent @90,99,168,258 — cuts to
fan/replay/celebration). gemma4 reads it correctly when present (102, 261
confirmed by Claude). Cost ~$0.10 (free bulk + ~7-15 Claude confirmations).
Scanner beats Gemini+Claude hybrid on accuracy (found the goal Gemini missed
at 250-254, no phantom) AND cost (~$0.10 vs ~$0.30-0.41). Persistence filter
now implemented in the script (run-based blip absorption, MIN_PERSIST=9s;
verified on the saved timeline — yields exactly the 3 real goals).

## Build one video: words match pictures (2026-09-08)

End-to-end cut-list-first build on Arsenal-Chelsea. No arrows, no renderer,
no crowd fix (boards + clean footage only). tools/assemble_words_match.py
(new) cuts footage at exact verified timestamps, replacing produce_v2's
fixed-5s-offset bug. Output final_video.mp4 45.2s, 1280x720.
PRIVATE: https://www.youtube.com/watch?v=WFi2LBwXINU

Step 7 (the never-run test) PASSES: relay on the mid-frame of each of 7
sections of the uploaded video — 6/7 clean match, 1 partial. All 3 goals
match narration via on-screen scorer captions (ROGERS 1-0, HAVERTZ 1-1,
ODEGAARD 1-2). The 8s pre-roll rule FAILED (offset varies in sign:
+17/+10/-8s); fix = search +-20s around each bug-update, relay-verified.
Gemini invented a goal in a celebration segment (discarded by relay) —
confirms Gemini-only is unsafe; cut-list-first + relay-verify works.
Possession board has a pre-existing data bug (39.3/32.7 vs stat_card
54.6/45.4), not fixed (renderer off-limits this build). Honest length 45.2s.

## Broadcast/filler signal (2026-09-08, Part 2 — verified)

The score bug is NOT a broadcast/filler signal. Mayo's independent frame check
found the uploader burned the bug in across ALL footage types (frame 261 =
fan-shot phone footage with the ARS 2-1 bug top-left). Re-checking what the
scanner's NONE reads correspond to (10 frames, gemma4, in
frames/2026-09-06_arsenal-chelsea/nonecheck_*.png on the mirror):
- NONE/PRE in the scanner = "gemma4 failed to OCR the bug", NOT "bug absent"
  (gemma4 re-read saw the bug at 150s/400s where the scanner said NONE).
- NONE frames are mostly BROADCAST (pre-match, in-play wide, post-match),
  not filler. Bug-present frames are MIXED (goals = broadcast, 200/261s =
  crowd/celebration filler). So bug presence does not split broadcast/filler.

Replacement signal: Gemini inventory content classification (action_type /
shot_type), already found accurate for WHAT. broadcast = {shot, build-up} with
shot_type != replay; filler = {non-action} or {replay}. Goal times cross-checked
against match_data + scoreboard timeline (NOT trusted from Gemini timestamps).
On the 482s reel: NEW signal = 43.0s broadcast (9%) / 440.0s filler (21 segments,
artifacts/broadcast_filler/ on the mirror). The old bug signal claimed ~210s
(44%), inflated ~5x by bug-burned-in celebration/crowd; the 43s matches the 45.2s
video actually produced. Scene-change alone is insufficient: cv_annotate
shot_boundaries has only 7 cuts for 482s and cuts do not classify type.
Tool: tools/broadcast_filler.py. Goal-finding is unaffected (scanner still works).

## Possession board bug (2026-09-08, Part 4a — fixed) + board ratings (4b)

Possession bug FIXED. Root cause was NOT a data misread (match_data has the
correct 54.6/45.4 and the code parses it correctly; the static possession.png
was already right). The bug was the ANIMATED possession.mp4: it filled the bar
over 75% of frames, so it broadcast intermediate values (e.g. 39.3/32.7 = 72%
of the real 54.6/45.4, with a 28% gap) for 3 of 4 seconds; the assemble step
loops/trims the mp4, so captured frames landed mid-fill. Fix in tactical_boards
_render_possession_animation: fill in first ~6% of frames, hold full ~94%,
label only with final values once full. Verified by Opus (54.6/45.4, full
width, sum 100). Backup: tools/tactical_boards.py.bak.

Board appearance RATED by Opus 5 (authoritative), NOT fixed (per instruction):
  formation 3/10, possession 4/10, stat_card 5/10. Common fails: matplotlib
  defaults, flat (no depth/shadows), default fonts (no Bebas/Barlow), no
  narrative furniture, stat-card bars both grow the same way (Arsenal should
  mirror). Verbatim responses: artifacts/board_ratings/boards_opus_ratings.md.
Source frames: frames/2026-09-06_arsenal-chelsea/board_*_full.png.

## Goal cut-window diagnosis (2026-09-08, Part 5)

Per-goal relay (gemma4) + Opus-authoritative read of the first/mid/last second
of each final cut window (frames at frames/.../goal_*.png):
- Rogers (Chelsea), 119-127 (8s): 100% celebration + ROGERS 1-0 caption, NO
  shot in the reel (Opus: "no goal shot whatsoever"; relay-verified search found
  none). Script's "finds the bottom corner" (a shot claim) FIXED to narrate the
  celebration: "gives the visitors a shock lead, and the travelling fans erupt."
  match_data fact kept, no shot claimed. Backup scripts/*.md.bak2.
- Havertz (Arsenal), 181-191 (10s): first second = ball-in-net aftermath (Opus:
  keeper beaten, ball in net, no player shooting) + HAVERTZ 1-1 caption; the live
  STRIKE is NOT in the window (~163-166, before it). ~1s goal-result + ~9s
  non-goal. Narration shows the result, not the strike.
- Odegaard (Arsenal), 253-262 (9s): first second (253) = LIVE STRIKE (Opus:
  player striking, bug still 1-1 pre-update); then ~6-8s aftermath/celebration,
  ØDEGAARD 1-2 caption @262. Strike is the first second.
Why goals 2 and 3 read wrong: both windows are mostly aftermath, so the
strike-implying narration plays over mostly-celebration. Goal 1 "matched"
because its window is coherently all celebration+caption (no mixed content),
even though it shows no strike. Video not re-rendered (narration fix applied
to the script; re-render is a separate pipeline run).