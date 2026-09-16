# STATUS.md — verified current state

> **Purpose:** verified current state of the pipeline (the status spine).
> **Reader:** every session (CLAUDE.md @STATUS.md).
> **Last verified against code:** 2026-09-14.

Verified against code on 2026-09-14 (brief 0610: stale-voice hole closed +
re-score + 2D-vs-3D contradiction settled + Gemini litmus). See the new
"Stage 15 — stale-voice hole + re-score + litmus" section below for the
episode score, the voice failure, the skip-logic fix, and the Gemini
vs scanner head-to-head. Earlier verification 2026-09-13 (Stage 12B:
produce_v2.py rewritten 858 lines; Stage 14: tactical_render.py + step4b
RETIRED, 3D formation board wired into step2_boards). Re-verified all line citations against
the current produce_v2.py (858 lines, post-Stage-12B rewrite). Stage 14
changes retired tactical_render.py + step4b, wired the 3D formation board into
step2_boards, added a .script_verified voice gate, rewrote step5_assemble to
cap footage at 20%, and added a 3rd SessionStart hook (check_doc_stamps.sh).
This rebuild supersedes the 2026-09-05 version: the tactical_overlay wiring
it described (pre-Stage-12B `produce_v2.py:232`, line number from the old
code) no longer exists, and the "no upload" claim is reversed (2 uploads
happened 2026-09-06). No plans, no hopes, no hand-typed quality scores. Every
claim has the command that proved it.

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

NOTE: file overwritten Sep 12 by a later run. Current state (verified
2026-09-13): 1920x1080, 724.000000s, 210477037 bytes, Sep 12 21:43.
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

### 1. Footage cuts via cut_list_gen (was: fixed 5s offsets, FIXED Stage 12B)
The old `clip_idx*5` fixed-offset logic (pre-Stage-12B `produce_v2.py:433`)
is REMOVED. Footage cutting now uses `cut_list_gen.py` wired into
`step4a_cutlist` (`produce_v2.py:371`), which calls `broadcast_filler.py`
(line 404) then `cut_list_gen.py` (line 414) to produce timestamp windows.
`step5_assemble` (`produce_v2.py:430`) reads `cut_list.json` and caps footage
at 20% of runtime (line 512: `footage_budget = 0.20 * total_duration`).
The script's `[VISUAL: footage=]` tags select board-vs-footage (line 472
`is_footage = tag.startswith("footage=")`, was `has_footage` at pre-rewrite
line 361). `assemble_words_match.py` still exists as a standalone tool; its
content-matched cut logic is now folded into produce_v2 via cut_list_gen
(step4a, line 371).

### 2. Script IS validated against match data (was broken, FIXED Stage 10B+14)
validate_script.py is now wired into produce_v2.py (`grep -c validate_script
tools/produce_v2.py` → 6). Called at `produce_v2.py:362` as STEP 1c.
A `.script_verified` gate (`produce_v2.py:648`) blocks voice generation until
a human fact-checks the script (Stage 14). CONTEXT example from the old
broken state: script said "Gravenberch receives" but match data has him as a
71st-minute sub.

### 3. Output is 720x1280, not 1080x1920 (broken format)
`shorts_crop.py:64` `OUT_W, OUT_H = 720, 1280`. Deliberate downscale to avoid a
soft 1.78x upscale. produce_v2 now caps the source at **720p** (`height<=720`,
`produce_v2.py:218`, Stage 2), so the 1080p/2160p source path is gone —
`shorts_crop`'s 4K→1080x1920 downscale branch (comment, line 78) is now
unreachable through produce_v2. Output stays 720x1280.

