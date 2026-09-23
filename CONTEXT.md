# CONTEXT.md — soccer-channel session brief

> **Purpose:** the session brief — goal, where things live, what's broken, measured facts.
> **Reader:** every session (CLAUDE.md @CONTEXT.md).
> **Last verified against code:** 2026-09-22.

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
- /mnt/f = raw-footage staging (mounted, fstab drvfs). Nothing raw or heavy
  is written inside /home/muads (ext4.vhdx grows and never shrinks).

## THE PIPELINE (flash era)
Entry: tools/pod_build.py (Modal). render3d <spec> = three_scene_v2.js;
render2d <spec> = boards_2d.py; assemble = cut footage from /vol/in/reel.mp4
+ loop boards + mix voice/ambience → /vol/out/final_video.mp4.
tools/produce_v2.py remains the older end-to-end entry (ESPN → script →
validate → boards → download → cut list → voice → assemble → merge →
shorts). Both parse scripts/<slug>.md with the same [VISUAL:] contract.

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
- Judge wobble is ±1-2 on identical frames; judge load-bearing scores more
  than once and report the spread.

## WHAT IS BROKEN, WORST FIRST
See STATUS.md "What is broken" — judge wobble, footage-overrun freezes
(4 sections, 0.1-1.1s), public-mirror content risk, unclaimed possible
disallowed goal at reel t=90, migration phases 5-8 unfinished.

## MEASURED FACTS, DO NOT RE-DERIVE
- EP001 (Real Madrid 2-1 Inter, 2026-09-08) final: 140.92s, 76.5MB,
  1920x1080 (ffprobe verified 2026-09-22). Voice 155.5 WPM (ElevenLabs).
- Output boards: render3d for formation/counter/goal-pattern/strike boards,
  render2d (matplotlib) for stats/possession, momentum, outro. All specs in
  renders/<slug>/board_specs/.
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

## CONTENT RESEARCH (hand-run, kept in brief 03)
- tools/viral_angle.py — trending topics with viral potential (YouTube API).
- tools/agent_reach_research.py — multi-platform fan-out research.
- tools/fresh_fetch.py — dated news fetch with age stamps (kills stale
  rumor classification).

## STANDING RULES
1. Work only in /home/muads/yt-digest/soccer-channel. Print pwd at the
   start of every response.
2. Nothing heavy on the local N150; Modal (or RunPod) for >2min work.
   Raw footage: residential download → /mnt/f staging → pod → delete
   locally. Never raw footage to YouTube (CLAUDE.md §3).
3. Every factual claim names the command you ran and pastes its output.
   If you did not run a command, write "not checked."
4. Never print full API keys. Mask them.
5. Git is the backup. Commit (with push) before any cleanup; there is no
   Trash (CLAUDE.md §2 gives standing approval).
6. One change, one run, one verification. Never batch changes.
7. Ask before guessing on unclear paths, but do not stop when a path is
   blocked — name the alternatives, pick the best, keep going (CLAUDE.md §7).
8. Batch independent tool calls in one response.
9. Use run_in_background for any command expected to take >2 minutes.
10. Session-start hooks: check_pods.sh (pod leaks), check_mnt_f.sh (mount),
    check_doc_stamps.sh (stale canonical docs). If the pod check warns,
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
   /mnt/f, shipped to the pod, and deleted locally. Nothing raw is written
   inside /home/muads. No local GPU work.
2. No tasking without calling the available tools, skills, connectors, web
   fetch, research or MCPs where they apply.
3. No orphaned, standalone or untested item, tool or aspect of the pipeline
   may exist. Everything is wired, tested, or retired. (Brief 03 retired the
   25 files that violated this; the census lives in STATUS.md.)
4. Every brief carries this doctrine as a footer.
5. NO DEAD ENDS. When a path blocks the work, name the alternatives, pick
   the best available, and keep going.