# STATUS.md — verified current state

> **Purpose:** verified current state of the pipeline (the status spine).
> **Reader:** every session (CLAUDE.md @STATUS.md).
> **Last verified against code:** 2026-09-23 (brief 05).

Rewritten 2026-09-23 (brief 04) against the code on disk. Brief 03 history and
pre-flash history (Gemini judge era, tactical render lane, staged migration
plan) live in DECISIONS.md, PROGRESS.md, and retired/. Every claim below
carries the command that proved it.

## The two entry points

1. **tools/pod_build.py** — the flash pipeline (what built EP001). Runs on
   Modal, volume "soccer-build". The episode slug is the CLI argument
   (`--slug` on assemble; `pull <slug>`), never a constant (brief 04 job 7).
   Subcommands:
   - `render3d <spec> --slug <slug>` — node three_scene_v2.js on Modal;
     specs at /vol/specs/<slug>/, output at /vol/out/<slug>/. A spec whose
     slug field contradicts the episode is REFUSED (brief 05 job 8).
   - `render2d <spec> --slug <slug>` — board_html.js drives board_page.html
     in headless Chromium on Modal (the ONE 2D board renderer; matplotlib
     deleted), same per-episode namespace and refusal.
   - `assemble [--slug X]` — parses scripts/<slug>.md ([VISUAL: board=/footage=]
     contract), cuts footage from /vol/in/reel.mp4, loops boards from
     /vol/out/<slug>/ (verifying every board's spec slug), mixes voice (1.6x)
     + ambience (0.3x), writes /vol/out/<slug>/final_video.mp4 and pulls it
     home to renders/<slug>/ automatically. A retime pass caps every footage
     section at its verified window (brief 04 job 2).
   - `pull <slug>` — pull the finished final home without Modal compute.
2. **tools/produce_v2.py** — the older full-pipeline entry (ESPN match data →
   script_gen → validate_script → board spec emitters + 3-step 2D/3D boards →
   yt-dlp download (local or --pod-download) → cut list → ElevenLabs voice →
   assemble → merge → shorts crop). Its board step renders through the SAME
   HTML renderer via pod_build render2d (brief 04 job 3).

## EP001 — 2026-09-08 Real Madrid 2-1 Inter (the flash reboot episode)

$ ffprobe -v error -show_entries format=duration,size -of csv=p=0 \
    renders/2026-09-08_real-madrid-inter/final_video.mp4
140.960000,<size> (v6, brief 05 boards — ffprobe at the Job 11 close)

- Six builds: flash v1 → goal3 board fix (v2) → brief 03 board fixes (v3) →
  brief 04 retime (v4) → brief 04 HTML-renderer boards (v5) → brief 05
  data-fix + balanced outro boards (v6). Voice unchanged (155.5 WPM,
  ElevenLabs). Facts file: renders/2026-09-08_real-madrid-inter/match_data.json
  (tracked, Sofascore event 16938768, fetched_at inside; lineups + average
  positions included).
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

## Tool census (after brief 05 job 7)

$ ls tools/ | grep -v "__pycache__\|USAGE" | wc -l  →  38

35 .py + 2 .js (three_scene_v2.js, board_html.js) + watchlist.yaml.
24 WIRED .py + 2 .js, 13 HAND-RUN (full table in TOOLS.md, computed by a
transitive walk 2026-09-23). three_render3d.py + three_scene.js (the v1 3D
engine) retired to retired/ — v1 required a full puppeteer install that
exists nowhere in this environment; v2 is spec-driven and produced EP001's
3D boards.

## What is broken, worst first

1. **Judge wobble is a model property.** ±2 across 5 runs at temperature 0 +
   seed 0 (measured 2026-09-23). Median-of-N (--runs) is the mitigation;
   single-shot scores are not load-bearing.
2. **Every number is now mechanically checked** (brief 05 job 3):
   tools/board_data_check.py runs as produce_v2 step 1d and as gate B16; any
   board number contradicting match_data.json fails the build. GAPS #1-#4 of
   the brief-05 brief are closed: stats rows, momentum fill, facts file,
   scripts/ tracked, vault removed, one 3D engine, spec namespaces.

## Hand-run command reference

| Task | Command |
|------|---------|
| Render a 3D board (Modal) | ~/yt-digest/.venv/bin/python tools/pod_build.py render3d <spec> --slug <slug> |
| Render a 2D board (Modal) | ~/yt-digest/.venv/bin/python tools/pod_build.py render2d <spec> --slug <slug> |
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
  docs, empty vault list, secret scan blocks the push on any hit. The 16
  legacy vault files were removed from the mirror HEAD on 2026-09-23
  (scanned clean first; history kept, brief 05 job 5).
- Sofascore average-positions averageX/averageY are 0-100 already (do NOT
  scale); lineups give formation + 11 starters per side.