### 4. OVERLAYS ARE GONE; step4b RETIRED (was broken, then resolved, then retired)
The 2026-09-05 STATUS said `tactical_overlay.py` guesses coords via
pre-Stage-12B `produce_v2.py:232`. That wiring is GONE (`grep -n tactical_overlay
tools/produce_v2.py` → exit 1). The step was then `step4b_tactical_render`
(pre-rewrite line 202) → `runpod_fulltrack.py` + `tactical_render.py`. As of
Stage 14, step4b_tactical_render is RETIRED (`produce_v2.py:313`,
`tactical_render.py` DELETED, `produce_v2.py:791` sets `tactical_path = None`).
The 3D formation board (three_scene.js via three_render3d) now covers the formation
render in `step2_boards` (`produce_v2.py:91`). `tactical_overlay.py` is DEAD.

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
- **E (top-down tactical renderer)**: `tactical_render.py` (RETIRED Stage 14,
  DELETED) rendered dark pitch with mowing stripes, player dots in team colors,
  movement trails, Bezier arrows. Outputs PNG or MP4. Screen-space projection
  (no homography). Opus assessment: 8/10 then 8.5/10 across tasks 2-4
  (authoritative). All elements visible. Superseded by the 3D formation board
  (three_scene.js via three_render3d, wired into step2_boards).
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
- GEMINI_API_KEY valid (prepaid AI Studio). ~$0.11/clip on gemini-2.5-pro,
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
665) generates `crowd_ambience.mp3`; `merge_voice.py` mixes it under voice at
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
- 27 STANDALONE smoke test: 26/27 launch; luminance_pod crashes (RETIRED
  Stage 6, deleted), runpod_stage1 hangs (RETIRED Stage 6, deleted).
  Rule 3 not closed per-tool (e2e owed).
- Rule 1: step3 (download) + step4b (tracking) now have pod paths (NOTE:
  step4b is RETIRED Stage 14, tactical_render.py deleted); the CPU/storage
  stages (boards, assemble, voice, ambience, merge, shorts) + cut-list generation
  still run locally (DECISIONS.md gap list updated).
- 5A.5: 720p cap KEPT for the local path (pod yt-dlp bot-blocked, so 1080p-on-pod
  untested); pod path is 1080p-ready for when a proxy unblocks it.

## Stage 6 — /mnt/f staging + name overlap + beliefs closed + keys, 2026-09-09

- NEW tools: staging.py (/mnt/f staging), backup_env.py (encrypted .env backup),
  b2_upload.py (B2 archive, pending key). RETIRED 5: luminance_pod, runpod_stage1,
  runpod_annotate, vastai_shorts, runpod_shorts. Tool count 46 -> 37.
  Current tool count (2026-09-13): 47 (`ls tools/*.py | wc -l`).
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
  still blocked by rule 1 + YouTube datacenter block + /mnt/f unmounted
  (NOTE: /mnt/f is now MOUNTED since Stage 11A, fstab `F: /mnt/f drvfs`).
- /dev/shm used for 9A under a one-time rule-1 exception (RAM, no vhdx bloat);
  cleaned up. /mnt/f now mounted (Stage 11A fstab fix).

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
  re-checked twice). 10C.4 deletion HELD at the time — tactical_render.py +
  tactical_boards.py stay until a 3D replacement is proven at/above spec.
  UPDATE Stage 14: tactical_render.py is now DELETED (RETIRED); step4b
  retired. tactical_boards.py still exists. TOOLS.md annotated. Scope +
  downstream list in LANE_PLAN.md §Stage 10.
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
  sudo). settings.json now has 3 SessionStart hooks (check_pods.sh,
  check_mnt_f.sh, check_doc_stamps.sh added Stage 14; additive, rule 12).
- 11B: Vast.ai has capacity (RTX 3090 $0.17/h); RunPod 0/48. Primary=Vast,
  fallback=Modal. render3d_vast.py = provider-agnostic layer (Vast backend).
  vastai_shorts fault was stale classic-API auth; Vast works now.
  (NOTE: vastai_shorts.py is RETIRED Stage 6, deleted.)
- 11C: 3D PoC BLOCKED on retrieval. Vast instance created/destroyed cleanly
  (capacity+credit OK) but the SDK has no SSH/logs, env-drop is broken,
  catbox/webhook are blocked from the sandbox, no port-exposure. 10C.4 HELD
  (tactical_render.py + tactical_boards.py stay). UPDATE Stage 14:
  tactical_render.py is now DELETED (RETIRED); tactical_boards.py still
  exists. Block = one-time user action:
  Modal token (three_render3d.py ready) or Vast SSH key. Awaiting user.
