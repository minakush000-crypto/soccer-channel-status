# DECISIONS.md — running log of choices and why

Newest first. Each entry is dated. "Why" is the actual reason, not a
retcon. If a decision is reversed, add a new entry above, do not edit the old
one.

## 2026-09-05 (session 2)

### Option C Stage 1: cv_annotate.py exports per-frame player positions
Why: tactical_overlay.py guesses coordinates from text via glm-5.2:cloud
(line ~120 `generate_overlay_spec`). No pixel data is read. The fix is to
make cv_annotate.py export per-frame tracker positions so stage 2 can place
arrows at real coordinates. Added `player_positions` list (one entry per
frame, each with `{frame, players: [{id, bbox, team}]}`) to the tracking
JSON. Existing exports intact. Backup at `tools/cv_annotate.py.bak`.
Verified on RunPod: 300 frames, 129 tracker IDs, 1.16MB JSON, 23s inference.

### No path from tracker ID to player name exists
Why: grep of cv_annotate.py and match_data.py found no jersey OCR, no
position heuristics, no manual mapping. `extract_jersey_color` (line 72)
reads RGB for K-means team clustering only. match_data.py fetches jersey
numbers from ESPN (line 141) but nothing connects them to tracker IDs.
Consequence: Stage 2 is arrows on unnamed tracked players (team + bbox
only), or arrows placed by hand with a human naming the tracker. Building
an automated path (jersey OCR on the bbox → match to ESPN lineup) would be
a separate stage.

### Tracker fragmentation is high on this source
Why: 129 unique ByteTrack IDs in 10s of 1920x1080 footage. The compressed
wide-shot reupload causes the tracker to lose and re-ID constantly. Tracker
37 survives 5 frames (39-43). This limits how long any one arrow can
follow a player. Noted, not fixed — fixing it would need a better source
clip or a re-ID / appearance-embedding pass.

### runpod_stage1.py created as a one-off RunPod runner
Why: runpod_annotate.py processes all clips in a slug and doesn't upload
the tracking JSON back. For a single 10s test, a minimal one-off was
cleaner than modifying the existing tool. It uploads clip + tools to
catbox.moe, creates a webhook, runs cv_annotate.py --max-frames 300,
uploads results, and terminates the pod. Known bug: the webhook parser
fails when track_preview contains unescaped newlines (caught by
except:pass), so the script appeared to hang even though the pod finished.
Results were retrieved directly from the webhook API. Fix the parser
before reusing this script.

### Cost measured: $0.05 per 10s clip on L4
Why: pod ran 720s (12 min, including apt-get + pip install) at $0.25/hr =
$0.05. Actual inference 23s. The old CONTEXT.md estimate of "$0.15 / 5 min
per episode" was for 7 clips and is plausible but unverified for the full
set. This single-clip run is the first measured data point.

## 2026-09-05

### produce_v2.py is the authoritative entry point
Why: produce_episode.py is older and its step ordering and wiring
(generate_ambience, validate_script, cv_annotate) diverge from what the
verified output comes from. produce_v2.py produced a 720x1280 64.2s Short on
Sep 5 (ffprobe confirmed). produce_episode.py has no verified output today.
Caveat: produce_episode.py:288 still calls cv_annotate.py and must not break.

### Cloud-only for any GPU work
Why: this machine is an Intel N150 with no GPU. Local YOLOv8 is 1.30s/frame at
1080p (CONTEXT.md, measured), so 900 frames = 19.5 min, full clip = 95 min.
RunPod and Vast.ai both authenticate (CONTEXT.md). runpod_annotate.py tarballs
tools/ and ships it, so edits propagate. ~$0.15 and ~5 min per episode on an
L4 (CONTEXT.md). Local allowed only for ls, grep, ffprobe, single-frame
checks, reads, edits.

### Dropped all ball-dependent features
Why: ball detection is dead. 8 of 12 frames had no ball from either YOLOv8 or
roboflow (CONTEXT.md, measured on the compressed wide-shot reupload source).
No ball trail, no ball-following crop, no ball-tied arrows. Anchor to players
only.

### Rejected roboflow/sports
Why: its football-specific model found the ball in 4/12 frames vs 3/12 for
generic COCO (CONTEXT.md). Not a meaningful gain. PLAYER_DETECTION exceeded
2s/frame and BALL_DETECTION with InferenceSlicer was 14x slower than baseline.

