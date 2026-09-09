# ARCHITECTURE.md — soccer-channel code map

Verified against code on 2026-09-09 (Stage 2: 720p cap + 180-1200s filter on produce_v2.py). Every line number is from `wc -l` /
`grep -n` against the file on disk today. If a line moved, re-read. This
rebuild supersedes the 2026-09-05 version, whose step-4 wiring (`tactical_overlay`
at `produce_v2.py:232`) no longer exists in the code.

## Entry points (two)

| Entry point | Status | Evidence |
|---|---|---|
| `tools/produce_v2.py` (625 lines) | Authoritative | `grep -n "^def main"` → `529:def main():`; nothing calls it |
| `tools/produce_episode.py` (519 lines) | Older, separate | calls `cv_annotate.py:288`, `assemble_video.py:303`, `validate_script.py:155`, `tactical_boards.py:173`, `generate_ambience.py:324`, `generate_voice.py:339`, `merge_voice.py:336`. NOT reachable from produce_v2 |

Run command (matches `produce_v2.py:10`):
`~/yt-digest/.venv/bin/python tools/produce_v2.py <slug> --query "<match>" --date-range YYYYMMDD-YYYYMMDD`

## produce_v2.py — the 9 steps, in actual execution order

Execution order is from `main()` (line 529), read directly:

```
$ grep -nE "^def step|^def main" tools/produce_v2.py
67:def step1_match_data(slug, query, date_range):
76:def step2_boards(slug):
95:def step3_download_clips(slug, query, render_dir):
211:def step4b_tactical_render(slug, clip_path, render_dir, match_data):
301:def step5_assemble(slug, render_dir, boards_dir, clip_path, match_data,
467:def step6_voice(slug, render_dir):
479:def step6b_ambience(slug, render_dir):
498:def step7_merge(slug, render_dir):
522:def step8_shorts(slug, render_dir):
529:def main():
```

`main()` calls (lines 549-588): step1 (549) → step2 (553) → step3 (557) →
step4b (567) → step6_voice (570) → step6b_ambience (575) → step5_assemble
(579) → step7_merge (584) → step8_shorts (588).

| # | Function (line) | What it runs | Output |
|---|---|---|---|
| 1 | `step1_match_data` (67) | `match_data.py <slug> --query <q>` (line 69) | `renders/<slug>/match_data.json` |
| 2 | `step2_boards` (76) | `tactical_boards.py <slug>` (line 78) | `renders/<slug>/boards/*.png + *.mp4` |
| 3 | `step3_download_clips` (95) | `yt-dlp` inline (line 175), format capped at **720p** (`height<=720`, line 176), duration filter **180-1200s** (3-20min, line 141), `--max-filesize 200M` (line 177), 720p gate (line 193), 200MB guard (line 195), cookies at `secrets/yt_cookies.txt` (line 155) | `renders/<slug>/clips/clip_<id>.mp4` |
| 4 | `step4b_tactical_render` (211) | `runpod_fulltrack.py --clip <clip>` (line 235, ships `cv_annotate.py`+`pitch_radar.py`+`ffmpeg_utils.py` to RunPod, runs cv_annotate on the pod) then `tactical_render.py <tracking.json>` (line 285) | `renders/<slug>/clips/<prefix>_full.tracking.json` + `tactical_view.mp4` |
| 5 | `step6_voice` (467) | `generate_voice.py <slug> --tts-only` (line 474) | `renders/<slug>/voice_elevenlabs.mp3` |
| 6 | `step6b_ambience` (479) | `generate_ambience.py <slug> 30` (line 493). Non-fatal. | `renders/<slug>/crowd_ambience.mp3` |
| 7 | `step5_assemble` (301) | inline ffmpeg concat, board/tactical/footage segments. Footage cuts at `clip_idx*5` (line 433) — the fixed-offset bug, see RECONCILIATION 1.2 | `renders/<slug>/clips/video_footage.mp4` |
| 8 | `step7_merge` (498) | `merge_voice.py <slug> <voice.mp3>` (line 514) mixes crowd ambience under voice at 30% | `renders/<slug>/final_video.mp4` |
| 9 | `step8_shorts` (522) | `shorts_crop.py <slug>` (line 524) | `renders/<slug>/shorts/final_video_shorts.mp4` |

Final step copies to `/mnt/c/Users/muads/Downloads/<slug>_Short.mp4` (main, line 618).

## What produce_v2.py does NOT call (verified by grep)

