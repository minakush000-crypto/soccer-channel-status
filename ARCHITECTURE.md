# ARCHITECTURE.md — soccer-channel code map

> **Purpose:** code map — which tool calls which, the produce_v2 pipeline.
> **Reader:** every session (CLAUDE.md loads it).
> **Last verified against code:** 2026-09-13.

Verified against code on 2026-09-13 (Stage 14: step4b RETIRED, 3D formation
board wired into step2_boards, .script_verified voice gate, step5_assemble
rewritten with footage <=20% cap, 3rd SessionStart hook). Every line number is
from `wc -l` / `grep -n` against the file on disk today. If a line moved,
re-read. This rebuild supersedes the 2026-09-05 and 2026-09-09 versions; the
2026-09-05 step-4 wiring (`tactical_overlay` at `produce_v2.py:232`) and the
2026-09-09 step4b_tactical_render (at `produce_v2.py:211`) are both gone.

## Entry points (two)

| Entry point | Status | Evidence |
|---|---|---|
| `tools/produce_v2.py` (858 lines) | Authoritative | `grep -n "^def main"` → `715:def main():`; nothing calls it |
| `tools/produce_episode.py` (529 lines) | Older, separate | calls `cv_annotate.py:298`, `assemble_video.py:313`, `validate_script.py:162`, `tactical_boards.py:180`, `generate_ambience.py:334`, `generate_voice.py:349`, `merge_voice.py:346`. NOT reachable from produce_v2 |

Run command (matches `produce_v2.py:10`):
`~/yt-digest/.venv/bin/python tools/produce_v2.py <slug> --query "<match>" --date-range YYYYMMDD-YYYYMMDD`

## produce_v2.py — the steps, in actual execution order

