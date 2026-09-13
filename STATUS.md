# STATUS.md — verified current state

> **Purpose:** verified current state of the pipeline (the status spine).
> **Reader:** every session (CLAUDE.md @STATUS.md).
> **Last verified against code:** 2026-09-09.

Verified against code on 2026-09-09 (Stage 3: broadcast_filler phantom bug fixed 3A.1, produce_episode 800M guard 3A.4; Stage 2: 720p cap + 180-1200s filter on produce_v2.py). This rebuild supersedes the 2026-09-05
version: the tactical_overlay wiring it described (`produce_v2.py:232`) no
longer exists, and the "no upload" claim is reversed (2 uploads happened
2026-09-06). No plans, no hopes, no hand-typed quality scores. Every claim has
the command that proved it.

## What runs today

### produce_v2.py (the authoritative 9-step pipeline, ARCHITECTURE.md)

Latest produce_v2.py output (liverpool-forest):

```
$ ffprobe -v error -show_entries stream=width,height -of csv=p=0:s=x renders/2026-08-30_liverpool-forest/shorts/final_video_shorts.mp4
720x1280
$ ffprobe -v error -show_entries format=duration -of csv=p=0 renders/2026-08-30_liverpool-forest/shorts/final_video_shorts.mp4
62.276009
$ stat -c '%s %n' renders/2026-08-30_liverpool-forest/shorts/final_video_shorts.mp4
24811062 ... Sep  7 00:07
```

### assemble_words_match.py (standalone, words-match-pictures path)

Arsenal-Chelsea, built 2026-09-08 (NOT a produce_v2 run):

```
$ ffprobe -v error -show_entries stream=width,height -of csv=p=0:s=x renders/2026-09-06_arsenal-chelsea/final_video.mp4
1280x720
$ ffprobe -v error -show_entries format=duration -of csv=p=0 renders/2026-09-06_arsenal-chelsea/final_video.mp4
45.200000
$ stat -c '%s %n' renders/2026-09-06_arsenal-chelsea/final_video.mp4
15034830 ... Sep  8 19:33
```
PRIVATE upload: https://www.youtube.com/watch?v=WFi2LBwXINU (and a second
PsEz5ITTIpM). `artifacts/publish-log/` has 2 entries:

```
$ ls artifacts/publish-log/
2026-09-06_arsenal-chelsea_PsEz5ITTIpM.json
2026-09-06_arsenal-chelsea_WFi2LBwXINU.json
```

### 200MB local-video guard (new, 2026-09-08)

`ffmpeg_utils.py` now enforces `LOCAL_VIDEO_LIMIT_MB = 200` on every local
video download (produce_v2 yt-dlp, runpod_fulltrack annotated-MP4 return,
cloud_produce download_url). See ARCHITECTURE.md "200MB guard".

## What is broken, worst first

### 1. Footage cuts ignore narration (broken logic)
`produce_v2.py:433`:
```
start = (seg.get("clip_idx", 0) * 5) % max(1, int(clip_total) - 5)
```
Fixed 5-second offsets indexed by `clip_idx`. The script's `[VISUAL: footage=]`
tags only select board-vs-footage (line 361 `has_footage = "footage=" in
tag.lower()`), not which part of the clip. `assemble_words_match.py` holds the
content-matched fix (parses `footage=START-END` windows) but is NOT folded in
— blocked by missing timestamp-window data + schema mismatch, see
RECONCILIATION 1.2.

### 2. Script is not validated against match data (broken correctness)
No call to `validate_script.py` in produce_v2.py (`grep -c validate_script
tools/produce_v2.py` → 0). CONTEXT example: script said "Gravenberch receives"
but match data has him as a 71st-minute sub.

### 3. Output is 720x1280, not 1080x1920 (broken format)
`shorts_crop.py:64` `OUT_W, OUT_H = 720, 1280`. Deliberate downscale to avoid a
soft 1.78x upscale. produce_v2 now caps the source at **720p** (`height<=720`,
`produce_v2.py:176`, Stage 2), so the 1080p/2160p source path is gone —
`shorts_crop`'s 4K→1080x1920 downscale branch (comment, line 78) is now
unreachable through produce_v2. Output stays 720x1280.

