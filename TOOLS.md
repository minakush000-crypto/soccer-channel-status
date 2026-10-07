# TOOLS.md — one row per file in tools/

> **Purpose:** one row per file in tools/ (status: WIRED / HAND-RUN).
> **Reader:** every session; mirrored to the public status repo.
> **Last verified against code:** 2026-10-07 (brief 17 phase B: brief17_stage.py + brief17_reel.py + brief17_stills.py wired; keeper-row truncation fixed in brief17_cards.py; rows below verified.)
> 56 files / 35 WIRED / 21 HAND-RUN; three new rows added brief 13).

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

$ ls -p tools/ | grep -v / | wc -l  →  53  (USAGE.md exempt from rows)

40 .py + 3 .js + 7 .sh + board_page.html + watchlist.yaml +
sofascore_team_ids.json.
35 WIRED, 21 HAND-RUN (deliberately kept operating tools/data). No DEAD class
exists — anything dead is deleted or in retired/.

## The 53 rows

| File | Class | Called by (file:line) | Notes |
|---|---|---|---|
| `produce_v2.py` | WIRED (entry) | nothing (entry point) | end-to-end pipeline; `--flash` runs the flash lane (facts→boards→Modal assemble) |
| `pod_build.py` | WIRED (entry) | nothing (entry point) | flash pipeline on Modal: render3d/render2d/assemble/pull; `--slug` is the episode argument |
| `sofascore_client.py` | WIRED | produce_v2.py step1 | THE fact source, every competition (brief 06 job 1): search resolver + retries, stats mapping, raw cache sofascore_raw.json, fetched_at/source_url stamps; brief 12: per-round Chromium fallback (`sofascore_chrome_fetch.js`) when Akamai challenges every impersonation |
| `sofascore_chrome_fetch.js` | WIRED | sofascore_client.py `_chrome_fetch` | the SAME endpoints through the real cached Chromium (genuine TLS/JS stack) when curl_cffi impersonations are challenged 40+ min (measured 2026-09-28); a transport, not a second fact source (brief 12) |
| `sofascore_team_ids.json` | HAND-RUN | consumed by sofascore_client.py | team-id registry the search resolver sweeps (accumulated by every fetch) |
| `stats_spec.py` | WIRED | produce_v2.py:102 | emits stat_card + possession specs from match_data (replaces tactical_boards, brief 04) |
| `momentum_board.py` | WIRED | produce_v2.py:102 | emits momentum.json from the RAW Sofascore /graph cache (rewritten brief 06 job 2 — the old spec was hand-typed) |
| `xg_flow_board.py` | WIRED | produce_v2.py:102 | emits xg_flow.json from the RAW /shotmap cache |
| `shotmap_board.py` | WIRED | produce_v2.py:102 | emits shotmap.json from the RAW /shotmap cache |
| `avgpositions_board.py` | WIRED | produce_v2.py:102 | emits avgpositions.json from match_data |
| `board_html.js` | WIRED | pod_build.py render_2d (Modal) | puppeteer-core driver: renders board_page.html frames in headless Chromium + the rendered-page audit vs the facts file (brief 06 job 4) |
| `board_page.html` | WIRED | board_html.js | the one HTML/CSS/canvas 2D board page (kinds: stats, momentum, outro, xgflow, shotmap, avgpos) + the draw recorder the audit reads |
| `three_scene_v2.js` | WIRED | pod_build.py render_3d (Modal) | spec-driven 3D board renderer — the ONE 3D engine (v1 retired brief 05 job 7) |
| `board_data_check.py` | WIRED | produce_v2.py step 1d; GATES B16/B18 | every spec number must match the RAW Sofascore responses by team NAME (extended brief 06 job 3); brief 12: kind timeline (name+jersey pairs vs raw XI, x/y vs raw average positions, step targets, schematic path bounds) |
| `brief12_check.py` | WIRED | GATES B56-B61 | brief-12 gate oracle: specs/durations/keyframes/swap/sheets/numbers over reports/brief12/render_manifest.json (writes only under /tmp) |
| `brief13_check.py` | WIRED | GATES B62-B65 | brief-13 gate oracle: sizing/overlap/identity/drift with positive controls (writes only under /tmp) |
| `brief14_check.py` | WIRED | GATES B71-B79 | brief-14 gate oracle: plan/applied/hooks/mcp/memory/after_table/quality over reports/brief14/plan.json (reads settings and ~/.claude.json; writes only under /tmp) |
| `brief15_check.py` | WIRED | GATES B83-B89 | brief-15 gate oracle: kit/overlay/freeze/cards/renders/publish (page audits via node board_html.js --audit-only, cached chrome; writes only under /tmp) |
| `brief15_cards.py` | WIRED | GATES B87 (5 card renders) | brief-15 card spec emitter: the 5 bench-B card specs to reports/brief15/cards/, every number walked from the cached Sofascore raws, audit_T mirrored from the page reveal math |
| `brief15_photo.py` | WIRED | GATES B89 (photo rows) | pod-side subject cutout for the title cards: Modal + rembg, parameterized since brief 17 job 0b (in-name/out-name/vol-dir; the Yamal default unchanged) |
| `brief15_sheets.py` | WIRED | GATES B88/B89 (sheets + keyframes) | job-6 publisher: before/after overlay sheets, per-card sheets (640px) + committed keyframes from the rendered MP4s |
| `brief13_overlays.py` | WIRED | pod_build.py render_footage; brief 13 jobs 3-5 | freeze-frame overlay spec emitter from reports/brief13/marks.json (image-space steps, identity sources) |
| `brief13_frames.py` | HAND-RUN | hand-invoked (brief 13 jobs 4-6) | pod-side stamped/clean/full-res frame extraction for marking + visual check (frames live on the volume) |
| `brief16_data.py` | WIRED | GATES B92-B95 oracle + hand-run | brief-16 driver: re-fetch/chosen matches + press_facts.json rows + md renderer matches.md/press_facts.md/clip_list.md from the JSON sidecars; every fetch through the batch chrome transport (SOFA_CHROME_FIRST) |
| `brief16_check.py` | WIRED | GATES B92-B100 (9 subcommands) | brief-16 gate oracle: matches/freshness/facts/claims/footage/clips/sheet/sizing/overlay, each with planted positive controls; local page audits via cached chrome |
| `brief16_frames.py` | HAND-RUN | brief 16 job 5 (frames) | pod-side 4s stamped survey + arbitrary-range 1s/0.5s drills to out/brief16/frames/, one pull tarball with integrity retry |
| `brief16_clips.py` | HAND-RUN | brief 16 job 6 (cuts) | pod-side clip cutter from clip_list.json (accurate seek, h264, prints CCLIP lines) |
| `brief16_footage.py` | HAND-RUN | brief 16 job 4 | official-highlights chain: staged-reuse download -> b2_archive + sha1 verify -> modal volume put + ls verify -> local delete; adopt path for the brief-13 volume files |
| `brief16_sheets.py` | HAND-RUN | brief 16 jobs 0/5/7 sheets | before/after pair sheets, 4-across review tiles (320px), 640px contact sheet |
| `brief16_retry_fetch.sh` | HAND-RUN | brief 16 job 1 (Akamai loop) | single-request re-fetch loop (~20 min apart, 5 attempts), logs fetch_attempts.log, copies the fresh list to /tmp on success |
| `brief17_check.py` | WIRED | GATES B102-B107 oracle | brief-17 phase-A oracle: chips (crops+kit rule+planted), title (subject share vs benchmark B), draw (arrows on entities + overlap walk + planted), claims (re-side trace + keeper axis), script (rows/assets/number trace + pace), voice (sample + mirror sha). Reads only cached raws + renders; never touches Sofascore |
| `brief17_stage.py` | WIRED | brief17_reel.py, pod_build.py assemble | brief-17 phase-B staging: parses the script the way the pod's assemble does, sizes every board spec to its allocated segment (duration_s = ceil(alloc+1.5); card hold sized so the page's fade-out clears the row), sets slug + per-spec facts (timeline -> match_data_racing.json); prints the section table + the volume put lines |
| `brief17_reel.py` | WIRED | pod_build.py assemble | brief-17 phase-B reel builder: 13 pieces (6 CLs + 5 overlays + rac 63-69 + sev 36-44) normalized (1920x1080/25fps/an) and concat-encoded pod-side to in/reel.mp4; writes reel_manifest.json (the jobs-6/7 share + audit mapping); REFUSES on any piece/window drift |
| `brief17_stills.py` | WIRED | brief17_check.py wmp | brief-17 phase-B stills: extracts one stamped mid-frame per manifest row + 4 probes per footage section from the final on the pod, into out/<episode>/wmp_frames/ |
| `brief17_cards.py` | WIRED | brief17_check.py title/script | emits the episode-one card specs: title (Yamal + Flick, photo_box from spec), stat cards C1/C3/C7/C9/C10 (rows stamped to press_facts rows / raw recomputes), chapter cards; audit_T = 1.2 + rows*0.85 + 0.65 |
| `brief12_timeline.py` | WIRED | brief 12 job 5 (proof boards); brief 16 assembly | timeline spec emitter (formation / runners / move) from match_data + raw cache, per docs/board_timeline.md |
| `brief12_data.py` | HAND-RUN | hand-invoked (brief 12 job 1) | Barcelona team-id + 4 most recent completed matches driver; every fetch through sofascore_client.py; id verified against the payload events |
| `census.py` | WIRED | GATES B20 | script-driven census: TOOLS.md must match tools/ exactly (brief 06 job 5) |
| `pagecheck_proof.sh` | WIRED | GATES B19 | runs the rendered-page audit on a deliberately mirrored spec (must FAIL) then the real spec (must PASS), locally via the cached Chrome (brief 08 job 3) |
| `pod_frames.py` | HAND-RUN | brief 09 jobs 1-2 (bench study) | Modal ship/extract/pull for benchmark frames: 2s frames + scene-cut frames (threshold printed) + scdet score sweep; reuses volume soccer-build |
| `brief09_check.py` | WIRED | GATES B24 | gate oracle: frames_<L>.csv row counts vs duration/2 within 1 (brief 09 job 8) |
| `f_mount.sh` | HAND-RUN (stub) | nothing (stub errors) | F: retired 2026-09-28 (brief 11 job 5): stick disconnects under sustained write; stub prints "F: retired 2026-09-28, use D:" and exits 1 |
| `scratch_mount.sh` | WIRED | b2_mount.sh; check_scratch.sh docs; GATES.md B47 | scratch-drive (D: at /mnt/d) mount+write-probe check and remount via the root-owned d-remount helper; takes a drive letter, default D (brief 11 job 2) |
| `brief11_elevated_setup.sh` | HAND-RUN | Mayo one-time elevated run | installs d-remount/d-trim/d-clean + /etc/sudoers.d/d-mount (visudo-validated), mounts D:, creates /mnt/d/scratch tree (brief 11 job 2) |
| `disk_guard.sh` | WIRED | produce_v2.py, pod_build.py, pod_frames.py, runpod_download.py, check_scratch.sh hook | C: floor 15G + scratch(/mnt/d)-paths write probe + retired-F check + vhdx growth log + ECC 10MB cap; --selftest 5 controls (brief 10 job 3, brief 11 job 4/5) |
| `b2_mount.sh` | HAND-RUN | brief 10 job 7 (started by hand) | read-only rclone mount of b2: at ~/b2, VFS cache on /mnt/d/scratch/b2-cache capped 2G, refuses without healthy scratch drive (cache moved from F: by brief 11) |
| `brief10_elevated_setup.sh` | HAND-RUN | Mayo, one-time (sudo) | installs f-remount/f-trim/f-clean helpers + narrow sudoers, validates with visudo, remounts and trims |
| `script_gen.py` | WIRED | produce_v2.py:372 | lane script draft |
| `validate_script.py` | WIRED | produce_v2.py:386; tests | grammar + source validation |
| `transformation_gate.py` | WIRED | produce_v2.py:667 | lane D floor |
| `broadcast_filler.py` | WIRED | produce_v2.py:428 | broadcast/filler classification |
| `cut_list_gen.py` | WIRED | produce_v2.py:438 | footage windows (PySceneDetect snap) |
| `ffmpeg_utils.py` | WIRED | produce_v2.py:47 + importers | 200MB guard, cleanup, get_duration |
| `staging.py` | WIRED | produce_v2.py:48 | /mnt/d/scratch staging (fails loudly if unmounted) |
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
2. check_scratch.sh (SessionStart; renamed from check_mnt_f.sh, brief 11) — runs tools/disk_guard.sh --report (write-probes /mnt/d, C: floor, vhdx log, retired-F check). Still matters.
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