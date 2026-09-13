# DECISIONS.md — running log of choices and why

> **Purpose:** running log of decisions and why, incl. the doctrine rule-1/rule-3 gap list.
> **Reader:** every session.
> **Last verified against code:** 2026-09-09.

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
The only real project is /home/muads/yt-digest/soccer-channel. ~/retired/ is
dead, never read or run. The soccer-channel SKILL.md (both global and parent)
still points at ~/soccer-pipeline/ and is DEPRECATED — do not follow it for
new work (see SKILLS.md).

### 720p rejection gate added to produce_v2.py
Why: produce_v2.py was silently accepting 360p downloads and upscaling. Now
loops up to 5 candidates, ffprobes each, rejects <720p, uses cookies from
secrets/yt_cookies.txt, fails loudly. Verified: source is now 1920x1080
(CONTEXT.md). Backup at tools/produce_v2.py.bak.

## STANDING OPERATING DOCTRINE (added 2026-09-09, Stage 4A)

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

### Gap: what does NOT yet meet the doctrine (updated 2026-09-09, Stage 6)

**Rule 1 (nothing local) — VIOLATED by the CPU/storage stages, not by the
download.** Stage 5A put step3 (download+cut) and step4b (tracking) on the pod.
Stage 6A moved the RAW DOWNLOAD to /mnt/f USB staging (`tools/staging.py`,
fail-loudly if /mnt/f not mounted, no /home/muads fallback), so a local download
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

## Stage 7 — rule 3 applies to documents (2026-09-09)

Rule 3 ("nothing orphaned may exist") covers documents, not just tools. A stale
doc that reads as authoritative is the exact failure that cost a session
(PROJECT_REPORT.md read as current for two weeks after the code moved on).

