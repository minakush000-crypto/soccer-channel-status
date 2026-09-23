# ARCHITECTURE.md — soccer-channel code map

> **Purpose:** code map — which tool calls which, the produce_v2 pipeline.
> **Reader:** every session (CLAUDE.md loads it).
> **Last verified against code:** 2026-09-22.

Verified against code on 2026-09-22 (brief 03: 25 tools retired, entry points
are produce_v2.py (910 lines) + pod_build.py (204 lines, the flash pipeline
that built EP001); produce_episode.py and the whole tracking lane are
retired; the dead tactical_path plumbing was removed). Line numbers re-grepped
against the file on disk today; if a line moved, re-read. Older versions of
this doc described the tactical_overlay (2026-09-05) and step4b (2026-09-09)
wiring; both are long gone.

## Entry points (two)

| Entry point | Status | Evidence |
|---|---|---|
| `tools/pod_build.py` (204 lines) | The flash pipeline (built EP001) | Modal app "soccer-flash-build"; `render3d <spec>` → node three_scene_v2.js; `render2d <spec>` → boards_2d.py; `assemble` → script parse + footage cuts + board loops + voice/ambience mix on volume "soccer-build". SLUG hardcoded at pod_build.py:25. |
| `tools/produce_v2.py` (910 lines) | Older end-to-end entry | `grep -n "^def main"`; nothing calls it. `RENDERS` (line 39) is the local working-copy dir; B2 is the archive of record. |
| `tools/produce_episode.py` | RETIRED 2026-09-22 | was the oldest entry point; deleted with the other 24 retired tools (git 3d31f0e). |

Run command (matches `produce_v2.py:10`):
`~/yt-digest/.venv/bin/python tools/produce_v2.py <slug> --query "<match>" --date-range YYYYMMDD-YYYYMMDD`

## produce_v2.py — the steps, in actual execution order

Execution order is from `main()` (line 724), read directly:

```
$ grep -nE "^def step|^def main" tools/produce_v2.py
82:def step1_match_data(slug, query, date_range):
91:def step2_boards(slug):
133:def step3_download_clips(slug, query, render_dir):
256:def step3_download_clips_pod(slug, query, render_dir):
321:def step1b_script_gen(slug, lane):
353:def step1c_validate(slug, lane):
371:def step4a_cutlist(slug, clip_path, render_dir):
430:def step5_assemble(slug, render_dir, boards_dir, clip_path, match_data, lane,
626:def step5b_transformation_gate(slug, render_dir, lane):
648:def step6_voice(slug, render_dir):
674:def step6b_ambience(slug, render_dir):
693:def step7_merge(slug, render_dir):
717:def step8_shorts(slug, render_dir):
724:def main():
```

`main()` calls (lines 758-831): step1 (758) → step1b_script_gen (765) →
step1c_validate (767) → step2 (770) → step3 (781/783, local or --pod-download)
→ step4a_cutlist (796) → step4b RETIRED (800 comment) → step6_voice (803) →
step6b_ambience (808) → step5_assemble (812) → step5b_transformation_gate
(817) → step7_merge (822) → step8_shorts (827) → staging.cleanup_staging (831).

