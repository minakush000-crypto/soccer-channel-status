# soccer-channel — combined status (auto-generated)

Every status doc concatenated on each push. Source of truth is the
individual files; this page is a single-fetch convenience. It does
NOT include the artifacts/ or frames/ trees — fetch those directly.


---

# FILE: CONTEXT.md

# CONTEXT.md — soccer-channel session brief

> **Purpose:** the session brief — goal, where things live, what's broken, measured facts.
> **Reader:** every session (CLAUDE.md @CONTEXT.md); mirrored to the public status repo.
> **Last verified against code:** 2026-09-09.

Verified against code on 2026-09-09 (Stage 2: produce_v2 capped at 720p, duration filter 180-1200s; Stage 3: broadcast_filler + produce_episode fixed). Drifted sections (the tactical_overlay
wiring, "no upload", the 8-step list, the cv_annotate export list) were
regenerated from grep; see ARCHITECTURE.md / STATUS.md / RECONCILIATION.md.

Read this at the start of every session instead of pasting the brief.
The project is ~/yt-digest/soccer-channel. Nothing else.

## THE GOAL
Produce soccer tactical analysis videos for a YouTube channel, matching
the quality of Coaches' Voice (dark pitch, orange accents, data-driven
graphics, functional arrows). Not Tifo, which needs a human illustrator.

## WHERE THINGS LIVE
The project is ~/yt-digest/soccer-channel. Nothing else.
~/retired/ holds two dead folders (soccer-pipeline, soccer-channel).
Never read or run anything in ~/retired/.
There used to be two folders named soccer-channel. That caused weeks of
confusion. Only the one inside yt-digest is real.

The .env is at ~/yt-digest/.env. yolov8s.pt is at ~/yolov8s.pt, outside
the project folder.

## STATUS MIRROR (public)
Current project state is mirrored to a PUBLIC docs-only repo so any
session (or the coordinator) can read it via web fetch instead of
pasting fragments.
Repo: https://github.com/minakush000-crypto/soccer-channel-status
Contains only: the seven docs (CONTEXT.md, STATUS.md, PROGRESS.md, GAPS.md,
DECISIONS.md, TOOLS.md, ARCHITECTURE.md) plus README.md and ALL_STATUS.md.
No code, no keys, no renders. A Stop hook (.claude/hooks/push_status.sh)
re-pushes these after any change, so the mirror is always current.

HOW TO READ IT (important — the github.com URL does NOT work for fetchers):
GitHub's github.com/.../blob/... HTML view is not reliably fetchable; it
returns the page chrome, not the content. Use the RAW host instead:
  https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/<FILE>
One-fetch full state (all seven docs concatenated):
  https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/ALL_STATUS.md
Index of every raw URL:
  https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/README.md
Caveat: WebFetch summarizes through a small model and can rewrite
headings, so it understands the state but is NOT a verbatim source. For
exact text or numbers, pull the raw bytes with curl (no auth needed).
Real Claude (Opus 5) confirmed all these URLs return HTTP 200 unauthenticated.

## ARTIFACTS and FRAMES (public, in the mirror repo)
The mirror repo also carries the results themselves, not just the doc
descriptions, so Claude can recompute and check the arithmetic:
  https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/artifacts/<type>/<file>
  https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/frames/<file>
- artifacts/ — small machine-readable JSON: scoreboard scan timeline + changes
  (artifacts/scoreboard/), Gemini footage inventory (artifacts/gemini_inventory/),
  YouTube publish-log entries (artifacts/publish-log/), and trimmed per-tracker
  tracking summaries (artifacts/tracking_summary/, NOT the 8MB per-frame files).
- frames/ — downscaled 640px-wide PNGs behind every visual claim, named by
  timestamp (e.g. frame_0253.png). Claude reads these directly; the relay's word
  is not needed when the frame is in the repo.
Both are staged in this project under artifacts/ and frames/ (gitignored here),
redacted (~ -> ~), secret-scanned, and synced to the mirror by the
same push_status.sh Stop hook. The mirror .gitignore allows only the two
subtrees plus the docs, and blocks renders/clips/.env/keys. Never put a
render, clip, or secret in artifacts/ or frames/. NOTE on the scoreboard
artifact: count goals from timeline[] (first occurrence of each new
scoreline), NOT from len(changes) — changes[] misses the first goal when the
bug was absent before it (see artifacts/scoreboard/README.md in the mirror).

## THE PIPELINE
Entry point: tools/produce_v2.py
Run it with: ~/yt-digest/.venv/bin/python tools/produce_v2.py <slug>
  --query "<match>" --date-range YYYYMMDD-YYYYMMDD
Working example: 2026-08-30_liverpool-forest --query "Liverpool Forest"
  --date-range 20260801-20260831

9 steps (verified 2026-09-08, see ARCHITECTURE.md): match data (ESPN)
-> boards -> download clip (720p cap + 200MB guard, 3-20min filter) -> tactical
render (runpod_fulltrack ships cv_annotate to RunPod, then tactical_render.py
draws the top-down view) -> voice (ElevenLabs) -> crowd ambience -> assemble ->
merge -> shorts crop. Latest produce_v2 output: 720x1280, 62.3s, 24.8MB
(liverpool-forest, Sep 7). Source is now 720p (Stage 2: height<=720 at
produce_v2.py:176; duration 180-1200s at :141).
A separate words-match path (assemble_words_match.py) built arsenal-chelsea:
1280x720, 45.2s, 15.0MB (Sep 8).

tools/produce_episode.py is an older entry point. It calls cv_annotate.py
at line 288. Do not break that. It is NOT reachable from produce_v2.py.

## WHAT WAS FIXED (verified)
FIX 1: produce_v2.py was downloading 360p clips and accepting them
silently. It now loops up to 5 candidates, ffprobes each, rejects below
720p, uses cookies from secrets/yt_cookies.txt, and fails loudly rather
than falling back. Verified: source is now 1920x1080.
Backup at tools/produce_v2.py.bak.

## WHAT IS STILL BROKEN, WORST FIRST
1. Footage does not match narration. produce_v2.py line 433 cuts at
   fixed 5-second offsets (`clip_idx*5`). The script's [VISUAL: footage=]
   tag is treated as a boolean (line 361), never as a timestamp window.
   assemble_words_match.py holds the content-matched fix but is NOT folded
   in — blocked by missing timestamp-window data + schema mismatch
   (RECONCILIATION 1.2). A cut-list generator is a prerequisite for every
   footage lane.
2. The script is factually wrong. It says "Gravenberch receives" but
   match data shows he is a 71st-minute sub for Frimpong. Nothing
   validates the script against match data (grep -c validate_script
   tools/produce_v2.py -> 0).
3. Output is 720x1280. shorts_crop.py downscales deliberately to avoid
   a soft 1.78x upscale to 1080x1920.
4. No upload step in produce_v2.py. youtube_upload.py is invoked by hand;
   2 private uploads happened 2026-09-06 (artifacts/publish-log/ has 2
   entries). produce_v2 does not wire upload.