```
$ for t in tactical_overlay validate_script check_and_download assemble_video youtube_upload; do c=$(grep -c "$t" tools/produce_v2.py); echo "$t: $c mention(s)"; done
tactical_overlay: 0 mention(s)
validate_script: 0 mention(s)
check_and_download: 0 mention(s)
assemble_video: 0 mention(s)
youtube_upload: 0 mention(s)
```
- `tactical_overlay.py` — NOT called (the old 2026-09-05 doc said line 232; that wiring is gone). DEAD.
- `validate_script.py` — no script-vs-matchdata validation in produce_v2.
- `generate_ambience.py` — IS now called (step6b, line 493). (The 2026-09-05 doc said NOT called; that is reversed.)
- `youtube_upload.py` — no upload step in produce_v2. Uploads happen by hand (2 entries in `artifacts/publish-log/`).
- `cv_annotate.py` — 3 mentions, all comments; not called locally. It runs on the RunPod pod, shipped by `runpod_fulltrack.py:72`.

## 200MB local-video guard (new, 2026-09-08)

`ffmpeg_utils.py` defines `LOCAL_VIDEO_LIMIT_MB = 200` and
`assert_video_under_limit(path)` (deletes + raises if a local video write
exceeds 200MB) and `cleanup_part_files(dir)`. Wired into every local
download path:

```
$ grep -n "assert_video_under_limit\|max-filesize\|cleanup_part_files" tools/produce_v2.py tools/runpod_fulltrack.py tools/cloud_produce.py
tools/produce_v2.py:177:  "--max-filesize", "200M",
tools/produce_v2.py:187:  cleanup_part_files(clips_dir)
tools/produce_v2.py:195:  assert_video_under_limit(clip_path)
tools/runpod_fulltrack.py:174:  assert_video_under_limit(local)
tools/cloud_produce.py:110:  assert_video_under_limit(local_path)
```

## External dependencies (outside this project)

| Dependency | Path | Evidence |
|---|---|---|
| Python venv | `~/yt-digest/.venv` | used by every run command |
| Env file | `~/yt-digest/.env` | `YOUTUBE_API_KEY`, `LLM_*`, `ELEVENLABS_API_KEY`, RunPod/Vast keys |
| YOLOv8 weights | `~/yolov8s.pt` | outside the project folder (CONTEXT.md) |
| yt-dlp cookies | `secrets/yt_cookies.txt` | produce_v2 line 150 |
| YouTube OAuth | `secrets/client_secret.json`, `secrets/youtube_token.json` | youtube_upload.py |
| RunPod / Vast.ai keys | in `~/yt-digest/.env` | not opened (rule 4) |

## Output layout

```
renders/<slug>/
  match_data.json            # step 1
  boards/                    # step 2 (PNG + MP4 per board type)
  clips/
    clip_<id>.mp4            # step 3 source (guarded ≤200MB)
    <prefix>_full.tracking.json  # step 4b (8MB, per-frame positions)
    tactical_view.mp4        # step 4b
    video_footage.mp4        # step 7 assembled
  voice_elevenlabs.mp3       # step 5
  crowd_ambience.mp3         # step 6
  concat_list.txt            # step 7 ffmpeg concat
  tmp_segments/              # step 7 scratch
  final_video.mp4            # step 8 merged
  shorts/final_video_shorts.mp4  # step 9 final
```

## Known render directories (ls renders/)

```
$ ls -d renders/*/
renders/2026-08-18_iraola-liverpool/
renders/2026-08-30_liverpool-forest/
renders/2026-08-30_liverpool-forest_sep1/
renders/2026-09-06_arsenal-chelsea/
renders/_fullmatch_arsenal-chelsea-carabao/
renders/_real_soccer_test/
renders/_sharp_test/
```

Latest produce_v2.py output (liverpool-forest): 720x1280, 62.3s, 24.8MB, Sep 7.
The 2026-09-06_arsenal-chelsea dir was built with `assemble_words_match.py`
(a standalone, words-match-pictures path), NOT produce_v2: 1280x720, 45.2s,
15.0MB, Sep 8. `_fullmatch_arsenal-chelsea-carabao` is an incomplete manual
download (only match_data.json; the .part was deleted 2026-09-08, see
RECONCILIATION 1.5/1.6).

## STANDING OPERATING DOCTRINE
1. NOTHING RUNS ON THE LOCAL MACHINE. No local downloads, except raw source
   staging on the USB flash drive at /mnt/f (staging area ONLY: download lands
   there, ships to the pod, gets deleted; never a working directory). No local GPU work.
   All compute and all storage go to the cloud GPU providers.
2. No tasking without calling the available tools, skills, connectors, web
   fetch, research or MCPs where they apply.
3. No orphaned, standalone or untested item, tool or aspect of the pipeline
   may exist. Everything is wired, tested, or retired.
4. Every brief carries this doctrine as a footer.

The truthful gap list (which tools violate rules 1 and 3 today) lives in
DECISIONS.md under this same heading. The current pipeline violates rule 1
(produce_v2 runs locally except step4b); rule 3 (4 DEAD + ~18 untested
STANDALONE). Not yet fixed.