| # | Function (line) | What it runs | Output |
|---|---|---|---|
| 1 | `step1_match_data` (82) | `match_data.py <slug> --query <q>` (line 84) | `renders/<slug>/match_data.json` |
| 1b | `step1b_script_gen` (321) | `script_gen.py <slug> --lane <lane>` (line 348). DRAFT — human must fact-check. Stage 12B. | `scripts/<slug>.md` |
| 1c | `step1c_validate` (353) | `validate_script.py <slug> --lane <lane>` (line 362). Non-fatal. Stage 12B. | prints errors |
| 2 | `step2_boards` (91) | `tactical_boards.py <slug>` (line 97) for 2D possession + stat_card + 2D formation fallback, then `three_render3d.py <slug>` (line 110, Modal T4) overwrites formation.mp4 with 3D. Stage 14. | `renders/<slug>/boards/*.png + *.mp4` |
| 3 | `step3_download_clips` (133) | `yt-dlp` inline (line 225), format capped at **720p** (`height<=720`, line 218), duration filter **180-1200s** (3-20min, line 179), `--max-filesize 200M` (line 219), 720p gate `MIN_HEIGHT=720` (line 50, reject at 208), 200MB guard (line 237), cookies at `secrets/yt_cookies.txt` (line 49/193). `--pod-download` uses `step3_download_clips_pod` (256) → `runpod_download.py` (line 288) | `renders/<slug>/clips/clip_<id>.mp4` |
| 4a | `step4a_cutlist` (371) | `broadcast_filler.py` (line 404) + `cut_list_gen.py` (line 414). Stage 12B. | `renders/<slug>/cut_list.json` |
| 4b | RETIRED Stage 14 | `step4b_tactical_render` removed. `tactical_render.py` git-rm'd. `runpod_fulltrack.py` no longer called by produce_v2. `pitch_radar.py` deleted (was never shipped; `runpod_fulltrack.py:101` ships only `cv_annotate.py` + `ffmpeg_utils.py`). | — |
| 5 | `step5_assemble` (430) | inline ffmpeg concat, sub-cut into 40-80 shots. Footage capped at **20% of runtime** (`footage_budget = 0.20 * total_duration`, line 512). Stage 12B rewrite; the old `clip_idx*5` fixed-offset bug is gone. | `renders/<slug>/clips/video_footage.mp4` |
| 5b | `step5b_transformation_gate` (626) | `transformation_gate.py <manifest>` (line 643). Lane D floor. Stage 12B. | pass/fail |
| 6 | `step6_voice` (648) | `generate_voice.py <slug> --tts-only` (line 669). Gated by `.script_verified` marker (line 657): no voice render until a human creates `renders/<slug>/.script_verified`. Stage 14. | `renders/<slug>/voice_elevenlabs.mp3` |
| 6b | `step6b_ambience` (674) | `generate_ambience.py <slug> 30` (line 688). Non-fatal. | `renders/<slug>/crowd_ambience.mp3` |
| 7 | `step7_merge` (693) | `merge_voice.py <slug> <voice.mp3>` (line 709) mixes crowd ambience under voice at 30% | `renders/<slug>/final_video.mp4` |
| 8 | `step8_shorts` (717) | `shorts_crop.py <slug>` (line 719). Non-fatal (16:9 final is primary). | `renders/<slug>/shorts/final_video_shorts.mp4` |

Final step cleans up the /mnt/f-staged raw source via `staging.cleanup_staging`
(line 831) and deletes a `--clip`-provided file if it was on /mnt/f. No copy to
/mnt/c/Downloads (removed Stage 12B).

## What produce_v2.py does NOT call (verified by grep)

