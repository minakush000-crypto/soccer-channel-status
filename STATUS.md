# STATUS.md — verified current state

> **Purpose:** verified current state of the pipeline (the status spine).
> **Reader:** every session (CLAUDE.md @STATUS.md).
> **Last verified against code:** 2026-09-22.

Rewritten 2026-09-22 (brief 03) against the code on disk. Pre-flash history
(Gemini judge era, tactical render lane, 46-tool census, staged migration
plan) lives in DECISIONS.md, PROGRESS.md, and LANE_PLAN.md. Every claim
below carries the command that proved it.

## The two entry points

1. **tools/pod_build.py** — the flash pipeline (what built EP001). Runs on
   Modal, volume "soccer-build". Subcommands:
   - `render3d <spec>` — node three_scene_v2.js on Modal, frames → MP4.
   - `render2d <spec>` — boards_2d.py (matplotlib) on Modal.
   - `assemble` — parses scripts/<slug>.md ([VISUAL: board=/footage=]
     contract), cuts footage from /vol/in/reel.mp4, loops boards from
     /vol/out, concats, mixes voice (1.6x) + crowd ambience (0.3x),
     writes /vol/out/final_video.mp4. Bring home with
     `modal volume get --force soccer-build out/final_video.mp4 <local>`.
2. **tools/produce_v2.py** — the older full-pipeline entry (ESPN match data →
   script_gen → validate_script → tactical_boards + 3-step 2D/3D boards →
   yt-dlp download (local or --pod-download) → cut list → ElevenLabs voice →
   assemble → merge → shorts crop). 910 lines. Still functional; the flash
   pipeline superseded it for EP001's build.

## EP001 — 2026-09-08 Real Madrid 2-1 Inter (the flash reboot episode)

$ ffprobe -v error -show_entries format=duration,size -of csv=p=0 \
    renders/2026-09-08_real-madrid-inter/final_video.mp4
140.920000,76519745

- Third build (v3): original flash build → goal3 board fix (Inter sub numbers
  from the CBS lineup graphic) → brief 03 board fixes (momentum labels,
  outro card). Voice unchanged (155.5 WPM, ElevenLabs).
- 19 sections, 365 words: 12 footage + 7 board. Boards: 4 render3d
  (formation_clash, counter_map, goal3_pattern, valverde_strike), 3 render2d
  (stats_possession, momentum, scoreline_outro).
- Judge (session model, JUDGE_FLASH.md): 6-8 per frame. Mayo's verdict on
  v2: "an improvement from previous outputs, it's a step in the right
  direction."
- Brief 03 fixes verified by frames viewed in the re-assembled final:
  momentum goal labels staggered on two lanes (no overlap, title unclipped);
  outro rises as one composition and holds complete (no four-section stepping).

## Tool census (after brief 03 retirement)

$ ls tools/ | grep -v "__pycache__\|USAGE" | wc -l
40

37 .py + 2 .js + watchlist.yaml. 25 files retired 2026-09-22 per call graph
(tarball + manifest in b2:mendymax-archive/soccer-channel/2026-09-23/; git
commit 3d31f0e). Live tool classes:
- Pipeline (reachable): produce_v2, pod_build, match_data, script_gen,
  validate_script, transformation_gate, tactical_boards, board_design,
  xg_flow_board, shotmap_board, momentum_board, avgpositions_board,
  boards_2d, three_render3d, three_scene.js, three_scene_v2.js,
  broadcast_filler, cut_list_gen, ffmpeg_utils, staging, runpod_download,
  generate_voice, generate_ambience, merge_voice, shorts_crop, script_utils.
- Hand-run operating tools (kept deliberately, each import-smoke-tested):
  glm_judge (the judge), scoreboard_scan (ground-truth goal finding),
  viral_angle + agent_reach_research + fresh_fetch (topic research),
  youtube_upload + oauth_setup (publishing), thumbnail_generator,
  b2_archive (B2 + provenance), backup_env, pod_check + doc_stamp_check
  (both wired into SessionStart hooks).
- Models: glm-5.3-flash:cloud only (session model, judge, vision). Gemini is
  RETIRED (API key 402 RESOURCE_EXHAUSTED since 2026-09-21; gemini_judge.py
  sits in retired/ as a record). ElevenLabs: voice + ambience, working.

## What is broken, worst first

1. **Judge wobble.** The session judge scores ±1-2 on identical frames
   (measured 2026-09-22: nine runs at temperature 0 gave 3-5 on one frame).
   reasoning_effort does not reach the model through Ollama. Known, not
   fixed; load-bearing scores need multiple judge runs with the spread
   reported.
2. **Footage overruns produce short freezes.** 4 of 12 footage sections get
   a tpad last-frame clone (holds 0.1-1.1s; worst: "The save" seg12, 1.1s)
   because allocated duration exceeds the verified window. Cosmetic but
   visible. Fix would reallocate duration or trim windows; not done in
   brief 03.
3. **Public mirror content risk (flagged, not fixed).** push_status.sh
   copies vault markdown to github.com/minakush000-crypto/
   soccer-channel-status with no content filter (only files literally named
   .env are deleted). Anything sensitive in a vault markdown file is
   published. Needs an owner decision on a content filter.
4. **Unclaimed possible disallowed goal at reel t=90.** Unresolved; would
   change goal3 narration if confirmed.
5. **Migration phases 5-8 unfinished.** Superseded in priority by the
   benchmark-first approach; the flash pipeline is the working path.

## Hand-run command reference

| Task | Command |
|------|---------|
| Render a 3D board (Modal) | ~/yt-digest/.venv/bin/python tools/pod_build.py render3d <spec> |
| Render a 2D board (Modal) | ~/yt-digest/.venv/bin/python tools/pod_build.py render2d <spec> |
| Assemble the episode (Modal) | ~/yt-digest/.venv/bin/python tools/pod_build.py assemble |
| Pull the final home | ~/yt-digest/.venv/bin/modal volume get --force soccer-build out/final_video.mp4 renders/<slug>/final_video.mp4 |
| Judge a frame | ~/yt-digest/.venv/bin/python tools/glm_judge.py --image <file> "prompt" |
| Full pipeline (older path) | ~/yt-digest/.venv/bin/python tools/produce_v2.py <slug> --query "<match>" |
| Archive to B2 with manifest | ~/yt-digest/.venv/bin/python tools/b2_archive.py --file <f> --project soccer-channel --task "<t>" --produced-by "<model> via Claude Code" |
| Upload to YouTube (private) | ~/yt-digest/.venv/bin/python tools/youtube_upload.py |
| Topic research | tools/viral_angle.py, tools/agent_reach_research.py, tools/fresh_fetch.py |
| Goal ground truth | tools/scoreboard_scan.py |

## Facts that were measured, do not re-derive

- Output is 1920x1080 (pod_build FOOT/SCALE filters). ffprobe verified.
- Ball detection on compressed wide-shot reuploads is dead (8/12 frames no
  ball). Anchor to players only. cv_annotate (retired 2026-09-22) was the
  tracking implementation; git history holds it.
- Scoreboard scanner beats LLM video reading on timestamps (Stage 15 litmus:
  scanner 4/4 goals, 0 phantoms, free vs Gemini 2/4, 1 phantom, paid).
  Gemini never returned for this; glm video reading has the same known
  weakness on WHEN.
- The B2 bucket is the archive of record: rclone remote "b2", bucket
  "mendymax-archive", per-project prefix, manifests written by
  tools/b2_archive.py at archive time.