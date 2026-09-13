# TOOLS.md — one row per file in tools/

> **Purpose:** one row per file in tools/ (status: WIRED / STANDALONE / DEAD).
> **Reader:** every session; mirrored to the public status repo.
> **Last verified against code:** 2026-09-09.

Verified against code on 2026-09-08. Caller column is from
`grep -rnE "import <mod>|TOOLS / \"<name>\"|\"<name>.py\"" tools/ --include="*.py"`
limited to real call sites (subprocess `cmd=[PYTHON,...]`, `import`, or curl
download), not docstring/test mentions. "ORPHANED" = no caller anywhere.
Line numbers are where the tool is invoked or imported; re-read before relying.
This rebuild supersedes the 2026-09-05 version (33 tools, tactical_overlay
wiring): the repo now has 43 tools and produce_v2 no longer calls
tactical_overlay.

## Classification (from RECONCILIATION.md 1.1)

- **WIRED** = reachable from produce_v2.py (13 tools, +runpod_download via --pod-download).
- **STANDALONE** = CLI entry point or hand-run utility, or wired only into
  another standalone entry point (27 tools).
- **DEAD** = no caller anywhere (4 tools) — RETIRED 2026-09-09 (Stage 5C.2),
  deleted from the tree; in git history.

## The 40 tools (was 43; +runpod_download, -4 DEAD retired)

