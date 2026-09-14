# CONTEXT.md — soccer-channel session brief

> **Purpose:** the session brief — goal, where things live, what's broken, measured facts.
> **Reader:** every session (CLAUDE.md @CONTEXT.md).
> **Last verified against code:** 2026-09-13.

Verified against code on 2026-09-13 (Stage 12B: produce_v2.py rewritten 858 lines, cut-list + assembler wired, validate_script wired; Stage 14: tactical_render.py + step4b RETIRED, 3D formation board wired into step2_boards, .script_verified voice gate, step5_assemble caps footage at 20%, 3rd SessionStart hook check_doc_stamps.sh, tools/sofascore_client.py + tools/doc_stamp_check.py added; B2 migration: rclone v1.75.1 verified, b2:mendymax-archive verified, ext4.vhdx 36.82 GB verified, /mnt/f EINVAL verified, vault ~/claude verified, .env + yolov8s.pt paths verified). Earlier verification 2026-09-09 (Stage 2: 720p cap, duration filter; Stage 3: broadcast_filler + produce_episode fixed). See ARCHITECTURE.md / STATUS.md.

Read this at the start of every session instead of pasting the brief.
The project is /home/muads/yt-digest/soccer-channel. Nothing else.

## THE GOAL
Produce soccer tactical analysis videos for a YouTube channel, matching
the quality of Coaches' Voice (dark pitch, orange accents, data-driven
graphics, functional arrows). Not Tifo, which needs a human illustrator.

## WHERE THINGS LIVE
The project is /home/muads/yt-digest/soccer-channel. Nothing else.
~/retired/ holds two dead folders (soccer-pipeline, soccer-channel).
Never read or run anything in ~/retired/.
There used to be two folders named soccer-channel. That caused weeks of
confusion. Only the one inside yt-digest is real.

The .env is at ~/yt-digest/.env. yolov8s.pt is at ~/yolov8s.pt, outside
the project folder.

B2 archive of record: rclone v1.75.1, remote "b2", bucket
"mendymax-archive" (verified `rclone lsd b2:`). NEW STANDING RULE:
everything produced by any task (renders, frames, transcripts, artifacts,
backups) is archived to B2; local copies are working copies, not the
record. Doctrine rule 1 (raw footage -> /mnt/f -> pod -> delete locally)
is unchanged; this extends it to generated outputs.

/mnt/f = raw-footage staging. Writes are currently EINVAL (`touch /mnt/f/x`
fails with "Invalid input"), so /dev/shm is the volatile fallback (RAM,
no vhdx bloat, cleared on reboot).

The vault at ~/claude (mirrored to the public status mirror by push_status.sh
via the $VAULT/10-projects..40-lessons dirs; see STATUS MIRROR below).

Public status mirror: github.com/minakush000-crypto/soccer-channel-status
(see STATUS MIRROR section below for the dual-allowlist constraint).

## DISK REALITY
ext4.vhdx is 36.82 GB (39530266624 bytes at
C:\Users\muads\AppData\Local\wsl\{f2ea779f-e5f1-4c82-a1b2-0608e6ab4883}\
ext4.vhdx). It grows and never shrinks on its own (deleting inside Ubuntu
does not shrink it); compaction requires WSL fully stopped (a separate
operation Mayo runs). C: has ~15 GB free of 119 GB; / shows ~29 GB used of
1007 GB after cache clearing. CLOSING PASS: free-space-after-deletions,
what was deleted, and bucket size (the archive-then-delete in progress
fills these).

## STATUS MIRROR (public, active)
Current project state is mirrored to a PUBLIC docs-only repo so any session
(or the coordinator) can read it via web fetch instead of pasting fragments.
Repo: https://github.com/minakush000-crypto/soccer-channel-status
A Stop hook (.claude/hooks/push_status.sh) re-pushes an ALLOWLIST of safe doc
files (CONTEXT/STATUS/PROGRESS/GAPS/DECISIONS/TOOLS/ARCHITECTURE/EPISODE_SPEC/
LANE_PLAN/GATES/SKILLS + a ~/claude/vault dir) after any session end. No code,
no keys, no renders, no .env (the script deletes any .env it finds). The mirror
repo lives at ~/soccer-channel-status (a separate working clone).