### Retired ~/soccer-pipeline and ~/retired/soccer-*
Why: two folders named soccer-channel caused weeks of confusion (CONTEXT.md).
The only real project is ~/yt-digest/soccer-channel. ~/retired/ is
dead, never read or run. The soccer-channel SKILL.md (both global and parent)
still points at ~/soccer-pipeline/ and is DEPRECATED — do not follow it for
new work (see SKILLS.md).

### 720p rejection gate added to produce_v2.py
Why: produce_v2.py was silently accepting 360p downloads and upscaling. Now
loops up to 5 candidates, ffprobes each, rejects <720p, uses cookies from
secrets/yt_cookies.txt, fails loudly. Verified: source is now 1920x1080
(CONTEXT.md). Backup at tools/produce_v2.py.bak.

## STANDING OPERATING DOCTRINE (added 2026-09-09, Stage 4A)

1. NOTHING RUNS ON THE LOCAL MACHINE. No local downloads. No local GPU work.
   All compute and all storage go to the cloud GPU providers.
2. No tasking without calling the available tools, skills, connectors, web
   fetch, research or MCPs where they apply.
3. No orphaned, standalone or untested item, tool or aspect of the pipeline
   may exist. Everything is wired, tested, or retired.
4. Every brief carries this doctrine as a footer.

### Gap: what does NOT yet meet the doctrine (truthful list, not fixed this run)

**Rule 1 (nothing local) — VIOLATED by the whole pipeline today.** produce_v2
downloads and processes locally; only step4b (tracking) runs on RunPod. Verified
by grep (`yt-dlp`, `subprocess.run([FFMPEG,...])`, `shutil.copy2` to `/mnt/c`):

```
$ grep -nE "yt-dlp|/mnt/c|subprocess.run\(\[FFMPEG|shutil.copy2" tools/produce_v2.py
122: search_cmd yt-dlp   175: dl_cmd yt-dlp   404/413/437/448: ffmpeg assemble
618: shutil.copy2 -> /mnt/c/Users/muads/Downloads
```

Local-download / local-compute / local-storage violators (WIRED + STANDALONE):
`produce_v2.py` (download + ffmpeg assemble/merge + shorts + /mnt/c copy),
`produce_episode.py`, `assemble_words_match.py`, `scoreboard_scan.py`,
`broadcast_filler.py`, `segment_scorer.py`, `tactical_render.py`,
`tactical_boards.py`, `match_data.py`, `generate_voice.py`,
`generate_ambience.py`, `merge_voice.py`, `shorts_crop.py`,
`gemini_inventory_test.py` (reads local clip), `trim_tracking.py`,
`validate_script.py`, `assemble_video.py`, `enhance_clips.py`,
`generate_captions.py`, `thumbnail_generator.py`, `sharpness_check.py`,
`render_video.py`, `tactical_overlay.py`, `check_and_download.py`. The cloud
tools (`runpod_fulltrack`, `runpod_annotate`, `runpod_shorts`, `runpod_superres`,
`runpod_stage1`, `gpu_superres`, `vastai_shorts`, `cloud_produce`,
`luminance_pod`) are compute-compliant (pod) but `cloud_produce` still downloads
raw clips locally (`cloud_produce.py:223`). Rule 1 cannot be met until the
pod-side download path (LANE_PLAN 3B.5, ~2-3h) is built.

**Rule 3 (no orphaned/untested) — VIOLATED by 4 DEAD + ~18 untested STANDALONE.**
- DEAD (must retire): `tactical_overlay.py`, `pitch_radar.py`, `render_video.py`,
  `check_and_download.py` (RECONCILIATION 1.1).
- STANDALONE, never run end-to-end (import OK per 3D.1, execution unverified):
  `produce_episode.py`, `cloud_produce.py`, `runpod_annotate.py`,
  `runpod_shorts.py`, `runpod_superres.py`, `gpu_superres.py`,
  `luminance_pod.py`, `sharpness_check.py`, `assemble_video.py`,
  `validate_script.py`, `generate_captions.py`, `thumbnail_generator.py`,
  `enhance_clips.py`, `viral_angle.py`, `agent_reach_research.py`,
  `fresh_fetch.py`, `oauth_setup.py`, `ltx_enhance.py`.
- Hand-run (tested by execution, not wired): `scoreboard_scan.py`,
  `broadcast_filler.py`, `segment_scorer.py`, `assemble_words_match.py`,
  `trim_tracking.py`, `gemini_inventory_test.py`, `youtube_upload.py`,
  `runpod_stage1.py`. These meet "tested" but not "wired."
- `runpod_fulltrack.py` is WIRED + tested (the only fully-compliant tool).

This list is the work queue for rules 1 and 3. It is not fixed in this run.