# Brief 07 summary — for Mayo (STOP point reached)

**Read this line first: the F: backup (job 1) could not be made, the Memorex stick drops
under sustained writes. Nothing below risked it: jobs 2-4 only cloned a repo and ran a
--dry-run. No file in ~/.claude was touched. Do not type APPROVED ECC until the backup lands.**

## What happens if you approve (jobs 5-8, preview from the dry run)

`node scripts/ecc.js install --target claude --profile developer --with lang:python
--with lang:typescript --enable-hooks` into ~/.claude:

| What | Count (measured from the plan) |
|---|---|
| Files copied | 814 (815 planned minus 1 the installer itself skips) + 1 settings merge |
| Skill dirs | 125 (227 files), flat into ~/.claude/skills/ next to your existing skills |
| Rule files | 122, all under ~/.claude/rules/ecc/ |
| Commands | 94 (listed in counts.txt) |
| Agents | 68 files (agents/) + 89 files (.agents/) |
| Hook scripts | 53 under scripts/hooks/ + 4 under hooks/ |
| Installed size | 4.9 MB total (tiny) |
| Repo on disk | ~/ECC at tag v2.2.1, 266 MB with node_modules |

Numbers in the brief's "verified facts" do not match the real plan: the coordinator said
146 skills / 123 rules / 83 hook files; the actual v2.2.1 plan says 125 / 122 / 57. The
tag claim "v2.2.0, v2.2.1 only" is wrong literally (25 tags exist) but right in intent:
v2.2.1 is the newest tag, there is no v2.2.2. Same dry-run command, same tag, run here.

## Clash analysis (what already exists that ECC would touch)

- **One clash: `~/.claude/skills/design-system/SKILL.md`.** The installer itself detects it,
  refuses to overwrite, and records a skip (warning printed in the plan). Your skill stays.
- `~/.claude/rules/` exists but is empty: all 122 rule files are new files, no clash.
- `~/.claude/settings.json` is modified IN PLACE (merge-hook-ids): it appends 24 ECC hook
  entries and changes nothing else. Your 3 existing entries there (unlazy Stop hook,
  block-retired, warn-local-gpu) survive unchanged, verified against the merge source code.
  Zero id collisions. No hook is overwritten or disabled.
- Hooks that live OUTSIDE settings.json are untouched by construction: the disk-guard
  session-start hook, pod check and doc stamps are in the PROJECT settings.json; claude-mem
  hooks are plugin-managed.

## Hooks ECC registers (24 entries, 7 events; full table in hooks.txt)

PreToolUse x9, Stop x7, SessionStart x2, PostToolUse x2, PostToolUseFailure x2,
PreCompact x1, SessionEnd x1. Every command is a node bootstrap that resolves the ECC
install root and runs scripts/hooks/*.js. Findings that matter:

- **Network**: only two scripts really touch the network, both loopback-local:
  mcp-health-check probes MCP servers (all 6 of yours are stdio, so that path is dormant)
  and plan-canvas-pending talks to its own localhost browser server (not installed).
  Everything else that greps "http" is comment URLs.
- **Writes outside ~/.claude**: gateguard writes state to ~/.gateguard/ (home level).
  Several hooks write ephemeral state to /tmp. `stop:format-typecheck` runs
  Biome/Prettier/tsc on JS/TS files edited in a response, which edits project files by
  design. This repo is Python/bash, so nothing for it to touch today.
- **Context load**: every hook command embeds a base64 hardcode of /home/muads/.claude.
- The dispatcher pattern (pre-bash-dispatcher, post:dispatcher) runs many child checks in
  one process, so per-tool latency is bounded by one node startup, not 9.

## Session-start context BEFORE (job 3f)

Claude Code does not expose a dedicated "system context tokens" field. Measured by
throwaway headless session (`claude -p "Say OK" --model glm-5.3-flash:cloud --output-format
json --verbose`): first model turn input_tokens = **39,837** (per-message API usage; the
result aggregate reports 37,826, both printed in context_before.txt). The AFTER run (job 6)
must repeat the exact command; the delta is the ECC context cost, measured not estimated.

## The F: blocker (job 1, BLOCKED)

Backup did NOT happen. The Memorex USB stick disconnected from Windows twice under
sustained write today (NTFS event 140, 9:35 PM; 1.6 MB/s with readback failures; the
tarball died at 147 MB of ~1.7 GB). Volume is NTFS-Healthy; the hardware flaps. Full
evidence and options in f_drive_failure.md. Gates B32/B33 are ABANDONed (handoff), not
passed. Also on that stick: soccer-channel-keys, with no other local copy. My suggestion:
re-plug or replace the stick; when it is readable again I rerun job 1 (tar + B2 copy)
and archive the keys to B2, and only then does an approval make sense.

## What approval will NOT do

It will not overwrite unlazy, block-retired, warn-local-gpu, the disk guard, pod check,
doc-stamp check, claude-mem, or agent-reach. The merge source was read and verified:
ECC's installer refuses to overwrite hooks it does not own (it even refused to touch your
design-system skill on its own).

## Stop

Per the brief: nothing further happens until you type APPROVED ECC (and the F: backup is
completed first, B32/B33 met). Artifacts: reports/brief07/{ecc_plan.json, ecc_plan.txt,
counts.txt, clashes.txt, hooks.txt, settings_diff.txt, size_estimate.txt, context_before.txt,
f_drive_failure.md, summary.md}.