# CONTEXT.md — soccer-channel session brief

> **Purpose:** the session brief — goal, where things live, what's broken, measured facts.
> **Reader:** every session (CLAUDE.md @CONTEXT.md).
> **Last verified against code:** 2026-10-03 (brief 15: card types + audits
> grepped in board_page.html/board_html.js, oracle re-runs green; brief 11
> staging/cache-path facts below unchanged and still true).

Rewritten 2026-09-22 (brief 03) against the code on disk. Read STATUS.md for
the verified state spine and DECISIONS.md for why things are the way they are.

## THE GOAL
Produce soccer tactical analysis videos for a YouTube channel that meet the
owner benchmark (CLAUDE.md §1): high-level match analysis from highlight
clips, insights of the play, much infotainment, graphs and editing that an
audience reads as really capturing the game. Engaging.

## WHERE THINGS LIVE
The project is /home/muads/yt-digest/soccer-channel. Nothing else.
~/retired/ is off limits. Never read or run anything there.

- .env at ~/yt-digest/.env. Canonical YOLO weights at ~/yolov8s.pt.
- Episode inputs: scripts/<slug>.md ([VISUAL: board=/footage=] contract),
  briefs/<slug>/sources.json (source-of-truth registry), renders/<slug>/
  (board_specs/, boards/, final_video.mp4, judge + report artifacts).
- B2 archive of record: rclone remote "b2", bucket "mendymax-archive",
  prefix soccer-channel/<date>/, manifests via tools/b2_archive.py.
- Modal volume "soccer-build" holds /vol/in (reel, voice, ambience, script),
  /vol/specs, /vol/out (boards + final), /vol/tools.
- /mnt/d = raw-footage staging + all caches, at /mnt/d/scratch/ (scratch SD
  card, Disk 1, NTFS; moved off the retired F: stick by brief 11 on
  2026-09-28; remount helper tools/scratch_mount.sh via passwordless
  /etc/sudoers.d/d-mount). F: (Memorex stick, Disk 2) is RETIRED: writing to
  it is a disk-guard failure. Nothing raw or heavy is written inside
  /home/muads (ext4.vhdx grows and never shrinks).

## THE PIPELINE (flash era, brief 06)
Entry: tools/produce_v2.py — `--flash` runs the flash lane in one command
(facts → data check → script-referenced boards → Modal assemble + pull);
tools/pod_build.py is the Modal build it drives. render2d/render3d take
--slug: specs at /vol/specs/<slug>/, boards AND the final at
/vol/out/<slug>/; a spec whose slug contradicts the episode is REFUSED.
render2d = board_html.js driving board_page.html in headless Chromium (the
ONE 2D renderer) and it AUDITS the drawn page against
/vol/specs/<slug>/match_data.json before writing frames — a wrong number or
a wrong side fails the render (brief 06 job 4); an episode render with no
facts file is refused. render3d = three_scene_v2.js (the ONE 3D engine).
assemble cuts footage + loops boards + mixes voice/ambience with a retime
pass, writes /vol/out/<slug>/assemble_manifest.json (the actual cut list),
and auto-pulls the final home. FACTS: renders/<slug>/match_data.json is
tracked (input of record) and tools/board_data_check.py (produce_v2 step 1d,
gates B16/B18) checks EVERY number in EVERY spec against the RAW Sofascore
responses cached at renders/<slug>/sofascore_raw.json (also tracked) — every
spec carries source + fetched_at; a spec with no source is a fact nobody
fetched. Sofascore (via tools/sofascore_client.py) is the only fact source,
for every competition; ESPN match_data.py is retired.
The long lane (produce_v2 without --flash) keeps the local
download/assemble/merge/shorts steps; scripts are keep-if-exists (a tracked
script is never regenerated).

Footage windows are cut from the CBS reel at verified timestamps
(cut_list_flash.json lineage). The reel's CBS Sports / @CBSSPORTSGOLAZO /
Paramount+ watermarks are accepted (CLAUDE.md §3).

## MODELS
- glm-5.3-flash:cloud (Ollama) is the only model: session, judge
  (tools/glm_judge.py), and vision (multimodal — read frames directly with
  the Read tool; do not reason about footage you have not looked at).
