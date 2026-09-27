# TOOLS.md — one row per file in tools/

> **Purpose:** one row per file in tools/ (status: WIRED / HAND-RUN).
> **Reader:** every session; mirrored to the public status repo.
> **Last verified against code:** 2026-09-27 (brief 06; census prose line
> re-run brief 09 job 0).

Rebuilt 2026-09-23 (brief 05 job 12) by command; **now enforced by command**
(brief 06 job 5): tools/census.py compares this table against `ls tools/`
and gate B20 fails on any drift — a file with no row, a row naming a missing
file, or class totals that do not equal the file count. Retirements: 25
files (brief 03, git 3d31f0e), boards_2d/board_design/tactical_boards (brief
04 job 3, matplotlib deletion), three_scene.js + three_render3d.py (brief 05
job 7), match_data.py (brief 06 job 1 — ESPN eng.1-only, superseded by
sofascore_client.py as the one fact source). Full history in git,
DECISIONS.md and retired/RETIREMENT_NOTES.md.

## Census (computed by tools/census.py, brief 06 job 5)

$ ls -p tools/ | grep -v / | wc -l  →  41  (USAGE.md exempt from rows)

37 .py + 2 .js + pagecheck_proof.sh + board_page.html + watchlist.yaml +
sofascore_team_ids.json.
28 WIRED, 15 HAND-RUN (deliberately kept operating tools/data). No DEAD class
exists — anything dead is deleted or in retired/.

## The 43 rows

| File | Class | Called by (file:line) | Notes |
|---|---|---|---|
| `produce_v2.py` | WIRED (entry) | nothing (entry point) | end-to-end pipeline; `--flash` runs the flash lane (facts→boards→Modal assemble) |
| `pod_build.py` | WIRED (entry) | nothing (entry point) | flash pipeline on Modal: render3d/render2d/assemble/pull; `--slug` is the episode argument |
| `sofascore_client.py` | WIRED | produce_v2.py step1 | THE fact source, every competition (brief 06 job 1): search resolver + retries, stats mapping, raw cache sofascore_raw.json, fetched_at/source_url stamps |
| `sofascore_team_ids.json` | HAND-RUN | consumed by sofascore_client.py | team-id registry the search resolver sweeps (accumulated by every fetch) |
| `stats_spec.py` | WIRED | produce_v2.py:102 | emits stat_card + possession specs from match_data (replaces tactical_boards, brief 04) |
| `momentum_board.py` | WIRED | produce_v2.py:102 | emits momentum.json from the RAW Sofascore /graph cache (rewritten brief 06 job 2 — the old spec was hand-typed) |
| `xg_flow_board.py` | WIRED | produce_v2.py:102 | emits xg_flow.json from the RAW /shotmap cache |
| `shotmap_board.py` | WIRED | produce_v2.py:102 | emits shotmap.json from the RAW /shotmap cache |
| `avgpositions_board.py` | WIRED | produce_v2.py:102 | emits avgpositions.json from match_data |
| `board_html.js` | WIRED | pod_build.py render_2d (Modal) | puppeteer-core driver: renders board_page.html frames in headless Chromium + the rendered-page audit vs the facts file (brief 06 job 4) |
| `board_page.html` | WIRED | board_html.js | the one HTML/CSS/canvas 2D board page (kinds: stats, momentum, outro, xgflow, shotmap, avgpos) + the draw recorder the audit reads |
| `three_scene_v2.js` | WIRED | pod_build.py render_3d (Modal) | spec-driven 3D board renderer — the ONE 3D engine (v1 retired brief 05 job 7) |
| `board_data_check.py` | WIRED | produce_v2.py step 1d; GATES B16/B18 | every spec number must match the RAW Sofascore responses by team NAME (extended brief 06 job 3) |
| `census.py` | WIRED | GATES B20 | script-driven census: TOOLS.md must match tools/ exactly (brief 06 job 5) |
| `pagecheck_proof.sh` | WIRED | GATES B19 | runs the rendered-page audit on a deliberately mirrored spec (must FAIL) then the real spec (must PASS), locally via the cached Chrome (brief 08 job 3) |
| `pod_frames.py` | HAND-RUN | brief 09 jobs 1-2 (bench study) | Modal ship/extract/pull for benchmark frames: 2s frames + scene-cut frames (threshold printed) + scdet score sweep; reuses volume soccer-build |
| `brief09_check.py` | WIRED | GATES B24 | gate oracle: frames_<L>.csv row counts vs duration/2 within 1 (brief 09 job 8) |
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