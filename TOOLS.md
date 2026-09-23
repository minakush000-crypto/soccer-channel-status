# TOOLS.md — one row per file in tools/

> **Purpose:** one row per file in tools/ (status: WIRED / HAND-RUN).
> **Reader:** every session; mirrored to the public status repo.
> **Last verified against code:** 2026-09-23 (brief 05).

Rebuilt 2026-09-23 (brief 05 job 12) by command, not by hand: the census
script walks the two entry points over python imports and subprocess/argv
invocations (including stem-name invocation without the .py suffix, which is
how produce_v2.py calls the spec emitters). Retirements: 25 files (brief 03,
git 3d31f0e), boards_2d/board_design/tactical_boards (brief 04 job 3,
matplotlib deletion), three_scene.js + three_render3d.py (brief 05 job 7,
the v1 3D engine that cannot run in this environment). Full history of
retired tools is in git, DECISIONS.md and retired/RETIREMENT_NOTES.md.

## Census (computed 2026-09-23 by the transitive walk in the session log)

$ ls tools/ | grep -v "__pycache__\|USAGE" | wc -l  →  38

35 .py + 2 .js (three_scene_v2.js, board_html.js) + watchlist.yaml.
24 WIRED .py + 2 .js, 13 HAND-RUN (deliberately kept operating tools),
1 data file. No DEAD class exists — anything dead is deleted or in retired/.

## The 38 files

| File | Class | Called by (file:line) | Notes |
|---|---|---|---|
| `produce_v2.py` | WIRED (entry) | nothing (entry point) | older end-to-end pipeline (long lane) |
| `pod_build.py` | WIRED (entry) | nothing (entry point) | flash pipeline on Modal: render3d/render2d/assemble/pull; `--slug` is the episode argument (brief 04 job 7) |
| `match_data.py` | WIRED | produce_v2.py:85 | ESPN facts (the only fact source) |
| `stats_spec.py` | WIRED | produce_v2.py:102 | emits stat_card + possession specs from match_data (replaces tactical_boards, brief 04) |
| `momentum_board.py` | WIRED | produce_v2.py:102 | emits momentum.json from Sofascore /graph (spec emitter; renders nothing — brief 04) |
| `xg_flow_board.py` | WIRED | produce_v2.py:102 | emits xg_flow.json from Sofascore /shotmap |
| `shotmap_board.py` | WIRED | produce_v2.py:102 | emits shotmap.json from Sofascore /shotmap |
| `avgpositions_board.py` | WIRED | produce_v2.py:102 | emits avgpositions.json from match_data |
| `board_html.js` | WIRED | pod_build.py:88 (Modal) | puppeteer-core driver: renders board_page.html frames in headless Chromium — THE 2D board renderer |
| `board_page.html` | WIRED | board_html.js | the one HTML/CSS/canvas 2D board page (kinds: stats, momentum, outro, xgflow, shotmap, avgpos) |
| `three_scene_v2.js` | WIRED | pod_build.py:~71 (Modal) | spec-driven 3D board renderer — the ONE 3D engine (v1 retired brief 05 job 7) |
| `board_data_check.py` | WIRED | produce_v2.py step 1d; GATES B16 | every board number must match match_data.json by team NAME (brief 05 job 3) |
| `script_gen.py` | WIRED | produce_v2.py:372 | lane script draft |
| `validate_script.py` | WIRED | produce_v2.py:386; tests | grammar + source validation |
| `transformation_gate.py` | WIRED | produce_v2.py:667 | lane D floor |
| `broadcast_filler.py` | WIRED | produce_v2.py:428 | broadcast/filler classification |
| `cut_list_gen.py` | WIRED | produce_v2.py:438 | footage windows (PySceneDetect snap) |
| `ffmpeg_utils.py` | WIRED | produce_v2.py:47 + importers | 200MB guard, cleanup, get_duration |
| `staging.py` | WIRED | produce_v2.py:48 | /mnt/f staging (fails loudly if unmounted) |
| `runpod_download.py` | WIRED | produce_v2.py:311 (--pod-download) | pod-side download |
| `generate_voice.py` | WIRED | produce_v2.py:708 | ElevenLabs TTS |
| `generate_ambience.py` | WIRED | produce_v2.py:730 | crowd ambience |
| `merge_voice.py` | WIRED | produce_v2.py:751; imported by generate_voice | voice+ambience mix |
| `shorts_crop.py` | WIRED | produce_v2.py:761 | 9:16 crop |
| `script_utils.py` | WIRED | imported by generate_voice.py | canonical clean_script_for_tts |
| `glm_judge.py` | HAND-RUN | hand-invoked; CLAUDE.md routing | THE judge (session model; --seed + --runs N median, brief 04 job 10a) |
| `scoreboard_scan.py` | HAND-RUN | hand-invoked | goal ground truth from the score bug (Stage 15 litmus winner) |
| `viral_angle.py` | HAND-RUN | hand-invoked | topic research (YouTube API) |
| `agent_reach_research.py` | HAND-RUN | hand-invoked | multi-platform research |
| `fresh_fetch.py` | HAND-RUN | hand-invoked | dated news fetch (kills stale rumors) |
| `youtube_upload.py` | HAND-RUN | hand-invoked | publishing (private uploads) |
| `oauth_setup.py` | HAND-RUN | hand-invoked | YouTube OAuth |
| `thumbnail_generator.py` | HAND-RUN | hand-invoked | thumbnails |
| `b2_archive.py` | HAND-RUN | hand-invoked per task | B2 upload + provenance manifest (archive of record) |
| `backup_env.py` | HAND-RUN | hand-invoked | encrypted .env backup |
| `pod_check.py` | HAND-RUN (hook-adjacent) | .claude/hooks/check_pods.sh | pod-leak check + --terminate |
| `doc_stamp_check.py` | HAND-RUN (hook-adjacent) | .claude/hooks/check_doc_stamps.sh | stale canonical-doc stamp check |
| `watchlist.yaml` | HAND-RUN | consumed by fresh_fetch.py | watchlist config |

## Project hooks (8 registered in .claude/settings.json, read 2026-09-23)

1. check_pods.sh (SessionStart) — surfaces leaked RunPod pods. Still matters (memory: pod leaks bill).
2. check_mnt_f.sh (SessionStart) — warns if /mnt/f is unmounted. Still matters (footage staging).
3. check_doc_stamps.sh (SessionStart) — flags canonical docs with stale stamps. Still matters.
4. skill-activation-prompt.sh (UserPromptSubmit) — Node skill suggestions + session intelligence (claude-mem). Matters (active).
5. skill-verification-guard.sh (PreToolUse Edit|MultiEdit|Write) — Node skill enforcement. Matters (active).
6. post-tool-use-tracker.sh (PostToolUse) — tracks edited files/repos (feeds claude-mem). Matters (active).
7. skill-activation-tracker.sh (PostToolUse) — clears activated skills from pending lists. Matters (active).
8. push_status.sh (Stop) — publishes the public mirror under the brief 04 allowlist + secret scan. Critical.
Unregistered leftovers in the folder: block-image-read.sh (dead — its hook was
removed 2026-09-22; candidate for retirement), block-retired.sh and
warn-local-gpu.sh (registered only in ~/.claude/settings.json, global — not
project), _run-node-hook.sh + *.ts (helpers for 4-7).

## Archive tooling

rclone, remote "b2", bucket "mendymax-archive"
(`rclone lsd b2:` lists it). tools/b2_archive.py writes the provenance
manifest and uploads artifact + manifest together. Verified in use
2026-09-23 (q_research_scratch tarball).