- 11D: assembler scoped (~1-2 days); est. score ~6.9/10 with 2D boards + 1400w
  + footage + assembler. Assembler-first beats 3D on score-per-hour and is a
  prerequisite for 3D. Build the assembler next.
- No episode has scored >= 7/10; freeze holds.

## Stage 12 re-run — 3D board on Modal, design vs dimension, 2026-09-12

Relay (2026-09-12/13) next action executed: three_render3d.py as-written on Modal
against arsenal-chelsea match_data.json, frame extracted, judged vs the 2D
possession.png control through a 5-lens vision workflow (4 Opus + 1 gemma4) +
synthesis. Provider Modal T4; output /mnt/f/soccer-staging/3dpoc_2026-09-06_arsenal-chelsea.mp4
(1280x720, h264, 5.0s, 534892 bytes; ffprobe verified). NOTE: file
overwritten Sep 13; current size 442966 bytes (stat -c '%s' verified
2026-09-13). Wall ~408.9s, cost
~$0.08 (12A same-code baseline; not re-timed). Frame
artifacts/frames/3dpoc_arsenal-chelsea_t2500.png.

- Authoritative (Opus): 3D frame 5/10; 2D possession control 4/10 (confirms
  prior). gemma4 cross-check 3/10 (disagreement flagged, Opus retained).
- VERDICT: DESIGN not DIMENSION. A well-designed 2D board would beat this 3D
  frame. CONFIRMS Stage 12A. CONTRADICTS the relay's Part 5 "scrap 2D, build
  3D" decision (overruled 12A without new evidence; this is the new evidence
  and it confirms 12A). Do NOT scrap 2D; build the design layer on 2D first.
- 12A "5/10" does not reproduce from the surviving manc-derby artifact:
  re-judged by Opus (1/10) + gemma4 (1/10) at t=2.5s, camera clipped inside
  the scene, 0 players; t=4.5s = 0/10. The 3D renderer is not reliably
  reproducible (one run produced a broken render despite the commit claiming
  "reproducible, encoding variance only"). Report says success, artifact says
  broken.
- Full verdict: artifacts/frames/synthesis_verdict.md. Freeze holds (no
  episode >= 7/10).

## Stage 12 re-run (cont.) — Gemini authoritative + bug-fix re-renders, 2026-09-13

Mayo brief: Modal funded ($29.15). Gemini AUTHORITATIVE visual judge
(supersedes Opus in CLAUDE.md/CONTEXT.md; DECISIONS.md updated). Scrap 2D,
build 3D (relay Part 5, final). 12A held the git rm (nothing removed; git log
--diff-filter=D empty). The 5/10 3D frame was buggy; fixed and re-rendered,
one change per run.

- tools/gemini_judge.py built (gemini-2.5-pro) + wired as AUTHORITATIVE
  in CLAUDE.md tool routing. Gemini judged both: 3D buggy=5/10, 2D control=4/10
  (matches Opus).
- CHANGE 1 camera framing: keyframes (0,-90,160)->(0,-55,150) replace the
  cropped (55,-68,72). Render 1 wall 707.61s. Gemini 4/10, all 22 tokens visible.
- CHANGE 2 pitch lines: torus rotation (90,0,0)->(0,0,0) fixes the centre
  circle; six-yard offset 4.125->2.75. Render 2 wall 555.34s. Gemini 6/10,
  centre circle clean flat ring YES, penalty + six-yard complete YES.
- SETTLED: non-buggy 3D = 6/10 (Gemini=Opus=gemma4 agree). 2D control 4/10.
  Non-buggy 3D beats 2D by +2 (depth). Cap is DESIGN (narrative furniture,
  dead void, typography), dimension-independent. Opus flagged compressed
  formation layout + flat tokens as 3D-build items.
- Spend: render 1 707.61s, render 2 555.34s (Modal T4 $0.000164/s). Running
  total 3D ~$0.27-0.34. Modal credit $29.15. Cross-provider total unknown.
- Next: scrap 2D / build 3D, build-then-retire order (wire 3D into produce_v2
  first, verify, then retire 2D) so the pipeline is never broken.

## Stage B2 — archive of record + cache clearing, 2026-09-13