### 4. OVERLAYS ARE GONE (was broken, now resolved)
The 2026-09-05 STATUS said `tactical_overlay.py` guesses coords via
`produce_v2.py:232`. That wiring is GONE (`grep -n tactical_overlay
tools/produce_v2.py` → exit 1). The step is now `step4b_tactical_render`
(line 202) → `runpod_fulltrack.py` + `tactical_render.py` (real tracking data,
top-down renderer, Opus 8/8.5). `tactical_overlay.py` is DEAD.

## Option C Stage 1: per-frame player positions — DONE (2026-09-05)

`cv_annotate.py` exports `player_positions` in its tracking JSON: one entry per
processed frame, each with `{frame, players: [{id, bbox, team}]}`. Existing
exports (team_assignment, ball_positions) intact.

Verified on RunPod (L4, 300 frames = 10s of 1920x1080 footage):
- 300 frames, 129 unique tracker IDs, 38 team_assignment entries
- Pod inference: 23s (0.077s/frame on L4 vs 1.30s/frame local)
- Pod cost: ~$0.05 (720s uptime at $0.25/hr L4)
- Tracker fragmentation: 129 IDs span 3 camera shots (cuts at frames 130 and
  245). Within shots, tracking is usable: median consecutive run 32 frames
  (1.07s), 72 IDs survive >25 frames. Shot 2 (frames 131-244) cleanest: 35
  IDs, median run 41 frames, 2 survive the full 3.8s shot.

No path from tracker ID to player name exists (DECISIONS 2026-09-05, question A).

## Option C+E: segment selection + top-down tactical renderer (2026-09-05 session 3)

- **C (segment selection)**: DONE. `segment_scorer.py` scores 1-second windows
  by detection, persistence, stability, team classification. Full 146s: 15/146
  segments score >=65 (15s usable). Best sec 1 (85.6), worst sec 45 (7.5).
- **E (top-down tactical renderer)**: `tactical_render.py` renders dark pitch
  with mowing stripes, player dots in team colors, movement trails, Bezier
  arrows. Outputs PNG or MP4. Screen-space projection (no homography). Opus
  assessment: 8/10 then 8.5/10 across tasks 2-4 (authoritative). All elements
  visible.
- Known gaps for E (from Opus): no context layer (title, team names, ball
  marker, attacking direction); pitch layout off-centre, missing 6-yard
  boxes/penalty spots/arcs/corner arcs/goals, stripe contrast too high;
  movement encoding has trail/dot color conflict, no hierarchy, raw polylines,
  no min arrow length filter.

Full-clip tracking: DONE. 146s on RunPod L4, 156s, $0.011.
`clip_nxPNT4TU5_Q_full.tracking.json` (8MB, 4380 frames).

## Gemini video inventory assessment (2026-09-08)

Tested whether Gemini can produce a timestamped footage inventory. Half-works;
the half that's wrong is the half that matters for cutting.
- GEMINI_API_KEY valid (prepaid AI Studio). ~$0.11/clip on gemini-3.1-pro-preview,
  ~$0.04 on gemini-3.6-flash. 91 video tokens/sec @720p.
- Content classification accurate (goals/celebration/replay/crowd/subs).
- Timestamps NOT cut-accurate: boundaries 1-2s early, goal windows bloated,
  wrong team in open play. Found 2/3 real goals, missed the winner, invented 1
  phantom.
- Player names work via jersey+lineup lookup (all 7 tested correct).
- ~5-6 usable action passages (~50s) in 482s; reel ~90% non-action.
- DECISION: match_data.json is ground truth for events; Gemini is only a
  segment-finder. Never trust Gemini timestamps as cut points without
  match_data cross-check.

## Scoreboard scanner — goal-finding by scoreline change (2026-09-08)

