# TOOLS.md — one row per file in tools/

Built 2026-09-05. Caller column is from `grep -rn "<toolname>" tools/ scripts/`,
limited to real call sites (subprocess.run, import, or inline shell), not
docstring mentions. "ORPHANED" means no caller found anywhere in the repo.
Line numbers are where the tool is invoked or imported; re-read before relying.

| File | What it does | Called by (file:line) | Last modified | Last evidence of execution | Works? |
|---|---|---|---|---|---|
| `produce_v2.py` | End-to-end pipeline, 8 steps (authoritative) | ORPHANED (entry point) | 2026-09-05 | output `renders/2026-08-30_liverpool-forest/shorts/final_video_shorts.mp4` Sep 5 20:08, ffprobe 720x1280 64.2s | works |
| `produce_episode.py` | Older end-to-end pipeline | ORPHANED (entry point) | 2026-08-30 | not checked | untested |
| `match_data.py` | Fetch match data from ESPN API | `produce_v2.py:64`, `cloud_produce.py` (referenced) | 2026-09-01 | `match_data.json` 9176B Sep 5 20:02 | works |
| `tactical_boards.py` | Generate data-driven boards with mplsoccer | `produce_v2.py:73`, `produce_episode.py:173` | 2026-09-01 | `boards/` dir Sep 5 19:17 | works |
| `tactical_overlay.py` | Draw arrows/zones/circles on footage; LLM guesses coords (line ~120 `generate_overlay_spec`) | `produce_v2.py:232` | 2026-09-01 | overlay output Sep 5 (clips dir) | runs, output is the "PowerPoint clipart" problem (broken quality) |
| `generate_voice.py` | ElevenLabs TTS voiceover | `produce_v2.py:397`, `produce_episode.py:339` | 2026-08-30 | `voice_elevenlabs.mp3` 514KB Sep 5 19:18 | works |
| `merge_voice.py` | Multi-layer audio mix: video + voice + crowd | `produce_v2.py:416` | 2026-08-25 | `final_video.mp4` 47.2MB Sep 5 20:07 | works |
| `shorts_crop.py` | Crop to 9:16 Shorts | `produce_v2.py:423`, `cloud_produce.py:592` | 2026-08-30 | `final_video_shorts.mp4` Sep 5 20:08 | works |
| `cv_annotate.py` | YOLOv8 + ByteTrack + KMeans tracking overlays | `produce_episode.py:288` ONLY. NOT called by produce_v2.py | 2026-08-25 | produce_episode run: not checked today | works (per CONTEXT.md vision-model verdict), but isolated from the v2 pipeline |
| `pitch_radar.py` | 2D pitch radar overlay (library) | `cv_annotate.py:27` (import), shipped by `runpod_annotate.py:229` | 2026-08-23 | not checked | untested |
| `ffmpeg_utils.py` | Shared ffmpeg selection + duration helper (library) | imported by `assemble_video.py:9`, `render_video.py:17`, `merge_voice.py:25`, `generate_captions.py:21`, `thumbnail_generator.py:15`, `generate_voice.py:24` | 2026-08-25 | runs whenever callers run | works |
| `script_utils.py` | Shared script-parsing utilities (library) | imported by `generate_captions.py:22`, `generate_voice.py:25`; listed in `cloud_produce.py:58` | 2026-08-25 | runs with generate_voice | works |
| `assemble_video.py` | Commentary-driven timeline assembly | ORPHANED. produce_v2.py does its own inline ffmpeg concat (lines 326-373) instead | 2026-08-30 | not checked | untested |
| `render_video.py` | Combine boards + voice into 1080p MP4 | ORPHANED. Its own docstring (line 48) says "DEPRECATED: use assemble_video.py" | 2026-08-25 | not checked | deprecated |
| `cloud_produce.py` | Run entire pipeline on a cloud GPU pod | ORPHANED (entry point) | 2026-09-01 | not checked | untested |
| `runpod_annotate.py` | Run cv_annotate on RunPod GPU (multi-clip) | ORPHANED (entry point) | 2026-08-23 | not checked end-to-end (see GAPS.md) | untested |
| `runpod_stage1.py` | One-off RunPod runner for single 10s clip (Stage 1 test) | ORPHANED (one-off) | 2026-09-05 | ran Sep 5: 300 frames, 23s, $0.05, tracking JSON downloaded | works (webhook parser has a bug, see DECISIONS.md) |
| `runpod_shorts.py` | Encode Shorts on RunPod with NVENC | ORPHANED (entry point) | 2026-08-29 | not checked | untested |
| `runpod_superres.py` | Real-ESRGAN super-res on RunPod | ORPHANED (entry point) | 2026-08-30 | not checked | untested |
| `vastai_shorts.py` | Encode Shorts on Vast.ai with NVENC | ORPHANED (entry point) | 2026-08-30 | not checked | untested |
| `gpu_superres.py` | Real-ESRGAN super-res on Vast.ai | ORPHANED (entry point) | 2026-08-30 | not checked | untested |
| `luminance_pod.py` | Luminance analysis for smart crop (runs on pod) | `runpod_shorts.py:254`, `vastai_shorts.py:334` (uploaded to pod) | 2026-08-29 | not checked | untested |
| `validate_script.py` | Verify [SRC]/[RUMOR] tags resolve to sources.json | `produce_episode.py:155` ONLY. NOT in produce_v2.py | 2026-08-25 | not checked | untested |
| `generate_ambience.py` | Crowd ambience via ElevenLabs Sound API | `produce_episode.py:324`, `cloud_produce.py:554`. NOT in produce_v2.py | 2026-08-22 | `crowd_ambience.mp3` Aug 23 (iraola run) | works |
| `generate_captions.py` | SRT captions from script + voice | ORPHANED | 2026-08-25 | `captions.srt` Aug 23 (iraola run, maybe manual) | untested |
| `youtube_upload.py` | Upload final video to YouTube Data API | ORPHANED. `publish-log/` is empty (ls confirmed) | 2026-08-23 | never (publish-log empty) | untested |
| `oauth_setup.py` | One-time YouTube OAuth | ORPHANED (one-time setup) | 2026-08-23 | `youtube_token.json` exists Aug 23 | ran once |
| `thumbnail_generator.py` | Auto YouTube thumbnail from final video | ORPHANED | 2026-08-25 | not checked | untested |
| `viral_angle.py` | Find trending soccer topics | ORPHANED | 2026-08-25 | not checked | untested |
| `enhance_clips.py` | FFmpeg upscale/enhance clips | ORPHANED | 2026-08-23 | not checked | untested |
| `ltx_enhance.py` | Animate static board PNGs via LTX Studio | ORPHANED | 2026-08-23 | not checked | untested |
| `fresh_fetch.py` | Dated sports news from RSS | ORPHANED. `USAGE.md` documents it | 2026-08-18 | not checked | untested |
| `check_and_download.py` | Download annotated clips from RunPod webhook | ORPHANED | 2026-08-23 | not checked | untested |
| `sharpness_check.py` | Laplacian blur metric on a frame | `cloud_produce.py:984` | 2026-08-30 | not checked today | untested |
| `agent_reach_research.py` | Multi-platform research layer | ORPHANED | 2026-08-30 | not checked | untested |

## Orphan count

Of 33 Python tools: 7 are called by produce_v2.py (match_data, tactical_boards,
tactical_overlay, generate_voice, merge_voice, shorts_crop + yt-dlp inline).
2 libraries (ffmpeg_utils, script_utils) are transitively used. 1 (cv_annotate)
is called only by the older entry point. The remaining ~20 are ORPHANED or
alternate entry points nothing invokes. UNVERIFIED that all 20 are truly dead —
they may be invoked by hand or by cloud_produce.py on the pod. GAPS.md tracks
this.