B2 archive of record established: rclone v1.75.1, remote "b2", bucket
"mendymax-archive" (verified `rclone lsd b2:` → mendymax-archive). Bucket
already holds `backups/`, `retired-archive/`, `yt-digest/` dirs. NEW STANDING
RULE: everything produced by any task (renders, frames, transcripts, artifacts,
backups) is archived to B2; local copies are working copies, not the record.
Doctrine rule 1 (raw footage -> /mnt/f -> pod -> delete locally) is unchanged;
this extends it to generated outputs.

Disk reality (verified 2026-09-13): ext4.vhdx is 37G at
`/mnt/c/Users/muads/AppData/Local/wsl/{f2ea779f-e5f1-4c82-a1b2-0608e6ab4883}/ext4.vhdx`
(`ls -lh` verified); it grows and never shrinks on its own (deleting inside
Ubuntu does not shrink it; compaction needs WSL fully stopped). C: has ~15 GB
free of 119 GB; `/` shows 29G used of 1007G after cache clearing (`df -h /`
verified).

Regenerable caches CLEARED (2026-09-13): ~/.cache (was 3.3G, now 40K),
~/.npm, ~/.cargo/registry, ~/.nvm/.cache, ~/.bun/install/cache,
~/.rustup/downloads, __pycache__ 388M, ~/.linkedin-mcp/patchright-browsers
394M. `/` went 35G -> 29G used (~6G freed). Caches are regenerable, no archive
needed.

Archive-then-delete IN PROGRESS to B2 (local deletions happen AFTER docs
amended + uploads verified):
- sep5 backups (~2.6G) — archive to B2, then delete local.
- ~/retired (1.4G, `du -sh` verified) — archive to B2, then delete local.
- Old render folders (~3.5G total, `du -sh` per dir verified):
  2026-08-18_iraola-liverpool (1.3G), 2026-08-30_liverpool-forest (457M),
  2026-08-30_liverpool-forest_sep1 (538M), 2026-09-06_arsenal-chelsea (753M),
  _20min_test (245M), _real_soccer_test (169M), _sharp_test (39M),
  _fullmatch_arsenal-chelsea-carabao (20K), 2026-09-12_preview-manc-derby
  (1.7M). KEEPING 2026-09-12_bournemouth-brentford (416M) local.

CLOSING PASS: after-numbers still owed — (1) free space on `/` after local
deletions complete, (2) confirmed-deleted list, (3) B2 bucket size after
uploads. These are not fabricated; placeholders until the closing pass fills
them. /mnt/f writes are currently EINVAL, so /dev/shm is the volatile fallback
for staging.

## Stage 15 — stale-voice hole + re-score + 2D-vs-3D litmus, 2026-09-14

### The voice failure (closed)
The 2026-09-12_bournemouth-brentford episode scored **5.5/10** (below the 7/10
gate; freeze held). The single disqualifier was §10 (narration source-of-truth,
MUST FAIL): the voice track named a fabricated scorer. Root cause verified:
`voice_elevenlabs.mp3` mtime 2026-09-13 02:16 was TTS'd from the PRE-FIX script
(1911 spoken words, containing "Igor Thiago needed just one big chance to
score"). The script was fixed at 20:33 (Igor Thiago -> Kevin Schade, who scored
both Brentford goals per match_data.json: Schade 34', 56'). The
`.script_verified` gate was set at 20:33 — AFTER the voice was already
generated. Every re-run reused the pre-fix voice because `step6_voice` skipped
on `voice_path.exists()` alone (`produce_v2.py` line 691-694, pre-fix). The
gate checked the marker existed but not that the voice matched the current
script. ◑ the audio says Schade not Thiago (inferred from the hash + script
content; no speech-to-text tool exists to prove it directly).

### The fix (one code change, verified)
`step6_voice` now records the sha256 of `scripts/<slug>.md` to
`.voice_script_hash` at voice-generation time, and on the skip check compares
the recorded hash to the current script hash. "Voice file exists" is no longer
a valid skip condition; mismatch or missing hash -> regenerate. `import
hashlib` added. Diff pasted in the handoff report (brief 0610). Parse OK.
PROOF (all ●, commands run 2026-09-14):
- script spoken words (clean_script_for_tts): **1819** (the prior "1751" was approximate)
- old voice: **719.078s**; new voice: **794.676s** (ffprobe)
- timestamps in order: script 09-13 20:33:27 -> .script_verified 20:33:38 -> voice 09-14 01:12:21 -> .voice_script_hash 01:12:21 (gate precedes voice)
- hash match: recorded `a4a70df3...` = computed `a4a70df3...` (sha256sum) — voice provably from the fixed script