HISTORY: the mirror was RETIRED 2026-09-12 (Stage 11, rule 3) as redundant +
fragile (Stop-hook-only; a 3-day gap 2026-09-09 to 2026-09-12 when crashed
sessions did not fire the hook), and push_status.sh was git-rm'd. It was
RE-ENABLED the same day (2026-09-12 ~22:39): push_status.sh re-created on disk,
re-registered as a project Stop hook, and `git add`'d back (un-retired). The
docs were updated to match (this section). The artifacts/ and frames/ subtrees
remain in this project (gitignored, local) for Claude's own use and are NOT in
push_status.sh's ALLOW_PROJECT list (so never copied to the mirror). The mirror
.gitignore no longer un-ignores artifacts/ or frames/ (removed 2026-09-14);
they were previously tracked from old pushes but have been git-rm'd from the
mirror and the un-ignore lines removed.

DUAL ALLOWLIST CONSTRAINT (standing — a file must pass BOTH or it vanishes
silently with no error):
1. push_status.sh carries an ALLOWLIST of files it copies into the mirror
   (CONTEXT/STATUS/PROGRESS/GAPS/DECISIONS/TOOLS/ARCHITECTURE/EPISODE_SPEC/
   LANE_PLAN/GATES/SKILLS + the ~/claude/vault dir). A file not in this list
   is never copied.
