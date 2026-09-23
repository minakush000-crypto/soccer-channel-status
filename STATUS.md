# STATUS.md — verified current state

> **Purpose:** verified current state of the pipeline (the status spine).
> **Reader:** every session (CLAUDE.md @STATUS.md).
> **Last verified against code:** 2026-09-23.

Rewritten 2026-09-23 (brief 04) against the code on disk. Brief 03 history and
pre-flash history (Gemini judge era, tactical render lane, staged migration
plan) live in DECISIONS.md, PROGRESS.md, and retired/. Every claim below
carries the command that proved it.

## The two entry points

1. **tools/pod_build.py** — the flash pipeline (what built EP001). Runs on
   Modal, volume "soccer-build". The episode slug is the CLI argument
   (`--slug` on assemble; `pull <slug>`), never a constant (brief 04 job 7).
   Subcommands:
   - `render3d <spec>` — node three_scene_v2.js on Modal, frames → MP4.
   - `render2d <spec>` — board_html.js drives board_page.html in headless
     Chromium on Modal (the ONE 2D board renderer; matplotlib deleted).
   - `assemble [--slug X]` — parses scripts/<slug>.md ([VISUAL: board=/footage=]
     contract), cuts footage from /vol/in/reel.mp4, loops boards from /vol/out,
     mixes voice (1.6x) + ambience (0.3x), writes /vol/out/final_video.mp4.
     A retime pass caps every footage section at its verified window and gives
     the surplus to board sections (brief 04 job 2); with --slug the final is
     pulled home to renders/<slug>/ automatically.
   - `pull <slug>` — pull the finished final home without Modal compute.
2. **tools/produce_v2.py** — the older full-pipeline entry (ESPN match data →
   script_gen → validate_script → board spec emitters + 3-step 2D/3D boards →
   yt-dlp download (local or --pod-download) → cut list → ElevenLabs voice →
   assemble → merge → shorts crop). Its board step renders through the SAME
   HTML renderer via pod_build render2d (brief 04 job 3).

## EP001 — 2026-09-08 Real Madrid 2-1 Inter (the flash reboot episode)

$ ffprobe -v error -show_entries format=duration,size -of csv=p=0 \
    renders/2026-09-08_real-madrid-inter/final_video.mp4
140.960000,75684053 (v5, brief 04 boards)

- Five builds: flash v1 → goal3 board fix (v2) → brief 03 board fixes (v3) →
  brief 04 retime (v4) → brief 04 HTML-renderer boards (v5). Voice unchanged
  (155.5 WPM, ElevenLabs).
- 19 sections, 365 words: 12 footage + 7 board. Boards: 4 render3d
  (formation_clash, counter_map, goal3_pattern, valverde_strike), 3 render2d
  (stats_possession, momentum, scoreline_outro) — all 2D boards now come out
  of board_page.html (proofs in reports/brief04/board_*_old.png vs
  board_*_new.png).
- The "tpad freezes" of GAPS.md #3 never existed: the tpad filter is dead in
  this invocation (input stream never ends mid-segment); the real defect was
  footage overrunning its verified window by 0.02-1.13s. Fixed by the retime
  pass; verified by dense frame scan + the allocation table (brief 04 job 2).
- Judge (session model): wobble is a model property — temperature 0 + fixed
  seed still gave 6,8,6,6,8 on one board. Rule: judge with --runs 5 and use
  the median (glm_judge.py implements it, brief 04 job 10a).

## Tool census (after brief 04 job 3/11)

$ ls tools/ | grep -v "__pycache__\|USAGE" | wc -l  →  39

35 .py + 3 .js (three_scene.js, three_scene_v2.js, board_html.js) +
watchlist.yaml. 25 WIRED, 13 HAND-RUN (full table in TOOLS.md, computed by a
transitive walk 2026-09-23). matplotlib is gone from requirements.txt and
from every tool (grep: 0 importers).

## What is broken, worst first

1. **Judge wobble is a model property.** ±2 across 5 runs at temperature 0 +
   seed 0 (measured 2026-09-23). Median-of-N (--runs) is the mitigation;
   single-shot scores are not load-bearing.
2. **/vol/specs is a single-episode namespace.** Two episodes emitting the
   same spec name (momentum.json) collide on the Modal volume; the collision
   put a Bournemouth board into an EP001 assemble before it was caught
   (2026-09-23). pod_build needs per-slug spec paths or a sync step that
   puts only the current episode's specs.
3. **Two 3D scene engines coexist.** three_scene.js (produce_v2 formation)
   and three_scene_v2.js (flash named boards) overlap in capability. Both
   are wired; consolidation is a candidate for a later brief.
4. **scripts/ is gitignored** (soccer-channel/.gitignore:4). Episode scripts
   exist only locally + on the Modal volume. A copy loss loses the pipeline
   input of record.
5. **Legacy vault files remain published** in the public mirror (17 files,
   published before the brief 04 allowlist). Removal is Mayo's decision.

## Hand-run command reference

| Task | Command |
|------|---------|
| Render a 3D board (Modal) | ~/yt-digest/.venv/bin/python tools/pod_build.py render3d <spec> |
| Render a 2D board (Modal) | ~/yt-digest/.venv/bin/python tools/pod_build.py render2d <spec> |
| Assemble + pull home | ~/yt-digest/.venv/bin/python tools/pod_build.py assemble --slug <slug> |
| Pull the final home | ~/yt-digest/.venv/bin/python tools/pod_build.py pull <slug> |
| Judge a frame (stable) | ~/yt-digest/.venv/bin/python tools/glm_judge.py --image <file> --runs 5 "<prompt>" (use the median) |
| Emit board specs from live data | tools/stats_spec.py / momentum_board.py / xg_flow_board.py / shotmap_board.py / avgpositions_board.py --slug <slug> |
| Full pipeline (long lane) | ~/yt-digest/.venv/bin/python tools/produce_v2.py <slug> --query "<match>" |
| Archive to B2 with manifest | ~/yt-digest/.venv/bin/python tools/b2_archive.py --file <f> --project soccer-channel --task "<t>" --produced-by "<model> via Claude Code" |
| Upload to YouTube (private) | ~/yt-digest/.venv/bin/python tools/youtube_upload.py |
| Topic research | tools/viral_angle.py, tools/agent_reach_research.py, tools/fresh_fetch.py |
| Goal ground truth | tools/scoreboard_scan.py |

## Facts that were measured, do not re-derive

- Output is 1920x1080. ffprobe verified every build.
- Sofascore shotmap playerCoordinates: x measures distance from the goal the
  team ATTACKS (bournemouth event: goals at x=11-15, per-side means ~13,
  measured 2026-09-23). The board page maps home to (100-x) [right goal] and
  away to x [left goal]. The old matplotlib renderer plotted raw x and showed
  teams attacking the wrong end.
- Scoreboard scanner beats LLM video reading on timestamps (Stage 15 litmus).
  Use the scanner for WHEN; the session model for WHAT.
- Modal costs (measured 2026-09-23 from `modal billing report --for today`):
  14 runs = $0.1004; month-to-date metered $1.78, billed $0.00 (credits).
- The B2 bucket is the archive of record: rclone remote "b2", bucket
  "mendymax-archive", per-project prefix, manifests written by
  tools/b2_archive.py at archive time.
- The public mirror is allowlist+scan locked (brief 04 job 4): 11 project
  docs, empty vault list, secret scan blocks the push on any hit.