```
$ for t in tactical_overlay check_and_download assemble_video youtube_upload runpod_fulltrack; do c=$(grep -c "$t" tools/produce_v2.py); echo "$t: $c mention(s)"; done
tactical_overlay: 0 mention(s)
check_and_download: 0 mention(s)
assemble_video: 0 mention(s)
youtube_upload: 0 mention(s)
runpod_fulltrack: 0 mention(s)
```
- `tactical_overlay.py` — NOT called (the old 2026-09-05 doc said line 232; that wiring is gone). DELETED.
- `validate_script.py` — IS now wired (step1c, line 362). Stage 12B reversed the old "0 mentions."
- `generate_ambience.py` — IS now called (step6b, line 688). (The 2026-09-05 doc said NOT called; that is reversed.)
- `broadcast_filler.py`, `cut_list_gen.py`, `script_gen.py`, `transformation_gate.py` — IS now wired (step4a/step1b/step5b). Stage 12B.
- `three_render3d.py` — IS now wired (step2_boards, line 110). Stage 14.
- `youtube_upload.py` — no upload step in produce_v2. Uploads happen by hand (2 entries in `artifacts/publish-log/`).
- `cv_annotate.py`, `runpod_fulltrack.py`, `segment_scorer.py` — RETIRED 2026-09-22 (tracking lane; nothing live reads tracking JSONs). Git + B2 hold them.
- `tactical_render.py` — DELETED (git-rm'd Stage 14). `pitch_radar.py` — DELETED earlier.
- `produce_episode.py`, `cloud_produce.py`, `assemble_video.py`, `assemble_words_match.py` — RETIRED 2026-09-22 (old entries/pipelines).
- `youtube_upload.py` — no upload step in produce_v2. Uploads happen by hand (2 entries in `publish-log/`).

## 200MB local-video guard (new, 2026-09-08)

`ffmpeg_utils.py` defines `LOCAL_VIDEO_LIMIT_MB = 200` and
`assert_video_under_limit(path)` (deletes + raises if a local video write
exceeds 200MB) and `cleanup_part_files(dir)`. Wired into every local
download path:

```
$ grep -n "assert_video_under_limit\|max-filesize\|cleanup_part_files" tools/produce_v2.py tools/runpod_fulltrack.py tools/cloud_produce.py
tools/produce_v2.py:219:  "--max-filesize", "200M",
tools/produce_v2.py:229:  cleanup_part_files(clips_dir)
tools/produce_v2.py:237:  assert_video_under_limit(dl_path)
tools/produce_v2.py:308:  assert_video_under_limit(clip_path)
tools/runpod_fulltrack.py:198:  assert_video_under_limit(local)
tools/cloud_produce.py:110:  assert_video_under_limit(local_path)
```

## External dependencies (outside this project)

| Dependency | Path | Evidence |
|---|---|---|
| Python venv | `~/yt-digest/.venv` | used by every run command |
| Env file | `~/yt-digest/.env` | `YOUTUBE_API_KEY`, `LLM_*`, `ELEVENLABS_API_KEY`, RunPod/Vast keys |
| YOLOv8 weights | `~/yolov8s.pt` | outside the project folder (CONTEXT.md) |
| yt-dlp cookies | `secrets/yt_cookies.txt` | produce_v2 line 49 (COOKIE_PATH), used at 193 |
| YouTube OAuth | `secrets/client_secret.json`, `secrets/youtube_token.json` | youtube_upload.py |
| RunPod / Vast.ai keys | in `~/yt-digest/.env` | not opened (rule 4) |

## Output layout

```
renders/<slug>/
  match_data.json            # step 1
  .script_verified           # human-created marker (step 6 gate, Stage 14)
  boards/                    # step 2 (PNG + MP4 per board type; formation.mp4 = 3D)
  clips/
    clip_<id>.mp4            # step 3 source (guarded ≤200MB)
    video_footage.mp4        # step 5 assembled
  cut_list.json              # step 4a (broadcast_filler + cut_list_gen)
  broadcast_filler.json      # step 4a
  voice_elevenlabs.mp3       # step 6
  crowd_ambience.mp3         # step 6b
  concat_list.txt            # step 5 ffmpeg concat
  tmp_segments/              # step 5 scratch
  final_video.mp4            # step 7 merged (16:9, primary output)
  shorts/final_video_shorts.mp4  # step 8 (vertical, non-fatal)
```

## B2 archive of record (Stage 14 / B2 migration, 2026-09-13)

`RENDERS = SCRIPT_DIR / "renders"` (`produce_v2.py:38`) is the **local
working-copy** directory. The **archive of record** is B2
(`b2:mendymax-archive`, bucket verified `rclone lsd b2:`). Every output the
pipeline produces — boards, renders, frames, transcripts, artifacts, backups
— has a B2 destination under `b2:mendymax-archive/yt-digest/...`. Local copies
are working copies, not the record.

This extends doctrine rule 1 (raw footage -> /mnt/f -> pod -> delete locally)
to generated outputs: the local `renders/` tree is ephemeral, B2 is durable.

B2 path mapping (the local path maps to the same relative path under the B2
`yt-digest/` prefix):

| Local (working copy) | B2 (archive of record) |
|---|---|
| `renders/<slug>/` | `b2:mendymax-archive/yt-digest/renders/<slug>/` |
| `renders/<slug>/boards/*.png + *.mp4` | `b2:mendymax-archive/yt-digest/renders/<slug>/boards/` |
| `renders/<slug>/clips/*.mp4` | `b2:mendymax-archive/yt-digest/renders/<slug>/clips/` |
| `renders/<slug>/voice_elevenlabs.mp3` | `b2:mendymax-archive/yt-digest/renders/<slug>/` |
| `renders/<slug>/final_video.mp4` | `b2:mendymax-archive/yt-digest/renders/<slug>/` |
| `renders/<slug>/shorts/final_video_shorts.mp4` | `b2:mendymax-archive/yt-digest/renders/<slug>/shorts/` |
| `scripts/<slug>.md` | `b2:mendymax-archive/yt-digest/scripts/` |
| `artifacts/` (frames, transcripts, publish logs) | `b2:mendymax-archive/yt-digest/artifacts/` |

B2 is not yet wired into produce_v2.py as an automatic upload step (no
`step9_archive` exists today). Archiving is a manual `rclone copy` until it is
wired. CLOSING PASS: confirm which tool, if any, auto-uploads to B2 after this
migration, and whether a `step9_archive` function has been added.

## Known render directories (ls renders/)

```
$ ls -d renders/*/
renders/2026-09-08_real-madrid-inter/
renders/2026-09-08_real-madrid-inter2/
renders/2026-09-12_bournemouth-brentford/
renders/2026-09-12_bournemouth-brentford-gemini/
```

(The ten pre-flash render dirs were archived to B2 and deleted locally,
2026-09-13.) Latest episode output: renders/2026-09-08_real-madrid-inter/
final_video.mp4 — 140.92s, 76.5MB, 1920x1080 (EP001 v3, brief 03 board fixes;
ffprobe verified 2026-09-22). `_fullmatch_arsenal-chelsea-carabao` and the
other scratch dirs are gone.

## STANDING OPERATING DOCTRINE
1. Raw footage is acquired over the residential connection, staged on /mnt/f,
   shipped to the pod, and deleted locally. Nothing raw is written inside
   /home/muads. No local GPU work. All heavy compute and all archival storage
   go to the cloud. The purpose of this rule is that the ext4.vhdx never grows
   and no GPU work runs on the N150; staging on /mnt/f satisfies both.
   (Amended 2026-09-12, Stage 10A.3: residential YouTube download works —
   proven on 6 reference videos Stage 9A; the obstacle was the old rule-1
   wording, not YouTube.)
2. No tasking without calling the available tools, skills, connectors, web
   fetch, research or MCPs where they apply.
3. No orphaned, standalone or untested item, tool or aspect of the pipeline
   may exist. Everything is wired, tested, or retired.
4. Every brief carries this doctrine as a footer.
5. NO DEAD ENDS. When a tool, path, provider or piece of infrastructure blocks
   the work, do not stop and report it blocked. Find the next best available
   option and take it. Report the block, the alternatives considered, and which
   you chose. Stopping at the first wall is only acceptable when every
   alternative has been named and priced.

Rule 3 status after brief 03 (2026-09-22): the census is 26 WIRED + 13
HAND-RUN (each a deliberate operating tool: judge, research, publishing,
archive, hook checks) + 1 data file; 25 files retired. No DEAD class exists.
The full census lives in TOOLS.md. Rule 1 (nothing heavy on the local N150)
is satisfied for the flash pipeline (pod_build runs on Modal); produce_v2's
local download path remains the exception (--pod-download is the compliant
path).
## Skill auto-activation (Stage 12C)

A skill-activation layer sits in the project's `.claude/` (installed from
diet103/claude-code-infrastructure-showcase, regex-only mode). It is NOT part of
the video pipeline (produce_v2.py); it is Claude-Code-session infrastructure
that suggests skills when a prompt matches a keyword/intentPattern in
`.claude/skills/skill-rules.json`.