RESOLVED (was #1): the tactical_overlay "PowerPoint clipart" problem is
gone. produce_v2.py no longer calls tactical_overlay.py (grep exit 1);
the step is now step4b_tactical_render (runpod_fulltrack + tactical_render,
real tracking, Opus 8/8.5). tactical_overlay.py is DEAD.

## MEASURED FACTS, DO NOT RE-DERIVE
- cv_annotate.py (YOLOv8 + ByteTrack + KMeans team classification) works
  and produces real tracking. A vision model called its output
  "professional broadcast tracking." produce_v2.py does NOT call it locally;
  runpod_fulltrack.py ships it to RunPod and runs it on the pod (line 72,91).
- Local speed: 1.30s/frame at 1080p. 900 frames = 19.5 min. Full clip
  = 95 min. Too slow.
- roboflow/sports was evaluated and REJECTED. Its football-specific
  model found the ball in 4 of 12 frames vs 3 of 12 for generic COCO.
  Not a meaningful gain. Its PLAYER_DETECTION exceeded 2s/frame and its
  BALL_DETECTION with InferenceSlicer was 14x slower than baseline.
- BALL DETECTION IS DEAD. 8 of 12 frames had no ball from either model.
  The source is a compressed wide-shot reupload. Drop every
  ball-dependent feature: no ball trail, no ball-following crop, no
  arrows tied to the ball. Anchor to PLAYERS ONLY.
- cv_annotate.py exports team_assignment, ball_positions, AND per-frame
  player_positions (Stage 1 done 2026-09-05: {frame, players:[{id,bbox,team}]}).
  No path from tracker ID to player name exists (no jersey OCR, no mapping).
- Cloud: RunPod and Vast.ai both authenticate. runpod_annotate.py is the
  tool for tracking; it tarballs tools/ and ships it, so edits propagate
  automatically. Roughly $0.15 and 5 minutes per episode on an L4.
- The vision quality scores in the old STATE.md (7/10, 8/10, 9/10) were
  typed by hand. No code produces them. They are not a benchmark.
- Working vision check: ~/tools/vision_analyze.py, ~2.75s per
  frame. Use it to judge output instead of asserting quality.

## CONTENT RESEARCH
Three tools exist for topic research and freshness checks. None were
documented before 2026-09-05, which is why they stopped being used.

- tools/viral_angle.py — finds trending soccer topics with viral potential
  using the YouTube Data API. Searches for tactical analysis videos, then
  flags high-view videos from low-subscriber channels (proven demand, low
  competition). Run: ~/yt-digest/.venv/bin/python tools/viral_angle.py "topic"
- tools/agent_reach_research.py — multi-platform research layer. Fans out
  across exa (semantic web search), web (Jina Reader), github, twitter, and
  reddit. Degrades gracefully when a channel needs a login. Run:
  ~/yt-digest/.venv/bin/python tools/agent_reach_research.py "query"
- tools/fresh_fetch.py — freshness fetcher. Pulls dated sports news from
  RSS feeds and football-data.org API. Stamps every item with published
  time and age in hours. Exists because the old pipeline classified a
  completed transfer as a "rumor" using a stale article. Run:
  ~/yt-digest/.venv/bin/python tools/fresh_fetch.py

## THE BENCHMARK (from RESUME_RESEARCH.md lines 70-77)
1. Layered depth (drop shadows, gradients, z-ordering)
2. Desaturated pitch + high-contrast accents
3. Selective visibility (show only what matters)
4. Functional arrow language (tapered, Bezier, round caps)
5. Contextual cropping (zoom to the relevant zone)
6. Condensed athletic fonts (Bebas Neue, Barlow Condensed)
7. Dark muted palette
Current output violates 1, 3, 4, and 5.

## STANDING RULES
1. Work only in ~/yt-digest/soccer-channel. Print pwd at the
   start of every response.
2. CLOUD ONLY. This is an Intel N150 with no GPU. Any GPU work goes to
   RunPod. Local is allowed only for ls, grep, ffprobe, single-frame
   checks, reading files, and edits. Anything over 2 minutes locally,
   stop and ask me first.
3. Every factual claim names the command you ran and pastes its output.
   If you did not run a command, write "not checked."
4. Never print full API keys. Mask them (<masked>).
5. Git is the backup. The .bak files were deleted 2026-09-08 (superseded by
   git, RECONCILIATION Amendment 2). Do not recreate .bak; commit first if you
   want a rollback point.
6. One change, one run, one verification. Never batch changes.
7. Ask before guessing. If a required argument or path is unclear, stop
   and ask rather than assuming.

## DONE (2026-09-05 session 2)
- Documentation spine built: ARCHITECTURE.md, TOOLS.md, STATUS.md,
  DECISIONS.md, GAPS.md, SKILLS.md, PROGRESS.md.
- soccer-channel skill neutralized (both copies now RETIRED, no longer
  point at ~/soccer-pipeline).
- Option C Stage 1: cv_annotate.py exports per-frame player positions.
  Verified on RunPod (L4, 300 frames, 129 tracker IDs, 23s, $0.05).
  One-off runner at tools/runpod_stage1.py. (Pre-edit backups were .bak
  files, deleted 2026-09-08; git is the backup now.)
- Key finding: NO path from tracker ID to player name exists. No jersey OCR,
  no position heuristics, no manual mapping. match_data.py has jersey numbers
  but nothing reads them from video. Stage 2 is arrows on unnamed tracked
  players, or hand-mapped. Tracker fragmentation is high (129 IDs in 10s).

## NEXT TASK (2026-09-08)
The C+E prototype is done (tactical_render 8/8.5, full 146s tracking).
The lane expansion is planned in LANE_PLAN.md (four lanes A/B/C/D). Shared
prerequisite for every footage lane: a cut-list generator that emits
`[VISUAL: footage=START-END]` timestamp windows (scoreboard_scan + ±20s
relay-verified search) — no wired tool generates them today (RECONCILIATION
1.2). E iteration priorities from Opus remain open but are now lower than
the lane expansion and the cut-list generator.
See PROGRESS.md and STATUS.md for full detail.

## STANDING OPERATING DOCTRINE
1. NOTHING RUNS ON THE LOCAL MACHINE. No local downloads, except raw source
   staging on the USB flash drive at /mnt/f (staging area ONLY: download lands
   there, ships to the pod, gets deleted; never a working directory, because the
   vhdx never shrinks and /mnt/f frees normally). No local GPU work.
   All compute and all storage go to the cloud GPU providers.
2. No tasking without calling the available tools, skills, connectors, web
   fetch, research or MCPs where they apply.
3. No orphaned, standalone or untested item, tool or aspect of the pipeline
   may exist. Everything is wired, tested, or retired.
4. Every brief carries this doctrine as a footer.

The truthful gap list (which tools violate rules 1 and 3 today) lives in
DECISIONS.md under this same heading. Rule 1 is violated by the whole pipeline
(produce_v2 downloads + processes locally); rule 3 by 4 DEAD + ~18 untested
STANDALONE tools. Not yet fixed.

---

# FILE: STATUS.md

# STATUS.md — verified current state

> **Purpose:** verified current state of the pipeline (the status spine).
> **Reader:** every session (CLAUDE.md @STATUS.md); mirrored to the public status repo.
> **Last verified against code:** 2026-09-09.

Verified against code on 2026-09-09 (Stage 3: broadcast_filler phantom bug fixed 3A.1, produce_episode 800M guard 3A.4; Stage 2: 720p cap + 180-1200s filter on produce_v2.py). This rebuild supersedes the 2026-09-05
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
soft 1.78x upscale. produce_v2 now caps the source at **720p** (`height<=720`,
`produce_v2.py:176`, Stage 2), so the 1080p/2160p source path is gone —
`shorts_crop`'s 4K→1080x1920 downscale branch (comment, line 78) is now
unreachable through produce_v2. Output stays 720x1280.

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
~5x. Tool: `broadcast_filler.py` (FIXED 3A.1: now clips segments to the video
duration and discards past-end phantoms via --clip/--duration; the 43s and 192s
figures stand, the 18.7min figure corrected 644→373s). Goal-finding is
unaffected (scanner works).

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
## Stage 5 — pod-side download path + rule-3 debt, 2026-09-09

- NEW: tools/runpod_download.py (WIRED via --pod-download). Pod downloads +
  cuts excerpts; only guarded excerpts come home. Proven: 2/3 windows, 4MB each,
  128MB source never local, $0.005/run. YouTube bot-blocks pod yt-dlp (all
  clients) — --source-url (catbox) is the working path today.
- 4 DEAD RETIRED: tactical_overlay, pitch_radar, render_video, check_and_download
  deleted (git history); pitch_radar ship-list refs removed. Tool count 46 -> 42.
- 27 STANDALONE smoke test: 26/27 launch; luminance_pod crashes, runpod_stage1
  hangs. Rule 3 not closed per-tool (e2e owed).
- Rule 1: step3 (download) + step4b (tracking) now have pod paths; the CPU/storage
  stages (boards, assemble, voice, ambience, merge, shorts) + cut-list generation
  still run locally (DECISIONS.md gap list updated).
- 5A.5: 720p cap KEPT for the local path (pod yt-dlp bot-blocked, so 1080p-on-pod
  untested); pod path is 1080p-ready for when a proxy unblocks it.

## Stage 6 — /mnt/f staging + name overlap + beliefs closed + keys, 2026-09-09

- NEW tools: staging.py (/mnt/f staging), backup_env.py (encrypted .env backup),
  b2_upload.py (B2 archive, pending key). RETIRED 5: luminance_pod, runpod_stage1,
  runpod_annotate, vastai_shorts, runpod_shorts. Tool count 46 -> 37.
- Doctrine rule 1: +/mnt/f staging exception (4 docs).
- produce_v2: canonical-path assertion (6B.4) + step3 downloads to /mnt/f (6A).
- runpod_download: keyframe fix (stream-copy + re-encode fallback, timeout-bounded).
- 6C closures: OAuth valid, 95 Mbps upload, mirror redaction proven, runpod_annotate
  broken (retired), cut-list 2nd-source partial, 1200s estimate, SoccerNet reasoned.
- Open: 6A.2/6E.1 real /mnt/f proofs (pending mount: sudo mount -t drvfs F: /mnt/f),
  6E.2 real B2 upload (pending key), 6D.1 3/3 (pending non-throttled pod), cut-list
  on pod (6D.4), SCRIPT_TEMPLATE 4-template blocker.


---

# FILE: PROGRESS.md

# PROGRESS.md — chronological log

> **Purpose:** dated chronological log of what was done when.
> **Reader:** every session; mirrored to the public status repo.
> **Last verified against code:** 2026-09-09.

## 2026-09-05 session 1: documentation spine + skill fix

- Built CONTEXT.md, ARCHITECTURE.md, TOOLS.md, STATUS.md, DECISIONS.md,
  GAPS.md, SKILLS.md from code reading and command output, not intent.
- Neutralized the soccer-channel skill (both copies): frontmatter now says
  RETIRED, body redirects to ~/yt-digest/soccer-channel, all
  references to ~/soccer-pipeline deleted. Backups at *.bak.
- Added GAPS.md note: skill firing is model judgment, CONTEXT.md is the
  only reliable primer.

## 2026-09-05 session 2: Option C Stage 1

- Backed up tools/cv_annotate.py to tools/cv_annotate.py.bak.
- Added `player_positions` export to cv_annotate.py: one entry per processed
  frame, each with {frame, players: [{id, bbox, team}]}. Existing exports
  (team_assignment, ball_positions) intact. Syntax checked.
- Created tools/runpod_stage1.py: one-off RunPod runner for a single 10s clip.
  Uploads clip + tools to catbox.moe, creates webhook, runs cv_annotate.py
  --max-frames 300, uploads tracking JSON + annotated MP4, terminates pod.
  Known bug: webhook parser fails on unescaped newlines in track_preview.
- Ran on RunPod L4: 300 frames, 129 tracker IDs, 23s inference, $0.05 cost.
  Output: renders/2026-08-30_liverpool-forest/clips/clip_nxPNT4TU5_Q_annotated.tracking.json
  (1.16MB) + clip_nxPNT4TU5_Q_annotated.mp4 (1.5MB, 1920x1080, 10s).
- Confirmed: produce_episode.py:288 call to cv_annotate.py is NOT broken
  by the change (signature and CLI unchanged).
- Confirmed: NO path from tracker ID to player name exists in the repo.
  match_data.py has jersey numbers but nothing reads them from video frames.
  Stage 2 is arrows on unnamed tracked players or hand-mapped.
- Updated STATUS.md, DECISIONS.md, GAPS.md, CONTEXT.md.

## 2026-09-05 session 3: tracking measurement + direction pick (C+E)

- Ran three measurements on the Stage 1 tracking JSON (300 frames, 10s).
  Key finding: the "129 IDs" is from 3 camera shots, not pure fragmentation.
  The clip has shot boundaries at frames 130 and 245. Within each shot,
  tracking is usable: median consecutive run 32 frames (1.07s), 72 of 129
  IDs survive >25 frames, 7 survive >75 frames. Shot 2 (frames 131-244)
  is the cleanest: 35 IDs, median run 41 frames, 2 survive the full shot.
- Detection is healthy: median 20.0/frame (near 22), mode 21 (58 frames).
  Max 27 (mild referee/crowd contamination). Team classification only
  works in shot 1 (shots 2-3 are 100% team 2 = unclassified).
- Long-lived trackers cluster by shot. New-ID rate spikes at shot boundaries
  (22 new IDs in frames 130-139, 20 in frames 240-249) and is near zero
  in stable passages (0-2 new IDs/sec in sec 1, 5, 6).
- Analysed options A-G against the RESUME_RESEARCH.md benchmark (7 criteria).
  Option E (top-down tactical graphics) scores 5/7. No other option scores
  above 1/7. Options B (fix tracking) and D (jersey OCR) are losing fights
  on compressed wide-shot footage. Picked C+E: segment selection by tracking
  quality + top-down tactical renderer.
- Built tools/segment_scorer.py: standalone scorer that ranks 1-second
  windows by detection count, tracker persistence, new-ID rate, and team
  classification quality. Composite score 0-100, threshold 65. Validated
  on the 10s tracking data: 4/10 segments recommended, correctly flags
  camera cuts and close-up shots as low-quality.
- pitch_radar.py confirmed dead code: shipped to RunPod but never imported
  by cv_annotate.py. Only mention is a comment on line 370. It is a
  starting point for option E but needs major expansion.
- Next: run cv_annotate on the full 146s clip on RunPod to score all
  segments, then prototype the top-down tactical renderer.

## 2026-09-05 session 3 (continued): C+E prototype

- Ran cv_annotate on the full 146s clip on RunPod L4: 4380 frames,
  156s inference, $0.011 cost. Output: clip_nxPNT4TU5_Q_full.tracking.json
  (8MB). Fixed webhook bug (removed track_preview field that broke JSON
  on newlines). Runner: tools/runpod_fulltrack.py.
- Scored all 146 segments: 15/146 score >=65 (15s of usable footage).
  Best: sec 1 (85.6), sec 5 (82.5), sec 113 (72.5). Worst: sec 45 (7.5),
  sec 68 (6.5), sec 89 (6.5). Many segments are close-ups/replays with
  1-5 detections — correctly flagged as low-quality.
- Built tools/tactical_render.py: full-frame top-down tactical renderer.
  Reads tracking JSON, renders dark pitch with mowing stripes, player
  dots in team colors, movement trails, and Bezier arrows. Outputs PNG
  or MP4. Screen-space projection (no homography model needed).
- Vision assessment: Claude Opus 5 rates 5.5/10 (authoritative). All
  elements visible: stripes, arrows with arrowheads, team-colored dots,
  trails. Local gemma4:cloud rated higher but is not the judge of record.
  Claude identifies
  3 key gaps: (1) no context layer (title, team names, ball marker,
  attacking direction), (2) pitch layout off-centre with missing
  markings, (3) movement encoding needs hierarchy (all players equal
  weight, trail color conflicts with team dot color).
- Deliverables this session:
  - tools/segment_scorer.py (C: segment quality scorer)
  - tools/runpod_fulltrack.py (RunPod runner for full-clip tracking)
  - tools/tactical_render.py (E: top-down tactical renderer prototype)
  - renders/2026-08-30_liverpool-forest/clips/clip_nxPNT4TU5_Q_full.tracking.json
  - /tmp/tactical_seg1_8s_v2.png (proof-of-concept image, Opus 5.5/10)
  - /tmp/tactical_seg1_5s.mp4 (proof-of-concept video, 1280x720, 5s)

## 2026-09-05 session 4: capability audit, doc fixes, C+E risk analysis

- Rewrote soccer-channel/CLAUDE.md (43 lines, under 50): project path,
  standing rules, image rule, judge rule (Opus authoritative), tool routing
  table with 10 tools, @CONTEXT.md and @STATUS.md references. Backup at
  CLAUDE.md.bak.
- Fixed yt-digest/CLAUDE.md: soccer-channel skill marked RETIRED (was
  listed as active). Backup at CLAUDE.md.bak.
- Added CONTENT RESEARCH section to CONTEXT.md covering viral_angle.py,
  agent_reach_research.py, fresh_fetch.py. These tools existed but were
  undocumented, which is why they stopped being used. Backup at CONTEXT.md.bak.
- Removed 8/10 gemma4 score from STATUS.md and PROGRESS.md. Only 5.5/10
  (Claude Opus 5, authoritative) remains as the current assessment.
  Backups at STATUS.md.bak and PROGRESS.md.bak.
- Risk 1 DIAGNOSED: K-means fits once on first 60 frames (line 258,
  classification_threshold=60, confirmed by grep). classified=True after
  fitting, never re-fits. team_assignment has 38 entries (shot 1 only).
  1020 unique tracker IDs in full 146s clip. 982 IDs (96%) unclassified,
  default to team 2 (referee). Shots 2-14: 0% classified. This is a hard
  limitation without per-shot re-fitting or a different classification
  approach.
- Risk 1b ANSWERED: 2 of 15 segments scoring >=65 have working team
  classification (sec 1 at 85.6, team%=74; sec 3 at 65.3, team%=22).
  The other 13 have team%=0. E does not have enough two-team footage.
  The 15s of "usable" footage is really 2s of two-team footage.
- produce_v2.py confirmed: syntax OK (ast.parse), imports all standard
  library, no references to new files (segment_scorer, tactical_render,
  runpod_fulltrack, runpod_stage1). The four new files are standalone and
  do not affect produce_v2.py.
- MCP memory server graph is empty (read_graph returned no entities).
- Auto memory is DISABLED (CLAUDE_CODE_DISABLE_AUTO_MEMORY=1 in settings.json).

## 2026-09-06 session 5: capability enablement

### Crash recovery
- git status: branch master, 2 commits ahead, modified files are expected
  pipeline tools. Nothing broken by the crash.
- ls tools/*.bak: cv_annotate.py.bak, produce_v2.py.bak, produce_v2.py.bak2,
  runpod_stage1.py.bak all present.
- ast.parse(produce_v2.py): ok.
- Claude Code version: 2.1.263 (matches npm registry 2.1.263, update is current).

### Settings backup
- Prior session backed up to ~/.claude/settings.json.bak-sep6 (1071 bytes,
  Sep 5 23:51, pre-edit state).
- This session backed up to ~/.claude/settings.json.bak-sep6-session5
  (1768 bytes, Sep 6 00:05, post-prior-session state).

### Settings audit (what was off, what was already on)

Prior session already fixed:
- CLAUDE_CODE_DISABLE_AUTO_MEMORY=1 → removed. autoMemoryEnabled: true set.
- No PreToolUse hooks → 3 installed (block-retired, block-image-read,
  warn-local-gpu).
- bypassPermissions: already set (user's deliberate choice, not touched).

This session enabled (before → after, all in ~/.claude/settings.json,
affects ALL projects on this machine):
- enableWorkflows: false → true (enables the workflows feature)
- workflowKeywordTriggerEnabled: false → true (enables keyword-triggered
  workflow activation)
- awaySummaryEnabled: false → true (enables away summaries, generates
  session summaries when user is away)

Already on, not touched:
- effortLevel: xhigh, fastMode: true, autoCompactEnabled: true,
  switchModelsOnFlag: true, skipDangerousModePermissionPrompt: true,
  autoMemoryEnabled: true (prior session).

### Auto memory verified
- Memory directory: ~/.claude/projects/-home-muads-yt-digest/memory/
  (exists, was empty before this session).
- Wrote MEMORY.md (index) and soccer-channel-capability-enablement.md
  (first memory file). Both confirmed created by Write tool.
- Auto memory is functional: the setting enables the system prompt
  instruction to write memory files, and the Write tool successfully
  creates them in the correct directory.

### MCP memory graph decision
- claude-mem is the memory of record. It has 50+ project observations
  with semantic search, timeline, and corpus building. The MCP memory
  graph (read_graph) is empty and has a simpler entity-relation model.
- Decision: do NOT populate the MCP memory graph. It would duplicate
  claude-mem with a less capable query system and create a sync problem.
  claude-mem's search (mcp__plugin_claude-mem_mcp-search__search) and
  timeline tools are more powerful for this project's needs.

### Hooks: all 3 proven to fire live
1. block-retired.sh (matcher: Read|Edit|Write|Bash)
   - PROVEN: blocked a Bash command containing ~/retired/test.py.
   - Exit 2, error: "BLOCKED: ~/retired/ is off limits."
2. block-image-read.sh (matcher: Read)
   - PROVEN: blocked Read of /tmp/nonexistent-test.png.
   - Exit 2, error: "BLOCKED: Image/video file detected. glm-5.2:cloud
     cannot see images. Use: ~/tools/vision_analyze.py or ask_claude.py."
3. warn-local-gpu.sh (matcher: Bash)
   - PROVEN: script test outputs warning and exits 0. Live test with
     echo "yolo" allowed through (exit 0). Warning goes to stderr but
     is NOT surfaced to the model for exit-0 hooks (Claude Code
     framework behavior). The hook fires but its warning is invisible
     to the model in practice. This is a known limitation: exit 0 =
     allow, stderr swallowed. For visible enforcement, use exit 2.
   - Hook script content verified: checks for
     cv2|yolo|torch|cuda|bytetrack|annotate|inference|detect.py|
     segment_scorer|tactical_render, exempts commands containing
     runpod|ssh|pod|remote|cloud.

### @ syntax settled
- WebFetch on https://code.claude.com/docs/en/memory confirms:
  "CLAUDE.md files can import additional files using @path/to/import
  syntax. Imported files are expanded and loaded into context at launch
  alongside the CLAUDE.md that references them."
- @CONTEXT.md and @STATUS.md in soccer-channel/CLAUDE.md ARE working.
  Their full contents are in the session system prompt. A session-start
  hook to read them is unnecessary.
- Relative paths resolve relative to the file containing the import.
  Maximum import depth: 4 hops. Code spans and fenced code blocks are
  skipped.

### Parallel and background rules (prior session, verified in CLAUDE.md)
- Soccer-channel CLAUDE.md standing rules 8 and 9 already contain:
  "Batch independent tool calls in one response. Serialize only when
  one task's output is needed as input for the next." and "Use
  run_in_background for any command expected to take >2 minutes."
- yt-digest CLAUDE.md section 5 already contains subagent delegation
  pattern with Explore subagents.

### MCP server tests (all 6 reachable and functional)
1. github: search_repositories "soccer tactical analysis computer vision"
   → 8 results. Found SoccerVisionAI (YOLOv8+ByteTrack+player assignment)
   and ugo-soccer-analytics (YOLOv8+ByteTrack+homography+possession).
   Use: research existing soccer CV pipelines, borrow approaches.
2. brave-search: brave_web_search "soccer tactical analysis video pipeline"
   → Nature paper on Tactiformer/StratGaze + Catapult MatchTracker.
   Use: research tactical analysis methods and benchmark tools.
3. filesystem: list_allowed_directories → ~/yt-digest/soccer-channel.
   Use: file operations within project scope. Redundant with Read/Write
   tools but available if needed.
4. sequential-thinking: single thought returned successfully.
   Use: break down complex pipeline design decisions step by step.
5. ide: getDiagnostics → [] (no VS Code file open). Use: catch lint/type
   errors when editing in VS Code.
6. puppeteer: navigated to about:blank successfully. Use: YouTube upload
   fallback (assessed below).

### Puppeteer and YouTube upload assessment
- youtube_upload.py uses YouTube Data API v3 with OAuth2 (line 133:
  build("youtube", "v3", credentials=creds)). Token file:
  secrets/youtube_token.json (Aug 23, 13 days old). Code checks for
  expired credentials and refreshes via refresh_token (line 117).
- client_secret.json: Aug 23. yt_cookies.txt: Sep 5 (fresh).
- API path assessment: likely functional. Google OAuth refresh tokens
  are long-lived (don't expire unless revoked). The code handles token
  refresh automatically. The API path should be tried first.
- Puppeteer fallback assessment: technically possible (browser launches,
  can navigate). Could use yt_cookies.txt for cookie-based auth. But:
  YouTube has strong anti-bot measures (CAPTCHA, device fingerprinting).
  Upload UI selectors change frequently. Slower and more fragile than
  API. Cookie auth might trigger security challenges.
- Recommendation: try youtube_upload.py first. If OAuth refresh fails,
  Puppeteer with cookies is a viable fallback but should be a last
  resort. Do not attempt upload yet (per user instruction).

### Task tracking
- 6 tasks created (TaskCreate):
  #1 Fix team classification: re-fit K-means per camera shot
  #2 Add context layer to tactical renderer
  #3 Fix pitch layout: centre, complete markings, lower stripe contrast
  #4 Rebuild movement encoding with hierarchy
  #5 Wire tactical_render into produce_v2.py (blocked by #1-#4)
  #6 First YouTube upload (blocked by #5)
- Task tracking is worth using: the project has multiple sequential
  workstreams with dependencies. Tasks give the user visibility into
  progress and prevent skipping steps.

### Cron
- Available but NOT set up. Cron jobs are session-only (not persistent
  across sessions). This project uses short interactive sessions, so
  cron is not useful for long-term scheduling. For periodic content
  research (fresh_fetch.py, viral_angle.py), system cron or a dedicated
  scheduler would be better. Cron is available if a long-running session
  needs periodic checks.

### Restart guidance
- No urgent restart needed. All settings changes are already in effect:
  hooks proved firing, auto memory proved writing, @ imports proved
  loading. The installed version (2.1.263) matches npm registry (2.1.263),
  so the update is current.
- After restart, verify:
  1. Run /context — check CONTEXT.md and STATUS.md appear under Memory
     files (proves @ imports load).
  2. Try Read on a .png file — should be blocked by block-image-read hook.
  3. Run /memory — check auto memory folder shows MEMORY.md and
     soccer-channel-capability-enablement.md.
  4. grep autoMemoryEnabled ~/.claude/settings.json — should be true.
  5. git status in project — verify nothing broken.

## 2026-09-06 session 5 (continued): task 1 — per-shot team classification

### Validation before building
Checked 6 frames from the raw 146s source clip (clip_nxPNT4TU5_Q.mp4,
1920x1080, 4380 frames) via vision relay:
- Frame 30: crowd/stands shot (no players). Opus confirmed.
- Frame 120: crowd shot (no players). Scored 82.5 by segment_scorer
  but is crowd, not pitch. Segment_scorer is contaminated by crowd
  detections (YOLOv8 detects spectators as "person" class).
- Frame 1522: close-up of one sky-blue player (#4). Opus says this
  looks like Manchester City, not Nottingham Forest. Clip may be
  mislabelled.
- Frame 2500: goalkeeper close-up (teal jersey).
- Frame 3500: crowd shot (no players).
- Frame 3360: WIDE PITCH SHOT with both teams visible. Red vs light
  blue, 10+ players. This is the first confirmed wide shot.
- Team assignment from existing tracking JSON: shot 1 found 2 balanced
  clusters (team 0 = 16, team 1 = 14, referee = 8). K-means works
  when both teams are visible.
- Opus assessment: red vs light blue are ~155 degrees apart in hue,
  "one of the easier cases" for hue-based anchoring. Raw RGB centroid
  matching will NOT hold across shots (illumination shifts); must use
  HSV hue matching. Must sample torso ROI only (discard low-saturation
  and dark pixels). Must cluster inside person-detector bounding boxes,
  not on whole frame (crowd is red at Anfield).

### Changes to tools/cv_annotate.py (backup: tools/cv_annotate.py.bak2)
1. TEAM_COLORS: team 1 changed from green (50,200,50) to light blue
   (50,150,255) to match the actual away kit.
2. extract_jersey_color: added HSV filtering. Converts torso ROI to
   HSV, keeps only pixels with S>60 and V>50 (discards white shorts,
   dark shadows, pitch grass). Falls back to grey if <5 qualifying
   pixels.
3. classify_teams: rewritten to cluster on (cos(hue), sin(hue),
   saturation) instead of raw RGB. Hue is treated as circular (red
   at 0/180 wraps). Returns (team_assignment, team_hues) tuple where
   team_hues is [(team_id, mean_hue)] for cross-shot anchoring.
4. New function circular_hue_distance: handles hue wraparound.
5. New function _anchor_labels: compares current shot's team hues to
   reference hues (from first shot) by circular distance. Swaps team
   0/1 labels if the swapped assignment gives a smaller total hue
   distance. This fixes the label-flip problem.
6. Main loop: added shot boundary detection (>10 new tracker IDs in
   one frame after a stable passage). At each boundary: resets
   detection_frames, classified, and tracker_colors. Keeps
   team_assignment and reference_hues. Only collects colors for
   trackers NOT already in team_assignment.
7. Main loop: classification now calls classify_teams with
   reference_hues. First shot sets reference_hues. Subsequent shots
   are anchored to the reference.
8. Export: added shot_count and shot_boundaries to tracking JSON.
9. End-of-function: unique tracker count now uses total_trackers set
   (accumulates across all shots) instead of tracker_colors (which
   resets per shot).

### Changes to tools/runpod_fulltrack.py (backup: tools/runpod_fulltrack.py.bak)
Added --verbose flag to pod command, log file capture and upload, and
log download. This gives shot boundary detection messages and per-shot
classification output in the downloaded log.

### RunPod test (in progress)
Running cv_annotate on full 146s clip with --verbose. Output prefix:
clip_nxPNT4TU5_Q_full_v2 (preserves old tracking JSON for comparison).
Background task ID: bi6023uj1. Awaiting results.

### Bug fix: --verbose flag + BGR colors
First RunPod run failed: cv_annotate.py rejected --verbose (argparse
error, flag not defined). verbose defaults to True in annotate_clip()
but was never exposed as a CLI argument. Fix: removed --verbose from
the pod command in runpod_fulltrack.py (verbose is already on by
default). Also fixed BGR color bug: team 0 was (255,50,50) BGR = blue
not red, team 1 was (50,150,255) BGR = orange not light blue. Changed
to (50,50,255) BGR = red and (255,200,100) BGR = light blue. Both
files re-syntax-checked. Re-running on RunPod (task bhm4anz4p).

### Bug fix: axis parameter in extract_jersey_color
Second RunPod run crashed at line 117 (cv2.cvtColor channel error).
Root cause: region[mask] has shape (N, 3) but I used mean(axis=(0,1))
which collapses to a scalar, not a 3-element array. The original code
used region.mean(axis=(0,1)) on the full (H,W,3) region which correctly
gives (3,). Fix: changed to region[mask].mean(axis=0) which preserves
the 3 color channels. Pod ran 8s before crash. Re-running (third attempt).

### RunPod test 3: SUCCESS (2026-09-06)
Third run completed: exit 0, 4380 frames, 1020 trackers, 93MB output.
Log shows 11 shot boundaries detected, all shots 2-11 anchored.

VERIFIED RESULTS (analysis script on v2 tracking JSON):
- Shot count: 11 (old: 1, no metadata)
- Shot boundaries: [0, 131, 245, 1268, 1370, 2660, 3369, 3639, 3845, 3926, 4062]
- Total classified trackers: 379 (old: 38) — 10x improvement
- Team assignment: {0: 222, 1: 110, 2: 47}
- Frames with both teams: 1018/4380 (old: 129/4380) — 8x improvement
- ALL 11 shots have both teams classified

Log classification messages:
- Shot 1: Team A=25, Team B=12, Ref=1 (sets reference hues)
- Shot 2: Team A=6, Team B=13, Ref=5 (anchored)
- Shot 3: Team A=20, Team B=9, Ref=6 (anchored)
- Shot 4: Team A=57, Team B=5, Ref=4 (anchored, unbalanced — close-up)
- Shot 5: Team A=39, Team B=3, Ref=2 (anchored, unbalanced — close-up)
- Shot 6: Team A=14, Team B=28, Ref=10 (anchored)
- Shot 7: Team A=10, Team B=12, Ref=5 (anchored, balanced — wide shot)
- Shots 8-11: all anchored, 5-9 per team

Known limitation: crowd contamination. 222 team-0 trackers vs 110 team-1
suggests red Anfield crowd detected as team 0. Per-shot re-fitting
mitigates but doesn't eliminate this. Shots 4-5 are close-ups with
one team dominant (57v5, 39v3).

TASK 1 COMPLETE. Backup: tools/cv_annotate.py.bak2.

## 2026-09-06 session 5 (continued): task 2 — context layer

### Changes to tools/tactical_render.py (backup: tools/tactical_render.py.bak)
1. TEAM_COLORS updated to match cv_annotate.py BGR values (team 0 = red
   (50,50,255), team 1 = light blue (255,200,100)).
2. New function draw_context_layer: top bar with team names, color dots,
   attacking direction arrows, score (centred on canvas), minute (inline
   satellite, small grey), and bottom bar with tactical caption. Orange
   accent rules bracket the pitch top and bottom.
3. render_segment signature extended with home_team, away_team, score,
   minute_str, caption parameters. draw_context_layer called before
   frame write in both PNG and video paths.
4. CLI args added: --home-team, --away-team, --score, --minute, --caption.

### Opus assessment (12 iterations, judge of record)
- v1: 4.5/10 (basic layer, minute in wrong place)
- v5: 8/10 (enlarged arrows, differentiated minute)
- v11: 7.5/10 (inline minute on shared baseline)
- v12: 8/10 — "Yes, this is production-ready for the context layer.
  The remaining optical-centring question is a taste call, not a defect."

Key design decisions from Opus feedback:
- Score is the fixed center anchor (centred on canvas centre), minute
  hangs to the right as a satellite (14px gap, 0.65 scale, muted grey).
  This keeps the score stable regardless of minute width.
- All text elements share one baseline (y=31) for a single horizontal
  anchor line. Team names, score, and minute all align.
- Attacking direction arrows (13px, full-saturation team colors) sit
  between the color dot and the team name, mirrored left/right.
- Bottom bar holds a tactical caption (not the minute), with an orange
  divider rule matching the top bar.
- Bar height 52px, score thickness 2 (thickness 3 distorted the "0"
  glyph counter in OpenCV's HERSHEY_DUPLEX font).

Known limitation: OpenCV's built-in fonts lack tabular numerals, prime
characters, and en-dashes. A production system would use PIL/Pillow with
TrueType fonts for these typographic details.

TASK 2 COMPLETE. Backup: tools/tactical_render.py.bak.

## 2026-09-06 session 5 (continued): task 3 — pitch layout

### Changes to tools/tactical_render.py
1. Added pitch constants: PEN_SPOT_L=11m, GOAL_W=7.32m, CORNER_R=1m.
2. draw_pitch: added penalty spots (filled circles at 11m from each goal),
   penalty arcs (cv2.ellipse at ±53° from penalty spot, clipped to outside
   the penalty box), corner arcs (quarter circles at all 4 corners), goals
   (small rectangles outside the goal line at both ends).
3. Lowered stripe contrast: lighter shade changed from (34,64,44) to
   (24,46,33) — roughly 1.3x PITCH_COLOR instead of 1.9x.
4. Increased goal area thickness from 1 to 2 (was invisible at 35% alpha).
5. Increased goal thickness from 1 to 2.
6. Increased LINE_ALPHA from 0.35 to 0.50 (Opus suggested 0.55, went
   with 0.50 as a compromise to keep lines subtle but visible).

### Opus assessment (judge of record)
- Pitch-only render (no players): 8.5/10
- "Complete and correctly ordered element hierarchy; nothing overlaps
  incorrectly. Excellent left/right mirror symmetry. Penalty arcs are
  properly clipped so only the portion outside the 18-yard box shows."
- All markings confirmed visible: touchlines, halfway line, centre circle,
  centre spot, penalty areas, goal areas, penalty spots, penalty arcs,
  corner arcs, goals.
- The left goal area "invisibility" in earlier full renders was player
  occlusion + low alpha, not a drawing bug. Pitch-only test confirmed
  both goal areas are drawn correctly.

TASK 3 COMPLETE.

## 2026-09-06 session 5 (continued): task 4 — movement encoding

### Changes to tools/tactical_render.py
1. TRAIL_COLOR constant: (235,235,235) light grey for fallback/unclassified.
2. draw_trail: uses team colours at 0.6 alpha (via addWeighted) so trails
   show team ownership while remaining visually distinct from solid dots.
   Width increased to 3px. Path simplified with _simplify_path before drawing.
3. New _simplify_path: removes vertices where turn > 140 degrees (kills
   hairpin artifacts from sharp direction reversals in tracking data).
4. New _should_draw_glyph: gates on net displacement >= 25px AND
   net/arclength ratio >= 0.55 (filters hairpins where player returns
   to near origin).
5. draw_bezier_arrow: minimum length increased from 8 to 15px.
6. Combined trail+arrow loop in PNG path: ensures consistency (both drawn
   or neither). Arrow uses simplified path endpoints. Arrow tip inset
   by DOT_RADIUS + ARROW_HEAD + 2 = 28px past the dot. Minimum 43px
   displacement for any glyph (inset + 15px arrow minimum).
7. Video path: trail loop uses _should_draw_glyph for the same filtering.

### Opus assessment (7 iterations, judge of record)
- v1: 6/10 (two-tone trails, spline overshoot)
- v5: 6.5/10 (team colours, ratio filter)
- v7: 8.5/10 — "hairpins and stubs are genuinely eliminated and the
  both-or-neither trail/arrow pairing is consistent"

Key design decisions from Opus feedback:
- Team-coloured trails at 0.6 alpha (not pure grey, not full saturation)
- Straight polylines (Catmull-Rom splines caused overshoot artifacts on
  sharp direction reversals; straight lines are cleaner for tracking data)
- Per-vertex angle simplification (drop vertices with > 140 degree turns)
- Net/arclength ratio filter (0.55 threshold catches hairpins)
- Combined trail+arrow loop (both drawn or neither, no incomplete glyphs)
- Arrow tip inset past dot radius so arrowheads are never hidden under dots

TASK 4 COMPLETE.

## 2026-09-06 session 5 (continued): task 5 — wire tactical_render into produce_v2.py

### Changes to tools/produce_v2.py (backup: tools/produce_v2.py.bak3)
1. New step4b_tactical_render function: checks for existing tracking JSON
   (prefers _v2 with per-shot classification), runs runpod_fulltrack.py if
   none exists, finds the shot with the most balanced team classification
   (team 0 vs team 1 tracker counts), runs tactical_render.py with team
   names and score from match_data.json.
2. step5_assemble: accepts tactical_path parameter, handles "tactical"
   visual tags ([VISUAL: tactical] in the script), adds "tactical" segment
   type to the concat builder (scales 1280x720 to 1920x1080 with pad,
   same filter as board segments).
3. main(): loads match_data before step4b (was loaded after voice), calls
   step4b_tactical_render, passes tactical_path to step5_assemble.

### Verification
- produce_v2.py syntax ok (ast.parse)
- step4b shot balancing logic verified: picks shot 7 (frame 3369, 112.3s,
  balance=0.09, 10 red vs 12 blue trackers) — the most balanced two-team
  shot in the 146s clip.
- tactical_view.mp4 produced: 1280x720, 3.0s, 1.1MB
- ffmpeg scale+pad verified: 1280x720 → 1920x1080 with #0d1f16 padding
  (same filter as board segments, which already work in the pipeline)

The C+E pipeline is now wired into produce_v2.py:
  step3 (download) → step4 (overlay) → step4b (tactical render) →
  step5 (voice) → step6 (assemble, includes tactical segments) →
  step7 (merge) → step8 (shorts crop)

TASK 5 COMPLETE.

## 2026-09-06 session 5 (continued): task 6 — YouTube upload assessment

### Assessment (no upload attempted, per user instruction)
- Token: secrets/youtube_token.json has refresh_token (✅). Access token
  expired 2026-08-24 but refresh tokens are long-lived (don't expire
  unless revoked). Code auto-refreshes at line 117.
- Script: tools/youtube_upload.py syntax ok (ast.parse).
- Packages: google-auth-oauthlib, google-auth, google-api-python-client
  all installed in ~/yt-digest/.venv.
- Publish log: publish-log/ is empty (no prior uploads, confirmed by ls).
- API path: READY. No blockers identified. The upload can be attempted
  when the user is ready.
- Puppeteer fallback: NOT needed. The API path should work. Puppeteer
  remains available as a backup if the OAuth refresh fails (assessed in
  capability enablement: browser launches, can navigate, but YouTube
  anti-bot measures make it fragile).

### What would need to happen for the first upload
1. Run: ~/yt-digest/.venv/bin/python tools/youtube_upload.py <slug>
2. The script reads match_data.json for title/description/tags.
3. It refreshes the OAuth token automatically.
4. It uploads the final video (shorts/final_video_shorts.mp4).
5. It logs the upload to publish-log/.
6. The user should verify the video is public and set the thumbnail.

TASK 6 COMPLETE (assessment only, no upload attempted).

## 2026-09-07: three questions answered

### Q1: Match identity — SETTLED
- yt-dlp confirms: title "Liverpool ( 2 - 2 ) Nottingham", tags include
  #nottinghamforest, uploader "footzonex7", upload date 20260830.
- Opus frame 1 (1522): sky-blue #4 kit, "classic Manchester City home
  kit". Crowd in red and white. FOOTZONE watermark. No scoreboard.
- Opus frame 2 (2500): Liverpool goalkeeper (Alisson), Liverbird crest,
  Standard Chartered sponsor, Expedia sleeve. No scoreboard.
- Opus frame 3 (3360): sky blue vs red, scoring moment. No scoreboard.
- Verdict: match IS Liverpool vs Nottingham Forest (confirmed by video
  title and match_data.json). The sky-blue kit is Forest's away kit, not
  Man City. Opus misidentified it because sky blue is City's classic
  color. No scoreboard visible in any frame, so score can't be
  independently verified from footage. Residual risk: step3 downloads
  the first 720p+ clip from YouTube search without verifying the
  footage matches the claimed match.

### Q2: Crowd contamination — QUANTIFIED
- 379 classified trackers: 200 pitch (53%), 51 crowd (13%), 128 edge
  (34%).
- Team 0 (red): 94 pitch, 33 crowd, 95 edge = 222 total.
- Team 1 (blue): 76 pitch, 10 crowd, 24 edge = 110 total.
- The 10x gain (38 to 379) is real but inflated: pitch-only gain is 5x
  (38 to 200). Crowd contamination is mostly team 0 (red Anfield crowd
  classified as red team). Some large-bbox trackers (area >400K) are
  close-ups of single players/coaches, not crowd.

### Q3: Capability check — HONEST AUDIT
- Parallel tool calls: YES, used extensively.
- Background tasks: YES, 3 RunPod runs + pipeline run.
- Subagents: NO. Should have used Explore for produce_v2.py reading.
- Sequential-thinking: NO. Should have used for label-flip analysis.
- PostToolUse PROGRESS.md hook: NO. Does not exist in settings.json.
  Manually updated PROGRESS.md with Edit calls.

## 2026-09-07: tasks 4 and 5 — pipeline run and YouTube upload

### Task 4: produce_v2.py end-to-end with step4b
- Pipeline ran: 300.8s (5.0 min), exit code 0.
- Step 1 (match data): 1.9s. Step 2 (boards): 89.5s. Step 3 (download):
  skipped (1080p clip cached). Step 4b (tactical render): 6.8s, best shot
  7 (112.3s, balance=0.09). Step 5 (assemble): 5 sections including [3]
  Tactical View (tactical tag, 8.7s of 62.3s total). Step 7 (merge):
  3.5s. Step 8 (shorts): 77.9s.
- Output: final_video.mp4 (1920x1080, 62.28s, 43.2MB, h264+aac).
  Shorts: final_video_shorts.mp4 (720x1280, 62.27s, 24.8MB).
- Copied to /mnt/c/Users/muads/Downloads/2026-08-30_liverpool-forest_Short.mp4.

### Task 5: YouTube upload (first ever)
- First attempt: OAuth refresh_token expired/revoked (token from Aug 23,
  Google "testing" mode expires after 7 days). Crash: RefreshError not
  caught. Fix: wrapped creds.refresh() in try/except, falls through to
  OAuth flow. Backup: tools/youtube_upload.py.bak.
- Second attempt: OAuth flow started, printed authorization URL, timed
  out (user hadn't opened URL in time — run_local_server default ~5min
  timeout).
- Third attempt: user ran with `!` prefix, authorized in Windows
  browser, token saved at 00:15:40.
- Fourth attempt (fresh token): upload succeeded. Exit 0.
  Output: "Upload complete: https://www.youtube.com/watch?v=IVUg5oZLH5k
  Video is PRIVATE. Review and publish in YouTube Studio."
- Video URL: https://www.youtube.com/watch?v=IVUg5oZLH5k
- Privacy: PRIVATE (user publishes manually).
- Title: "Liverpool Forest — Soccer Tactical Analysis"
- Tags: tactical, analysis, tactics, forest, football, soccer, liverpool
- File uploaded: final_video.mp4 (41MB, 1920x1080, 62.3s)
- publish-log/ is empty (script prints URL to stdout, does not write
  a log file — minor gap for future tracking).
- FIRST EVER YouTube upload for the soccer-channel project.
## Session 2026-09-07: Structural overhaul (step4 removal + re-balance + crowd filter + new match)

User verdict: "The video is bad and the reason is structural. Tactical View
is 8.7s of a 62.3s video. The other 53.6s is step4's LLM-guessed arrows
(crayon scribbles). Stop adding. Start replacing."

### Current section breakdown (verified from pipeline output)
- [0] Hook: 30 words, 8.7s, board=formation (14%)
- [1] Setup: 57 words, 16.5s, board=possession (26%)
- [2] Point 1: 72 words, 20.8s, footage=buildup (33%) ← step4 overlay
- [3] Tactical View: 30 words, 8.7s, tactical [actual 3s] (14%)
- [4] Close: 33 words, 9.5s, board=stat_card (15%)
- Total: 222 words, 64.2s. Tactical: 3s/62.3s = 4.8%.

### Changes made (6 edits, all syntax-verified)

1. **REMOVE step4** (produce_v2.py, backup: .bak4):
   - Deleted step4_overlay function (was lines 202-239).
   - Removed call in main() (was lines 559-560).
   - Changed step5_assemble to receive clip_path (raw) not annotated_clip.
   - grep confirms: no step4_overlay, annotated_clip, or tactical_overlay
     references remain in produce_v2.py.
   - tactical_overlay.py now orphaned (no pipeline caller).

2. **RE-BALANCE** (produce_v2.py):
   - step4b: --duration "3" → dynamic min(45.0, clip_dur - start_sec).
   - step5_assemble: tactical cap min(3.0, sec_duration) → min(sec_duration, 45.0).
   - New script (scripts/2026-09-06_arsenal-chelsea.md): 4 sections,
     223 words, 65.5% tactical (146 words). Target: 60%+. Confirmed.

3. **FILTER CROWD** (cv_annotate.py, backup: .bak3):
   - Added bbox size/position filter before tracker.update_with_detections.
   - Rejects: area < 500 (far crowd), > 50000 (close-up), cy < 80 (top
     stands), cy > h-80 (bottom edge). Persons only, balls pass through.
   - Verification RunPod run in progress on old Liverpool-Forest clip.

4. **NEW MATCH**: Arsenal 2-1 Chelsea (Sep 6 2026, PL).
   - fresh_fetch.py run (briefs/fresh/2026-09-07_0539_fresh.md).
   - Arsenal 4-2-3-1 vs Chelsea 3-4-2-1. 54.6% possession, 16 shots.
   - Slug: 2026-09-06_arsenal-chelsea. Query: "Arsenal Chelsea".

5. **OAUTH FIX** (user action, no code): Google Cloud Console →
   APIs & Services → OAuth consent screen → Publish App. Testing mode
   = 7-day refresh expiry; production = no expiry.

6. **PUBLISH LOG** (youtube_upload.py, backup: .bak2):
   - Added from datetime import datetime.
   - Writes JSON to publish-log/{slug}_{video_id}.json after upload.

### End-to-end pipeline run (in progress)
- produce_v2.py 2026-09-06_arsenal-chelsea --query "Arsenal Chelsea"
  --date-range 20260901-20260930
- Background task, output: /tmp/pipeline_arsenal_chelsea.txt

### Bug fixes during pipeline run (2026-09-07)

1. **AV1 codec crash**: First pipeline run downloaded an AV1-encoded clip.
   The RunPod pod (L4) cannot decode AV1 (no hardware decoder, software
   decoder fails with "Missing Sequence Header"). Result: 0 frames
   processed, 0 trackers, 0KB tracking JSON, tactical render skipped.
   Fix: changed yt-dlp format in step3 to prefer H.264 (vcodec^=avc1)
   before falling back to other formats.
   Verification: pending re-run.

2. **OUT_DIR hardcoded**: runpod_fulltrack.py line 18 had OUT_DIR hardcoded
   to the Liverpool-Forest clips directory. Tracking JSON downloaded to
   the wrong render folder. Fix: OUT_DIR = clip_path.parent (derive from
   the --clip argument). Backup: runpod_fulltrack.py.bak2.

3. **Step4b timeout**: Increased RunPod tracking timeout from 600s to 1200s
   for longer clips (482s clip has 14,472 frames).

### End-to-end pipeline result (2026-09-07 01:12)

Pipeline: 2026-09-06_arsenal-chelsea --query "Arsenal Chelsea"
--date-range 20260901-20260930

**Output verified:**
- final_video.mp4: 1920x1080, h264+aac, 65.17s, 12MB
- shorts: 720x1280, h264+aac, 65.18s, 8.2MB
- tactical_view.mp4: 1280x720, 45.0s, 1350 frames
- step4_overlay references in produce_v2.py: 0 (fully removed)

**Section breakdown (with board fix):**
- Hook: 17 words, 5.7s, board=formation
- Setup: 20 words, 6.7s, board=possession
- Tactical Analysis: 146 words, 48.5s, tactical (45s view + 3.5s footage)
- Close: 13 words, 4.3s, board=stat_card
- Total: 196 words, 65.2s. Tactical view: 45s/65.2s = 69%

**Crowd filter verification (Arsenal-Chelsea 720p):**
- Total trackers: 3248, Pitch: 3248 (100%), Edge: 0, Crowd: 0
- Team assignments: 285 (all real players)

**YouTube upload:**
- URL: https://www.youtube.com/watch?v=PsEz5ITTIpM
- Privacy: PRIVATE
- Publish log: publish-log/2026-09-06_arsenal-chelsea_PsEz5ITTIpM.json
- First publish-log entry ever (directory was empty before)

**Bug fix: board duration (4th fix):**
- step5_assemble subtracted 4.0s for boards but only played
  min(4.0, sec_duration*0.5). Video was 59.6s vs voice 65.2s —
  the entire Close narration was cut off. Fixed: subtract actual
  board_dur. Video now 65.2s, matching voice.

**Total cost:** ~$0.04 RunPod (2 runs: $0.009 verification + $0.030
Arsenal-Chelsea tracking) + ElevenLabs voice + YouTube API (free).

## 2026-09-08 session: Gemini video inventory assessment (footage-first idea)

User hypothesis: reverse the pipeline, footage inventory first (Gemini
video understanding), then script from that inventory cross-referenced
with ESPN match data, then cut at named timestamps. Assessment only,
no pipeline built.

Setup:
- GEMINI_API_KEY in ~/yt-digest/.env authenticates (urllib GET
  /v1beta/models -> 50 models). Prepaid AI Studio key; was depleted
  (429 "prepayment credits are depleted"), user topped $25, then works.
- Installed google-genai into ~/yt-digest/.venv (no breakage: yt_digest,
  openai, requests all still import). tools/gemini_inventory_test.py
  written (File API upload + inline-bytes mode, model fallback). Backup
  at tools/gemini_inventory_test.py.bak.
- 2.5 model family is 404-gone ("no longer available to new users",
  redirects to gemini-3.6-flash). Only gemini-3.6-flash TEXT is free;
  ALL video (even 3s) is prepay-gated. Used gemini-3.1-pro-preview after
  credit.

Measured cost (usage_metadata, not pricing-page guesses):
- 91 video tokens/sec at 720p (43862 video tok / 482s; 4095/45s matches).
- gemini-3.1-pro-preview: ~$0.11/full-clip pass. gemini-3.6-flash: ~$0.04.
- $25 ~ 220 pro passes.

Results on renders/2026-09-06_arsenal-chelsea/clips/clip_PrCW_geeRAU.mp4
(482s, 1280x720, partly fan-shot per Claude at 250/330s):
- Inventory: 22 passages, clean JSON, saved to
  clip_PrCW_geeRAU.gemini_inventory.json. Content labels (goal/celebration/
  replay/crowd/subs/lineup) accurate, confirmed by Claude on 18 frames.
- Timestamps NOT accurate enough to cut on:
  * Pre-match boundaries 1-2s early (banner at 5.0 but 5.5 still goal;
    Chelsea lineup at 23.0 but 23.5 still banner; Arsenal at 33.0 but
    33.5 still Chelsea blue 21/45).
  * Goal windows bloated: shot at front, midpoint is post-goal restart
    (Chelsea 99-111: 105s board 0-1 restart; Havertz 161-172: 166s
    HAVERTZ 1-1 graphic, restart).
  * Wrong team in open play: "Chelsea advances" 88-99, but 93s is Arsenal.
  * 172 boundary wrong: Gemini says replay, 172.5s is live build-up 1-1.
  * WINNING GOAL mislocated ~60s: Gemini "Arsenal scores" 250-254, but
    250/251/252s is ARS 1-1 Chelsea attacking. Real winner Ødegaard
    (ARS 2-1, "ØDEGAARD 1-2" graphic) at ~305-315s, which Gemini labelled
    "celebration"+"replay" of a phantom goal. Found 2 of 3 real goals
    (Chelsea, Havertz), missed Ødegaard, invented 1 phantom.
- Player names WORK via jersey lookup: with match_data lineup, mapped
  10=Palmer,17=Rogers,9=Joao Pedro,3=Fofana,34=Acheampong,45=Lavia,
  21=Hato (all correct). Without lineup, reads numbers only, no guesses.
  Both name-test responses truncated ~80 output tokens (finish_reason
  unchecked).
- Usable passages: ~5-6 action moments (~50s) in a 482s reel; ~90% is
  celebration/crowd/replay/ceremony. Video length capped by available
  action. One 63s celebration block confirmed real (200/220/240s).

Verdict: footage-first half-works. Gemini good at WHAT (content, jerseys),
bad at WHEN (timestamps, team, goal grounding; does not track scoreboard).
match_data.json (real goals+scorers+scoreline) MUST be ground truth;
Gemini only the segment-finder, never the event-source. Do not trust
Gemini timestamps as cut points without match_data cross-check.

Cost this session: ~$0.45 of $25 (pro full-clip $0.11 + ~30 Claude frame
judgements ~$0.30 + small inline calls).

## 2026-09-08 session 4: scoreboard scanner + status mirror

PART 1 — status mirror (public docs repo):
- soccer-channel is a SUBDIR of the private yt-digest repo, not its own
  repo; the 7 docs were untracked (never in any remote).
- Created PUBLIC repo minakush000-crypto/soccer-channel-status, docs only
  (7 files + .gitignore). Scanned all 7 for secrets BEFORE push: only
  ~ paths + masked-key example; redacted on copy (~->~,
  <masked>-><masked>). Verified clean on remote.
- Automated: .claude/hooks/push_status.sh (idempotent, redacts, no-ops,
  exit 0) wired as 2nd Stop hook in ~/.claude/settings.json. Proven: manual
  run committed + pushed. URL written into CONTEXT.md.

PART 2 — scoreboard scanner (goal-finding by scoreline change):
- Built tools/scoreboard_scan.py. Goal-finding as a scanning problem (read
  the broadcast score bug), not model judgement. gemma4:cloud reads the
  top-left score bug every 3s (free, 0.25s/frame, 4 workers); tesseract OCR
  rejected (0/3 readable). NONE (bug-absent) frames carry last-known score.
- 482s arsenal-chelsea clip: 161 frames, 76s, free. 3 real scoreline changes
  (after dropping 1-frame blips "1-1 BUE"@213, "2-4"@288):
  102s 0-0->0-1 Rogers; 171s 0-1->1-1 Havertz; 261s 1-1->2-1 Ødegaard.
  Matches match_data.json exactly (Rogers 2', Havertz 25', Ødegaard 50').
- Walk-back (Claude full frames) to the shot: Chelsea offset ~1-3s, Havertz
  ~5-8s, Ødegaard 8s (bug 261, shot 253 "Arsenal player shooting" still 1-1).
  Offset variable 1-8s; fixed 8s pre-roll captures all 3.
- Corrected my own earlier error: "~305-315s" Ødegaard winner was the
  CELEBRATION, not the goal. Scanner pins bug-update 261s, shot 253s.
- Confirmations (ask_claude): 102 ARS 0-1 CHE (match), 261 ARS 2-1 CHE
  (match). 171 value confirmed via HAVERTZ 1-1 caption + bug @172.5
  (bug intermittent, ±1-2s on exact frame).

Answers to coordinator Q5-8:
- Q5 Scanner vs Gemini+Claude: scanner wins on accuracy (found the goal
  Gemini missed at 250-254, no phantom) AND cost (~$0.10 vs ~$0.30-0.41).
- Q6 Non-goal events (build-ups/presses/overloads): no scoreboard change.
  Use scanner bug-presence to segment broadcast-action vs
  celebration/replay/fan; run Gemini (content classification, WHAT is
  reliable) on broadcast-action segments only; scanner for goals, match_data
  validates, Claude confirms. Scanner removes the goal-mislocation failure
  mode for the highest-value events.
- Q7 Coordinates/crowd: tracking half-crowd (200/379, crowd as team, 14
  close-ups). Resolve: filter to pitch-region detections + render arrows only
  on wide shots (segment_scorer scores this); if fragmentation still too high
  on the reupload, cleaner source = full-match 1080p broadcast (~3-8GB,
  ~$1.5/episode tracking on RunPod, delete after render). Scanner doesn't
  need tracking; only arrows do.
- Q8 Scope: ~50s action in 482s. One reel = ~60-90s video (3 goals + 1-2
  build-ups). For Tifo-length (5-10min), need full-match broadcast (~3-8GB,
  ~$1.5 tracking) — scanner scales to it (1800 frames, free). Several reels
  redundant (overlap); not recommended.

Cost this session (part 2): ~$0.20 (scanner confirmations + walk-back frames).

Session 4 total: status mirror live at
https://github.com/minakush000-crypto/soccer-channel-status (auto-pushed by
Stop hook .claude/hooks/push_status.sh); scoreboard scanner built and
verified (3/3 goals, Ødegaard winner pinned to 261s bug-update / 253s shot).
Total session 4 cost: ~$0.65.

## 2026-09-08 session 5: build one video where words match pictures

End-to-end cut-list-first build on the Arsenal-Chelsea clip. No arrows, no
renderer, no crowd fix — boards + clean footage only.

Step 1 segmentation (scanner bug-presence): 482s reel = 210s broadcast-action
(43%) vs 273s filler (57% celebration/replay/fan). Windows 102-129, 171-198,
201-255, 261-363.

Step 2 goals — the 8s pre-roll rule FAILED (offset varies in SIGN): Chelsea
+17s (footage is celebration, no clean shot in the reel), Havertz +10s
(ball-in-net replay AFTER bug-update), Odegaard -8s (live shot BEFORE).
Fix: search +-20s around each bug-update for the goal-scoring visual,
relay-verified. Verified cut windows: Rogers [119-127] (celebration +
ROGERS 1-0 caption; the shot isn't in the reel), Havertz [181-191]
(ball-in-net @182 + HAVERTZ 1-1 caption), Odegaard [253-262] (shot @253 +
ODEGAARD 1-2 caption @259).

Step 3 non-goal: Gemini on broadcast segments invented a goal-shot at 275
that the relay showed was celebration (discarded — confirms Gemini-only
unsafe). Relay found sparse on-ball action; used Rice build-up [173-177]
(broadcast, on-ball, Rice named).

Step 4 script (GLM 5.2, 150 words), cut-list-first, each line tied to a
verified clip. Validated vs match_data: Rogers (Chelsea #17, outside-box
right foot), Havertz (Arsenal #29, left foot outside box), Odegaard (Arsenal
#8, centre of box), Rice (Arsenal #41, midfield). No wrong role, no invented
event. scripts/2026-09-06_arsenal-chelsea.md (backup .bak).

Step 5 assemble: tools/assemble_words_match.py (new) cuts footage at exact
[VISUAL: footage=START-END] timestamps (replaces produce_v2 fixed-5s-offset
bug), allocates time by narration word-count, scales to 1280x720, tpad-holds
shortfall, merges ElevenLabs voice. final_video.mp4 45.2s, 1280x720, 15MB.
(First run had a hold-last-frame bug producing a 0.04s segment; fixed with
tpad.)

Step 6 upload PRIVATE: https://www.youtube.com/watch?v=WFi2LBwXINU
publish-log/2026-09-06_arsenal-chelsea_WFi2LBwXINU.json

Step 7 words-match-pictures (the never-run test) — relay on the mid-frame of
each of 7 sections of the FINAL video:
  formation board @3.2s -> Arsenal vs Chelsea lineups MATCH
  Chelsea goal @10.8s -> crowd celebrating, ARS 0-1 CHE, ROGERS 1-0 caption MATCH
  Rice build-up @17.7s -> Arsenal attacking build-up MATCH
  Havertz goal @24.0s -> Arsenal #29 celebrating, HAVERTZ 1-1 caption MATCH
  possession board @30.2s -> Arsenal 39.3 vs Chelsea 32.7 PARTIAL (board
    numbers buggy: 39.3/32.7 vs stat_card 54.6/45.4; pre-existing
    tactical_boards.py bug, not fixed — renderer off-limits)
  Odegaard winner @36.9s -> Arsenal goal moment, keeper beaten (ODEGAARD 1-2
    caption elsewhere in clip) MATCH
  stat_card @43.2s -> MATCH STATS 16-13 shots, 54.6-45.4 poss MATCH
RESULT: 6/7 clean match, 1 partial (possession board data bug). All 3 goals
match narration via on-screen scorer captions. The picture matches the
words. Honest length 45.2s.

Cost this session: ~$0.30 (relay frames + ElevenLabs + Gemini inline).

## 2026-09-08 session 6: Mayo review + 6-part fix list, Part 1 done

Mayo watched the WFi2LBwXINU video. Goal 1 matched narration; goals 2 and 3
were off; the opening board section read "PowerPoint and crayon." Six-part
fix list, order 1 -> (2+3) -> 4 -> 5 -> 6, one change per run.

PART 1 (unblock status-mirror read loop) — DONE.
Problem: github.com/.../blob/... returns the HTML page chrome (317KB), not
content, so Claude's fetcher could not read STATUS.md or PROGRESS.md.
Fix: use the RAW host, which needs zero setup and returns plain text.
  https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/<FILE>
Added two generated pages to the mirror, wired into push_status.sh:
  - README.md: index of every raw URL.
  - ALL_STATUS.md: all seven docs concatenated (one fetch = full state).
Backups: .claude/hooks/push_status.sh.bak. CONTEXT.md STATUS MIRROR section
rewritten to point at the raw host (the old text sent sessions to the broken
github.com URL). .gitignore in the mirror now allows README.md + ALL_STATUS.md.
Verified:
  - curl: README 1210B, ALL_STATUS 88930B (7 FILE sections), STATUS 12119B,
    PROGRESS 49269B, all HTTP 200, no auth.
  - WebFetch (this session) reads STATUS.md + PROGRESS.md verbatim.
  - Real Claude Opus 5 (claude5 -p --allowedTools WebFetch) confirmed all
    three URLs return HTTP 200 unauthenticated and quoted first lines:
      STATUS.md     -> "# STATUS.md — verified current state"
      PROGRESS.md   -> "# PROGRESS.md — chronological log"
      ALL_STATUS.md -> "# soccer-channel — combined status (auto-generated)"
Caveat found (Opus 5): WebFetch summarizes through a small model and can
rewrite headings (it fabricated ALL_STATUS's title as "# Soccer-Channel
Status Summary"). WebFetch understands state but is NOT a verbatim source;
for exact text/numbers, pull raw bytes with curl. Documented in CONTEXT.md.

The URL to give any session: the combined page
  https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/ALL_STATUS.md
Cost Part 1: ~$0.02 (one Opus 5 WebFetch confirmation).

PART 3 (push artifacts/ + frames/ to mirror) — DONE.
Staged in soccer-channel under artifacts/ and frames/ (gitignored here, synced
to the public mirror by push_status.sh). Redacted (~ -> ~),
secret-scanned (no keys, tokens, /home paths), .gitignore allows only the two
subtrees + docs and blocks renders/clips/.env/keys.

artifacts/ (288K total, all under a few hundred KB each):
  - scoreboard/clip_PrCW_geeRAU.scoreboard.json (19KB) + README.md caveat
  - gemini_inventory/clip_PrCW_geeRAU.gemini_inventory.json (4.6KB) + .meta.json
  - publish-log/2026-09-06_arsenal-chelsea_*.json (2 x 228B)
  - tracking_summary/clip_PrCW_geeRAU.summary.json (175KB, 3248 trackers)
  - tracking_summary/clip_nxPNT4TU5_Q.summary.json (54KB, 1020 trackers)
frames/ — empty for now (Parts 4 and 5 populate it).

New tool: tools/trim_tracking.py — reduces an 8-16MB per-frame tracking JSON to
a compact per-tracker summary (id, team, norm centre, area_frac, n_frames,
first/last frame, location class via a documented pitch/crowd/edge heuristic).
Valid JSON, recomputable from the source per-frame file.

push_status.sh rewritten: now syncs artifacts/ + frames/ (rsync --delete,
excludes dangerous types, redacts text/json), always rewrites the mirror
.gitignore, regenerates README (lists the two trees) + ALL_STATUS, and commits
only when the git index actually changed. Backup at push_status.sh.bak.

Verified:
  - Live raw URLs: scoreboard JSON 18912B HTTP 200, scoreboard README 1960B
    HTTP 200, arsenal tracking summary 174798B HTTP 200. All unauthenticated.
  - Secret scan on mirror: clean (no sk-/AIza/rpa_/ghp_ keys, no /home paths).
  - Recheckable arithmetic: timeline[] first-occurrences = 0-1 @102s, 1-1
    @171s, 2-1 @261s, matching match_data (Rogers 2', Havertz 25', Odegaard 50').

FINDING (scoreboard changes[] gap): the changes[] array has 6 entries (2 real
goals + 4 OCR blips) and MISSES Rogers (first goal, 0-0->0-1 @102s) because the
bug was PRE/absent before ~99s, so there is no prior score to change from.
len(changes) undercounts goals by 1. Recheck goal count from timeline[]
first-occurrences, not len(changes). Documented in artifacts/scoreboard/README.md
and CONTEXT.md. This does not affect goal-finding; only naive len(changes).

Cost Part 3: $0 (all local + the mirror push).

PART 2 (re-examine NONE reads; find a non-bug broadcast/filler signal) — DONE.
Trigger: Mayo's independent frame check found the score bug is burned in by the
uploader across ALL footage types (frame 261 = fan-shot phone footage with the
ARS 2-1 bug top-left), so bug presence does not split broadcast from filler.

Re-check of what NONE reads correspond to (10 frames at 640px, gemma4 vision,
in frames/2026-09-06_arsenal-chelsea/nonecheck_*.png):
- NONE is NOT "bug absent". The scanner's NONE/PRE = "gemma4 failed to OCR the
  bug", not "no bug". gemma4 re-read saw the bug at 150s and 400s where the
  scanner recorded NONE. So NONE is an OCR-failure signal, not a presence signal.
- NONE frames are mostly BROADCAST: pre-match lineups/huddle (15/45/75s),
  in-play wide shot (150s), post-match (400/460s). Not filler.
- Bug-present frames are MIXED: broadcast goals (110s) AND filler crowd/
  celebration (200/240/261s). So bug presence does NOT separate broadcast from
  filler. Confirmed.

New signal (the replacement): Gemini inventory content classification
(action_type / shot_type), which the project already found accurate for WHAT.
  broadcast  = action_type in {shot, build-up} AND shot_type != replay
  filler     = action_type == non-action  OR  shot_type == replay
Goal times are NOT trusted from Gemini (timestamps drift); cross-check against
match_data + the scoreboard timeline (0-1@102, 1-1@171, 2-1@261).

Result on the 482s Arsenal-Chelsea reel (artifacts/broadcast_filler/):
  NEW signal: 43.0s broadcast (9%) / 440.0s filler (91%), 21 segments.
  OLD bug signal: ~210s "broadcast" (44%) — inflated ~5x by bug-burned-in
  celebration/crowd. The 43s matches the 45.2s video actually produced, which
  is independent confirmation the new signal is realistic.
Broadcast windows: 0-5, 88-99, 99-111, 161-172, 250-254s (4 goal shots + 1 build-up).

Scene-change detection alone is INSUFFICIENT: cv_annotate shot_boundaries has
only 7 cuts for the 482s reel (far too coarse), and cuts segment but do not
classify broadcast vs filler. Content classification is required either way.

Caveat (◑): gemma4 single-frame calls disagreed with Gemini's segment class at
two boundaries (150s: Gemini celebration vs gemma4 in-play wide; 400s: Gemini
crowd vs gemma4 player-on-pitch). Gemini's segment classification is the one
Mayo already validated as accurate, so the segmentation trusts Gemini; the
single-frame gemma4 calls are ◑ and not used for the split.

Goal-finding is UNAFFECTED: the scoreboard scanner still finds all 3 goals via
scoreline changes in the timeline (count from timeline[] first-occurrences, not
len(changes) — see Part 3 finding).

New tool: tools/broadcast_filler.py. New artifact: artifacts/broadcast_filler/.
10 verification frames pushed to frames/. All live on the mirror (HTTP 200).

Cost Part 2: $0 (gemma4 vision is free; no Opus needed for this finding).

PART 4 (fix possession board bug, then rate board appearance) — DONE.

4a. Possession bug — FIXED and verified.
The user's framing was "the board misreads the data (39.3/32.7 vs stat_card
54.6/45.4)." The actual root cause is NOT a data misread:
  - match_data.json has possession '54.6' / '45.4' (strings, no %). Both the
    possession board (bare float) and stat_card (strip %) parse them to
    54.6 / 45.4 correctly. The on-disk possession.png is correct (Opus: 54.6/
    45.4, full width, sum 100).
  - The bug is the ANIMATED possession.mp4: _render_possession_animation filled
    the bar from 0 to the values over 75% of frames (frac = frame/(ANIM_FRAMES*
    0.75)), broadcasting INTERMEDIATE values for 3 of 4 seconds. assemble_
    words_match.py loops/trims the board mp4, so the captured frame at @30.2s
    landed mid-fill at frac ~= 0.72 -> 54.6*0.72 = 39.3, 45.4*0.72 = 32.7, with
    a 28% gap. Exactly the "wrong" numbers.
Fix (tools/tactical_boards.py, _render_possession_animation): fill in the
first ~6% of frames, hold full for ~94%, and show ONLY the final values once
the bar is full (no intermediate labels ever). Backup at tools/tactical_boards.py.bak.
Verified: re-rendered possession.mp4; Opus read of the previously-broken 1.5s
frame now = Arsenal 54.6%, Chelsea 45.4%, full width, no gap, sum 100 (matches
match_data). gemma4 confirmed 54.6/45.4 full at 1.5s and 2.0s too.
Frame evidence: frames/.../board_possession_video_0302.png (the bug) and
board_possession_fixed_0150.png (the fix), both on the mirror.

4b. Board appearance — RATED ONLY (not fixed, per instruction).
Opus 5 (authoritative judge) rated one full-state frame of each board against
the 7-criterion Coaches' Voice benchmark. Verbatim responses saved to
artifacts/board_ratings/boards_opus_ratings.md on the mirror. Scores:
  - formation : 3/10  (flat; matplotlib defaults; chips collide/clip pitch
    lines; ~5px unreadable numbers; default font; no narrative furniture)
  - possession: 4/10  (flat single plane; hard 90 deg butt-join; square caps;
    default font; no 50% marker; ~40% dead space)
  - stat_card : 5/10  (both bar sets grow left-to-right, Arsenal should mirror;
    numerals colliding with bar ends; inconsistent scaling; no crests/scoreline)
Common thread: matplotlib fingerprints (default fonts, hairline uniform
strokes, flat fills, no drop shadows), no layered depth, no condensed athletic
fonts. Confirms Mayo's "PowerPoint and crayon". Appearance NOT fixed yet.

Cost Part 4: ~$0.10 (1 Opus possession confirm + 3 Opus board ratings).

PART 5 (diagnose goals 2 and 3 cut timing) — DONE.
Mayo: goal 1 matched, goals 2 and 3 read wrong. Diagnosis per goal, with the
relay (gemma4) description of the first/middle/last second of each window and
the Opus-authoritative shot/no-shot call. Frames at frames/.../goal_*.png.

GOAL 1 — Rogers (Chelsea), window footage=119-127 (8s):
  first 119: CELEBRATION, ROGERS 1-0 caption, bug ARS 0-1
  mid   123: CELEBRATION, ROGERS 1-0, bug ARS 0-1
  last  127: CELEBRATION, ROGERS 1-0, bug ARS 0-1
  Opus (mid): "no goal shot whatsoever — no pitch, ball, or player striking;
  purely crowd celebration." The relay-verified search found no shot anywhere
  in the reel (searched +-20s around the bug-update @102). The actual strike is
  NOT in the reel. So 8/8s (100%) of the window is not the shot — it is all
  celebration + the ROGERS 1-0 caption.
  FIX applied: the script's Rogers line said "finds the bottom corner" (claims a
  shot). Rewritten to state the goal fact and describe the celebration instead:
  "Morgan Rogers, right-footed from outside the box, gives the visitors a shock
  lead, and the travelling fans erupt." (match_data fact kept; no shot claimed.)
  Backup scripts/2026-09-06_arsenal-chelsea.md.bak2. Video NOT re-rendered yet
  (separate pipeline run; narration fix is the Part 5 deliverable).

GOAL 2 — Havertz (Arsenal), window footage=181-191 (10s):
  first 181: ball-in-net aftermath (Opus: "keeper lying face-down beaten, ball
            over by the goal line, no player shooting"), HAVERTZ 1-1 caption,
            bug ARS 1-1
  mid   186: OTHER, no caption, bug ARS 1-1
  last  191: CELEBRATION, no caption, bug ARS 1-1
  The actual live STRIKE is NOT in the window (scoreboard scan inferred ~163-166,
  before the bug-update @171 and before this window). The window's first second
  is the ball-in-net result, not the strike. So ~1s goal-result + ~9s non-goal;
  0s live strike in the window. Narration "finds the bottom corner" shows the
  result (ball in net) but not the strike.

GOAL 3 — Odegaard (Arsenal), window footage=253-262 (9s):
  first 253: LIVE STRIKE (Opus: "player striking the ball, Chelsea #3 closing
            down, keeper stranded"), no caption, bug still ARS 1-1 (pre-update)
  mid   258: shot/aftermath, no caption, bug none
  last  262: CELEBRATION, ODEGAARD 1-2 caption, bug ARS 2-1
  The actual strike IS in the window (first second, 253). So ~1-3s strike +
  ~6-8s aftermath/celebration; the strike is the first second, the rest is not
  the goal.

Why 2 and 3 read wrong to the viewer: goal 2 shows the ball-in-net result, not
the strike, then celebration; goal 3 shows 1s of strike then ~8s of celebration.
Both windows are mostly aftermath, so the narration (which implies a strike)
plays over mostly-celebration footage. Goal 1 read as "matched" because its
window is coherently all celebration+caption (no mixed shot/celebration), even
though it never shows a strike.

Recommendation (not yet applied, per "diagnose" scope): for goals with a strike
in the reel (Odegaard), start the window ON the strike and keep it short (~3s);
for goals with only the ball-in-net (Havertz), trim to the ball-in-net moment;
for goals with no shot at all (Rogers), narrate the celebration (done) and do
not lengthen the window looking for a shot that is not there.

Cost Part 5: ~$0.10 (3 Opus shot/no-shot confirmations; gemma4 bulk was free).

PART 6 (audio: crowd ambience + British narrator) — DONE.

6a. Crowd ambience wired into produce_v2.py (the orphan).
generate_ambience.py was called by produce_episode.py and cloud_produce.py but
NOT produce_v2.py; produce_v2 step7_merge called merge_voice.py without the
crowd mp3. Fix in tools/produce_v2.py:
  - Added step6b_ambience(): calls generate_ambience.py <slug> 30 (idempotent,
    skips if crowd_ambience.mp3 exists; non-fatal on failure).
  - step7_merge() now passes renders/<slug>/crowd_ambience.mp3 to merge_voice.py
    when it exists; merge_voice mixes it under the voice at 30% volume with
    fade in/out (voice is 1.6x), so the crowd is an audible presence that never
    competes with the narration.
  - Called in main after the voice step, before the merge.
Backup tools/produce_v2.py.bak.
Verified: generate_ambience produced crowd_ambience.mp3 for arsenal-chelsea
(30s, 0.5MB, ElevenLabs Sound API). merge_voice on a TEMP slug (non-destructive;
published source untouched) ran the multi-layer mix — log: "Layers: voice(1.6x)
+ crowd(0.3x)", output 45.2s with audio. Temp cleaned up. Note: the existing
merge_voice mix is a fixed 30% crowd (no sidechain ducking); if it ever
competes during quiet speech, add sidechain compression to merge_voice.py
(separate change, not needed now).

6b. British narrator voice (samples for Mayo to pick).
Listed all 23 ElevenLabs voices on the account; 5 are British. The current .env
VOICE_ID is 1stSYyl7ZVPJk2ECrNlo = the "british soccer commentator" custom clone
(NOT a default, despite the brief saying so). Sampled the top 3 fitting an
analytical British football broadcaster, same ~15s script excerpt with the
production TTS settings, saved LOCAL (gitignored) for Mayo to listen:
  experiments/voice-test/audio/
  - voice1_daniel_steady_broadcaster.mp3  (onwK4e9ZLuTAKqWW03F9, 20s)
  - voice2_george_storyteller.mp3        (JBFqnCBsd6RMkjVDRZzb, 16.5s)
  - voice3_current_british_soccer_commentator.mp3 (1stSYyl7ZVPJk2ECrNlo, 14.7s)
Did NOT choose for Mayo. experiments/voice-test/audio/VOICES.md lists the 3 with
voice IDs, how to play (mpv/ffplay/Explorer), and how to set the choice.
Per-episode config added: tools/generate_voice.py now accepts
  --voice-id <id>  (overrides .env VOICE_ID for that run)
so each episode can use a different narrator. The .env VOICE_ID stays the
default. Syntax OK. Backup tools/generate_voice.py.bak.
Other British voices available (not sampled): Alice (Xb7hH8MSUJpSbSDYk0k2),
Lily (pFZP5JQG7iQjIQuC4Bku). Mayo picks; I will set the default or wire per
episode.

Cost Part 6: ~$0.02 (1 ambience sound-gen + 3 short TTS samples; all tiny).
TOTAL session cost (all 6 parts): ~$0.27 (Part 1 Opus ~0.02, Part 4 ~0.10,
Part 5 ~0.10, Part 6 ~0.02; Parts 2-3 free).

## Stage 5 — pod-side download path + rule-3 debt, 2026-09-09

5A (built): tools/runpod_download.py — pod-side yt-dlp + excerpt cut; only
guarded excerpts come home, full source stays on pod. Wired into produce_v2 as
--pod-download. PROVEN with real runs: 2/3 goal-window excerpts (4MB each) came
home under the 200MB guard; 128MB source never written locally; $0.005/run.
YouTube bot-blocks the RunPod datacenter IP on every player client (web_safari,
ios, android, default) — "Sign in to confirm you're not a bot" — so pod yt-dlp
is not viable today; --source-url (catbox) is the working cut path. 720p cap
kept for the local path; pod path is 1080p-ready. Cost: ~$0.03 across 7 pod runs.
5B (report): B2 archive designed (boto3 already installed, no S3 code yet,
~1.5-2.5h to build; archive only finals + publish-log ~100-200MB, not the 2.9G
renders; pod->B2 possible without touching laptop; .env key handling proposed).
5C.1: --help smoke test on 27 STANDALONE — 26/27 launch; luminance_pod crashes
(IndexError), runpod_stage1 hangs; rule 3 not closed per-tool.
5C.2: 4 DEAD tools retired (deleted, in git history); tool count 46 -> 42.
5C.3: runpod_annotate e2e costed (~$0.01-0.02), not run per brief.
DECISIONS.md doctrine gap + TOOLS.md updated. LANE_PLAN.md Stage 5 appended.
Pushed: 0fb4e96..80adff9.

## Stage 6 — /mnt/f staging, name overlap, beliefs closed, keys, 2026-09-09

6A: tools/staging.py — raw downloads stage on /mnt/f (fail-loudly if not mounted,
no ~ fallback). produce_v2 step3 downloads to /mnt/f + cleanup at end.
Doctrine rule 1 updated in 4 docs. 6A.2 real proof pending /mnt/f mount (mechanics proven).
6B: name overlap resolved — canonical path proven (6B.1), retired fenced by hook
(6B.2), yt_digest live (6B.3), canonical-path assertion in produce_v2 fires from
wrong tree (6B.4).
6C: all 8 beliefs closed — runpod_annotate RUN (0 results, retired), 1200s yield
estimate (no 1200s source), SoccerNet reasoned-not-measured, mirror redaction
proven, OAuth VALID, cut-list 2nd source partial (snapping holds, goals need
scoreboard), 5 tools retired (luminance_pod/runpod_stage1/runpod_annotate/
vastai_shorts/runpod_shorts; 46->37), upload 95 Mbps.
6D: keyframe bug diagnosed+fixed+bounded (stream-copy + re-encode fallback +
timeout), proxies researched, recommend AGAINST proxy for now, Gemini-cloud
switch scoped ~1.5-2.5h.
6E: encrypted .env backup (backup_env.py, gpg AES256, restore proven), b2_upload.py
built (pending B2 key), pod-to-B2 scoped. Tool count 46 -> 37.

## 2026-09-09 session: Stage 7 — rule 3 for documents + four-template blocker

7A (document audit): grep proved ZERO root docs are read by any pipeline tool at
runtime (tools read scripts/<slug>.md, never a root doc). Classified all 21 root
docs: 11 CANONICAL, 5 RECORD, 4 ORPHANED. Retired the 4 orphaned via git rm
(CONFIG, HANDOFF, PLAN_ARTIFACT.html, PROJECT_REPORT — all contradicted by code:
vastai_shorts deleted yet HANDOFF says it works; 29 tools hard-code
"soccer-channel" yet CONFIG claims nothing does; PROJECT_REPORT wrong per
REPORT_AUDIT; PLAN_ARTIFACT claims 9/10 with no CV annotation). Added a
purpose/reader/last-verified header to every surviving doc. Rule 3 now covers
documents. Retirement recorded in DECISIONS.md.

7B (four-template blocker, BUILT + PROVEN): SCRIPT_TEMPLATE.md rewritten to 4
lane shapes (A tactical / B preview / C hot-topic / D historical). validate_script.py
gained --lane A|B|C|D with per-lane structural rules: visual vocabulary (A
board/footage/tactical; B/C board/stills; D board/footage), B/C no footage= +
same-day fresh-brief required (fresh_fetch wired), D narration-first +
footage=START-END format + [CUTLIST:] ref. Existing [SRC]/[RUMOR] freshness
preserved for all lanes. Proven with real runs: lane B PASS, lane C PASS,
deliberate B-with-footage FAIL, lane D PASS, deliberate D-narration-first FAIL.
Lane D + transformation_gate composition proven (footage=119-130 passes
validate_script format; the 11s excerpt fails transformation_gate rule 2 — they
check different artifacts, no conflict). fresh_fetch run live (43 items, exit 0).

7C (open items): 7C.1 keyframe 3/3 verify attempted on a fresh L4 — pod throttled
(hung 540s+, zero webhook posts, exit 124), same as 6A.2/6D.1; left open with
bound (fix is diagnosed+fixed+bounded; pending a non-throttled pod; tonight's
L4s throttled). Found runpod_download pod-leak on timeout (SIGTERM kills python
before pod cleanup) — terminated the leaked verify pod. 7C.2 found TWO leftover
"soccer-stage1" pods still RUNNING (billing) despite prior stages reporting them
terminated; both terminated; pod count 0; known RunPod spend ~$0.20-0.25 (API
exposes no balance field). 7C.3 Gemini-cloud switch scope confirmed 1.5-2.5h,
unchanged by 7B (report only, not built).

New fragile: runpod_download/runpod_fulltrack leak the pod on timeout/crash
(manual termination required until a finally/signal-handler cleanup is added).
Committed 6b349f8, pushed to master.


---

# FILE: GAPS.md

# GAPS.md — what nobody has verified

> **Purpose:** registry of unverified claims and open uncertainties.
> **Reader:** every session; mirrored to the public status repo.
> **Last verified against code:** 2026-09-09.

This file existing and being short is a warning sign, not a success. Each
entry is something that could be true or false and nobody has run the command
to find out. If you verify one, move it to STATUS.md or DECISIONS.md and date
it.

## Cloud / GPU

- **runpod_annotate.py end-to-end run**: UNVERIFIED. It exists, it
  tarballs tools/ and ships them, but no log of a full pod run is in the repo.
  CONTEXT.md says "RunPod and Vast.ai both authenticate" but does not show a
  completed annotation job. Falsifier: run it with a slug and check the pod
  log + downloaded tracking JSON.

- **runpod_shorts.py / vastai_shorts.py produce a 1080x1920 output**: UNVERIFIED
  today. The 1080x1920 iraola output (Aug 29) predates the v2 pipeline. Could
  be that cloud encoding works but nobody ran it after the v2 changes.

- **$0.15 / 5 min per episode on an L4**: PARTIALLY VERIFIED 2026-09-05.
  One 10s clip cost $0.05 (720s pod uptime at $0.25/hr, 23s inference). The
  old estimate was for 7 clips; extrapolating linearly gives $0.35 for 7
  clips, but setup is one-time so amortized cost would be lower. The 5-min
  estimate is plausible for inference alone but not for full pod uptime.

## Tracker-to-player mapping

- **Tracker IDs can map to player names**: VERIFIED 2026-09-05 — no path
  exists. grep of cv_annotate.py and match_data.py found no jersey OCR, no
  position heuristics, no manual mapping. match_data.py has jersey numbers
  (line 141) but nothing reads them from video. cv_annotate.py now exports
  per-frame player_positions (Stage 1 done) but every tracker is unnamed.
  Stage 2 is arrows on unnamed players, or hand-mapped. An automated path
  (jersey OCR on the bbox → match to ESPN lineup) would be a separate stage.

## Upload

- **youtube_upload.py OAuth tokens are valid**: UNVERIFIED.
  `secrets/youtube_token.json` exists (Aug 23) but tokens expire. publish-log/
  is empty so no upload has confirmed them. Falsifier: run
  `youtube_upload.py <slug>` and see if it 401s.

## Visual quality

- **cv_annotate.py's output looks acceptable today**: UNVERIFIED. The
  "professional broadcast tracking" verdict in CONTEXT.md is from a prior
  session. Re-run `~/tools/vision_analyze.py` on a current annotated frame
  before relying on it.

- **The latest produce_v2.py output (Sep 5) is visually acceptable**: UNVERIFIED.
  ffprobe confirms dimensions and duration, not quality. Run
  `~/tools/vision_analyze.py` on a frame from
  `renders/2026-08-30_liverpool-forest/shorts/final_video_shorts.mp4` before
  claiming the pipeline produces good video.

## Skill reliability

- **Skill firing is model judgment, not enforced.** The harness lists skills
  and the model decides whether to invoke one; nothing forces a match
  (SKILLS.md). So CONTEXT.md is the only reliable primer — it is read every
  session, skills are not.

## Orphans

- **The ~20 ORPHANED tools in TOOLS.md are truly dead**: UNVERIFIED. They may
  be invoked by hand, by cloud_produce.py on the pod (not grepped line-by-line
  for every tool), or by scripts/ not yet examined. Before deleting any, grep
  the whole tree including scripts/ and briefs/.

---

# FILE: DECISIONS.md

# DECISIONS.md — running log of choices and why

> **Purpose:** running log of decisions and why, incl. the doctrine rule-1/rule-3 gap list.
> **Reader:** every session; mirrored to the public status repo.
> **Last verified against code:** 2026-09-09.

Newest first. Each entry is dated. "Why" is the actual reason, not a
retcon. If a decision is reversed, add a new entry above, do not edit the old
one.

## 2026-09-05 (session 2)

### Option C Stage 1: cv_annotate.py exports per-frame player positions
Why: tactical_overlay.py guesses coordinates from text via glm-5.2:cloud
(line ~120 `generate_overlay_spec`). No pixel data is read. The fix is to
make cv_annotate.py export per-frame tracker positions so stage 2 can place
arrows at real coordinates. Added `player_positions` list (one entry per
frame, each with `{frame, players: [{id, bbox, team}]}`) to the tracking
JSON. Existing exports intact. Backup at `tools/cv_annotate.py.bak`.
Verified on RunPod: 300 frames, 129 tracker IDs, 1.16MB JSON, 23s inference.

### No path from tracker ID to player name exists
Why: grep of cv_annotate.py and match_data.py found no jersey OCR, no
position heuristics, no manual mapping. `extract_jersey_color` (line 72)
reads RGB for K-means team clustering only. match_data.py fetches jersey
numbers from ESPN (line 141) but nothing connects them to tracker IDs.
Consequence: Stage 2 is arrows on unnamed tracked players (team + bbox
only), or arrows placed by hand with a human naming the tracker. Building
an automated path (jersey OCR on the bbox → match to ESPN lineup) would be
a separate stage.

### Tracker fragmentation is high on this source
Why: 129 unique ByteTrack IDs in 10s of 1920x1080 footage. The compressed
wide-shot reupload causes the tracker to lose and re-ID constantly. Tracker
37 survives 5 frames (39-43). This limits how long any one arrow can
follow a player. Noted, not fixed — fixing it would need a better source
clip or a re-ID / appearance-embedding pass.

### runpod_stage1.py created as a one-off RunPod runner
Why: runpod_annotate.py processes all clips in a slug and doesn't upload
the tracking JSON back. For a single 10s test, a minimal one-off was
cleaner than modifying the existing tool. It uploads clip + tools to
catbox.moe, creates a webhook, runs cv_annotate.py --max-frames 300,
uploads results, and terminates the pod. Known bug: the webhook parser
fails when track_preview contains unescaped newlines (caught by
except:pass), so the script appeared to hang even though the pod finished.
Results were retrieved directly from the webhook API. Fix the parser
before reusing this script.

### Cost measured: $0.05 per 10s clip on L4
Why: pod ran 720s (12 min, including apt-get + pip install) at $0.25/hr =
$0.05. Actual inference 23s. The old CONTEXT.md estimate of "$0.15 / 5 min
per episode" was for 7 clips and is plausible but unverified for the full
set. This single-clip run is the first measured data point.

## 2026-09-05

### produce_v2.py is the authoritative entry point
Why: produce_episode.py is older and its step ordering and wiring
(generate_ambience, validate_script, cv_annotate) diverge from what the
verified output comes from. produce_v2.py produced a 720x1280 64.2s Short on
Sep 5 (ffprobe confirmed). produce_episode.py has no verified output today.
Caveat: produce_episode.py:288 still calls cv_annotate.py and must not break.

### Cloud-only for any GPU work
Why: this machine is an Intel N150 with no GPU. Local YOLOv8 is 1.30s/frame at
1080p (CONTEXT.md, measured), so 900 frames = 19.5 min, full clip = 95 min.
RunPod and Vast.ai both authenticate (CONTEXT.md). runpod_annotate.py tarballs
tools/ and ships it, so edits propagate. ~$0.15 and ~5 min per episode on an
L4 (CONTEXT.md). Local allowed only for ls, grep, ffprobe, single-frame
checks, reads, edits.

### Dropped all ball-dependent features
Why: ball detection is dead. 8 of 12 frames had no ball from either YOLOv8 or
roboflow (CONTEXT.md, measured on the compressed wide-shot reupload source).
No ball trail, no ball-following crop, no ball-tied arrows. Anchor to players
only.

### Rejected roboflow/sports
Why: its football-specific model found the ball in 4/12 frames vs 3/12 for
generic COCO (CONTEXT.md). Not a meaningful gain. PLAYER_DETECTION exceeded
2s/frame and BALL_DETECTION with InferenceSlicer was 14x slower than baseline.

### Retired ~/soccer-pipeline and ~/retired/soccer-*
Why: two folders named soccer-channel caused weeks of confusion (CONTEXT.md).
The only real project is ~/yt-digest/soccer-channel. ~/retired/ is
dead, never read or run. The soccer-channel SKILL.md (both global and parent)
still points at ~/soccer-pipeline/ and is DEPRECATED — do not follow it for
new work (see SKILLS.md).

### 720p rejection gate added to produce_v2.py
Why: produce_v2.py was silently accepting 360p downloads and upscaling. Now
loops up to 5 candidates, ffprobes each, rejects <720p, uses cookies from
secrets/yt_cookies.txt, fails loudly. Verified: source is now 1920x1080
(CONTEXT.md). Backup at tools/produce_v2.py.bak.

## STANDING OPERATING DOCTRINE (added 2026-09-09, Stage 4A)

1. NOTHING RUNS ON THE LOCAL MACHINE. No local downloads, except raw source
   staging on the USB flash drive at /mnt/f (staging area ONLY: download lands
   there, ships to the pod, gets deleted; never a working directory). No local GPU work.
   All compute and all storage go to the cloud GPU providers.
2. No tasking without calling the available tools, skills, connectors, web
   fetch, research or MCPs where they apply.
3. No orphaned, standalone or untested item, tool or aspect of the pipeline
   may exist. Everything is wired, tested, or retired.
4. Every brief carries this doctrine as a footer.

### Gap: what does NOT yet meet the doctrine (updated 2026-09-09, Stage 6)

**Rule 1 (nothing local) — VIOLATED by the CPU/storage stages, not by the
download.** Stage 5A put step3 (download+cut) and step4b (tracking) on the pod.
Stage 6A moved the RAW DOWNLOAD to /mnt/f USB staging (`tools/staging.py`,
fail-loudly if /mnt/f not mounted, no ~ fallback), so a local download
no longer permanently grows the vhdx. The remaining local violators:

- step1 match_data (local ESPN API), step2 boards (local matplotlib),
  step5 assemble (local ffmpeg), step6 voice (local ElevenLabs API),
  step6b ambience (local ffmpeg), step7 merge (local ffmpeg),
  step8 shorts (local ffmpeg crop), and the final `shutil.copy2` to
  /mnt/c/Downloads (the one permitted local hand-off of the finished file).
- Transitional: until step5 moves to the pod, the staged clip on /mnt/f is read
  during local processing (slow 9p, unplug risk) — a temporary breach of the
  "staging-only" limit that closes when step5 runs on the pod (5A.4 gap).
- cut-list GENERATION (`scoreboard_scan.py` + `broadcast_filler.py` +
  `cut_list_gen.py` + PySceneDetect) runs locally and needs the clip. Blocked by
  scoreboard_scan's local-vision dependency (gemma4 via localhost:11434; switch
  to GEMINI_API_KEY cloud vision — scoped 6D.4).
- `produce_episode.py`, `assemble_words_match.py` (separate local pipelines).
- `cloud_produce.py` still downloads raw clips locally (`cloud_produce.py:223`).
- Caveat: pod yt-dlp is bot-blocked by YouTube (datacenter IP, 5A.3); the working
  rule-1 path is local-download → /mnt/f staging → catbox → pod cut → excerpts
  home → /mnt/f cleanup. A residential proxy would unblock pod yt-dlp (6D.2/6D.3).

**Rule 3 (no orphaned/untested) — 9 tools RETIRED total (5 in 5C.2, 5 in 6C.7
incl. vastai_shorts which was marked-retired but not deleted in 4B.4).**
- RETIRED (deleted, in git history): tactical_overlay, pitch_radar, render_video,
  check_and_download (5C.2); luminance_pod (crashed, dead callers), runpod_stage1
  (one-off done, hung on --help), runpod_annotate (6C.1: pod ran ~60s/$0.004 but
  captured 0 results — broken webhook, superseded by runpod_fulltrack), vastai_shorts
  (4B.4 retired, file deleted now), runpod_shorts (untested, depended on retired
  luminance_pod). Tool count 46 → 37.
- 27 STANDALONE decisions (6C.7): 10 have proper argparse (launch fine); 15
  treat `--help` as a positional arg (launch but no `-h` — low-priority polish,
  not a rule-3 failure); the 5 broken/dead ones retired above. `--help` proves
  "launches," not "works" — e2e runs are still owed for the untested STANDALONE
  (cloud_produce, runpod_superres, gpu_superres, produce_episode, etc.).
- `runpod_fulltrack.py` + `runpod_download.py` are WIRED + tested (compliant).
- 6C closures: OAuth token VALID (refresh works, 6C.5); upload 95 Mbps (6C.8);
  mirror redaction PROVEN (6C.4, path+key-pattern+file-type, not a general
  scanner); cut-list gen on 2nd source PARTIAL (6C.6 — snapping holds, but
  broadcast_filler misses celebration-classified goals); 1200s yield stays an
  ESTIMATE (6C.2 — no 1200s source exists, longest is 1120s); SoccerNet is a
  dataset/benchmark not a drop-in tool (6C.3 — reasoned, never measured).

This list is the work queue for rules 1 and 3. Stage 6 closed the raw-download
vhdx-growth (6A), 5 more dead tools (6C.7), and every open belief (6C). The
CPU/storage stages, cut-list-on-pod, and untested-STANDALONE e2e runs remain.

## Stage 7 — rule 3 applies to documents (2026-09-09)

Rule 3 ("nothing orphaned may exist") covers documents, not just tools. A stale
doc that reads as authoritative is the exact failure that cost a session
(PROJECT_REPORT.md read as current for two weeks after the code moved on).

Audit method: `grep -rl "<doc>.md" tools/ scripts/ tests/` for every root doc
(7A.1). Result: NO root doc is read by any pipeline tool at runtime. Tools read
`scripts/<slug>.md` (the per-episode scripts), never a root doc. The one hit,
`SCRIPT_TEMPLATE.md` in `fresh_fetch.py:249`, is a string literal ("see
SCRIPT_TEMPLATE.md") embedded in output text, not a file read. So FUNCTIONAL
(read by code at runtime) = zero root docs. The rest split into CANONICAL /
RECORD / ORPHANED.

Retired (ORPHANED — read by nothing, superseded or contradicted by code):
- `PROJECT_REPORT.md` (dated 2026-08-23) — REPORT_AUDIT.md proved it wrong on
  every structural point. Retired per 7A.2.
- `HANDOFF.md` (dated 2026-08-29) — says `vastai_shorts.py` works and RunPod is
  dead. Code contradicts: `ls tools/vastai_shorts.py` → "No such file" (retired
  Stage 6); RunPod is the working path (Stage 5).
- `CONFIG.md` — claims "Nothing else in the workspace hard-codes the name;
  everything references CHANNEL_SLUG." Code contradicts: 29 of tools/*.py
  hard-code the string "soccer-channel" (grep). No tool reads CONFIG.md.
- `PLAN_ARTIFACT.html` (2026-08-25 blueprint) — claims "9/10 ACHIEVED" with "No
  CV annotation"; cv_annotate works now and the pipeline is 720x1280 with
  footage/narration mismatch. Superseded + contradicted.

Where retired docs go: DELETED from the working tree via `git rm` (all four were
tracked, so recoverable from git history with `git show <rev>:<path>`). The
retirement is recorded here. No `retired/` folder is kept in the tree (it would
itself be an orphan); git history is the archive.

Survivors (16) carry a header line: purpose / reader / last-verified-against-code
date (7A.4). CANONICAL (mirrored spine + active plan/instructions):
ARCHITECTURE, CLAUDE, CONTEXT, DECISIONS, GAPS, LANE_PLAN, PROGRESS, README,
SCRIPT_TEMPLATE, STATUS, TOOLS. RECORD (completed-stage artifacts, kept for
history): GATES (Stage 5 gates), RECONCILIATION (2026-09-08 audit),
REPORT_AUDIT (the audit that proved PROJECT_REPORT wrong), SKILLS (2026-09-05
skill analysis), VISUAL_QUALITY_ASSESSMENT (2026-08-25 snapshot, superseded by
STATUS for current state).

---

# FILE: TOOLS.md

# TOOLS.md — one row per file in tools/

> **Purpose:** one row per file in tools/ (status: WIRED / STANDALONE / DEAD).
> **Reader:** every session; mirrored to the public status repo.
> **Last verified against code:** 2026-09-09.

Verified against code on 2026-09-08. Caller column is from
`grep -rnE "import <mod>|TOOLS / \"<name>\"|\"<name>.py\"" tools/ --include="*.py"`
limited to real call sites (subprocess `cmd=[PYTHON,...]`, `import`, or curl
download), not docstring/test mentions. "ORPHANED" = no caller anywhere.
Line numbers are where the tool is invoked or imported; re-read before relying.
This rebuild supersedes the 2026-09-05 version (33 tools, tactical_overlay
wiring): the repo now has 43 tools and produce_v2 no longer calls
tactical_overlay.

## Classification (from RECONCILIATION.md 1.1)

- **WIRED** = reachable from produce_v2.py (13 tools, +runpod_download via --pod-download).
- **STANDALONE** = CLI entry point or hand-run utility, or wired only into
  another standalone entry point (27 tools).
- **DEAD** = no caller anywhere (4 tools) — RETIRED 2026-09-09 (Stage 5C.2),
  deleted from the tree; in git history.

## The 40 tools (was 43; +runpod_download, -4 DEAD retired)

| File | Class | Called by (file:line) | Last modified | Works? |
|---|---|---|---|---|
| `produce_v2.py` | WIRED (root) | nothing (entry point) | 2026-09-09 | works (liverpool-forest 720x1280 62.3s Sep 7); --pod-download routes step3 to runpod_download (5A) |
| `match_data.py` | WIRED | `produce_v2.py:69` | 2026-09-01 | works |
| `tactical_boards.py` | WIRED | `produce_v2.py:78` | 2026-09-08 | works |
| `runpod_fulltrack.py` | WIRED | `produce_v2.py:235` | 2026-09-09 | works (146s full-clip, $0.011, STATUS); pitch_radar removed from ship list (5C.2) |
| `runpod_download.py` | WIRED (--pod-download) | `produce_v2.py` step3_download_clips_pod | 2026-09-09 | built 5A; pod-side yt-dlp bot-blocked by YouTube (runs #1-3); excerpt-cut+guard proven via --source-url (2/3 windows, 4MB each, $0.005, LANE_PLAN 5A.3) |
| `tactical_render.py` | WIRED | `produce_v2.py:285` | 2026-09-06 | works (Opus 8/8.5, STATUS) |
| `generate_voice.py` | WIRED | `produce_v2.py:474` | 2026-09-08 | works |
| `generate_ambience.py` | WIRED | `produce_v2.py:493` | 2026-08-22 | works (wired 2026-09-08, STATUS Part 6) |
| `merge_voice.py` | WIRED | `produce_v2.py:514` | 2026-08-25 | works |
| `shorts_crop.py` | WIRED | `produce_v2.py:524` | 2026-08-30 | works (outputs 720x1280) |
| `ffmpeg_utils.py` | WIRED (lib) | imported by `generate_voice.py:24`, `merge_voice.py:25`, `shorts_crop.py:31`; shipped to pod by `runpod_fulltrack.py:72` | 2026-09-08 | works (now holds the 200MB guard) |
| `script_utils.py` | WIRED (lib) | imported by `generate_voice.py:25` | 2026-08-25 | works |
| `cv_annotate.py` | WIRED (pod) | shipped+run on pod by `runpod_fulltrack.py:72,91` | 2026-09-07 | works (per-frame positions export, STATUS) |
| `produce_episode.py` | STANDALONE | entry point; no caller | 2026-09-09 | untested (no verified output); 800M guard added (3A.4), keeps 1080p anti-blur source |
| `cloud_produce.py` | STANDALONE | entry point; no caller | 2026-09-08 | untested (pod-side yt-dlp + download_url guarded) |
| `runpod_annotate.py` | STANDALONE | entry point; no caller | 2026-08-23 | untested (GAPS) |
| `runpod_shorts.py` | STANDALONE | entry point; no caller; uploads `luminance_pod.py:254` | 2026-08-29 | untested |
| `vastai_shorts.py` | RETIRED (4B.4) | was STANDALONE; end-to-end run 2026-09-09: Vast key works, pod creates, but encoding_failed (size 0) deterministically — encode script broken on the pod | 2026-09-09 | retired (rule 3) |
| `runpod_superres.py` | STANDALONE | entry point; no caller | 2026-08-30 | untested |
| `gpu_superres.py` | STANDALONE | entry point; no caller | 2026-08-30 | untested |
| `luminance_pod.py` | STANDALONE (transitive) | `runpod_shorts.py:254`, `vastai_shorts.py:334` (uploaded to pod) | 2026-08-29 | untested |
| `sharpness_check.py` | STANDALONE (transitive) | `cloud_produce.py:984` | 2026-08-30 | untested |
| `assemble_video.py` | STANDALONE (legacy) | `produce_episode.py:303`, `cloud_produce.py:564`; NOT produce_v2 | 2026-08-30 | untested |
| `validate_script.py` | STANDALONE (legacy) | `produce_episode.py:155`; NOT produce_v2 | 2026-08-25 | untested |
| `generate_captions.py` | STANDALONE | no caller (only `tests/`) | 2026-08-25 | untested |
| `thumbnail_generator.py` | STANDALONE | no caller | 2026-08-25 | untested |
| `enhance_clips.py` | STANDALONE | no caller (only `tests/`) | 2026-08-23 | untested |
| `segment_scorer.py` | STANDALONE | no caller; `cv_annotate.py:332` is a comment only | 2026-09-05 | works (hand-run on full-clip data, STATUS) |
| `scoreboard_scan.py` | STANDALONE | no caller | 2026-09-08 | works (3/3 goals, STATUS) |
| `broadcast_filler.py` | STANDALONE | no caller | 2026-09-09 | works; FIXED 3A.1 (phantom-segment bug: --clip/--duration bounds); yields 43s/8min, 192s/13.5min, 373s/18.7min |
| `cut_list_gen.py` | STANDALONE (prerequisite) | no caller; run by hand before produce_v2 to write footage=START-END tags | 2026-09-09 | works (4C); PySceneDetect + broadcast_filler -> scripts/<slug>.md; proven on 18.7min (cuts at 26.6/214.7/222.1/245.4) |
| `assemble_words_match.py` | STANDALONE | no caller | 2026-09-08 | works (arsenal-chelsea 45.2s, STATUS) |
| `trim_tracking.py` | STANDALONE | no caller; has `main()` CLI; output in `artifacts/tracking_summary/` | 2026-09-08 | works (hand-run) |
| `viral_angle.py` | STANDALONE | no caller | 2026-08-25 | untested |
| `agent_reach_research.py` | STANDALONE | no caller | 2026-08-30 | untested |
| `fresh_fetch.py` | STANDALONE | no caller | 2026-08-18 | untested |
| `youtube_upload.py` | STANDALONE | no code caller; invoked by hand | 2026-09-07 | works (2 publish-log entries, Sep 6) |
| `oauth_setup.py` | STANDALONE | one-time; no caller | 2026-08-23 | ran once |
| `gemini_inventory_test.py` | STANDALONE | no caller | 2026-09-08 | works (hand-run, STATUS Gemini section) |
| `runpod_stage1.py` | STANDALONE (one-off) | no caller; docstring = one-off | 2026-09-05 | works (one-off, $0.05, STATUS) |
| `ltx_enhance.py` | STANDALONE | no caller | 2026-08-23 | untested |
| `tactical_overlay.py` | RETIRED (5C.2) | was DEAD | 2026-09-09 | DELETED 2026-09-09; in git history (superseded by tactical_render) |
| `pitch_radar.py` | RETIRED (5C.2) | was DEAD (shipped but never run) | 2026-09-09 | DELETED 2026-09-09; ship-list refs removed from runpod_fulltrack/stage1/annotate |
| `render_video.py` | RETIRED (5C.2) | was DEAD (DEPRECATED) | 2026-09-09 | DELETED 2026-09-09; in git history |
| `check_and_download.py` | RETIRED (5C.2) | was DEAD (superseded by runpod_fulltrack) | 2026-09-09 | DELETED 2026-09-09; in git history |

Counts: **13 WIRED, 27 STANDALONE, 0 DEAD (4 retired)** = 40 (+2 libs = 42 .py files).

## Wired set (reachable from produce_v2.py)

8 direct subprocess calls + 2 shared libs + 1 pod-shipped tracker:

```
$ grep -nE "cmd = \[PYTHON|str\(TOOLS / \"" tools/produce_v2.py
69:  match_data.py       78:  tactical_boards.py   235: runpod_fulltrack.py
285: tactical_render.py  474: generate_voice.py    493: generate_ambience.py
514: merge_voice.py      524: shorts_crop.py
```
Transitive libs: `ffmpeg_utils.py` (imported by generate_voice/merge_voice/shorts_crop),
`script_utils.py` (imported by generate_voice). Pod: `cv_annotate.py`
(shipped+run by runpod_fulltrack). `pitch_radar.py` is shipped but never run.

---

# FILE: ARCHITECTURE.md

# ARCHITECTURE.md — soccer-channel code map

> **Purpose:** code map — which tool calls which, the 9-step produce_v2 pipeline.
> **Reader:** every session (CLAUDE.md loads it); mirrored to the public status repo by `.claude/hooks/push_status.sh`.
> **Last verified against code:** 2026-09-09.

Verified against code on 2026-09-09 (Stage 2: 720p cap + 180-1200s filter on produce_v2.py). Every line number is from `wc -l` /
`grep -n` against the file on disk today. If a line moved, re-read. This
rebuild supersedes the 2026-09-05 version, whose step-4 wiring (`tactical_overlay`
at `produce_v2.py:232`) no longer exists in the code.

## Entry points (two)

| Entry point | Status | Evidence |
|---|---|---|
| `tools/produce_v2.py` (625 lines) | Authoritative | `grep -n "^def main"` → `529:def main():`; nothing calls it |
| `tools/produce_episode.py` (519 lines) | Older, separate | calls `cv_annotate.py:288`, `assemble_video.py:303`, `validate_script.py:155`, `tactical_boards.py:173`, `generate_ambience.py:324`, `generate_voice.py:339`, `merge_voice.py:336`. NOT reachable from produce_v2 |

Run command (matches `produce_v2.py:10`):
`~/yt-digest/.venv/bin/python tools/produce_v2.py <slug> --query "<match>" --date-range YYYYMMDD-YYYYMMDD`

## produce_v2.py — the 9 steps, in actual execution order

Execution order is from `main()` (line 529), read directly:

```
$ grep -nE "^def step|^def main" tools/produce_v2.py
67:def step1_match_data(slug, query, date_range):
76:def step2_boards(slug):
95:def step3_download_clips(slug, query, render_dir):
211:def step4b_tactical_render(slug, clip_path, render_dir, match_data):
301:def step5_assemble(slug, render_dir, boards_dir, clip_path, match_data,
467:def step6_voice(slug, render_dir):
479:def step6b_ambience(slug, render_dir):
498:def step7_merge(slug, render_dir):
522:def step8_shorts(slug, render_dir):
529:def main():
```

`main()` calls (lines 549-588): step1 (549) → step2 (553) → step3 (557) →
step4b (567) → step6_voice (570) → step6b_ambience (575) → step5_assemble
(579) → step7_merge (584) → step8_shorts (588).

| # | Function (line) | What it runs | Output |
|---|---|---|---|
| 1 | `step1_match_data` (67) | `match_data.py <slug> --query <q>` (line 69) | `renders/<slug>/match_data.json` |
| 2 | `step2_boards` (76) | `tactical_boards.py <slug>` (line 78) | `renders/<slug>/boards/*.png + *.mp4` |
| 3 | `step3_download_clips` (95) | `yt-dlp` inline (line 175), format capped at **720p** (`height<=720`, line 176), duration filter **180-1200s** (3-20min, line 141), `--max-filesize 200M` (line 177), 720p gate (line 193), 200MB guard (line 195), cookies at `secrets/yt_cookies.txt` (line 155) | `renders/<slug>/clips/clip_<id>.mp4` |
| 4 | `step4b_tactical_render` (211) | `runpod_fulltrack.py --clip <clip>` (line 235, ships `cv_annotate.py`+`pitch_radar.py`+`ffmpeg_utils.py` to RunPod, runs cv_annotate on the pod) then `tactical_render.py <tracking.json>` (line 285) | `renders/<slug>/clips/<prefix>_full.tracking.json` + `tactical_view.mp4` |
| 5 | `step6_voice` (467) | `generate_voice.py <slug> --tts-only` (line 474) | `renders/<slug>/voice_elevenlabs.mp3` |
| 6 | `step6b_ambience` (479) | `generate_ambience.py <slug> 30` (line 493). Non-fatal. | `renders/<slug>/crowd_ambience.mp3` |
| 7 | `step5_assemble` (301) | inline ffmpeg concat, board/tactical/footage segments. Footage cuts at `clip_idx*5` (line 433) — the fixed-offset bug, see RECONCILIATION 1.2 | `renders/<slug>/clips/video_footage.mp4` |
| 8 | `step7_merge` (498) | `merge_voice.py <slug> <voice.mp3>` (line 514) mixes crowd ambience under voice at 30% | `renders/<slug>/final_video.mp4` |
| 9 | `step8_shorts` (522) | `shorts_crop.py <slug>` (line 524) | `renders/<slug>/shorts/final_video_shorts.mp4` |

Final step copies to `/mnt/c/Users/muads/Downloads/<slug>_Short.mp4` (main, line 618).

## What produce_v2.py does NOT call (verified by grep)

```
$ for t in tactical_overlay validate_script check_and_download assemble_video youtube_upload; do c=$(grep -c "$t" tools/produce_v2.py); echo "$t: $c mention(s)"; done
tactical_overlay: 0 mention(s)
validate_script: 0 mention(s)
check_and_download: 0 mention(s)
assemble_video: 0 mention(s)
youtube_upload: 0 mention(s)
```
- `tactical_overlay.py` — NOT called (the old 2026-09-05 doc said line 232; that wiring is gone). DEAD.
- `validate_script.py` — no script-vs-matchdata validation in produce_v2.
- `generate_ambience.py` — IS now called (step6b, line 493). (The 2026-09-05 doc said NOT called; that is reversed.)
- `youtube_upload.py` — no upload step in produce_v2. Uploads happen by hand (2 entries in `artifacts/publish-log/`).
- `cv_annotate.py` — 3 mentions, all comments; not called locally. It runs on the RunPod pod, shipped by `runpod_fulltrack.py:72`.

## 200MB local-video guard (new, 2026-09-08)

`ffmpeg_utils.py` defines `LOCAL_VIDEO_LIMIT_MB = 200` and
`assert_video_under_limit(path)` (deletes + raises if a local video write
exceeds 200MB) and `cleanup_part_files(dir)`. Wired into every local
download path:

```
$ grep -n "assert_video_under_limit\|max-filesize\|cleanup_part_files" tools/produce_v2.py tools/runpod_fulltrack.py tools/cloud_produce.py
tools/produce_v2.py:177:  "--max-filesize", "200M",
tools/produce_v2.py:187:  cleanup_part_files(clips_dir)
tools/produce_v2.py:195:  assert_video_under_limit(clip_path)
tools/runpod_fulltrack.py:174:  assert_video_under_limit(local)
tools/cloud_produce.py:110:  assert_video_under_limit(local_path)
```

## External dependencies (outside this project)

| Dependency | Path | Evidence |
|---|---|---|
| Python venv | `~/yt-digest/.venv` | used by every run command |
| Env file | `~/yt-digest/.env` | `YOUTUBE_API_KEY`, `LLM_*`, `ELEVENLABS_API_KEY`, RunPod/Vast keys |
| YOLOv8 weights | `~/yolov8s.pt` | outside the project folder (CONTEXT.md) |
| yt-dlp cookies | `secrets/yt_cookies.txt` | produce_v2 line 150 |
| YouTube OAuth | `secrets/client_secret.json`, `secrets/youtube_token.json` | youtube_upload.py |
| RunPod / Vast.ai keys | in `~/yt-digest/.env` | not opened (rule 4) |

## Output layout

```
renders/<slug>/
  match_data.json            # step 1
  boards/                    # step 2 (PNG + MP4 per board type)
  clips/
    clip_<id>.mp4            # step 3 source (guarded ≤200MB)
    <prefix>_full.tracking.json  # step 4b (8MB, per-frame positions)
    tactical_view.mp4        # step 4b
    video_footage.mp4        # step 7 assembled
  voice_elevenlabs.mp3       # step 5
  crowd_ambience.mp3         # step 6
  concat_list.txt            # step 7 ffmpeg concat
  tmp_segments/              # step 7 scratch
  final_video.mp4            # step 8 merged
  shorts/final_video_shorts.mp4  # step 9 final
```

## Known render directories (ls renders/)

```
$ ls -d renders/*/
renders/2026-08-18_iraola-liverpool/
renders/2026-08-30_liverpool-forest/
renders/2026-08-30_liverpool-forest_sep1/
renders/2026-09-06_arsenal-chelsea/
renders/_fullmatch_arsenal-chelsea-carabao/
renders/_real_soccer_test/
renders/_sharp_test/
```

Latest produce_v2.py output (liverpool-forest): 720x1280, 62.3s, 24.8MB, Sep 7.
The 2026-09-06_arsenal-chelsea dir was built with `assemble_words_match.py`
(a standalone, words-match-pictures path), NOT produce_v2: 1280x720, 45.2s,
15.0MB, Sep 8. `_fullmatch_arsenal-chelsea-carabao` is an incomplete manual
download (only match_data.json; the .part was deleted 2026-09-08, see
RECONCILIATION 1.5/1.6).

## STANDING OPERATING DOCTRINE
1. NOTHING RUNS ON THE LOCAL MACHINE. No local downloads, except raw source
   staging on the USB flash drive at /mnt/f (staging area ONLY: download lands
   there, ships to the pod, gets deleted; never a working directory). No local GPU work.
   All compute and all storage go to the cloud GPU providers.
2. No tasking without calling the available tools, skills, connectors, web
   fetch, research or MCPs where they apply.
3. No orphaned, standalone or untested item, tool or aspect of the pipeline
   may exist. Everything is wired, tested, or retired.
4. Every brief carries this doctrine as a footer.

The truthful gap list (which tools violate rules 1 and 3 today) lives in
DECISIONS.md under this same heading. The current pipeline violates rule 1
(produce_v2 runs locally except step4b); rule 3 (4 DEAD + ~18 untested
STANDALONE). Not yet fixed.