`scoreboard_scan.py`: samples every 3s, crops the top-left score bug (420x150),
reads the scoreline with gemma4:cloud (free), carries last-known score across
NONE frames, detects changes. On the 482s arsenal-chelsea clip: 161 frames,
76s, free. 3 real scoreline changes found (after run-based blip absorption,
MIN_PERSIST=9s):
- ~102s: 0-0 -> 0-1 Rogers (Chelsea)
- 171s: 0-1 -> 1-1 Havertz
- 261s: 1-1 -> 2-1 Ødegaard
Matches match_data.json exactly (Rogers 2', Havertz 25', Ødegaard 50').
Walk-back offset to the shot is variable (1-8s). A fixed 8s pre-roll captures
every goal's shot; 1s walk-back gives the exact shot frame. The 8s pre-roll
rule FAILED on the words-match build (offset varies in sign: +17/+10/-8s);
fix = search ±20s around each bug-update, relay-verified.

## Broadcast/filler signal (2026-09-08, Part 2 — verified)

The score bug is NOT a broadcast/filler signal (uploader burned it in across
ALL footage types, including fan-shot phone frames). NONE = "gemma4 failed to
OCR the bug", not "bug absent". Replacement signal: Gemini inventory
content classification. broadcast = {shot, build-up} with shot_type != replay;
filler = {non-action} or {replay}. On the 482s reel: 43.0s broadcast (9%) /
440.0s filler (21 segments). The old bug signal claimed ~210s (44%), inflated
~5x. Tool: `broadcast_filler.py` (FIXED 3A.1: now clips segments to the video
duration and discards past-end phantoms via --clip/--duration; the 43s and 192s
figures stand, the 18.7min figure corrected 644→373s). Goal-finding is
unaffected (scanner works).

## Build one video: words match pictures (2026-09-08)

End-to-end cut-list-first build on Arsenal-Chelsea via `assemble_words_match.py`
(no arrows, no renderer, boards + clean footage only). Cuts footage at exact
verified timestamps, replacing produce_v2's fixed-5s-offset bug. Output
final_video.mp4 45.2s, 1280x720. PRIVATE: WFi2LBwXINU.
Step 7 (the never-run test) PASSES: relay on the mid-frame of each of 7
sections — 6/7 clean match, 1 partial. All 3 goals match narration via
on-screen scorer captions. Possession board had a pre-existing data bug
(39.3/32.7 vs 54.6/45.4) — fixed (see below).

## Possession board bug (2026-09-08, Part 4a — fixed) + board ratings (4b)

Possession bug FIXED. Root cause: the ANIMATED possession.mp4 filled the bar
over 75% of frames, broadcasting intermediate values. Fix: fill in first ~6%
of frames, hold full ~94%, label only final values. Verified by Opus
(54.6/45.4, full width, sum 100).
Board appearance RATED by Opus 5 (authoritative), NOT fixed: formation 3/10,
possession 4/10, stat_card 5/10. Common fails: matplotlib defaults, flat, no
Bebas/Barlow, no narrative furniture, stat-card bars both grow the same way.

## Goal cut-window diagnosis (2026-09-08, Part 5)

- Rogers (Chelsea), 119-127 (8s): 100% celebration + ROGERS 1-0 caption, NO
  shot in the reel. Script fixed to narrate the celebration, no shot claimed.
- Havertz (Arsenal), 181-191 (10s): first second = ball-in-net aftermath +
  HAVERTZ 1-1 caption; the live STRIKE is NOT in the window (~163-166).
- Odegaard (Arsenal), 253-262 (9s): first second (253) = LIVE STRIKE (bug still
  1-1 pre-update); then aftermath/celebration, ØDEGAARD 1-2 caption @262.
Goals 2 and 3 read wrong because both windows are mostly aftermath, so
strike-implying narration plays over mostly-celebration.

## Audio (2026-09-08, Part 6)

Crowd ambience: `generate_ambience.py` was an orphan (produce_episode/cloud_produce
called it, produce_v2 did not). FIXED: produce_v2.py `step6b_ambience` (line
479) generates `crowd_ambience.mp3`; `merge_voice.py` mixes it under voice at
30% volume with fade. Verified on a temp slug.
Narrator voice: VOICE_ID is per-episode config. `generate_voice.py` accepts
`--voice-id <id>`. 3 candidate samples saved LOCAL at experiments/voice-test/audio/
for Mayo to pick. Current .env VOICE_ID = 1stSYyl7ZVPJk2ECrNlo.

## Numbers that were hand-typed, not measured

The old STATE.md's 7/10, 8/10, 9/10 vision quality scores were typed by hand.
No code produces a score. Do not cite them. `sharpness_check.py` exists and
could produce a real number but is not in produce_v2.py.
## Stage 5 — pod-side download path + rule-3 debt, 2026-09-09

- NEW: tools/runpod_download.py (WIRED via --pod-download). Pod downloads +
  cuts excerpts; only guarded excerpts come home. Proven: 2/3 windows, 4MB each,
  128MB source never local, $0.005/run. YouTube bot-blocks pod yt-dlp (all
  clients) — --source-url (catbox) is the working path today.
- 4 DEAD RETIRED: tactical_overlay, pitch_radar, render_video, check_and_download
  deleted (git history); pitch_radar ship-list refs removed. Tool count 46 -> 42.
- 27 STANDALONE smoke test: 26/27 launch; luminance_pod crashes, runpod_stage1
  hangs. Rule 3 not closed per-tool (e2e owed).
- Rule 1: step3 (download) + step4b (tracking) now have pod paths; the CPU/storage
  stages (boards, assemble, voice, ambience, merge, shorts) + cut-list generation
  still run locally (DECISIONS.md gap list updated).
- 5A.5: 720p cap KEPT for the local path (pod yt-dlp bot-blocked, so 1080p-on-pod
  untested); pod path is 1080p-ready for when a proxy unblocks it.

## Stage 6 — /mnt/f staging + name overlap + beliefs closed + keys, 2026-09-09

- NEW tools: staging.py (/mnt/f staging), backup_env.py (encrypted .env backup),
  b2_upload.py (B2 archive, pending key). RETIRED 5: luminance_pod, runpod_stage1,
  runpod_annotate, vastai_shorts, runpod_shorts. Tool count 46 -> 37.
- Doctrine rule 1: +/mnt/f staging exception (4 docs).
- produce_v2: canonical-path assertion (6B.4) + step3 downloads to /mnt/f (6A).
- runpod_download: keyframe fix (stream-copy + re-encode fallback, timeout-bounded).
- 6C closures: OAuth valid, 95 Mbps upload, mirror redaction proven, runpod_annotate
  broken (retired), cut-list 2nd-source partial, 1200s estimate, SoccerNet reasoned.
- Open: 6A.2/6E.1 real /mnt/f proofs (pending mount: sudo mount -t drvfs F: /mnt/f),
  6E.2 real B2 upload (pending key), 6D.1 3/3 (pending non-throttled pod), cut-list
  on pod (6D.4), SCRIPT_TEMPLATE 4-template blocker.

## Stage 9 — benchmark spec + three fixes, 2026-09-12

- NEW: EPISODE_SPEC.md — measurable definition of a finished episode, every
  MUST traced to a 9A measurement of 6 reference videos. Production freeze
  gate until approved. Full 9A data + 9D in LANE_PLAN.md §Stage 9.
- 9C.1 FIXED (was: tactical_boards fabricates 0 stats + shows 0-0 for
  previews): stat_card skips unavailable rows + outcome stats for previews;
  possession board refuses when absent; formation hides scoreline for
  previews. Proven. match_data.json "preview": true (render artifact, gitignored).
- 9C.2 FIXED (was: pillarboxed 1920x1080): boards normalize to 1920x1080
  (fit + pitch-bg pad). Edge check before black 14/14 -> after 0/14. All
  board PNGs/MP4s now 1920x1080.
- 9C.3 diagnosed: 41s is the script (~140 words). Need 1100-1900 words for
  8-14 min. Word budget not yet in SCRIPT_TEMPLATE.
- 9D: boards-only lane B resembles none of the 6 references; recommend drop
  as publishable format (OK as sub-60s preview Short). Footage acquisition
  still blocked by rule 1 + YouTube datacenter block + /mnt/f unmounted.
- /dev/shm used for 9A under a one-time rule-1 exception (RAM, no vhdx bloat);
  cleaned up. /mnt/f still unmounted.

## Stage 10 — amendments + script budget + opening gate + 3D scope, 2026-09-12

- EPISODE_SPEC.md APPROVED with 3 amendments applied: talking-head is a
  format decision (voiceover-only); 3D renders promoted to a build target;
  footage acquisition removed from impossible.
- Doctrine Rule 1 amended in all 4 canonical docs (CLAUDE/CONTEXT/DECISIONS/
  ARCHITECTURE): raw footage via residential -> /mnt/f -> pod -> delete
  locally. Verified: old rule-1 text 0 remaining, new text 1 in each doc.
- 10B: SCRIPT_TEMPLATE.md §WORD BUDGET (A/D 1400-2100, B/C 1050-1750).
  validate_script.py enforces the lane min (proven: 17w FAIL). tools/
  script_gen.py built (glm-5.2:cloud, per-section, reads match_data + fresh
  brief; 1439w draft proven). Human must fact-check + resolve [SRC].
- 10D: validate_script.py check_opening_hook rejects a static formation-board
  open (spec §6) before assembly. Proven: formation FAIL, footage PASS.
- 10C: Blender confirmed right tool. PoC BLOCKED on 0/48 GPU (2026-09-12,
  re-checked twice). 10C.4 deletion HELD — tactical_render.py +
  tactical_boards.py stay until a 3D replacement is proven at/above spec.
  TOOLS.md annotated. Scope + downstream list in LANE_PLAN.md §Stage 10.
- Production freeze stays until an episode scores >= 7/10.

## Stage 11 (part 1) — Rule 5 + mirror retirement, 2026-09-12

- Doctrine Rule 5 (NO DEAD ENDS) added to all 4 canonical docs.
- Public status mirror RETIRED 2026-09-12 (rule 3): redundant (GitHub connector
  syncs private repo; raw.githubusercontent not fetchable by Claude) + fragile
  (Stop-hook-only, 3-day gap 2026-09-09 to 2026-09-12). git rm push_status.sh;
  removed its global Stop-hook entry (kept unlazy); stripped mirror claims
  from 6 doc headers + CONTEXT.md. artifacts/ + frames/ stay local.
  **RE-ENABLED 2026-09-12 ~22:39 (same day):** push_status.sh re-created on
  disk, re-registered as a PROJECT Stop hook (settings.json, additive per rule
  12), and `git add`'d back (un-retired). Mirror resumed pushing (last push
  22:42). CONTEXT.md updated to "active." The Stop-hook fragility is accepted
  (the mirror is a convenience; the private repo is canonical). See CONTEXT.md
  "STATUS MIRROR (public, active)."
