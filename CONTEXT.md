# CONTEXT.md — soccer-channel session brief

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

## THE PIPELINE
Entry point: tools/produce_v2.py
Run it with: ~/yt-digest/.venv/bin/python tools/produce_v2.py <slug>
  --query "<match>" --date-range YYYYMMDD-YYYYMMDD
Working example: 2026-08-30_liverpool-forest --query "Liverpool Forest"
  --date-range 20260801-20260831

8 steps: match data (ESPN) -> boards -> download clip -> tactical
overlays -> voice (ElevenLabs) -> assemble -> merge -> shorts crop.
Confirmed working: exit 0 in 6m31s, output 720x1280, 64.2s, 27.3MB.

tools/produce_episode.py is an older entry point. It calls cv_annotate.py
at line 288. Do not break that.

## WHAT WAS FIXED (verified)
FIX 1: produce_v2.py was downloading 360p clips and accepting them
silently. It now loops up to 5 candidates, ffprobes each, rejects below
720p, uses cookies from secrets/yt_cookies.txt, and fails loudly rather
than falling back. Verified: source is now 1920x1080.
Backup at tools/produce_v2.py.bak.

## WHAT IS STILL BROKEN, WORST FIRST
1. Overlays look like PowerPoint clipart. tactical_overlay.py sends the
   text description to glm-5.2:cloud and asks it to GUESS coordinates
   between 0.0 and 1.0 (lines 88-108). draw_arrow stamps those fixed
   coordinates on all 900 frames (lines 58-61). Nothing looks at pixels.
   A vision model independently confirmed: "PowerPoint clipart, static
   markers, not tracking players."
2. Footage does not match narration. produce_v2.py line 307 cuts at
   fixed 5-second offsets. The script's [VISUAL: description] tags drive
   only the overlays, never the footage selection.
3. The script is factually wrong. It says "Gravenberch receives" but
   match data shows he is a 71st-minute sub for Frimpong. Nothing
   validates the script against match data.
4. No YouTube upload has ever happened. tools/youtube_upload.py exists
   (Aug 23) but publish-log/ is empty and nothing calls it.
5. Output is 720x1280. shorts_crop.py downscales deliberately to avoid
   a soft 1.78x upscale to 1080x1920.

## MEASURED FACTS, DO NOT RE-DERIVE
- cv_annotate.py (YOLOv8 + ByteTrack + KMeans team classification) works
  and produces real tracking. A vision model called its output
  "professional broadcast tracking." produce_v2.py does NOT call it.
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
- cv_annotate.py currently exports team_assignment and ball_positions,
  but NOT per-frame player positions.
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
5. cp <file> <file>.bak before editing anything in tools/.
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
  Backup at tools/cv_annotate.py.bak. One-off runner at tools/runpod_stage1.py.
- Key finding: NO path from tracker ID to player name exists. No jersey OCR,
  no position heuristics, no manual mapping. match_data.py has jersey numbers
  but nothing reads them from video. Stage 2 is arrows on unnamed tracked
  players, or hand-mapped. Tracker fragmentation is high (129 IDs in 10s).

## NEXT TASK (2026-09-05 session 3 end)
C+E prototype built. Next iteration priorities for E (from Claude Opus 5):
1. Add context layer (title, team names, ball marker, attacking direction).
2. Fix pitch layout (centre, complete markings, lower stripe contrast).
3. Rebuild movement encoding with hierarchy (highlight key players,
   smooth trails, separate trail color from team dot color).
C is functional. Full 146s tracking data available.
See PROGRESS.md and STATUS.md for full detail.