Audit method: `grep -rl "<doc>.md" tools/ scripts/ tests/` for every root doc
(7A.1). Result: NO root doc is read by any pipeline tool at runtime. Tools read
`scripts/<slug>.md` (the per-episode scripts), never a root doc. The one hit,
`SCRIPT_TEMPLATE.md` in `fresh_fetch.py:249`, is a string literal ("see
SCRIPT_TEMPLATE.md") embedded in output text, not a file read. So FUNCTIONAL
(read by code at runtime) = zero root docs. The rest split into CANONICAL /
RECORD / ORPHANED.

Retired (ORPHANED — read by nothing, superseded or contradicted by code):
- `PROJECT_REPORT.md` (dated 2026-08-23) — REPORT_AUDIT.md proved it wrong on
  every structural point. Retired per 7A.2.
- `HANDOFF.md` (dated 2026-08-29) — says `vastai_shorts.py` works and RunPod is
  dead. Code contradicts: `ls tools/vastai_shorts.py` → "No such file" (retired
  Stage 6); RunPod is the working path (Stage 5).
- `CONFIG.md` — claims "Nothing else in the workspace hard-codes the name;
  everything references CHANNEL_SLUG." Code contradicts: 29 of tools/*.py
  hard-code the string "soccer-channel" (grep). No tool reads CONFIG.md.
- `PLAN_ARTIFACT.html` (2026-08-25 blueprint) — claims "9/10 ACHIEVED" with "No
  CV annotation"; cv_annotate works now and the pipeline is 720x1280 with
  footage/narration mismatch. Superseded + contradicted.

Where retired docs go: DELETED from the working tree via `git rm` (all four were
tracked, so recoverable from git history with `git show <rev>:<path>`). The
retirement is recorded here. No `retired/` folder is kept in the tree (it would
itself be an orphan); git history is the archive.

Survivors (16) carry a header line: purpose / reader / last-verified-against-code
date (7A.4). CANONICAL (mirrored spine + active plan/instructions):
ARCHITECTURE, CLAUDE, CONTEXT, DECISIONS, GAPS, LANE_PLAN, PROGRESS, README,
SCRIPT_TEMPLATE, STATUS, TOOLS. RECORD (completed-stage artifacts, kept for
history): GATES (Stage 5 gates), RECONCILIATION (2026-09-08 audit),
REPORT_AUDIT (the audit that proved PROJECT_REPORT wrong), SKILLS (2026-09-05
skill analysis), VISUAL_QUALITY_ASSESSMENT (2026-08-25 snapshot, superseded by
STATUS for current state).

## 2026-09-13 — Gemini authoritative + scrap-2D/build-3D + spend tracking

### Gemini is the authoritative visual judge (supersedes Opus)
Decided by Mayo 2026-09-13 mid-task, now binding. Gemini
(tools/gemini_judge.py, gemini-3.1-pro-preview, fallback gemini-3.6-flash,
GEMINI_API_KEY in ~/yt-digest/.env) is the AUTHORITATIVE visual judge. Opus
(ask_claude.py --image) and gemma4 (vision_analyze.py) are cross-checks. On
disagreement, record the Gemini score as authority and flag it; never write
the more flattering (higher) number. This supersedes the prior Opus
authoritative rule that lived in CLAUDE.md (Judge rule) and was referenced in
CONTEXT.md. CLAUDE.md, CONTEXT.md, and this file updated 2026-09-13. Boundary
(doctrine): Gemini judges what it can see; it does NOT generate timestamps,
pitch coordinates, or cut decisions, those stay mechanical.
Proof of authority switch: a 5-lens Opus/gemma4 workflow scored the
arsenal-chelsea 3D frame 5/10 (Opus) / 3/10 (gemma4); Gemini (authoritative)
scored it 5/10 with verdict DESIGN and "a well-designed 2D board would beat
it" (yes). Gemini confirmed Opus on this frame.

### Scrap 2D, build 3D (relay Part 5, final)
Decided by Mayo 2026-09-13, final. Scrap the 2D matplotlib/tactical_render
renderers and build the 3D path (scene_gen.py via modal_render3d.py) next.
This contradicts the Stage 12A hold (12A.3 HELD the git rm; verdict "design
not dimension, prove design on 2D first") and contradicts the re-run evidence
(both Gemini and Opus say a well-designed 2D board beats the 3D frame).
Mayo reviewed the contradiction and chose relay Part 5 anyway. Mayo's call.
Execution order (doctrine #5, no dead ends): build and fix the 3D path first
(fix scene_gen camera + pitch-line bugs, re-render, verify), wire it into
produce_v2 as the board source, THEN retire the 2D tools. Deleting the wired
2D tools before the 3D replacement exists would break produce_v2 step2_boards
(:93), step4b (:364), produce_episode (:180), and the 12B assembler (step5,
:489+). That dead end is forbidden.

### 12A git rm hold: nothing was removed (recorded, no contradiction)
The brief asked: if 12A held a git rm, what was removed, when, by which
decision. Answer (verified `git log --diff-filter=D -- tools/tactical_boards.py
tools/tactical_render.py` -> no output): NOTHING was removed. Stage 12A
(commit b1ab579, 2026-09-12) recorded "12A.3 HOLDS the git rm of
tactical_render.py + tactical_boards.py" and did not execute it. Both files
remain on disk and wired (tactical_boards.py mtime 2026-09-12 17:38;
tactical_render.py mtime 2026-09-06 00:57). No removal contradicts a recorded
decision. The 12A hold stood, which is why the relay's 2026-09-09 "scrap
matplotlib" decision never reached the code.

### Spend tracking (new rule, this stage)
Report actual spend per render and a running total after each cloud run.
Modal T4 rate $0.000164/s (modal.com/pricing, from commit b1ab579). Total
provider spend across RunPod, Vast, Modal has never been measured and remains
an open unknown. Known point estimates: 12A 3D render 408.9s ~ $0.067;
arsenal-chelsea 3D re-run 2026-09-12 ~ same code/cached image (not separately
timed). Modal workspace minakush000-crypto, Starter plan, $29.15 credit as of
2026-09-13.

### 12A "5/10 reproducible" does not reproduce (artifact vs report)
The 12A commit reports "Opus scored the t=2.5s frame 5/10, reproducible,
encoding variance only." The surviving artifact
(3dpoc_2026-09-12_preview-manc-derby.mp4, run 2, 522523 bytes) does NOT
reproduce this: re-judged at t=2.5s by Gemini-authoritative + gemma4 + Opus,
all score 1/10 (camera clipped inside the scene, 0 players visible); the
t=4.5s frame is 0/10, fully broken. The manc-derby match_data is complete
(Man Utd 4-2-3-1 vs Man City 4-3-3, 11-v-11), so the scene should have had 22
tokens. Report said success; artifact says broken. Artifact wins. The 3D
renderer is not reliably reproducible (one run produced a broken render).