2. The mirror repo's own .gitignore carries a SECOND allowlist: it default-
   denies everything (`*`) then un-ignores specific files/dirs (!CONTEXT.md,
   !STATUS.md, ... !vault, !vault/**, etc.). A file
   that IS copied but is NOT un-ignored by the mirror .gitignore is gitignored
   → `git add` silently skips it → it never reaches the remote, with NO error.
So: to add a new doc to the mirror, add it to BOTH push_status.sh's
ALLOW_PROJECT list AND the mirror .gitignore's un-ignore list. Do NOT remove
the `!vault` or `!vault/**` lines (the vault dir is un-ignored by name; if
either is dropped, vault contents stop syncing). Defense-in-depth: the mirror
.gitignore also blocks *.mp4/*.mov/*.env/*.key/*.pt/*.pth/*.bak/*.cookies.txt
even inside allowed dirs.

The known fragility (Stop-hook-only, gap on crashed sessions) is accepted: the
mirror is a convenience, not a source of truth — the private yt-digest repo
(synced via the GitHub connector) is canonical, and this project's own files
are the live state. A crashed session simply means the mirror may lag until the
next clean Stop; it never means the mirror is wrong about what it does contain.

## THE PIPELINE
Entry point: tools/produce_v2.py
Run it with: ~/yt-digest/.venv/bin/python tools/produce_v2.py <slug>
  --query "<match>" --date-range YYYYMMDD-YYYYMMDD
Working example: 2026-08-30_liverpool-forest --query "Liverpool Forest"
  --date-range 20260801-20260831

12 steps (verified 2026-09-13, see ARCHITECTURE.md): match data (ESPN)
-> script gen (glm-5.2:cloud) -> validate_script -> boards (2D + 3D formation
via Modal) -> download clip (720p cap + 200MB guard, 3-20min filter) -> cut
list (broadcast_filler + cut_list_gen) -> assemble (footage capped at 20%)
-> transformation gate -> voice (ElevenLabs) -> crowd ambience -> merge ->
shorts crop. Step4b tactical render RETIRED Stage 14 (tactical_render.py
deleted; runpod_fulltrack.py still exists for standalone tracking). Latest
produce_v2 output: 720x1280, 62.3s, 24.8MB (liverpool-forest, Sep 7). Source
is now 720p (Stage 2: height<=720 at produce_v2.py:218; duration 180-1200s
at :179).
A separate words-match path (assemble_words_match.py) built arsenal-chelsea:
1280x720, 45.2s, 15.0MB (Sep 8) (file overwritten; now 1920x1080, 724.0s,
210MB).

tools/produce_episode.py is an older entry point. It calls cv_annotate.py
at line 298. Do not break that. It is NOT reachable from produce_v2.py.

## WHAT WAS FIXED (verified)
FIX 1: produce_v2.py was downloading 360p clips and accepting them
silently. It now loops up to 5 candidates, ffprobes each, rejects below
720p, uses cookies from secrets/yt_cookies.txt, and fails loudly rather
than falling back. Source is now 720p (Stage 2 cap, height<=720 at
produce_v2.py:218). The old "1920x1080" claim was from the pre-Stage-2
fix; Stage 2 later capped to 720p.

## WHAT IS STILL BROKEN, WORST FIRST
1. Footage cuts previously ignored narration. Stage 12B replaced the old
   fixed 5-second offsets (`clip_idx*5` at the former :433 — REMOVED) with
   a cut-list-driven assembler (step4a_cutlist + step5_assemble). The
   [VISUAL: footage=] tag is now parsed as a window at produce_v2.py:472
   (`is_footage`), not just a boolean. cut_list_gen.py IS wired (step4a,
   :371) and broadcast_filler IS wired (:374). Remaining gap: cut-list
   quality depends on Gemini inventory accuracy (see STATUS.md).
2. Script validation IS now wired. `grep -c validate_script
   tools/produce_v2.py` -> 6 (step1c_validate at :353-366). The old
   "Gravenberch receives" example was from a pre-validation draft.
3. Output is 720x1280. shorts_crop.py downscales deliberately to avoid
   a soft 1.78x upscale to 1080x1920.
4. No upload step in produce_v2.py. youtube_upload.py is invoked by hand;
   2 private uploads happened 2026-09-06 (artifacts/publish-log/ has 2
   entries). produce_v2 does not wire upload.
RESOLVED (was #1): the tactical_overlay "PowerPoint clipart" problem is
gone. produce_v2.py no longer calls tactical_overlay.py (grep exit 1).
tactical_overlay.py is DEAD. step4b_tactical_render is also RETIRED
Stage 14 (tactical_render.py deleted; the 3D formation board via
scene_gen/modal_render3d in step2_boards now covers the visual layer).
runpod_fulltrack.py still exists for standalone tracking but is not
called from produce_v2.py.

## MEASURED FACTS, DO NOT RE-DERIVE
- cv_annotate.py (YOLOv8 + ByteTrack + KMeans team classification) works
  and produces real tracking. A vision model called its output
  "professional broadcast tracking." produce_v2.py does NOT call it locally;
  runpod_fulltrack.py ships it to RunPod and runs it on the pod (line :101
  ship list, :120 run command). runpod_fulltrack.py is NOT called from
  produce_v2.py after Stage 14 retirement of step4b.
- Local speed: 1.30s/frame at 1080p. 900 frames = 19.5 min. Full clip
  = 95 min. Too slow.
- roboflow/sports was evaluated and REJECTED. Its football-specific
  model found the ball in 4 of 12 frames vs 3 of 12 for generic COCO.
  Not a meaningful gain. Its PLAYER_DETECTION exceeded 2s/frame and its
  BALL_DETECTION with InferenceSlicer was 14x slower than baseline.
- BALL DETECTION IS DEAD. 8 of 12 frames had no ball from either model.
  The source is a compressed wide-shot reupload. Drop every
  ball-dependent feature: no ball trail, no ball-following crop, no
  arrows tied to the ball. Anchor to PLAYERS ONLY.
- cv_annotate.py exports team_assignment, ball_positions, AND per-frame
  player_positions (Stage 1 done 2026-09-05: {frame, players:[{id,bbox,team}]}).
  No path from tracker ID to player name exists (no jersey OCR, no mapping).
- Cloud: RunPod and Vast.ai both authenticate. runpod_annotate.py is
  RETIRED Stage 6 (deleted). runpod_fulltrack.py is the current tracking
  tool; it tarballs tools/ and ships it, so edits propagate automatically.
  Roughly $0.15 and 5 minutes per episode on an L4.
- The vision quality scores in the old STATE.md (7/10, 8/10, 9/10) were
  typed by hand. No code produces them. They are not a benchmark.
- Working vision check: /home/muads/tools/vision_analyze.py, ~2.75s per
  frame. Use it to judge output instead of asserting quality.
- JUDGE RULE (decided 2026-09-13, supersedes the prior Opus-authoritative
  rule in CLAUDE.md): Gemini (tools/gemini_judge.py, gemini-3.1-pro-preview)
  is the AUTHORITATIVE visual judge. Opus (ask_claude.py --image) and gemma4
  (vision_analyze.py) are cross-checks. On disagreement, record the Gemini
  score as authority and flag it; never write the more flattering number.
  Boundary: Gemini judges what it can see; it does NOT generate timestamps,
  pitch coordinates, or cut decisions (those stay mechanical).

## CONTENT RESEARCH
Three tools exist for topic research and freshness checks. None were
documented before 2026-09-05, which is why they stopped being used.

- tools/viral_angle.py — finds trending soccer topics with viral potential
  using the YouTube Data API. Searches for tactical analysis videos, then
  flags high-view videos from low-subscriber channels (proven demand, low
  competition). Run: ~/yt-digest/.venv/bin/python tools/viral_angle.py "topic"
- tools/agent_reach_research.py — multi-platform research layer. Fans out
  across exa (semantic web search), web (Jina Reader), github, twitter, and
  reddit. Degrades gracefully when a channel needs a login. Run:
  ~/yt-digest/.venv/bin/python tools/agent_reach_research.py "query"
- tools/fresh_fetch.py — freshness fetcher. Pulls dated sports news from
  RSS feeds and football-data.org API. Stamps every item with published
  time and age in hours. Exists because the old pipeline classified a
  completed transfer as a "rumor" using a stale article. Run:
  ~/yt-digest/.venv/bin/python tools/fresh_fetch.py

## THE BENCHMARK (from RESUME_RESEARCH.md lines 70-77 — file no longer exists)
1. Layered depth (drop shadows, gradients, z-ordering)
2. Desaturated pitch + high-contrast accents
3. Selective visibility (show only what matters)
4. Functional arrow language (tapered, Bezier, round caps)
5. Contextual cropping (zoom to the relevant zone)
6. Condensed athletic fonts (Bebas Neue, Barlow Condensed)
7. Dark muted palette
Current output violates 1, 3, 4, and 5.

## STANDING RULES
1. Work only in /home/muads/yt-digest/soccer-channel. Print pwd at the
   start of every response.
2. CLOUD ONLY. This is an Intel N150 with no GPU. Any GPU work goes to
   RunPod. Local is allowed only for ls, grep, ffprobe, single-frame
   checks, reading files, and edits. Anything over 2 minutes locally,
   stop and ask me first.
3. Every factual claim names the command you ran and pastes its output.
   If you did not run a command, write "not checked."
4. Never print full API keys. Mask them (rpa_J40...RGT8).
5. Git is the backup. The .bak files were deleted 2026-09-08 (superseded by
   git, RECONCILIATION Amendment 2). Do not recreate .bak; commit first if you
   want a rollback point.
6. One change, one run, one verification. Never batch changes.
7. Ask before guessing. If a required argument or path is unclear, stop
   and ask rather than assuming.

## DONE (2026-09-05 session 2)
- Documentation spine built: ARCHITECTURE.md, TOOLS.md, STATUS.md,
  DECISIONS.md, GAPS.md, SKILLS.md, PROGRESS.md.
- soccer-channel skill neutralized (both copies now RETIRED, no longer
  point at ~/soccer-pipeline).
- Option C Stage 1: cv_annotate.py exports per-frame player positions.
  Verified on RunPod (L4, 300 frames, 129 tracker IDs, 23s, $0.05).
  One-off runner at tools/runpod_stage1.py — RETIRED Stage 6 (deleted).
  (Pre-edit backups were .bak files, deleted 2026-09-08; git is the
  backup now.)
- Key finding: NO path from tracker ID to player name exists. No jersey OCR,
  no position heuristics, no manual mapping. match_data.py has jersey numbers
  but nothing reads them from video. Stage 2 is arrows on unnamed tracked
  players, or hand-mapped. Tracker fragmentation is high (129 IDs in 10s).

## NEXT TASK (2026-09-08)
The C+E prototype is done (tactical_render 8/8.5, full 146s tracking;
tactical_render.py RETIRED Stage 14 — deleted, historical assessment).
The lane expansion is planned in LANE_PLAN.md (four lanes A/B/C/D). Shared
prerequisite for every footage lane: a cut-list generator that emits
`[VISUAL: footage=START-END]` timestamp windows (scoreboard_scan + ±20s
relay-verified search) — no wired tool generates them today (RECONCILIATION
1.2). E iteration priorities from Opus remain open but are now lower than
the lane expansion and the cut-list generator.
See PROGRESS.md and STATUS.md for full detail.

## STANDING OPERATING DOCTRINE
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

The truthful gap list (which tools violate rules 1 and 3 today) lives in
DECISIONS.md under this same heading. Rule 1 is violated by the whole pipeline
(produce_v2 downloads + processes locally); rule 3 by 4 DEAD + ~18 untested
STANDALONE tools. Not yet fixed.