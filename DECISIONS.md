# DECISIONS.md — running log of choices and why

> **Purpose:** running log of decisions and why, incl. the doctrine rule-1/rule-3 gap list.
> **Reader:** every session.
> **Last verified against code:** 2026-09-16.

Chronological order (oldest first); append new entries at the end. Each
entry is dated. "Why" is the actual reason, not a retcon. If a decision is
reversed, add a new entry, do not edit the old one.

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
timed). Modal workspace minakush000-crypto (verified in ~/.modal.toml); plan
tier and credit balance are NOT checkable from the filesystem and require the
Modal API/dashboard (last seen $29.15 Starter, 2026-09-13, unverified).

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

### CONDITIONAL — Gemini may generate cut timestamps only if it beats the scanner head-to-head (Mayo, 2026-09-13, binding)
The mechanical rule (Gemini judges what it sees; it does NOT generate
timestamps, pitch coordinates, or cut decisions — those stay mechanical, per
the Gemini-authoritative decision above) can be relaxed ONLY through this
litmus test. If an episode built under the current mechanical-only rule still
fails to reach 7/10, run a head-to-head BEFORE delegating cut-timestamp
generation to Gemini:
1. Gemini proposes cut timestamps for one episode.
2. The scoreboard scanner (tools/scoreboard_scan.py) proposes cut timestamps
   for the SAME episode.