### Re-score (same method as the 5.5 run, Gemini authoritative)
Re-assembled final_video.mp4: 1920x1080, 794.76s (13.25 min), 77.7MB. 9 frames
judged by gemini-2.5-pro (raw responses in the handoff report):
opening 3/10, footage 3/10, formation(3D) 4/10, stat_card 4/10, possession 5/10,
xg_flow 7/10, shotmap 6/10, momentum 7/10, avgpositions 6/10. (xg_flow dropped
10->7 vs the 5.5 run on the SAME board — Gemini judgment variance, same model.)

EPISODE_SPEC section-by-section:
| § | Section | Score | Verdict |
|---|---|---|---|
| 1 | Runtime 8-14min | 2/2 | 13.25 min. PASS |
| 2 | Shot rhythm | 1.5/2 | 51 segments, mean 15.6s (PASS), longest 27s (below 30s min) |
| 3 | Content mix | 2/2 | 87% graphics, 13% footage. PASS |
| 4 | Graphic types | 1.5/2 | All 3 MUSTs + 4 new; SHOULD arrows/lower-third absent |
| 5 | Typography/colour | 1/2 | 4 new 2D boards have Bebas/Barlow; 3D formation + stat_card + possession don't |
| 6 | Opening | 1.5/2 | Footage-led hook (PASS); basic aerial, not dynamic action |
| 7 | Audio | 0.5/2 | **137.3 WPM (BELOW 155-195 MUST)**; crowd ambience present; voice from fixed script |
| 8 | Framing | 2/2 | 1920x1080, no pillarbox. PASS |
| 9 | Source-footage provenance | 1.5/2 | Footage frames clean aerial (no watermark; the 5.5 "Google watermark" was a false positive) |
| 10 | Narration source-of-truth | **2/2** | **RESOLVED.** Voice from fixed script (hash proven), no fabricated scorer |
| 11 | Board design properties | 1/2 | 4 new 2D boards have the 4 properties; 3D formation + stat_card + possession don't |
| **Total** | | **16.5/22 (7.5)** | §10 disqualifier RESOLVED |

**Overall: 7.5/10.** The §10 credibility disqualifier is resolved (the freeze's
reason is gone). BUT §7 regressed: the new voice is 137 WPM (below the 155
MUST) because ElevenLabs paced the 1819-word TTS slower than the 1911-word
pre-fix voice (159 WPM). The episode numerically clears the 7.0 gate, but §7
is a MUST violation. **Recommendation: regenerate the voice with an ElevenLabs
speed setting that hits 155+ WPM before declaring the freeze lifted.** If §7
MUST-fail is treated as disqualifying (like §10 was), the freeze holds until
the WPM is fixed; if only §10 credibility is disqualifying, the freeze lifts at
7.5. Either way the fabricated-scorer hole is closed.

### Piece 2 — what medium practitioners use (research workflow, 13 agents)
~/claude/40-lessons/viz-design-findings.md was checked FIRST (covers the
2D-vs-3D principle per board type). The workflow (7 named channels + 5 tooling
sources) supplemented the per-practitioner medium table. Key findings:
- **Positional 3D verdict:** ONE credible practitioner renders positional
  boards in true 3D for analysis: Coaches' Voice (FM-powered Masterclass
  series, sponsored by Football Manager). Broadcasters (Sky/BT/BBC) use 3D AR
  on LED studio floors for reveals (spectacle, not analysis). EVERY other
  practitioner keeps positional boards 2D: Football Meta (fmstadio, explicitly
  2D), Football Made Simple (Once Video Analyzer, 2D top-down), mplsoccer
  (2D-only by architecture), StatsBomb/Opta (2D for positional; 3D only for
  single-shot reconstruction), Tableau (2D-only), After Effects (2D for
  analysis, 3D for pre-match fly-throughs). The dominant industry standard is
  2D top-down.