| File | Class | Called by (file:line) | Last modified | Works? |
|---|---|---|---|---|
| `produce_v2.py` | WIRED (root) | nothing (entry point) | 2026-09-09 | works (liverpool-forest 720x1280 62.3s Sep 7); --pod-download routes step3 to runpod_download (5A) |
| `match_data.py` | WIRED | `produce_v2.py:69` | 2026-09-01 | works |
| `tactical_boards.py` | WIRED | `produce_v2.py:78` | 2026-09-08 | works. 10C: 2D matplotlib board renderer; slated for replacement by scripted 3D (10C). NOT deleted — runs until a 3D replacement is proven at/above spec. 10C.4 deletion HELD: GPU capacity 0/48 on 2026-09-12, no 3D frame rendered yet. |
| `runpod_fulltrack.py` | WIRED | `produce_v2.py:235` | 2026-09-09 | works (146s full-clip, $0.011, STATUS); pitch_radar removed from ship list (5C.2) |
| `runpod_download.py` | WIRED (--pod-download) | `produce_v2.py` step3_download_clips_pod | 2026-09-09 | built 5A; pod-side yt-dlp bot-blocked by YouTube (runs #1-3); excerpt-cut+guard proven via --source-url (2/3 windows, 4MB each, $0.005, LANE_PLAN 5A.3) |
| `tactical_render.py` | WIRED | `produce_v2.py:285` | 2026-09-06 | works (Opus 8/8.5, STATUS). 10C: 2D top-down matplotlib renderer; slated for replacement by scripted 3D (10C). NOT deleted — runs until a 3D replacement is proven at/above spec. 10C.4 deletion HELD: GPU 0/48 on 2026-09-12. |
| `generate_voice.py` | WIRED | `produce_v2.py:474` | 2026-09-08 | works |
| `generate_ambience.py` | WIRED | `produce_v2.py:493` | 2026-08-22 | works (wired 2026-09-08, STATUS Part 6) |
| `merge_voice.py` | WIRED | `produce_v2.py:514` | 2026-08-25 | works |
| `shorts_crop.py` | WIRED | `produce_v2.py:524` | 2026-08-30 | works (outputs 720x1280) |
| `ffmpeg_utils.py` | WIRED (lib) | imported by `generate_voice.py:24`, `merge_voice.py:25`, `shorts_crop.py:31`; shipped to pod by `runpod_fulltrack.py:72` | 2026-09-08 | works (now holds the 200MB guard) |
| `script_utils.py` | WIRED (lib) | imported by `generate_voice.py:25` | 2026-08-25 | works |
| `cv_annotate.py` | WIRED (pod) | shipped+run on pod by `runpod_fulltrack.py:72,91` | 2026-09-07 | works (per-frame positions export, STATUS) |
| `produce_episode.py` | STANDALONE | entry point; no caller | 2026-09-09 | untested (no verified output); 800M guard added (3A.4), keeps 1080p anti-blur source |
| `cloud_produce.py` | STANDALONE | entry point; no caller | 2026-09-08 | untested (pod-side yt-dlp + download_url guarded) |
| `runpod_annotate.py` | STANDALONE | entry point; no caller | 2026-08-23 | untested (GAPS) |
| `runpod_shorts.py` | STANDALONE | entry point; no caller; uploads `luminance_pod.py:254` | 2026-08-29 | untested |
| `vastai_shorts.py` | RETIRED (4B.4) | was STANDALONE; end-to-end run 2026-09-09: Vast key works, pod creates, but encoding_failed (size 0) deterministically — encode script broken on the pod | 2026-09-09 | retired (rule 3) |
| `runpod_superres.py` | STANDALONE | entry point; no caller | 2026-08-30 | untested |
| `gpu_superres.py` | STANDALONE | entry point; no caller | 2026-08-30 | untested |
| `luminance_pod.py` | STANDALONE (transitive) | `runpod_shorts.py:254`, `vastai_shorts.py:334` (uploaded to pod) | 2026-08-29 | untested |
| `sharpness_check.py` | STANDALONE (transitive) | `cloud_produce.py:984` | 2026-08-30 | untested |
| `assemble_video.py` | STANDALONE (legacy) | `produce_episode.py:303`, `cloud_produce.py:564`; NOT produce_v2 | 2026-08-30 | untested |
| `validate_script.py` | STANDALONE (legacy) | `produce_episode.py:155`; NOT produce_v2 | 2026-08-25 | untested |
| `generate_captions.py` | STANDALONE | no caller (only `tests/`) | 2026-08-25 | untested |
| `thumbnail_generator.py` | STANDALONE | no caller | 2026-08-25 | untested |
| `enhance_clips.py` | STANDALONE | no caller (only `tests/`) | 2026-08-23 | untested |
| `segment_scorer.py` | STANDALONE | no caller; `cv_annotate.py:332` is a comment only | 2026-09-05 | works (hand-run on full-clip data, STATUS) |
| `scoreboard_scan.py` | STANDALONE | no caller | 2026-09-08 | works (3/3 goals, STATUS) |
| `broadcast_filler.py` | STANDALONE | no caller | 2026-09-09 | works; FIXED 3A.1 (phantom-segment bug: --clip/--duration bounds); yields 43s/8min, 192s/13.5min, 373s/18.7min |
| `cut_list_gen.py` | STANDALONE (prerequisite) | no caller; run by hand before produce_v2 to write footage=START-END tags | 2026-09-09 | works (4C); PySceneDetect + broadcast_filler -> scripts/<slug>.md; proven on 18.7min (cuts at 26.6/214.7/222.1/245.4) |
| `assemble_words_match.py` | STANDALONE | no caller | 2026-09-08 | works (arsenal-chelsea 45.2s, STATUS) |
| `trim_tracking.py` | STANDALONE | no caller; has `main()` CLI; output in `artifacts/tracking_summary/` | 2026-09-08 | works (hand-run) |
| `viral_angle.py` | STANDALONE | no caller | 2026-08-25 | untested |
| `agent_reach_research.py` | STANDALONE | no caller | 2026-08-30 | untested |
| `fresh_fetch.py` | STANDALONE | no caller | 2026-08-18 | untested |
| `youtube_upload.py` | STANDALONE | no code caller; invoked by hand | 2026-09-07 | works (2 publish-log entries, Sep 6) |
| `oauth_setup.py` | STANDALONE | one-time; no caller | 2026-08-23 | ran once |
| `gemini_inventory_test.py` | STANDALONE | no caller | 2026-09-08 | works (hand-run, STATUS Gemini section) |
| `runpod_stage1.py` | STANDALONE (one-off) | no caller; docstring = one-off | 2026-09-05 | works (one-off, $0.05, STATUS) |
| `ltx_enhance.py` | STANDALONE | no caller | 2026-08-23 | untested |
| `tactical_overlay.py` | RETIRED (5C.2) | was DEAD | 2026-09-09 | DELETED 2026-09-09; in git history (superseded by tactical_render) |
| `pitch_radar.py` | RETIRED (5C.2) | was DEAD (shipped but never run) | 2026-09-09 | DELETED 2026-09-09; ship-list refs removed from runpod_fulltrack/stage1/annotate |
| `render_video.py` | RETIRED (5C.2) | was DEAD (DEPRECATED) | 2026-09-09 | DELETED 2026-09-09; in git history |
| `check_and_download.py` | RETIRED (5C.2) | was DEAD (superseded by runpod_fulltrack) | 2026-09-09 | DELETED 2026-09-09; in git history |

Counts: **14 WIRED, 22 STANDALONE, 0 DEAD (4 retired)** = 36 (+2 libs = 38 .py files). Stage 12B wired 5 more (script_gen, validate_script, broadcast_filler, cut_list_gen, transformation_gate).

## Wired set (reachable from produce_v2.py)

14 direct subprocess calls + 2 shared libs + 1 pod-shipped tracker:

```
$ grep -oE 'TOOLS / "[a-z_]+\.py"' tools/produce_v2.py | sort -u
broadcast_filler.py  cut_list_gen.py  generate_ambience.py  generate_voice.py
match_data.py  merge_voice.py  runpod_download.py  runpod_fulltrack.py
script_gen.py  shorts_crop.py  tactical_boards.py  tactical_render.py
transformation_gate.py  validate_script.py
```
Transitive libs: `ffmpeg_utils.py` (imported by generate_voice/merge_voice/shorts_crop),
`script_utils.py` (imported by generate_voice). Pod: `cv_annotate.py`
(shipped+run by runpod_fulltrack). `pitch_radar.py` is shipped but never run.
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
frontend-error-fixer (no frontend here — stack-mismatched but harmless, on-demand),
plan-reviewer, refactor-planner, web-research-specialist. Invokable via
subagent_type. 6 are generic-relevant; 2 are stack-specific (auto-error-resolver
TS, frontend-error-fixer frontend).

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