3. Compare both against match_data.json (ground truth for events).
4. Report which is more accurate.
Delegate cut-timestamp generation to Gemini ONLY if Gemini wins that
comparison. The scoreboard scanner has already beaten Gemini on this job once
(Gemini video inventory: timestamps 1-2s early, bloated goal windows, wrong
team in open play, found 2/3 real goals, missed the winner, invented 1
phantom; the scanner matched match_data.json exactly on 3/3 scoreline changes
— Rogers 2', Havertz 25', Odegaard 50'). The comparison is therefore NOT a
formality. This is binding and the ONLY route by which the mechanical rule can
be relaxed. Recorded 2026-09-13 per Mayo's directive. PROCEED note attached.

### STAMP RULE + known weakness + 2026-09-13 reconciliation (binding)
STAMP RULE (also in CLAUDE.md, verbatim): any task that edits a canonical
document must, in the same task, either (a) update its "Last verified against
code" stamp to today having actually checked its claims against the code, or
(b) state in the task report which specific claims could not be verified and
leave the stamp unchanged. Never set a stamp you did not earn. The SessionStart
check (tools/doc_stamp_check.py, hooked via .claude/hooks/check_doc_stamps.sh)
is the backstop, not the process; if it fires, the stamp rule was skipped on a
previous task.

Known weakness (recorded so a future reader sees it as a known limitation,
not an oversight): this rule depends on being remembered — the same shape as
the status mirror that only updated when a push was remembered, which produced
a three-day silent gap (2026-09-09 to 2026-09-12). The SessionStart check makes
the failure visible rather than invisible, but it does not prevent it. A stale
stamp at next session start means someone edited a canonical doc without
applying the rule.

2026-09-13 reconciliation (this entry earns the 2026-09-13 stamp by checking
DECISIONS.md against the code, per PRECEDENCE). Verified this pass: Gemini is
the authoritative judge (gemini_judge.py, gemini-3.1-pro-preview); the
scrap-2D/build-3D execution order; the 12A git-rm hold (nothing removed);
spend tracking rule; reproducibility failure. Historical drift acknowledged,
NOT fixed per the don't-edit-old-entries convention: the Stage 5/6 era entries
contain stale line-number citations and file references — produce_episode.py:288
(actual :298), cloud_produce.py:223 (actual :228-229), "29 of tools/*.py
hard-code soccer-channel" (actual 33), the "27 STANDALONE = 10+15+5" arithmetic
(10+15+5=30, not 27), and the .bak references at tools/cv_annotate.py.bak and
tools/produce_v2.py.bak (both deleted 2026-09-08). These are records of state
at decision time; current code state lives in STATUS.md/ARCHITECTURE.md (which
carry their own drift, flagged separately, stamps NOT advanced this pass). The
"Newest first" header was wrong (the file is chronological) — corrected.

### PARTIAL 2D retirement — open debt (2026-09-13, confirmed by Mayo)
Decision: wire the 3D formation board (scene_gen.py via modal_render3d.py) into
produce_v2 as the formation board source; KEEP tactical_boards.py for the
possession and stat_card boards; RETIRE tactical_render.py + step4b (its
top-down tracking view is unused by the assembler — no hole). Rationale: the 3D
path covers ONLY the formation board (verified by reading scene_gen.py +
modal_render3d.py); deleting tactical_boards.py now would leave possession and
stat_card as holes, which the no-hole rule forbids. The "zero matplotlib" grep
cannot pass under partial retirement and is NOT claimed to.
OPEN DEBT: the 2026-09-09 "scrap 2D / build 3D" decision remains unfulfilled
for the possession and stat_card boards. This debt closes when 3D versions of
BOTH exist and pass Gemini (authoritative) at or above the 2D baseline (current
2D: possession 4/10, stat_card 5/10 per Opus — re-baseline with Gemini before
retiring).
Backlog (closes this debt): (1) build a 3D possession board in scene_gen.py /
modal_render3d.py; (2) build a 3D stat_card board. Both must Gemini-judge at or
above the 2D baseline before tactical_boards.py is retired.
Status: tactical_render.py retirement + 3D formation wiring — Stage 14 (in
progress). Possession/stat_card 3D — backlog, not started.

### 3D formation board scored 4/10 vs 2D 5/10 — partial retirement made that board worse (2026-09-13, measured result)
Result, not a setback. The Stage 14 episode (2026-09-12_bournemouth-brentford)
3D formation board (scene_gen.py via modal_render3d) was Gemini-judged 4/10,
cross-checked by Opus 4/10 and gemma4 4/10 (all three converge). The 2D
possession board scored 5/10 and the 2D stat_card 5/10 (Gemini). So the 3D
formation board, as wired, scored BELOW the 2D boards it replaced. This
contradicts the Stage 12 finding (3D 6/10 vs 2D 4/10) — the Stage 12 6/10 did
not reproduce in the actual episode. Opus diagnosis: "It isn't a formation.
Players parked in rows… no shape, no units… 3D adds cost but no information.
Would be stronger as clean 2D." The single biggest failure is
formation_positions() placing tokens in a synthetic grid, not a real shape.
Implication: the scrap-2D decision's payoff now depends ENTIRELY on (a) Sofascore
average-positions replacing the synthetic grid (verified direct fit, wired this
stage) + (b) design work (narrative furniture, bold condensed typography, arrows,
zone shading). 3D alone does not earn its place. If the average-positions
re-render is still at or below 5/10, scene_gen.py goes on the retirement list
alongside the other retired renderers.

### 3D formation board with real positions + distinct colours: 6/10 (Gemini), Opus disagrees (2026-09-13, measured)
The clean re-render (Sofascore average positions replacing the synthetic grid, +
kit-clash-distinct colours — both teams were red in the first attempt because a
sofascore_client re-run reset the colours; a durable kit-clash override now
prevents that) scored Gemini 6/10 (authoritative), up from 4/10 (synthetic grid).
The real-positions fix worked. ABOVE the <=5/10 retire threshold, so scene_gen.py
is NOT retired (per the standing instruction's literal condition). Opus cross-check
disagrees qualitatively: "raw plot, not a Coaches' Voice graphic... perspective is
the wrong tool for positional data: foreshortening makes vertical spacing
non-comparable... Red team's shape is unreadable as a formation." Per the Judge
rule, Gemini 6/10 is recorded as authority and the disagreement is flagged. Both
judges converge on the cap being DESIGN (zero narrative furniture, no selective
visibility, basic typography, label collisions) + a MEDIUM MISMATCH (3D perspective
distorts the 2D average-position data; a 2D top-down would preserve measurement
integrity — Opus: "flatten the perspective"). Still below the 7/10 gate. Path to
7/10 = design furniture + selective visibility + bold condensed typography + label
collision resolution, and consider a 2D top-down for the positions board. The
scrap-2D payoff still depends on design work, not 3D dimension. Running 3D Modal
spend: this re-render ~545s ~$0.09; running 3D total ~$0.45-0.52.

### No voice render on an unverified script (2026-09-13, binding)
An episode was rendered narrating 1911 words of un-fact-checked LLM draft;
validate_script flagged it and it rendered anyway — a confident voice asserting
unverified claims about a real match. The fact-check found one outright FALSE
goal attribution ("Igor Thiago needed just one big chance to score" — both
Brentford goals were Kevin Schade's, 34' and 56'), an unsupported "own-goal"
claim, a wrong match date in the header, unfilled <competition> placeholder,
untraceable manager/nationality/former-club colour, and no sources.json. Binding
rule: produce_v2 MUST refuse the voice step (step6_voice) until the script has
been fact-checked. The gate is a marker file renders/<slug>/.script_verified
(created by a human after fact-checking scripts/<slug>.md against match_data.json
+ sources.json); absence = hard stop, not a warning. The check runs before the
voice-exists short-circuit so an unverified-existing voice is not reused. A
warning that does not stop is not a check.
Prevention wired 2026-09-13 (multi-layer, not just the gate): (1) the
.script_verified gate in step6_voice; (2) script_gen.py SYSTEM prompt constrained
to facts in its facts block, and the goal events (scorers + minutes) are now
included in facts_blob so the generator cannot invent a scorer; (3) sources.json
built for the episode (managers, nationalities, former clubs cited — Thomas Frank
correctly flagged as NOT Brentford's manager, removed); (4) EPISODE_SPEC §10 makes
the source-of-truth rule a spec MUST. The fabricated-scorer class of failure
needs all four: a gate catches a bad script, but the constraint stops it being
written.

### B2 is the record, local is the working copy (2026-09-13, binding)
rclone v1.75.1, remote `b2`, bucket `mendymax-archive` (verified `rclone lsd b2:`).
Standing rule: everything produced by any task — renders, frames, transcripts,
artifacts, backups — is archived to B2. Local copies are working copies, not the
record. Doctrine rule 1 (raw footage → /mnt/f → pod → delete locally) is
unchanged; this rule extends it to generated outputs. Renders were landing
inside /home/muads because produce_v2.py:38 `RENDERS = SCRIPT_DIR/"renders"` —
generated outputs, not raw footage, so rule 1 did not cover them; the disk
filled anyway. Fix: archive old renders to B2 + delete locally, keep only the
current working render; the vhdx is compacted separately (Mayo, with WSL
stopped). Nothing is permanently deleted without either a B2 archive or a
one-line command that regenerates it.

### B2 provenance manifests (2026-09-13, binding)
Every artifact archived to B2 carries a manifest (who made it, when, with what,
for which task). An artifact without provenance is not archived. Bucket layout:
mendymax-archive/<project>/<yyyy-mm-dd>/<tagged-artifact> + .manifest.json.
produced_by names the AGENT (glm-5.2:cloud via Claude Code / claude.ai / gemini /
human), not the person; inputs carry dates so stale-input reuse is visible in
the archive. Enforcement tool: tools/b2_archive.py (built + tested 2026-09-13);
nothing goes to B2 by bare rclone copy after it. Backfilled manifests for the
pre-provenance uploads (~/yt-digest code snapshot, sep5 tar.gz backups, old
render folders) with produced_by="unknown (pre-provenance)" where genuinely
unknown — not guessed. Recorded in the global ~/.claude/CLAUDE.md (machine-wide)
+ here. The 3D-vs-2D question is RESOLVED AS 2D by the reference-viz extraction
(3D distorts 2D positional data; timelines inherently 2D); board build order:
#1 xG flow chart (2D timeline), #2 shot map (2D top-down), #3 momentum chart
(2D timeline), #4 average-positions scatter (2D top-down) — all 2D, all from
Sofascore data we have. Four design properties adopted as the standard boards
are judged against: direct labels not legends; three-colour discipline on a
dark background; full-frame use of space; brightest marks lead the eye +
title-as-message (trace: ~/claude/40-lessons/viz-design-findings.md §4).

### Briefs and reports are files, not messages (2026-09-14, binding standing order)
Every brief Mayo pastes is written verbatim (unedited, unsummarised) to
~/claude/30-briefs/YYYY-MM-DD-HHMM-<slug>.md before work starts. Every report
is written to ~/claude/20-handoffs/YYYY-MM-DD-HHMM-<slug>-report.md when work
ends, naming the brief file it answers in its first line. push_status.sh is run
explicitly after both (not relying on the Stop hook, which produced a 3-day
silent gap before). Any decision Mayo makes mid-task goes into DECISIONS.md with
the date before the work continues. Rationale: the coordinator reads the public
mirror; when brief + report are files, the brief can be checked against the report
by reading, not by trusting a pasted summary.

### Board build order + 3D-vs-2D resolved as 2D + four design properties (2026-09-13, binding)
Source: the reference-viz extraction (~/claude/40-lessons/viz-design-findings.md,
49238 bytes, verified `ls -la`). Gemini (authoritative) judged 10 design-focused
tutorial videos frame-by-frame; the four recurring design properties and the
build order below are taken from those verdicts, not re-derived here.

(1) BOARD BUILD ORDER (APPROVED). Four boards, all 2D, in this order:
- #1 xG flow chart (~50 lines, 2D timeline, HIGH narrative value, data=shotmap).
- #2 shot map (~150 lines, 2D top-down pitch, HIGH, data=shotmap).
- #3 momentum chart (~30 lines, 2D timeline, MED-HIGH, data=momentum).
- #4 average-positions scatter (~70 lines, 2D top-down, MED, data=avg-positions).
All four are fully supported by Sofascore data (shotmap, momentum,
average-positions endpoints documented in tools/sofascore_client.py:11;
average-positions is the only one currently fetched, at sofascore_client.py:290).
The extraction confirmed the shotmap and momentum endpoints exist in the
Sofascore API; wiring the fetches is build work, not a data-availability risk.

(2) 3D-vs-2D RESOLVED AS 2D. Part 1 of the extraction found 3D perspective
distorts 2D positional data (Opus: "perspective is the wrong tool for positional
data; foreshortening makes vertical spacing non-comparable"). Timelines (xG
flow, momentum) are inherently 2D. Positional charts (shot map,
average-positions) are 2D top-down. This confirms the Stage 12 "design not
dimension" verdict from the opposite direction: the extraction's evidence
(10 reference videos, Gemini-judged) says 2D is the right dimension for every
board on the build list. 3D is not the bottleneck; design is. This does not
edit the scrap-2D/build-3D decision above (don't-edit-old-entries convention);
it records the new evidence and the resolved direction for the build order.

(3) FOUR DESIGN PROPERTIES (the standard boards are judged against, from the
extraction's Gemini verdicts):
- (a) Direct labels on data marks, not a separate legend.
- (b) Three-colour discipline on a dark background: background + one accent +
  white. (McKay Johns: "exactly three colours — background + red + white";
  colour discipline scores 8-9/10 with this rule, 4/10 with 4+ default
  matplotlib colours, 1/10 when the accent matches the background.)
- (c) Full-frame use of space: the board fills 1920x1080, not a notebook or
  surrounded by UI chrome (space scores 7-9/10 full-frame, 3-6/10 when margins
  steal real estate).
- (d) Brightest/most-clustered marks lead the eye; the title is the largest
  text and states the argument (title-as-message, not a label). The eye goes
  to (1) the brightest/most-saturated colour block, (2) the largest text,
  (3) the densest cluster of marks. Make the focal data the brightest and most
  clustered thing, and the title the largest text.

CLOSING PASS: after-numbers not needed for this entry (no deletion or disk
claims here). The B2 entry above carries the after-numbers debt.

### STANDING RULE — learn from the best before designing (2026-09-14, binding)
Before executing any task that involves a design, a format, a structure or a
method, first study how the people who do it best actually do it. Sequence:
(1) check ~/claude/40-lessons/viz-design-findings.md (the reference-viz study);
(2) if not covered, extract transcripts + frames from the best practitioners;
(3) scour beyond given sources (cap 5, report why); (4) infer the principle,
not the artefact (never reproduce a design/layout/frame); (5) report before
building. Applies to visual design, script structure, pacing, sourcing,
tooling, workflow. If nothing relevant exists, say so explicitly + proceed.
Skipping the step is not valid. Recorded in CLAUDE.md + here as binding.
### 2026-09-14 — close stale-voice hole + settle 2D-vs-3D + Gemini litmus (brief 0610)

Task briefed: ~/claude/30-briefs/2026-09-14-0610-close-stale-voice-and-settle-two-questions.md
Three pieces, in order: (1) re-render voice from the fixed, .script_verified
script and prove the audio no longer names a fabricated scorer; fix the
step6_voice skip logic so "voice file exists" is not a valid skip condition
(compare the script hash recorded at voice-gen time vs the current script
hash, regenerate on mismatch). (2) Research what medium real practitioners
use for each football tactical board type, and settle the 2D-vs-3D
contradiction on record (STATUS.md 09-13: non-buggy 3D 6/10 vs 2D control
4/10, 3D ahead by 2; handoff: 2D beats 3D because momentum 9/10 beats
formation 3D 6/10 — those are unlike board types). (3) Litmus head-to-head:
Gemini vs scoreboard scanner on timestamps + pitch coordinates + cut
decisions, vs match_data.json. Trigger met: episode scored 5.5/10 < 7.

State at start (verified 2026-09-14):
- STATUS.md stamp 2026-09-13 (predates the 5.5 verdict + voice failure;
  must be updated to 2026-09-14 under the STAMP RULE as part of piece 1).
- voice_elevenlabs.mp3 mtime 2026-09-13 02:16; .script_verified mtime
  2026-09-13 20:33; scripts/<slug>.md mtime 2026-09-13 20:33. The voice
  predates the fix + the gate by ~18h — it was TTS'd from the pre-fix
  script, then reused on every re-run because step6_voice skips on
  voice_path.exists() (produce_v2.py:691-694). The gate runs before the
  short-circuit (:684-690) but only checks the marker exists, not that the
  voice matches the current script.
- Fixed script spoken word count (clean_script_for_tts): 1819 words (the
  prior report's "1751" was approximate; 1819 is measured). Pre-fix was
  ~1911. Old voice 719.1s @ ~159 WPM. New voice for 1819 words expected
  ~560-705s.
- match_data.json goals (ground truth for piece 3): Schade 34' (0-1),
  Kluivert 38' (1-1), Tavernier 52' (2-1), Schade 56' (2-2). Both Brentford
  goals = Schade. The fabricated pre-fix claim was "Igor Thiago needed just
  one big chance to score"; the fixed script attributes both to Schade and
  names Igor Thiago only as the striker.

### 2026-09-14 — Stage 15 outcomes (brief 0610)

Mid-task decisions (dated before work continued, per the brief):
- **Fix-then-render order.** The brief listed re-render before fixing the skip
  logic. I reversed it: fix the hash gate first, then render once. Reason: the
  fix records the script hash at voice-gen time; render-then-fix would leave no
  hash recorded and force a second ElevenLabs render on the next step6_voice
  call (re-assemble calls step6_voice again). Fix-then-render = one render,
  hash recorded, no wasted TTS cost. One change, one run.
- **§7 WPM regression accepted as a finding, not silently "fixed."** The new
  voice is 137 WPM (below the 155 MUST) because ElevenLabs paced the 1819-word
  TTS slower than the 1911-word pre-fix voice (159 WPM). This is an ElevenLabs
  pacing variance, not a script-content issue. The re-score reports it honestly
  (§7 0.5/2). The §10 credibility disqualifier is resolved (the freeze's
  reason is gone); the episode is 7.5/10 numerically. Recommendation: regenerate
  at an ElevenLabs speed setting that hits 155+ WPM before declaring the freeze
  lifted. Not done in this task (one change per run; the change was the hash
  gate).
- **Gemini delegation.** The litmus (Piece 3) is decisive: Gemini loses to the
  mechanical tools on every mechanical output (timestamps 2/4 + 1 phantom vs
  scanner 4/4 free; coordinates hallucinated with 3 wrong jerseys; cut
  decisions wrong scorer + no filtering). Delegate to Gemini ONLY for content
  classification (what kind of passage), not for when/where/what-numbers. The
  2026-09-08 finding holds on a second clip.

Outcomes (verified):
- Voice: from the fixed script (sha256 a4a70df3... matches), 794.676s, gate
  precedes voice. Skip logic fixed (hash gate, hashlib). Old voice moved to
  voice_elevenlabs_pre_fix.mp3.
- Re-score: 7.5/10 (16.5/22), §10 RESOLVED, §7 regressed (137 WPM).
- 2D-vs-3D: neither on-record claim was like-for-like; "DESIGN not DIMENSION"
  survives; industry standard for positional boards is 2D top-down (one
  sponsored exception: Coaches' Voice + Football Manager). A like-for-like
  same-board-same-data 2D-vs-3D test has never been run.
- Gemini litmus: timestamps scanner 4/4 free vs Gemini 2/4 + 1 phantom paid;
  coordinates Gemini hallucinated; cut decisions Gemini wrong scorer. Gemini
  wins nowhere on mechanical output.

STATUS.md updated to 2026-09-14 (STAMP RULE) with the 5.5 verdict, the voice
failure, the fix, the re-score, and the litmus (Stage 15 section).
Handoff: ~/claude/20-handoffs/2026-09-14-0610-close-stale-voice-report.md.

### gemini-pro-latest is the authoritative judge (2026-09-16, binding)
gemini-3.1-pro-preview was a PREVIEW endpoint. Replaced with gemini-pro-latest
(stable alias, same Pro tier). 3-run spread on 9 unchanged boards: max 1.0 point
(no board > 2). The 10/10 → 7/10 swing on xg_flow was BETWEEN models/sessions,
not within the same model. A single gemini-pro-latest score is stable within 1
point. DEFAULT_MODEL in tools/gemini_judge.py updated. gemini-2.5-pro and
gemini-2.5-flash are DEPRECATED (404 for this key). gemini-pro-latest accepts
both video and image input (verified).

### Gemini on raw clip: 4/4 goals, correct scorers (2026-09-16)
On the raw 792s Bournemouth-Brentford clip (360p h264 uploaded via Files API),
gemini-pro-latest found all 4 goals, 0 phantoms, 0 misses, all scorers correct
(Schade 274s, Kluivert 333s, Tavernier 477s, Schade 520s). Offset vs scanner:
+1 to +6s. The previous "Mbeumo" error was from the ASSEMBLED episode (which is
all boards, no match footage), not from Gemini watching the raw clip. Gemini's
value is content classification + video understanding on raw footage; it is NOT
a timestamp/cut-decision tool (the scanner wins on precision, Gemini wins on
scorer identification from on-screen captions).

### 3D positional boards: replace scene_gen.py with Three.js (2026-09-16, recommendation)
Decision taken by Mayo: positional boards go 3D. scene_gen.py (Blender on Modal)
is NOT the right tool: it silently broke (Stage 12, cause unknown), costs ~$0.10
and ~600s per render, is capped at 1280x720, and cannot use the shared design
layer (board_design.py is matplotlib-only). RECOMMEND: replace with Three.js in
headless Chromium (Puppeteer). Reason: deterministic (no silent breaks), $0 per
render, 1920x1080 native, CSS @font-face for Bebas Neue / Barlow Condensed (same
fonts as the 2D boards), ~5-10s render time. Build effort: ~150-200 lines JS.