Execution order is from `main()` (line 715), read directly:

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
617:def step5b_transformation_gate(slug, render_dir, lane):
639:def step6_voice(slug, render_dir):
665:def step6b_ambience(slug, render_dir):
684:def step7_merge(slug, render_dir):
708:def step8_shorts(slug, render_dir):
715:def main():
```

`main()` calls (lines 749-822): step1 (749) → step1b_script_gen (756) →
step1c_validate (758) → step2 (761) → step3 (772/774, local or --pod-download)
→ step4a_cutlist (787) → step4b RETIRED (791 comment) → step6_voice (794) →
step6b_ambience (799) → step5_assemble (803) → step5b_transformation_gate
(808) → step7_merge (813) → step8_shorts (818) → staging.cleanup_staging (822).

| # | Function (line) | What it runs | Output |
|---|---|---|---|
| 1 | `step1_match_data` (82) | `match_data.py <slug> --query <q>` (line 84) | `renders/<slug>/match_data.json` |
| 1b | `step1b_script_gen` (321) | `script_gen.py <slug> --lane <lane>` (line 348). DRAFT — human must fact-check. Stage 12B. | `scripts/<slug>.md` |
| 1c | `step1c_validate` (353) | `validate_script.py <slug> --lane <lane>` (line 362). Non-fatal. Stage 12B. | prints errors |
| 2 | `step2_boards` (91) | `tactical_boards.py <slug>` (line 97) for 2D possession + stat_card + 2D formation fallback, then `modal_render3d.py <slug>` (line 110, Modal T4) overwrites formation.mp4 with 3D. Stage 14. | `renders/<slug>/boards/*.png + *.mp4` |
| 3 | `step3_download_clips` (133) | `yt-dlp` inline (line 225), format capped at **720p** (`height<=720`, line 218), duration filter **180-1200s** (3-20min, line 179), `--max-filesize 200M` (line 219), 720p gate `MIN_HEIGHT=720` (line 50, reject at 208), 200MB guard (line 237), cookies at `secrets/yt_cookies.txt` (line 49/193). `--pod-download` uses `step3_download_clips_pod` (256) → `runpod_download.py` (line 288) | `renders/<slug>/clips/clip_<id>.mp4` |
| 4a | `step4a_cutlist` (371) | `broadcast_filler.py` (line 404) + `cut_list_gen.py` (line 414). Stage 12B. | `renders/<slug>/cut_list.json` |
| 4b | RETIRED Stage 14 | `step4b_tactical_render` removed. `tactical_render.py` git-rm'd. `runpod_fulltrack.py` no longer called by produce_v2. `pitch_radar.py` deleted (was never shipped; `runpod_fulltrack.py:101` ships only `cv_annotate.py` + `ffmpeg_utils.py`). | — |
| 5 | `step5_assemble` (430) | inline ffmpeg concat, sub-cut into 40-80 shots. Footage capped at **20% of runtime** (`footage_budget = 0.20 * total_duration`, line 512). Stage 12B rewrite; the old `clip_idx*5` fixed-offset bug is gone. | `renders/<slug>/clips/video_footage.mp4` |
| 5b | `step5b_transformation_gate` (617) | `transformation_gate.py <manifest>` (line 634). Lane D floor. Stage 12B. | pass/fail |
| 6 | `step6_voice` (639) | `generate_voice.py <slug> --tts-only` (line 660). Gated by `.script_verified` marker (line 648): no voice render until a human creates `renders/<slug>/.script_verified`. Stage 14. | `renders/<slug>/voice_elevenlabs.mp3` |
| 6b | `step6b_ambience` (665) | `generate_ambience.py <slug> 30` (line 679). Non-fatal. | `renders/<slug>/crowd_ambience.mp3` |
| 7 | `step7_merge` (684) | `merge_voice.py <slug> <voice.mp3>` (line 700) mixes crowd ambience under voice at 30% | `renders/<slug>/final_video.mp4` |
| 8 | `step8_shorts` (708) | `shorts_crop.py <slug>` (line 710). Non-fatal (16:9 final is primary). | `renders/<slug>/shorts/final_video_shorts.mp4` |

Final step cleans up the /mnt/f-staged raw source via `staging.cleanup_staging`
(line 822) and deletes a `--clip`-provided file if it was on /mnt/f. No copy to
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
- `generate_ambience.py` — IS now called (step6b, line 679). (The 2026-09-05 doc said NOT called; that is reversed.)
- `broadcast_filler.py`, `cut_list_gen.py`, `script_gen.py`, `transformation_gate.py` — IS now wired (step4a/step1b/step5b). Stage 12B.
- `modal_render3d.py` — IS now wired (step2_boards, line 110). Stage 14.
- `youtube_upload.py` — no upload step in produce_v2. Uploads happen by hand (2 entries in `artifacts/publish-log/`).
- `cv_annotate.py` — not called locally; not shipped by produce_v2. `runpod_fulltrack.py:101` ships it to RunPod, but runpod_fulltrack is no longer called by produce_v2 (step4b RETIRED Stage 14).
- `runpod_fulltrack.py` — 0 mentions in produce_v2 (step4b retired Stage 14). Still exists on disk; callable standalone.
- `tactical_render.py` — DELETED (git-rm'd Stage 14). RETIRED.
- `pitch_radar.py` — DELETED. Never shipped by runpod_fulltrack (line 101 ships only cv_annotate.py + ffmpeg_utils.py).

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

## Known render directories (ls renders/)

```
$ ls -d renders/*/
renders/2026-08-18_iraola-liverpool/
renders/2026-08-30_liverpool-forest/
renders/2026-08-30_liverpool-forest_sep1/
renders/2026-09-06_arsenal-chelsea/
renders/2026-09-12_bournemouth-brentford/
renders/2026-09-12_preview-manc-derby/
renders/_20min_test/
renders/_fullmatch_arsenal-chelsea-carabao/
renders/_real_soccer_test/
renders/_sharp_test/
```

Latest produce_v2.py output (liverpool-forest shorts): 720x1280, 62.3s,
24.8MB, Sep 7 (unchanged). The 2026-09-06_arsenal-chelsea dir was originally
built with `assemble_words_match.py` (a standalone, words-match-pictures
path), NOT produce_v2: 1280x720, 45.2s, 15.0MB, Sep 8. (File overwritten
Stage 12B; now 1920x1080, 724.0s, 210.5MB.) `_fullmatch_arsenal-chelsea-carabao`
is an incomplete manual download (only match_data.json; the .part was deleted
2026-09-08, see RECONCILIATION 1.5/1.6).

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

The truthful gap list (which tools violate rules 1 and 3 today) lives in
DECISIONS.md under this same heading. The current pipeline violates rule 1
(produce_v2 runs locally except step3 --pod-download). Rule 3 improved Stage
12B: 5 tools wired (script_gen, validate_script, broadcast_filler,
cut_list_gen, transformation_gate) → 14 WIRED, 22 STANDALONE remaining (was
27). Stage 14: runpod_fulltrack unwired (step4b retired), modal_render3d
wired (step2_boards) — net unchanged, 14 WIRED, 22 STANDALONE. Verified: 13
tools called via `TOOLS /` in produce_v2.py + `staging` import = 14 wired; 47
total tools. The 22 STANDALONE tools are the remaining rule-3 debt.
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
