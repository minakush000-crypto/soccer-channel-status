# TOOLS.md — one row per file in tools/

Verified against code on 2026-09-08. Caller column is from
`grep -rnE "import <mod>|TOOLS / \"<name>\"|\"<name>.py\"" tools/ --include="*.py"`
limited to real call sites (subprocess `cmd=[PYTHON,...]`, `import`, or curl
download), not docstring/test mentions. "ORPHANED" = no caller anywhere.
Line numbers are where the tool is invoked or imported; re-read before relying.
This rebuild supersedes the 2026-09-05 version (33 tools, tactical_overlay
wiring): the repo now has 43 tools and produce_v2 no longer calls
tactical_overlay.

## Classification (from RECONCILIATION.md 1.1)

- **WIRED** = reachable from produce_v2.py (12 tools).
- **STANDALONE** = CLI entry point or hand-run utility, or wired only into
  another standalone entry point (27 tools).
- **DEAD** = no caller anywhere (4 tools).

## The 43 tools

| File | Class | Called by (file:line) | Last modified | Works? |
|---|---|---|---|---|
| `produce_v2.py` | WIRED (root) | nothing (entry point) | 2026-09-08 | works (liverpool-forest 720x1280 62.3s Sep 7) |
| `match_data.py` | WIRED | `produce_v2.py:69` | 2026-09-01 | works |
| `tactical_boards.py` | WIRED | `produce_v2.py:78` | 2026-09-08 | works |
| `runpod_fulltrack.py` | WIRED | `produce_v2.py:235` | 2026-09-08 | works (146s full-clip, $0.011, STATUS) |
| `tactical_render.py` | WIRED | `produce_v2.py:285` | 2026-09-06 | works (Opus 8/8.5, STATUS) |
| `generate_voice.py` | WIRED | `produce_v2.py:474` | 2026-09-08 | works |
| `generate_ambience.py` | WIRED | `produce_v2.py:493` | 2026-08-22 | works (wired 2026-09-08, STATUS Part 6) |
| `merge_voice.py` | WIRED | `produce_v2.py:514` | 2026-08-25 | works |
| `shorts_crop.py` | WIRED | `produce_v2.py:524` | 2026-08-30 | works (outputs 720x1280) |
| `ffmpeg_utils.py` | WIRED (lib) | imported by `generate_voice.py:24`, `merge_voice.py:25`, `shorts_crop.py:31`; shipped to pod by `runpod_fulltrack.py:72` | 2026-09-08 | works (now holds the 200MB guard) |
| `script_utils.py` | WIRED (lib) | imported by `generate_voice.py:25` | 2026-08-25 | works |
| `cv_annotate.py` | WIRED (pod) | shipped+run on pod by `runpod_fulltrack.py:72,91` | 2026-09-07 | works (per-frame positions export, STATUS) |
| `produce_episode.py` | STANDALONE | entry point; no caller | 2026-08-30 | untested (no verified output) |
| `cloud_produce.py` | STANDALONE | entry point; no caller | 2026-09-08 | untested (pod-side yt-dlp + download_url guarded) |
| `runpod_annotate.py` | STANDALONE | entry point; no caller | 2026-08-23 | untested (GAPS) |
| `runpod_shorts.py` | STANDALONE | entry point; no caller; uploads `luminance_pod.py:254` | 2026-08-29 | untested |
| `vastai_shorts.py` | STANDALONE | entry point; no caller; uploads `luminance_pod.py:334` | 2026-08-30 | untested |
| `runpod_superres.py` | STANDALONE | entry point; no caller | 2026-08-30 | untested |
| `gpu_superres.py` | STANDALONE | entry point; no caller | 2026-08-30 | untested |
| `luminance_pod.py` | STANDALONE (transitive) | `runpod_shorts.py:254`, `vastai_shorts.py:334` (uploaded to pod) | 2026-08-29 | untested |
| `sharpness_check.py` | STANDALONE (transitive) | `cloud_produce.py:984` | 2026-08-30 | untested |
| `assemble_video.py` | STANDALONE (legacy) | `produce_episode.py:303`, `cloud_produce.py:564`; NOT produce_v2 | 2026-08-30 | untested |
| `validate_script.py` | STANDALONE (legacy) | `produce_episode.py:155`; NOT produce_v2 | 2026-08-25 | untested |
| `generate_captions.py` | STANDALONE | no caller (only `tests/`) | 2026-08-25 | untested |
| `thumbnail_generator.py` | STANDALONE | no caller | 2026-08-25 | untested |
| `enhance_clips.py` | STANDALONE | no caller (only `tests/`) | 2026-08-23 | untested |
| `segment_scorer.py` | STANDALONE | no caller; `cv_annotate.py:332` is a comment only | 2026-09-05 | works (hand-run on full-clip data, STATUS) |
| `scoreboard_scan.py` | STANDALONE | no caller | 2026-09-08 | works (3/3 goals, STATUS) |
| `broadcast_filler.py` | STANDALONE | no caller | 2026-09-08 | works (43s/482s broadcast, STATUS) |
| `assemble_words_match.py` | STANDALONE | no caller | 2026-09-08 | works (arsenal-chelsea 45.2s, STATUS) |
| `trim_tracking.py` | STANDALONE | no caller; has `main()` CLI; output in `artifacts/tracking_summary/` | 2026-09-08 | works (hand-run) |
| `viral_angle.py` | STANDALONE | no caller | 2026-08-25 | untested |
| `agent_reach_research.py` | STANDALONE | no caller | 2026-08-30 | untested |
| `fresh_fetch.py` | STANDALONE | no caller | 2026-08-18 | untested |
| `youtube_upload.py` | STANDALONE | no code caller; invoked by hand | 2026-09-07 | works (2 publish-log entries, Sep 6) |
| `oauth_setup.py` | STANDALONE | one-time; no caller | 2026-08-23 | ran once |
| `gemini_inventory_test.py` | STANDALONE | no caller | 2026-09-08 | works (hand-run, STATUS Gemini section) |
| `runpod_stage1.py` | STANDALONE (one-off) | no caller; docstring = one-off | 2026-09-05 | works (one-off, $0.05, STATUS) |
| `ltx_enhance.py` | STANDALONE | no caller | 2026-08-23 | untested |
| `tactical_overlay.py` | DEAD | no caller anywhere (`grep -rn` → only self) | 2026-09-01 | retire (superseded by tactical_render; LLM-guess-coords was the clipart root cause) |
| `pitch_radar.py` | DEAD | shipped to pod by 3 runpod tools but never imported/run | 2026-08-23 | wire or retire (2D radar library, unused) |
| `render_video.py` | DEAD | no caller; docstring says DEPRECATED | 2026-08-25 | retire |
| `check_and_download.py` | DEAD | no caller; RunPod-webhook downloader, superseded by runpod_fulltrack | 2026-08-23 | retire |

Counts: **12 WIRED, 27 STANDALONE, 4 DEAD** = 43.

## Wired set (reachable from produce_v2.py)

8 direct subprocess calls + 2 shared libs + 1 pod-shipped tracker:

```
$ grep -nE "cmd = \[PYTHON|str\(TOOLS / \"" tools/produce_v2.py
69:  match_data.py       78:  tactical_boards.py   235: runpod_fulltrack.py
285: tactical_render.py  474: generate_voice.py    493: generate_ambience.py
514: merge_voice.py      524: shorts_crop.py
```
Transitive libs: `ffmpeg_utils.py` (imported by generate_voice/merge_voice/shorts_crop),
`script_utils.py` (imported by generate_voice). Pod: `cv_annotate.py`
(shipped+run by runpod_fulltrack). `pitch_radar.py` is shipped but never run.