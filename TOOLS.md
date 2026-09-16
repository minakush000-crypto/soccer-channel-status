# TOOLS.md — one row per file in tools/

> **Purpose:** one row per file in tools/ (status: WIRED / STANDALONE / DEAD).
> **Reader:** every session; mirrored to the public status repo.
> **Last verified against code:** 2026-09-13.

Verified against code on 2026-09-13. Caller column is from
`grep -rnE "import <mod>|TOOLS / \"<name>\"|\"<name>.py\"" tools/ --include="*.py"`
limited to real call sites (subprocess `cmd=[PYTHON,...]`, `import`, or curl
download), not docstring/test mentions. "ORPHANED" = no caller anywhere.
Line numbers are where the tool is invoked or imported; re-read before relying.
This rebuild supersedes the 2026-09-05 version (33 tools, tactical_overlay
wiring): the repo now has 47 tools on disk and produce_v2 no longer calls
tactical_overlay or tactical_render (Stage 14 retirement). Archive tooling
(rclone v1.75.1 + B2 bucket mendymax-archive) verified this pass; see the
"Archive tooling" section below.

## Classification (from RECONCILIATION.md 1.1)

- **WIRED** = reachable from produce_v2.py (14 direct calls + 4 transitive libs + 1 transitive pod script).
- **STANDALONE** = CLI entry point or hand-run utility, or wired only into
  another standalone entry point (29 tools).
- **DEAD** = no caller anywhere — RETIRED (10 tools deleted from the tree; in git history).

## The 47 tools (was 40; +10 new, -10 retired deleted)