- **Principle:** "2D for information, 3D for spectacle." Positional/tactical =
  2D top-down; statistical timelines = 2D; shot maps = 2D (3D only for
  single-shot reconstruction); broadcast pre-match = 3D (spectacle); studio
  reveals = 3D AR (engagement).
- **Contradiction verdict:** NEITHER on-record claim survives. STATUS 09-13's
  "3D 6/10 vs 2D control 4/10, 3D ahead by 2" compares a 3D FORMATION vs a 2D
  POSSESSION board (unlike types). The handoff's "2D beats 3D because momentum
  9/10 beats formation 3D 6/10" compares a 2D MOMENTUM vs a 3D FORMATION board
  (unlike types). The SAME board with the SAME data has NEVER been rendered
  both 2D and 3D in this project. What survives: "DESIGN not DIMENSION" — score
  variance is driven by design quality (typography, labels, colour discipline,
  title-as-message), not dimension. A like-for-like test (same formation board,
  same data, one 2D top-down + one 3D, both using the shared design layer) has
  never been run and is the only test that would settle the dimension question.

### Piece 3 — Gemini vs scoreboard scanner litmus (mechanical output)
Same episode, same 718.04s footage. Model: gemini-2.5-pro (47388 video
tokens, ~$0.11). Scanner: gemma4:cloud via vision_analyze.py, free, 68.6s.

**Timestamps (per-goal offset, misses, phantoms):**
Scanner (ground truth via scoreboard, 4/4 found, 0 phantoms): 60s (0-1 Schade),
102s (1-1 Kluivert), 117s (2-1 Tavernier), 162s (2-2 Schade). The blips
(BRIG/BRU/2-0) are gemma4 OCR noise that reverts in 3-6s, not phantoms.
Gemini found 3 goals (97s, 113s, 155s): called the real 60s goal a "saved
shot" (MISS), invented a goal at 97s where no scoreline changed (PHANTOM),
missed the 117s goal (MISS), and placed goal 2 at 113s (+11s vs scanner 102s)
and goal 4 at 155s (-7s vs scanner 162s). **Gemini: 2/4 found, 2 misses, 1
phantom. Scanner: 4/4 found, 0 misses, 0 phantoms, free.** This reproduces the
2026-09-08 finding (Gemini invents phantoms, misses real goals) on a different
clip with the same model family. Gemini does NOT win on timestamps.

**Pitch coordinates (TASK2, never tested before):** Gemini "read" the
avg-positions board graphic and produced 11 Brentford coordinates. It got the
GK (jersey 1: 10,50 vs Sofascore 11,50.3) and striker (jersey 9: 60,50 vs
60.7,52.2) right, but used 3 WRONG jersey numbers (5, 8, 11 — none exist;
real are 44, 23, 7) and most coordinates were off by 30-60 y-units. Gemini
produced a plausible "football-shaped" 4-2-3-1 from its own prior knowledge,
NOT a reading of the board (the board shows away MIRRORED at 100-x, but Gemini
reported RAW coords — it did not actually read the dots). **Verdict: Gemini
cannot produce usable pitch coordinates; it hallucinates plausible-but-wrong
data and confuses its own football knowledge for a reading.**

**Cut decisions (TASK3):** Gemini picked the first goal, named the scorer
"Mbeumo" (WRONG — Schade; Mbeumo is not in this match's lineup), and produced
a cut list that just includes the whole 88-101s segment (no replay exclusion,
no filler filtering, no editorial choice). The mechanical cut_list_gen produces
±20s relay-verified windows with broadcast/filler classification. **Gemini's
cut decision is inferior: wrong scorer, no editorial discrimination.**

**Delegation verdict:** Delegate to Gemini ONLY where it wins on the pasted
numbers. It wins NOWHERE here: timestamps (scanner 4/4 free vs Gemini 2/4
paid), coordinates (Gemini hallucinates), cut decisions (Gemini wrong scorer,
no filtering). The mechanical tools (scanner + Sofascore data + cut_list_gen)
win on every mechanical output. Gemini's value is CONTENT CLASSIFICATION (what
kind of passage), not WHEN/WHERE/WHAT-NUMBERS.
