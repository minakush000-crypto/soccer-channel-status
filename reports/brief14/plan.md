# Brief 14 PLAN: lean harness (2026-09-28)

Author: Flash (soccer-channel-ae). Machine-readable twin: plan.json. The
oracles (tools/brief14_check.py) read plan.json; this file is the human
version. Every removal carries owner + reason; anything with an unclear
owner stays.

## Scope decision note (important, honest)
Closed decision 1 tells me to remove ONLY ECC's non-engineering/business
skills and agents. Measured reality: the ECC examples in the brief
(customs/trade compliance, energy procurement, visa translation, plus the
healthcare, logistics, investor and compute packs: 160 skills) were NEVER
installed. The installed 126 ECC skills are all engineering. The 127
foreign skills (the big marketing kit, document skills, etc.) are
untracked in ECC's install-state and their owner is unclear, so they STAY
per the brief's own rule (owner unclear stays). Therefore the removable
skill set is EMPTY. The real load levers this brief hits: hook dedup (50
fewer spawns per event), 4 Stop-hook removals, 3 MCP servers, 1 memory
merge, and the read-discipline rule.

## Counts before/after

| quantity | before | after |
|---|---|---|
| skills (~/.claude/skills) | 253 | 253 |
| agents (~/.claude/agents) | 68 | 65 |
| hook entries, all sources | 94 | 40 |
| hook entries, global settings | 75 | 23 |
| Stop-event entries | 25 | 7 |
| MCP servers (~/.claude.json) | 6 | 3 |
| MCP tools exposed headless | 15 | 15 |

## Removals

### Agents (3, ECC baseline:agents, files in ~/.claude/agents)
- chief-of-staff: email/Slack triage; decision-1 class; unused.
- healthcare-reviewer: healthcare-domain reviewer; no healthcare codebase.
- marketing-agent: marketing copywriter; decision-1 class.
Owner-supported way: ECC's catalog/uninstall cannot target single agents
(agents-core is one module), so files are removed directly, then
`ecc.js doctor` must report 0 warnings/0 errors for everything that
remains. If doctor flags missing managed files, restore from the ECC
source copy (reversible) and record why: the supported way does not permit
per-agent removal.

### Skills (0, documented empty set)
- Nothing removed. Reason: all installed ECC skills are engineering
  (verified by reading each SKILL.md description; heuristics pass plus
  manual sampling); Mayo's listed examples are ECC-source-only, not
  installed; the foreign kit stays (owner unclear). Recorded as a brief-15
  lever for Mayo: removing the foreign kit by name needs Mayo's decision
  because its owner is not ECC.

### Hooks (ECC, settings.json edits with a JSON tool)
Remove ALL copies of 4 Stop children (12 entries):
- stop:desktop-notify: 3,092ms x3 measured; no notification consumer.
- stop:evaluate-session: ECC self-eval, never read.
- stop:check-console-log: console.log scanner, no consumer on either pipeline.
- stop:format-typecheck: tsc/eslint at stop, spawnSync timeout 30s, the
  plausible brief-13 ETIMEDOUT hook.
Dedupe everything else: every identical ECC wrapper command (sha256-
identical, installed 3x) is listed exactly once. Kept unique children
(22 + unlazy): plan-canvas-pending, ecc session-end, cost-tracker,
pre-bash-dispatcher, powershell:gateguard-fact-force,
write:doc-file-warning, edit-write:suggest-compact, observe-runner,
governance-capture, config-protection, mcp-health-check (pre and
post-fail), posttooluse-dispatcher (sync and async), skill-run-tracker,
session-start-bootstrap, plan-canvas-sessions, pre-compact,
session-end-marker, block-retired.sh, warn-local-gpu.sh, unlazy stop-hook.
Global entries: 75 -> 23. Stop event: 25 -> 7 entries.

Untouched (protected by closed decision 2): unlazy Stop, soccer
push_status, soccer disk-guard/session hooks, jiheeye push_status, all
claude-mem plugin hooks (its hooks.json is not edited).

### MCP servers (3 of 6, ~/.claude.json, JSON tool)
Remove: filesystem, memory, sequential-thinking. Zero callers proven in
reports/brief14/mcp_callers.txt (both repos, all skills, rules, hooks
dirs); none exposes a live tool. Keep: github, brave-search, puppeteer
(brief scope). claude-mem's mcp-search tools are plugin-served and
untouched.

### Memory (1 file)
Merge soccer-channel-capability-enablement.md (yt-digest store, 2026-09-06
capability snapshot) into a short pointer to ~/.claude/SYSTEM.md sections
5/9 (which now reflect the end state). Update its MEMORY.md line. All
other memory files are gotchas or project facts and stay.

### Global CLAUDE.md (read discipline)
Append a Read discipline section: grep first, read only needed ranges,
never re-read a whole file already read this session, prefer subagents for
bulk reading. (Ordered by the brief's "Also in scope"; the job-1 backup is
the rollback.)

## What stays, deliberately
- The whole foreign marketing kit (127 skills): owner unclear.
- GateGuard and every pre/post ECC dispatcher: deduped, not removed; they
  carry the fact-forcing and protection behavior Mayo relies on.
- claude-mem, unlazy, both push_status, the disk-guard chain.
- 65 agents (engineering/review/testing/security set per decision 1).
- 3 MCP servers (github, brave-search, puppeteer).

## Execution order (job 4)
1. JSON edit ~/.claude/settings.json (dedupe + 4 Stop-child removals).
2. JSON edit ~/.claude.json (3 MCP removals).
3. Remove 3 agent files (mv to /tmp first, doctor-gated decision).
4. Merge the memory file into a pointer + fix its MEMORY.md line.
5. Add the CLAUDE.md read-discipline rule.
6. node ~/ECC/scripts/ecc.js doctor -> must be clean for what remains.
7. Any doctor failure: restore the removed item, record why.