- Gemini is RETIRED (key 402 since 2026-09-21; gemini_judge.py is in
  retired/ as a record). Cross-checks if ever needed: ~/tools/ask_claude.py
  --image (paid) and ~/tools/vision_analyze.py (gemma4, free).
- Judge wobble is a model property (6,8,6,6,8 over 5 runs at temperature 0 +
  seed 0, 2026-09-23). Judge load-bearing scores with --runs 5 and use the
  median (glm_judge.py implements it).
- Stats board team sides: home LEFT, away RIGHT everywhere (bar, header,
  rows). GATE: reports/brief05/ board pairs prove the brief-05 fixes.

## WHAT IS BROKEN, WORST FIRST
See STATUS.md "What is broken" — judge wobble, footage-overrun freezes
(4 sections, 0.1-1.1s), public-mirror content risk, unclaimed possible
disallowed goal at reel t=90, migration phases 5-8 unfinished.

## MEASURED FACTS, DO NOT RE-DERIVE
- EP001 (Real Madrid 2-1 Inter, 2026-09-08) final: 140.96s, 75.6MB,
  1920x1080 (ffprobe verified 2026-09-27, v7). Voice 155.5 WPM (ElevenLabs).
- Sofascore retires old events' sub-endpoints (event 16363640 served empty
  bodies 15 days post-match; 16938768, older, served everything). Cache the
  raw responses at fetch time — renders/<slug>/sofascore_raw.json is the
  only durable copy. /statistics has ALL/1ST/2ND periods; 2ND must never
  overwrite ALL.
- Output boards: render3d for formation/counter/goal-pattern/strike boards,
  render2d (audited) for stats/possession, momentum, outro. All specs in
  renders/<slug>/board_specs/, tracked, each with source + fetched_at.
- Scoreboard scanner (gemma4 OCR of the score bug) is the ground truth for
  goal timestamps; LLM video reading invents phantoms and misses goals
  (measured Stage 15). Use the scanner for WHEN; use the session model for
  WHAT (content classification).
- The 2026-09-08 match facts your training data gets wrong: Mourinho manages
  Real Madrid; Dumfries plays right back FOR REAL MADRID; Alexander-Arnold
  played midfield; Chivu manages Inter (see CLAUDE.md §9).
- BALL DETECTION IS DEAD on compressed wide-shot reuploads (8/12 frames no
  ball). Anchor to players only.
- Local speed: no GPU work on the N150. All heavy compute on Modal
  (assemble wall ~5-7 min; board renders ~2-12 min each).
- Animated boards (brief 12, 2026-09-28): kind "timeline" in the ONE 2D
  renderer — a spec of steps (reveal/highlight/number/line/zone/ball_path/
  caption/hide) played deterministically (state = f(spec, T)), audited at
  every step end vs the facts. Format doc: docs/board_timeline.md. Sofascore
  has NO pass/possession sequences (7 candidate endpoints probed, all empty —
  reports/brief12/probes.txt), so a traveling ball is SCHEMATIC and labeled
  on the board. Timeline renders measured: 8.5s board = 108s Modal wall,
  9s = 54s, 10s = 88s; brief-12 Modal window cost $0.0182 (billing report).
- Akamai can 403-challenge every curl_cffi impersonation for 40+ min
  (residential AND Modal egress, measured 2026-09-28). The real cached
  Chromium passes: fetch_sofascore falls back per round to
  tools/sofascore_chrome_fetch.js. /team/{id}/events/last/0 carries no
  "team" key — verify a team id by the event payloads.
- modal volume put remote paths are RELATIVE to the volume root: an
  absolute "/vol/tools/x" lands in a nested vol/ tree the functions never
  mount (cost brief 12 two failed smoke renders).
- Board sizing (brief 13, 2026-09-28): card 1760px = 91.7% of 1920 (floor
  88%), pitch fills the card; per-keyframe AUDITED minima at 1080p: disc
  r>=42, chip r>=26, label 36px (the 480p floor at 2.25x downscale); the
  audit also enforces chip-vs-label and label-vs-disc non-overlap. Spec
  field "frame" auto-frames the half the ACTIVE players occupy (jersey-
  targeted by any step; runners board -> right-half).
