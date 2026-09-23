# TOOLS.md — one row per file in tools/

> **Purpose:** one row per file in tools/ (status: WIRED / HAND-RUN).
> **Reader:** every session; mirrored to the public status repo.
> **Last verified against code:** 2026-09-22.

Rebuilt 2026-09-22 (brief 03 job 1). Reachability = BFS from the two entry
points (tools/produce_v2.py, tools/pod_build.py) over python imports and
subprocess/argv invocations (the same method as the retirement). 25 files
were retired 2026-09-22 (git commit 3d31f0e; tarball + manifest at
b2:mendymax-archive/soccer-channel/2026-09-23/). Full history of retired
tools is in git and in DECISIONS.md.

## Census

$ ls tools/ | grep -v "__pycache__\|USAGE" | wc -l
40

37 .py + 2 .js + watchlist.yaml. 26 WIRED (reachable from an entry point),
13 HAND-RUN (deliberately kept operating tools; each import-smoke-tested
2026-09-22), 1 data file. No DEAD class exists anymore — anything dead was
deleted.

## The 40 files

| File | Class | Called by (file:line) | Notes |
|---|---|---|---|
| `produce_v2.py` | WIRED (entry) | nothing (entry point) | older end-to-end pipeline, 910 lines |
| `pod_build.py` | WIRED (entry) | nothing (entry point) | flash pipeline on Modal: render3d/render2d/assemble |
| `match_data.py` | WIRED | produce_v2.py:85 | ESPN facts (the only fact source; parse_feed retired) |
| `script_gen.py` | WIRED | produce_v2.py:376 | lane script draft |
| `validate_script.py` | WIRED | produce_v2.py:390; tests | 2026-09-22: skips [SRC:] mentions in comment lines |
| `transformation_gate.py` | WIRED | produce_v2.py:671 | lane D floor |
| `tactical_boards.py` | WIRED | produce_v2.py:98 | 2D possession/stat_card/formation fallback |
| `three_render3d.py` | WIRED | produce_v2.py:138 | drives three_scene.js via Puppeteer |
| `three_scene.js` | WIRED | three_render3d.py:29 | 3D formation renderer (produce_v2 path) |
| `three_scene_v2.js` | WIRED | pod_build.py:59 (Modal) | spec-driven 3D board renderer (flash path) |
| `boards_2d.py` | WIRED | pod_build.py:76 (Modal) | animated 2D boards: stats/momentum/outro |
| `board_design.py` | WIRED | imported by the 4 board tools | shared matplotlib style |
| `xg_flow_board.py` | WIRED | produce_v2.py:109 | xG flow board |
| `shotmap_board.py` | WIRED | produce_v2.py:110 | shotmap board |
| `momentum_board.py` | WIRED | produce_v2.py:111 | produce_v2 momentum variant (flash uses boards_2d) |
| `avgpositions_board.py` | WIRED | produce_v2.py:112 | avg positions board |
| `broadcast_filler.py` | WIRED | produce_v2.py:432 | broadcast/filler classification |
| `cut_list_gen.py` | WIRED | produce_v2.py:442 | footage windows (PySceneDetect snap) |
| `ffmpeg_utils.py` | WIRED | produce_v2.py:47 + 5 importers | 200MB guard, cleanup, get_duration |
| `staging.py` | WIRED | produce_v2.py:48 | /mnt/f staging (fails loudly if unmounted) |
| `runpod_download.py` | WIRED | produce_v2.py:316 (--pod-download) | pod-side download, guarded excerpts home |
| `generate_voice.py` | WIRED | produce_v2.py:712 | ElevenLabs TTS |
| `generate_ambience.py` | WIRED | produce_v2.py:734 | crowd ambience |
| `merge_voice.py` | WIRED | produce_v2.py:755; imported by generate_voice.py:124 | voice+ambience mix |
| `shorts_crop.py` | WIRED | produce_v2.py:765 | 9:16 crop |
| `script_utils.py` | WIRED | generate_voice.py:25 | canonical clean_script_for_tts |
| `glm_judge.py` | HAND-RUN | hand-invoked; CLAUDE.md tool routing | THE judge (session model; writes model into score records) |
| `scoreboard_scan.py` | HAND-RUN | hand-invoked | goal ground truth from the score bug (Stage 15 litmus winner) |
| `viral_angle.py` | HAND-RUN | hand-invoked | topic research (YouTube API) |
| `agent_reach_research.py` | HAND-RUN | hand-invoked | multi-platform research |
| `fresh_fetch.py` | HAND-RUN | hand-invoked | dated news fetch (kills stale rumors) |
| `youtube_upload.py` | HAND-RUN | hand-invoked | publishing (private uploads) |
| `oauth_setup.py` | HAND-RUN | hand-invoked | YouTube OAuth |
| `thumbnail_generator.py` | HAND-RUN | hand-invoked | thumbnails |
| `b2_archive.py` | HAND-RUN | hand-invoked per task | B2 upload + provenance manifest (the archive of record) |
| `backup_env.py` | HAND-RUN | hand-invoked | encrypted .env backup |
| `pod_check.py` | WIRED (hook) | .claude/hooks/check_pods.sh | pod-leak check + --terminate |
| `doc_stamp_check.py` | WIRED (hook) | .claude/hooks/check_doc_stamps.sh | stale canonical-doc stamp check |
| `three_scene_v2.js` | WIRED | pod_build.py:59 | (listed above; .js) |
| `three_scene.js` | WIRED | three_render3d.py:29 | (listed above; .js) |
| `watchlist.yaml` | HAND-RUN | consumed by fresh_fetch.py | watchlist config |

## Archive tooling

rclone v1.75.1, remote "b2", bucket "mendymax-archive"
(`rclone lsd b2:` lists it). tools/b2_archive.py writes the provenance
manifest and uploads artifact + manifest together. Verified in use
2026-09-22 (retired-tools tarball + EP001 v3 final).