- Correction on record: push_status.sh WAS registered (global Stop hook, not
  project settings); mirror WAS pushed today. The 3-day gap was the real
  issue, not "not registered." Stage 8 did not replace a hook registration
  (project settings.json was new, no prior hooks).
- A4 audit: all other "automatic" claims hold (check_pods, block-retired,
  block-image-read, warn-local-gpu, unlazy Stop, OAuth auto-refresh). Only
  the mirror claim was false.
- A5: CLAUDE.md rule 12 — settings.json hook writes are additive.

## Stage 11 (part 2) — GPU unblock, 3D PoC attempt, assembler scope, 2026-09-12

- 11A: check_mnt_f.sh SessionStart hook (fires); fstab `F: /mnt/f drvfs
  defaults,noatime,nofail 0 0` is the durable reboot-surviving fix (one-time
  sudo). settings.json now has 2 SessionStart hooks (additive, rule 12).
- 11B: Vast.ai has capacity (RTX 3090 $0.17/h); RunPod 0/48. Primary=Vast,
  fallback=Modal. render3d_vast.py = provider-agnostic layer (Vast backend).
  vastai_shorts fault was stale classic-API auth; Vast works now.
- 11C: 3D PoC BLOCKED on retrieval. Vast instance created/destroyed cleanly
  (capacity+credit OK) but the SDK has no SSH/logs, env-drop is broken,
  catbox/webhook are blocked from the sandbox, no port-exposure. 10C.4 HELD
  (tactical_render.py + tactical_boards.py stay). Block = one-time user action:
  Modal token (modal_render3d.py ready) or Vast SSH key. Awaiting user.
- 11D: assembler scoped (~1-2 days); est. score ~6.9/10 with 2D boards + 1400w
  + footage + assembler. Assembler-first beats 3D on score-per-hour and is a
  prerequisite for 3D. Build the assembler next.
- No episode has scored >= 7/10; freeze holds.