- Freeze-frame overlays (brief 13, 2026-09-28): kind "footage_overlay" —
  base {clip, freeze_at} + image-space steps (ground_ellipse / chip / link /
  unit / arrow at 1920x1080 pixel coords); play-in, freeze, animate, hold,
  fade, resume as a pure function of (spec, T) with pod-side ffmpeg
  pre-extracted frames (no <video>). Identity gate: a chip naming a player
  needs source "lineup:<jersey>" and that jersey in a facts XI (either
  team; a jersey can exist in BOTH XIs, 14 = Adeyemi AND Gueye). Marked
  points + plans live in reports/brief13/marks.json; the specs are emitted
  (brief13_overlays.py), never hand-typed. Drift control: drawn ellipses
  are compared to the marks; a 150px shift FAILS (gate B65). Overlay render
  on Modal: ~6-8 min wall each (121 ffmpeg extracts + 315 page frames).

## CONTENT RESEARCH (hand-run, kept in brief 03)
- tools/viral_angle.py — trending topics with viral potential (YouTube API).
- tools/agent_reach_research.py — multi-platform fan-out research.
- tools/fresh_fetch.py — dated news fetch with age stamps (kills stale
  rumor classification).

## STANDING RULES
1. Work only in /home/muads/yt-digest/soccer-channel. Print pwd at the
   start of every response.
2. Nothing heavy on the local N150; Modal (or RunPod) for >2min work.
   Raw footage: residential download → /mnt/d staging → pod → delete
   locally. Never raw footage to YouTube (CLAUDE.md §3).
3. Every factual claim names the command you ran and pastes its output.
   If you did not run a command, write "not checked."
4. Never print full API keys. Mask them.
5. Git is the backup. Commit (with push) before any cleanup; there is no
   Trash (CLAUDE.md §2 gives standing approval). Exception: scripts/ is
   gitignored, so episode scripts exist only locally + on the Modal volume
   (GAPS #4).
6. One change, one run, one verification. Never batch changes.
7. Ask before guessing on unclear paths, but do not stop when a path is
   blocked — name the alternatives, pick the best, keep going (CLAUDE.md §7).
8. Batch independent tool calls in one response.
9. Use run_in_background for any command expected to take >2 minutes.
10. Session-start hooks: check_pods.sh (pod leaks), check_scratch.sh (D:
    mount; auto-remounts via scratch_mount.sh --fix on guard failure,
    brief 11 job 7), check_doc_stamps.sh (stale canonical docs). If the
    pod check warns,
    terminate with tools/pod_check.py --terminate.
11. Canonical docs (CLAUDE.md, CONTEXT.md, STATUS.md, DECISIONS.md,
    PROGRESS.md, GAPS.md, TOOLS.md, ARCHITECTURE.md, EPISODE_SPEC.md,
    LANE_PLAN.md, SCRIPT_TEMPLATE.md, GATES.md, SKILLS.md) carry a
    "Last verified against code" stamp. Any task that edits one must
    either re-verify its claims and move the stamp, or state which claims
    could not be verified and leave the stamp. Never set a stamp you did
    not earn.
12. settings.json hook writes are ADDITIVE. Never Write a fresh settings.json;
    read the current file and merge. (2026-09-22: global block-image-read.sh
    hook deleted — it was a no-op written for a text-only model; the
    model-dependent image policy lives in CLAUDE.md / the global CLAUDE.md.)
13. B2 is the archive of record. Everything produced is archived with a
    manifest before local deletion; local copies are working copies.

## DOCTRINE FOOTER
1. Raw footage is acquired over the residential connection, staged on
   /mnt/d, shipped to the pod, and deleted locally. Nothing raw is written
   inside /home/muads. No local GPU work.
2. No tasking without calling the available tools, skills, connectors, web
   fetch, research or MCPs where they apply.
3. No orphaned, standalone or untested item, tool or aspect of the pipeline
   may exist. Everything is wired, tested, or retired. (Brief 03 retired the
   25 files that violated this; the census lives in STATUS.md.)
4. Every brief carries this doctrine as a footer.
5. NO DEAD ENDS. When a path blocks the work, name the alternatives, pick
   the best available, and keep going.
6. "Done" requires --reverify output that post-dates the last code commit
   and is published to the mirror. --status is never proof. Never state an
   uncounted count. (Brief 08 rule 7; the brief 06 false-pass is the
   standing example.)