| File | Class | Called by (file:line) | Last modified | Works? |
|---|---|---|---|---|
| `produce_v2.py` | WIRED (root) | nothing (entry point) | 2026-09-13 | works (liverpool-forest 720x1280 62.3s Sep 7); --pod-download routes step3 to runpod_download (5A); step4b_tactical_render RETIRED Stage 14 |
| `match_data.py` | WIRED | `produce_v2.py:84` | 2026-09-01 | works |
| `tactical_boards.py` | WIRED | `produce_v2.py:97` | 2026-09-12 | works. Stage 14: 3D formation board (scene_gen via modal_render3d) overwrites formation.mp4 in step2_boards; tactical_boards still produces possession/stat_card boards. |
| `three_render3d.py` | WIRED | `produce_v2.py:110` | 2026-09-16 | works (headless Chromium, 3D formation board); replaced modal_render3d |
| `runpod_download.py` | WIRED (--pod-download) | `produce_v2.py:288` (step3_download_clips_pod) | 2026-09-12 | built 5A; pod-side yt-dlp bot-blocked by YouTube (runs #1-3); excerpt-cut+guard proven via --source-url (2/3 windows, 4MB each, $0.005, LANE_PLAN 5A.3) |
| `script_gen.py` | WIRED | `produce_v2.py:348` (step1b) | 2026-09-13 | works (Stage 12B; glm-5.2:cloud per-section, 1439w draft proven) |
| `validate_script.py` | WIRED | `produce_v2.py:362` (step1c); also `produce_episode.py:162` | 2026-09-12 | works (Stage 12B; enforces lane word-budget minimum; opening-hook check 10D) |
| `broadcast_filler.py` | WIRED | `produce_v2.py:404` (step4a) | 2026-09-08 | works; FIXED 3A.1 (phantom-segment bug: --clip/--duration bounds); yields 43s/8min, 192s/13.5min, 373s/18.7min |
| `cut_list_gen.py` | WIRED | `produce_v2.py:414` (step4a) | 2026-09-09 | works (4C); PySceneDetect + broadcast_filler -> scripts/<slug>.md; proven on 18.7min (cuts at 26.6/214.7/222.1/245.4) |
| `transformation_gate.py` | WIRED | `produce_v2.py:634` (step5b) | 2026-09-08 | works (Stage 12B; lane D floor gate) |
| `generate_voice.py` | WIRED | `produce_v2.py:660` | 2026-09-12 | works; Stage 14 .script_verified voice gate (produce_v2.py:648 checks renders/<slug>/.script_verified before TTS) |
| `generate_ambience.py` | WIRED | `produce_v2.py:679` | 2026-08-22 | works (wired 2026-09-08, STATUS Part 6) |
| `merge_voice.py` | WIRED | `produce_v2.py:700` | 2026-08-25 | works |
| `shorts_crop.py` | WIRED | `produce_v2.py:710` | 2026-08-30 | works (outputs 720x1280) |
| `ffmpeg_utils.py` | WIRED (lib) | imported by `produce_v2.py:46`, `generate_voice.py:24`, `merge_voice.py:25`, `shorts_crop.py:31`; shipped to pod by `runpod_fulltrack.py:101` | 2026-09-08 | works (now holds the 200MB guard) |
| `script_utils.py` | WIRED (lib) | imported by `generate_voice.py:25` | 2026-08-25 | works |
| `staging.py` | WIRED (lib) | imported by `produce_v2.py:47` | 2026-09-09 | works (/mnt/f USB staging for raw downloads, 6A) |
| `three_scene.js` | WIRED (transitive) | run locally by `three_render3d.py` | 2026-09-16 | works (3D formation board) |
| `cv_annotate.py` | STANDALONE (transitive) | shipped+run on pod by `runpod_fulltrack.py:101` (STANDALONE since step4b retired Stage 14) | 2026-09-07 | works (per-frame positions export, STATUS); no longer reachable from produce_v2 |
| `runpod_fulltrack.py` | STANDALONE (was WIRED) | was `produce_v2.py` step4b; step4b RETIRED Stage 14 (tactical_render.py git-rm'd) | 2026-09-12 | works (146s full-clip, $0.011, STATUS); no longer called from produce_v2; hand-run only |
| `produce_episode.py` | STANDALONE | entry point; no caller | 2026-09-09 | untested (no verified output); 800M guard added (3A.4), keeps 1080p anti-blur source |
| `cloud_produce.py` | STANDALONE | entry point; no caller | 2026-09-08 | untested (pod-side yt-dlp + download_url guarded) |
| `runpod_superres.py` | STANDALONE | entry point; no caller | 2026-08-30 | untested |
| `gpu_superres.py` | STANDALONE | entry point; no caller | 2026-08-30 | untested |
| `sharpness_check.py` | STANDALONE (transitive) | `cloud_produce.py:989` | 2026-08-30 | untested |
| `assemble_video.py` | STANDALONE (legacy) | `produce_episode.py:313`, `cloud_produce.py:569`; NOT produce_v2 | 2026-08-30 | untested |
| `generate_captions.py` | STANDALONE | no caller (only `tests/`) | 2026-08-25 | untested |
| `thumbnail_generator.py` | STANDALONE | no caller | 2026-08-25 | untested |
| `enhance_clips.py` | STANDALONE | no caller (only `tests/`) | 2026-08-23 | untested |
| `segment_scorer.py` | STANDALONE | no caller; `cv_annotate.py:332` is a comment only | 2026-09-05 | works (hand-run on full-clip data, STATUS) |
| `scoreboard_scan.py` | STANDALONE | no caller | 2026-09-08 | works (3/3 goals, STATUS) |
| `assemble_words_match.py` | STANDALONE | no caller | 2026-09-08 | works (arsenal-chelsea 45.2s 1280x720 15MB Sep 8; file overwritten 2026-09-12; now 1920x1080 724s 210MB) |
| `player_mapper.py` | STANDALONE | no caller | 2026-09-15 | untested, undocumented (violates rule 3) |
| `trim_tracking.py` | STANDALONE | no caller; has `main()` CLI; output in `artifacts/tracking_summary/` | 2026-09-08 | works (hand-run) |
| `viral_angle.py` | STANDALONE | no caller | 2026-08-25 | untested |
| `agent_reach_research.py` | STANDALONE | no caller | 2026-08-30 | untested |
| `fresh_fetch.py` | STANDALONE | no caller | 2026-08-18 | untested |
| `youtube_upload.py` | STANDALONE | no code caller; invoked by hand | 2026-09-07 | works (2 publish-log entries, Sep 6) |
| `oauth_setup.py` | STANDALONE | one-time; no caller | 2026-08-23 | ran once |
| `gemini_inventory_test.py` | STANDALONE | no caller | 2026-09-08 | works (hand-run, STATUS Gemini section) |
| `ltx_enhance.py` | STANDALONE | no caller | 2026-08-23 | untested |
| `gemini_judge.py` | STANDALONE | no caller; invoked by hand (AUTHORITATIVE visual judge, Stage 12) | 2026-09-13 | works (gemini-3.1-pro-preview, Stage 12) |
| `pod_check.py` | STANDALONE | SessionStart hook `.claude/hooks/check_pods.sh`; no pipeline caller | 2026-09-12 | works (leak check, STATUS) |
| `b2_upload.py` | STANDALONE | no caller | 2026-09-09 | untested via boto3 (B2_KEY_ID absent from .env, 6E.2); B2 itself IS reachable via rclone (see Archive tooling below) |
| `backup_env.py` | STANDALONE | no caller | 2026-09-09 | works (encrypted .env backup, Stage 6) |
| `render3d_vast.py` | STANDALONE | no caller | 2026-09-12 | untested (Vast SSH key blocked, Stage 11C) |
| `sofascore_client.py` | STANDALONE | no caller | 2026-09-13 | untested (Stage 14; SofaScore lineup fetch) |
| `doc_stamp_check.py` | STANDALONE | SessionStart hook `.claude/hooks/check_doc_stamps.sh` | 2026-09-13 | works (Stage 14; flags stale doc stamps) |
| `tactical_render.py` | RETIRED (Stage 14) | was `produce_v2.py:285` (step4b) | 2026-09-06 | DELETED Stage 14; git-rm'd; in git history (superseded by 3D formation board via modal_render3d) |
| `tactical_overlay.py` | RETIRED (5C.2) | was DEAD | 2026-09-09 | DELETED 2026-09-09; in git history (superseded by tactical_render, then tactical_render retired) |
| `pitch_radar.py` | RETIRED (5C.2) | was DEAD (shipped but never run) | 2026-09-09 | DELETED 2026-09-09; ship-list refs removed from runpod_fulltrack/stage1/annotate |
| `render_video.py` | RETIRED (5C.2) | was DEAD (DEPRECATED) | 2026-09-09 | DELETED 2026-09-09; in git history |
| `check_and_download.py` | RETIRED (5C.2) | was DEAD (superseded by runpod_fulltrack) | 2026-09-09 | DELETED 2026-09-09; in git history |
| `runpod_annotate.py` | RETIRED (6B) | was STANDALONE | 2026-08-23 | DELETED Stage 6B; in git history (superseded by runpod_fulltrack) |
| `runpod_shorts.py` | RETIRED (6B) | was STANDALONE | 2026-08-29 | DELETED Stage 6B; in git history |
| `vastai_shorts.py` | RETIRED (4B.4/6B) | was STANDALONE | 2026-09-09 | DELETED Stage 6B; in git history (encoding_failed deterministically) |
| `luminance_pod.py` | RETIRED (6B) | was STANDALONE (transitive, uploaded by runpod_shorts/vastai_shorts) | 2026-08-29 | DELETED Stage 6B; in git history |
| `runpod_stage1.py` | RETIRED (6B) | was STANDALONE (one-off) | 2026-09-05 | DELETED Stage 6B; in git history (one-off, $0.05, STATUS) |

Counts: **14 WIRED direct + 4 transitive (ffmpeg_utils, script_utils, staging, scene_gen) + 29 STANDALONE, 0 DEAD (10 retired deleted)** = 47 .py files on disk. Stage 12B wired 5 more (script_gen, validate_script, broadcast_filler, cut_list_gen, transformation_gate). Stage 14 retired tactical_render + step4b, wired modal_render3d + scene_gen.

## Wired set (reachable from produce_v2.py)

13 direct subprocess TOOLS/ calls + 1 root (produce_v2 itself) + 4 transitive libs + 1 transitive pod script:

```
$ grep -oE 'TOOLS / "[a-z_0-9]+\.py"' tools/produce_v2.py | sort -u
broadcast_filler.py  cut_list_gen.py  generate_ambience.py  generate_voice.py
match_data.py  merge_voice.py  modal_render3d.py  runpod_download.py
script_gen.py  shorts_crop.py  tactical_boards.py  transformation_gate.py
validate_script.py
```
Transitive libs: `ffmpeg_utils.py` (imported by produce_v2/generate_voice/merge_voice/shorts_crop),
`script_utils.py` (imported by generate_voice), `staging.py` (imported by produce_v2).
Pod: `scene_gen.py` (shipped+run by modal_render3d via Blender). `cv_annotate.py`
was shipped by runpod_fulltrack (now STANDALONE — step4b retired Stage 14).

## Skill auto-activation infrastructure (Stage 12C)

Installed from github.com/diet103/claude-code-infrastructure-showcase.
Regex-only mode (skill-rules.json `skill_activation_mode: disabled` — no AI
provider, free, offline). Verified: `bash .claude/scripts/verify-setup.sh` →
8/8 PASS.

**4 hooks (project .claude/settings.json, additive to the existing SessionStart):**
- `skill-activation-prompt.sh` (UserPromptSubmit) — suggests skills from
  skill-rules.json by keyword/intentPattern regex match. Confirmed: a
  "create a new skill" prompt → recommends skill-developer via regex.
- `skill-verification-guard.sh` (PreToolUse Edit|MultiEdit|Write) — blocks an
  edit if a mandatory skill is pending (two-try model). No-op when no skills
  are block-enforcement (this project has none).
- `post-tool-use-tracker.sh` (PostToolUse Edit|MultiEdit|Write) — logs edited
  files + their repo to .claude/tsc-cache/. Pure bash (jq). Designed for the
  monorepo tsc/build workflow; logs "unknown" repos here (no package.json) but
  is harmless.
- `skill-activation-tracker.sh` (PostToolUse Skill) — records when a skill is
  activated (session intelligence / better-sqlite3).
NOT installed (per brief): tsc-check, trigger-build-resolver,
stop-build-check-enhanced (monorepo build hooks, wrong stack), and
session-doc-updater (Stop, not in the 4).

The 3 skill-* hooks run via `_run-node-hook.sh` → tsx (.ts in .claude/hooks/).
Deps: .claude/hooks/node_modules (tsx, better-sqlite3, minimatch; npm install).
`.claude/hooks/.env` sets SESSION_DOCS_ENABLED=false (suppresses the dev-doc
reminder — this project uses its own doc spine, not /dev/active/ + /dev-docs).

**skill-rules.json** (.claude/skills/): mode=disabled, conservativeness=balanced.
Only `skill-developer` registered (the showcase's backend-dev-guidelines
Express/Prisma, frontend-dev-guidelines React/MUI, error-tracking Sentry skills
were NOT installed — wrong stack for a Python/ffmpeg soccer pipeline).

**8 agents** (.claude/agents/, all clean — no hardcoded paths): auto-error-resolver
(TS), code-architecture-reviewer, code-refactor-master, documentation-architect,
frontend-error-resolver (no frontend here — stack-mismatched but harmless, on-demand),
plan-reviewer, refactor-planner, web-research-specialist. Invokable via
subagent_type. 6 are generic-relevant; 2 are stack-specific (auto-error-resolver
TS, frontend-error-resolver frontend).

**dev-docs pattern: NOT adopted.** The showcase's dev-docs commands reference a
/dev/active/ task-dir structure. This project already has a doc spine
(CONTEXT/STATUS/PROGRESS/DECISIONS/ARCHITECTURE/TOOLS/GAPS + LANE_PLAN + the
public status mirror) + the unlazy GATES.md pattern. Adding dev-docs would create
a competing second doc system (rule 3). Skipped.

**~130 skill folders audit (rule 3):** 127 marketing/design skills in
~/.claude/skills (126 valid SKILL.md), 125 mirrored in the parent
yt-digest/.claude/skills (duplicates). They are a deliberate user library
(marketing work), manually invokable via /<skill-name>, NOT in skill-rules.json
so NOT auto-suggested. They are not pipeline orphans (not pipeline tools). To
auto-suggest them, add entries to skill-rules.json. 1 global skill folder lacks
SKILL.md (minor invalid entry). No action taken beyond documenting.

## Archive tooling (rclone + B2) — Stage B2 migration, 2026-09-13

**rclone is the archive of record.** Not a tools/ .py file — a system binary at
`/usr/bin/rclone` (v1.75.1, verified `rclone version`). Remote `b2` configured
via `rclone config` (type=b2, account+key in rclone's own config, NOT in .env).
Bucket `mendymax-archive` verified: `rclone lsd b2:` prints it; `rclone lsf
b2:mendymax-archive` lists 3 items (CLOSING PASS: bucket size in bytes +
itemized listing after the archive-then-delete pass completes).

**NEW STANDING RULE (extends doctrine rule 1 to generated outputs):**
everything produced by any task (renders, frames, transcripts, artifacts,
backups) is archived to B2; local copies are working copies, not the record.
Archive-then-delete IN PROGRESS (sep5 backups 2.6G + ~/retired 1.4G + old
render folders ~3.4G); local deletions happen AFTER docs are amended + uploads
verified. CLOSING PASS: free space after deletions, what was deleted, bucket
size after upload.

Status: **WIRED** (verified `rclone lsd b2:` → mendymax-archive bucket
reachable). rclone is not called from produce_v2.py (archive is a manual /
post-session step, not a pipeline stage). To archive a render:
`rclone copy renders/<slug>/shorts/final_video_shorts.mp4 b2:mendymax-archive/<slug>/`

**b2_upload.py** (tools/, STANDALONE) is the Python/boto3 alternative path. It
reads B2_KEY_ID + B2_APPLICATION_KEY from ~/yt-digest/.env — those keys are
ABSENT (grep `^B2_` .env → 0 matches), so b2_upload.py itself is untested. B2
is reachable through rclone's own config regardless. b2_upload.py is NOT the
archive of record; rclone is. CLOSING PASS: whether b2_upload.py gets retired
or wired once the .env key question is settled.