- **UserPromptSubmit** → `skill-activation-prompt.sh`: regex-matches the prompt
  against skill-rules.json, suggests matching skills. (mode=disabled = regex
  only, no AI provider.)
- **PreToolUse (Edit|MultiEdit|Write)** → `skill-verification-guard.sh`: blocks
  an edit if a mandatory (block-enforcement) skill is pending. No-op here (no
  block-enforcement skills installed).
- **PostToolUse (Edit|MultiEdit|Write)** → `post-tool-use-tracker.sh`: logs
  edited files (bash+jq).
- **PostToolUse (Skill)** → `skill-activation-tracker.sh`: records skill
  activations (session intel, better-sqlite3).
- **SessionStart** (pre-existing): check_pods.sh + check_mnt_f.sh +
  check_doc_stamps.sh (3 project SessionStart hooks; check_doc_stamps.sh added
  Stage 14).
- **Stop** (global, pre-existing): unlazy stop-hook.mjs (unchanged).
- **PreToolUse** (global, pre-existing): block-retired, block-image-read,
  warn-local-gpu (unchanged).

settings.json is merged additively (showcase pattern: extract + merge, never
overwrite). The 7 pre-existing hooks (3 project SessionStart + 1 global Stop +
3 global PreToolUse) are preserved; the 4 new hooks are added.

Only `skill-developer` is in skill-rules.json. The showcase's React/Express/Sentry
skills were not installed (wrong stack). The ~130 user marketing skills
(~/.claude/skills) are not in skill-rules.json → not auto-suggested, but
manually invokable. dev-docs commands not adopted (project has its own doc
spine). Verified: verify-setup.sh 8/8 PASS + a real trigger test (skill-developer
suggested via regex). See TOOLS.md and LANE_PLAN.md §Stage 12C.
