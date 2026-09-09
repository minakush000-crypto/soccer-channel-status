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

1. NOTHING RUNS ON THE LOCAL MACHINE. No local downloads, except raw source
   staging on the USB flash drive at /mnt/f (staging area ONLY: download lands
   there, ships to the pod, gets deleted; never a working directory). No local GPU work.
   All compute and all storage go to the cloud GPU providers.
2. No tasking without calling the available tools, skills, connectors, web
   fetch, research or MCPs where they apply.
3. No orphaned, standalone or untested item, tool or aspect of the pipeline
   may exist. Everything is wired, tested, or retired.
4. Every brief carries this doctrine as a footer.

### Gap: what does NOT yet meet the doctrine (updated 2026-09-09, Stage 6)

**Rule 1 (nothing local) — VIOLATED by the CPU/storage stages, not by the
download.** Stage 5A put step3 (download+cut) and step4b (tracking) on the pod.
Stage 6A moved the RAW DOWNLOAD to /mnt/f USB staging (`tools/staging.py`,
fail-loudly if /mnt/f not mounted, no ~ fallback), so a local download
no longer permanently grows the vhdx. The remaining local violators:

- step1 match_data (local ESPN API), step2 boards (local matplotlib),
  step5 assemble (local ffmpeg), step6 voice (local ElevenLabs API),
  step6b ambience (local ffmpeg), step7 merge (local ffmpeg),
  step8 shorts (local ffmpeg crop), and the final `shutil.copy2` to
  /mnt/c/Downloads (the one permitted local hand-off of the finished file).
- Transitional: until step5 moves to the pod, the staged clip on /mnt/f is read
  during local processing (slow 9p, unplug risk) — a temporary breach of the
  "staging-only" limit that closes when step5 runs on the pod (5A.4 gap).
- cut-list GENERATION (`scoreboard_scan.py` + `broadcast_filler.py` +
  `cut_list_gen.py` + PySceneDetect) runs locally and needs the clip. Blocked by
  scoreboard_scan's local-vision dependency (gemma4 via localhost:11434; switch
  to GEMINI_API_KEY cloud vision — scoped 6D.4).
- `produce_episode.py`, `assemble_words_match.py` (separate local pipelines).
- `cloud_produce.py` still downloads raw clips locally (`cloud_produce.py:223`).
- Caveat: pod yt-dlp is bot-blocked by YouTube (datacenter IP, 5A.3); the working
  rule-1 path is local-download → /mnt/f staging → catbox → pod cut → excerpts
  home → /mnt/f cleanup. A residential proxy would unblock pod yt-dlp (6D.2/6D.3).

**Rule 3 (no orphaned/untested) — 9 tools RETIRED total (5 in 5C.2, 5 in 6C.7
incl. vastai_shorts which was marked-retired but not deleted in 4B.4).**
- RETIRED (deleted, in git history): tactical_overlay, pitch_radar, render_video,
  check_and_download (5C.2); luminance_pod (crashed, dead callers), runpod_stage1
  (one-off done, hung on --help), runpod_annotate (6C.1: pod ran ~60s/$0.004 but
  captured 0 results — broken webhook, superseded by runpod_fulltrack), vastai_shorts
  (4B.4 retired, file deleted now), runpod_shorts (untested, depended on retired
  luminance_pod). Tool count 46 → 37.
- 27 STANDALONE decisions (6C.7): 10 have proper argparse (launch fine); 15
  treat `--help` as a positional arg (launch but no `-h` — low-priority polish,
  not a rule-3 failure); the 5 broken/dead ones retired above. `--help` proves
  "launches," not "works" — e2e runs are still owed for the untested STANDALONE
  (cloud_produce, runpod_superres, gpu_superres, produce_episode, etc.).
- `runpod_fulltrack.py` + `runpod_download.py` are WIRED + tested (compliant).
- 6C closures: OAuth token VALID (refresh works, 6C.5); upload 95 Mbps (6C.8);
  mirror redaction PROVEN (6C.4, path+key-pattern+file-type, not a general
  scanner); cut-list gen on 2nd source PARTIAL (6C.6 — snapping holds, but
  broadcast_filler misses celebration-classified goals); 1200s yield stays an
  ESTIMATE (6C.2 — no 1200s source exists, longest is 1120s); SoccerNet is a
  dataset/benchmark not a drop-in tool (6C.3 — reasoned, never measured).

This list is the work queue for rules 1 and 3. Stage 6 closed the raw-download
vhdx-growth (6A), 5 more dead tools (6C.7), and every open belief (6C). The
CPU/storage stages, cut-list-on-pod, and untested-STANDALONE e2e runs remain.