# CONTEXT.md — soccer-channel session brief

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
1. NOTHING RUNS ON THE LOCAL MACHINE. No local downloads. No local GPU work.
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