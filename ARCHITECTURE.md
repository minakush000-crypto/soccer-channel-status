# ARCHITECTURE.md — soccer-channel code map

Built 2026-09-05 by reading the code, not intent. Every line number is from
`wc -l` / `Read` against the file on disk today. If a line moved, re-read.

## Entry points (two)

| Entry point | Status | Evidence |
|---|---|---|
| `tools/produce_v2.py` (518 lines) | Authoritative | `grep` shows nothing calls it; `main()` at line 428 |
| `tools/produce_episode.py` (519 lines) | Older, still wired | calls `cv_annotate.py` at line 288; do not break that |

Run command (from CONTEXT.md, matches produce_v2.py:10):
`~/yt-digest/.venv/bin/python tools/produce_v2.py <slug> --query "<match>" --date-range YYYYMMDD-YYYYMMDD`

## produce_v2.py — the 8 steps, in actual execution order

Execution order is NOT the function-number order. `main()` (line 428) calls
voice before assembly so assembly can use the real voice duration
(comment at line 463). Verified by reading `main()` lines 447-482.

| # | Function (line) | What it runs | Output |
|---|---|---|---|
| 1 | `step1_match_data` (62) | `match_data.py <slug> --query <q>` (line 64) | `renders/<slug>/match_data.json` |
| 2 | `step2_boards` (71) | `tactical_boards.py <slug>` (line 73) | `renders/<slug>/boards/*.mp4` |
| 3 | `step3_download_clips` (90) | `yt-dlp` inline (line 170), ffprobe gate at line 184, rejects <720p (line 186), cookies at `secrets/yt_cookies.txt` (line 150) | `renders/<slug>/clips/clip_<id>.mp4` |
| 4 | `step4_overlay` (202) | `tactical_overlay.py <clip> --description <d> --output <o> --duration 30` (line 232) | `renders/<slug>/clips/<stem>_overlay.mp4` |
| 5 | `step6_voice` (390) | `generate_voice.py <slug> --tts-only` (line 397) | `renders/<slug>/voice_elevenlabs.mp3` |
| 6 | `step5_assemble` (242) | inline ffmpeg concat (lines 326-373), footage cuts at `clip_idx*5` (line 356) | `renders/<slug>/clips/video_footage.mp4` |
| 7 | `step7_merge` (402) | `merge_voice.py <slug> <voice.mp3>` (line 416) | `renders/<slug>/final_video.mp4` |
| 8 | `step8_shorts` (421) | `shorts_crop.py <slug>` (line 423) | `renders/<slug>/shorts/final_video_shorts.mp4` |

Final step also copies to `/mnt/c/Users/muads/Downloads/<slug>_Short.mp4` (line 510).

## What produce_v2.py does NOT call

These are wired into `produce_episode.py` but NOT into `produce_v2.py`
(verified by grep of produce_v2.py body):
- `cv_annotate.py` — the real CV tracker. produce_v2.py never runs it.
- `validate_script.py` — no script-vs-matchdata validation in produce_v2.py.
- `generate_ambience.py` — no crowd ambience in produce_v2.py.
- `youtube_upload.py` — no upload step anywhere.

## External dependencies (outside this project)

| Dependency | Path | Evidence |
|---|---|---|
| Python venv | `~/yt-digest/.venv` | `ls -d` confirmed |
| Env file | `~/yt-digest/.env` | `ls -la` confirmed (895 bytes, Aug 30) |
| YOLOv8 weights | `~/yolov8s.pt` AND `tools/yolov8s.pt` (22MB) | both exist; `ls -la` confirmed |
| yt-dlp cookies | `secrets/yt_cookies.txt` | exists, 1602 bytes, Sep 5 |
| YouTube OAuth | `secrets/client_secret.json`, `secrets/youtube_token.json` | both exist |
| RunPod / Vast.ai keys | in `~/yt-digest/.env` | not opened (rule 4) |

## Output layout

```
renders/<slug>/
  match_data.json          # step 1
  boards/                  # step 2 (PNG + MP4 per board type)
  clips/
    clip_<id>.mp4          # step 3 source
    <stem>_overlay.mp4     # step 4
    video_footage.mp4      # step 6 assembled
  voice_elevenlabs.mp3     # step 5
  concat_list.txt          # step 6 ffmpeg concat
  tmp_segments/            # step 6 scratch
  final_video.mp4          # step 7 merged
  shorts/
    final_video_shorts.mp4 # step 8 final
```

## Known render directories (ls renders/)

| Dir | final_video_shorts.mp4 | ffprobe (run today) |
|---|---|---|
| `2026-08-30_liverpool-forest` | 27.27MB, Sep 5 20:08 | 720x1280, 64.2s |
| `2026-08-30_liverpool-forest_sep1` | 23.29MB, Sep 1 23:10 | 720x1280, 59.4s |
| `2026-08-18_iraola-liverpool` | 135.1MB, Aug 29 22:41 | 1080x1920, 493.1s |
| `_real_soccer_test` | 91.9MB, Aug 30 01:58 | 720x1280, 128.7s |
| `_sharp_test` | 6.13MB, Aug 30 01:19 | 1080x1920, 12.0s |

The latest produce_v2.py output (Sep 5) is 720x1280, 64.2s, 27.27MB.
CONTEXT.md's "confirmed working" numbers match this run.