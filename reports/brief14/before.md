# Brief 14 BEFORE measurements (2026-09-28, session soccer-channel-ae)

Inputs: brief14.md (copied into briefs/ this session), brief-07 method file
reports/brief07/context_before.txt (produced 2026-09-27 ~21:55 CDT), all
settings files read 2026-09-28 ~19:30-20:30 CDT. Raw files kept in
reports/brief14/raw/. Jiheeye session confirmed idle before any measurement
(ListAgents: jiheeye-ultra-64, interactive, idle, 19:31 CDT; tmux shows
jiheeye session attached but its Flash session idle).

## SESSION-START-TOKENS (method = brief-07 throwaway headless session)

Command (exact, run from the repo):

  claude -p 'Say OK' --model glm-5.3-flash:cloud --output-format stream-json --verbose --max-turns 1

Raw: reports/brief14/raw/ctx_before.jsonl (stream-json). Extracted:

  init: tools=39, mcp_servers=7 (6 configured servers + claude-mem)
  assistant#1 input_tokens=51297 output_tokens=0
  assistant#2 input_tokens=51297 output_tokens=0
  result: subtype=error_max_turns is_error=True num_turns=2
          input=48552 cache_read=0 output=43
  wall 69.40s, exit 1 (max-turns reached because a hook re-prompted; the
  same shape as brief 07's run, which measured 39,837/37,826 pre-ECC)

SESSION-START TOKENS (headline, first model turn = the fixed system load):
**51,297** (aggregate 48,552). Brief 07 comparison point pre-ECC: 39,837.

## BENCH-RUN-1 and BENCH-RUN-2 (fixed task, fresh headless session each)

Prompt (exact, from the brief): "Read GATES.md and tools/census.py and
report the number of live gates and census totals"

Method: each run in a THROWAWAY git worktree (git worktree add --detach) with
(a) GATES.md filtered to the 55 evidenced standing gates, so the unlazy Stop
hook sees an all-met ledger (a checked box without an EVIDENCE line still
counts unmet; verified live), and (b) the repo's push_status Stop hook
removed from the copy so a bench session cannot publish the public mirror.
Everything else identical to the repo. Same filter will be applied to the
AFTER worktree, so before/after comparability is exact.

Two contaminated runs are recorded as observations, not as the benchmark:

  (a) repo-cwd run with the 14 authored but unmet brief-14 gates: wall
      152.53s, 23 assistant messages, aggregate input 169,335; the session
      spent 19 of 23 turns talking about the unmet-gate Stop hook loop and
      buried the answer. Cause: unmet-gate loop, not the harness.
  (b) worktree run with all 69 boxes checked but no EVIDENCE lines: wall
      189.69s, 20 messages, aggregate 175,559; hook still saw 14 unmet
      (evidence rule). Cause of design change to the filtered ledger.

Benchmark truth at run time (counted seconds before each run):
  live gates in the bench ledger = 55 (boxes: grep -c "^- \[")
  census = "CENSUS OK files=57 wired=36 handrun=21 (exempt from rows: USAGE.md)"
  (tools/brief14_check.py registered in TOOLS.md before measuring, so the
  census number is stable across both phases)

### BENCH-RUN-1 (reports/brief14/raw/bench_run1.jsonl)
  wall 118.37s, exit 0, model turns 6 (num_turns=6), tool calls 6
  first-turn input 51007, aggregate input 109,455
  ANSWER: 55 live gates (all met) on the bench ledger; census CENSUS OK
  files=57 wired=36 handrun=21.
  VERDICT: CORRECT (55 = filtered ledger count; census line matches exactly).
  The answer also reports HEAD's committed ledger as 69 boxes, 14 unchecked,
  which is true of the repo and is the reason a filtered worktree exists.

### BENCH-RUN-2 (reports/brief14/raw/bench_run2.jsonl)
  wall 244.72s, exit 0, num_turns 7, tool calls 6
  first-turn input 51,252, aggregate input 90,049
  ANSWER: "Live gates: 55 in GATES.md on disk (all 55 met, 0 unmet)...
  Census totals: files=57 wired=36 handrun=21, CENSUS OK, exit 0."
  VERDICT: CORRECT.

Wall-time variance between identical runs (118s vs 245s) is large; both runs
are kept as the brief requires and the AFTER phase keeps both again and
compares run-for-run plus mean. First-turn input is the stable token proxy:
51007/51252 before (mean 51,129.5).

## HOOKS-TABLE (every hook, owner, measured runtime)

Full machine inventory (commands, matchers, timeouts):
reports/brief14/raw/b14_hooks_full.json. 94 command hook entries total:

  global ~/.claude/settings.json   75 entries = 25 unique ECC wrappers x 3
  soccer .claude/settings.json      8 entries (3 SessionStart: check_pods,
                                    check_scratch, check_doc_stamps;
                                    1 UserPromptSubmit skill-activation-prompt;
                                    1 PreToolUse skill-verification-guard;
                                    2 PostToolUse post-tool-use-tracker,
                                    skill-activation-tracker;
                                    1 Stop push_status)
  jiheeye .claude/settings.json     1 entry (Stop push_status twin)
  claude-mem plugin hooks.json      8 entries (Setup, SessionStart x2,
                                    UserPromptSubmit, PreToolUse, PostToolUse,
                                    Stop, SessionEnd)

### The duplication (verified by sha256 of exact command strings)
Every ECC wrapper command appears EXACTLY 3 times (three install passes
appended instead of replaced; brief 07 install plus later runs). Each wrapper
spawned per event therefore runs 3x. Examples (x3 each):
  Stop: plan-canvas-pending, stop-format-typecheck, check-console-log,
  session-end(ecc), evaluate-session, cost-tracker, desktop-notify
  PreToolUse: pre-bash-dispatcher, powershell:gateguard-fact-force,
  write:doc-file-warning, edit-write:suggest-compact, observe-runner,
  governance-capture, config-protection, mcp-health-check (8 wrappers)
  PostToolUse: posttooluse-dispatcher (sync + async), x3 each
  PreCompact: pre-compact x3; SessionEnd: session-end-marker x3
  SessionStart: session-start-bootstrap, plan-canvas-sessions (x3 each)
  PostToolUseFailure: mcp-health-check, skill-run-tracker (x3 each)

### Measured runtimes (unique child scripts run directly with the wrapper's
exact argv and a synthetic b14probe payload; wrapper pass = 2-19ms because
the synthetic payload rejects early; the child script is the honest
per-invocation floor. push_status is EXCLUDED from execution on purpose:
timing it would publish the public mirror; its real cost is dominated by
rclone plus git push.)

Stop event (per turn end, before dedup: sum of 7 children x 3 = 16.3s):
  unlazy stop-hook.mjs        390ms  exit 0  (protected, keep)
  plan-canvas-pending.js       91ms  exit 0  (dedupe, keep)
  session-end.js              390ms  exit 0  (dedupe, keep)
  cost-tracker.js             866ms  exit 0  (dedupe, keep)
  desktop-notify.js         3,092ms  exit 0  REMOVE (WSL has no notification
                                        consumer; x3 = 9.3s per turn end)
  evaluate-session.js         240ms  exit 0  REMOVE (ECC self-evaluation;
                                        never read on this pipeline)
  check-console-log.js        613ms  exit 0  REMOVE (scans for console.log;
                                        soccer is python, jiheeye has no
                                        dev loop; zero findings ever used)
  stop-format-typecheck.js    143ms  exit 0  REMOVE (tsc/eslint at stop; the
                                        spawnSync timeout:30000 inside the
                                        wrapper is the plausible source of
                                        the brief-13 ETIMEDOUT stop hook)
PreToolUse (per tool call; unique wrappers x3 today):
  pre-bash-dispatcher.js      374ms  keep+dedupe (runs GateGuard fact-force
                                    live; it fired 15 times this session)
  gateguard-fact-force.js     225ms  keep+dedupe (Mayo's fact gate)
  doc-file-warning.js         264ms  keep+dedupe
  suggest-compact.js          456ms  keep+dedupe
  observe-runner.js           645ms  keep+dedupe (exited 1 in the synthetic
                                        env for ECC-HOME reasons; real env fine)
  governance-capture.js       209ms  keep+dedupe
  config-protection.js        148ms  keep+dedupe
  mcp-health-check.js         395ms  keep+dedupe (pre) and 336ms (post-fail)
  block-retired.sh            180ms  exit 0 (soccer-era global hook, protected
                                        family)
  warn-local-gpu.sh           219ms  exit 0 (same)
Session start:
  session-start-bootstrap.js  665ms  keep+dedupe
  plan-canvas-sessions.js     167ms  keep+dedupe
PreCompact: pre-compact.js          398ms  keep+dedupe
SessionEnd: session-end-marker.js (wrapper 2,019 chars) keep+dedupe
PostToolUseFailure: skill-run-tracker.js 220ms keep+dedupe
soccer hooks: timed via synthetic run only (2-14ms with exit 127 because
  $CLAUDE_PROJECT_DIR was unset in the synthetic env; they are
  project-protected hooks and untouched by brief 14; push_status skipped)
claude-mem: 4-10ms each on the synthetic payload (worker not reached);
protected by closed decision 2, untouched.

Not timed: push_status.sh (soccer) and tools/push_status.sh (jiheeye):
publishing excluded by design.

Real-session cost anchors: 'Say OK' wall 69.40s (2 turns incl. the hook
re-prompt turn); bench 6-7 model turns 118-245s.

## MCP, skills, agents, memory (before)

- MCP servers configured in ~/.claude.json (6): puppeteer, filesystem,
  memory, sequential-thinking, github, brave-search.
- Tools actually exposed live (headless init): 39 total, of which mcp =
  15, ALL claude-mem. In this interactive session additionally
  brave-search (2 brave_* tools). None of filesystem / memory /
  sequential-thinking / github / puppeteer currently expose a tool.
- No-caller grep (reports/brief14/mcp_callers.txt): zero hits for
  mcp__filesystem, mcp__memory, mcp__sequential(=sequential-thinking),
  mcp__puppeteer, mcp__github across both repos, ~/.claude/skills, hooks
  and rules. Only textual mentions are the brief and ledger prose.
- Skills installed: 253 (126 ECC-managed + 127 foreign). Agents: 68, all
  ECC-managed. ECC source holds 286 skills/68 agents; 160 business-domain
  ECC skills (customs-trade-compliance, energy-procurement,
  visa-doc-translate, healthcare/HIPAA pack, logistics, ito/compute,
  investor materials, etc.) are NOT installed: decision 1's examples are
  already absent from the machine.
- claude-mem skills: plugin (15 mcp tools), untouched in this brief.
- Memory files: yt-digest store 18 files / 28,787 B; soccer store 2 files.
  One SYSTEM.md duplicate found: soccer-channel-capability-enablement.md
  (2026-09-06 hook/MCP capability snapshot; superseded by SYSTEM.md
  sections 5 and 9). Every other file is a gotcha or a project fact.
- Gate ledger: 55 standing evidenced gates + 14 authored brief-14 gates
  (B69-B82) = 69 boxes in the live repo GATES.md at this moment.

## Verdicts
  ● measured: all figures above (commands shown)
  ◑ believed: wrapper-level 3x duplication is the dominant cause of real
    per-event latency (child timings measured in isolation; real events
    also carry bigger payloads)
  ○ unchecked: none
  ★ fragile: GATES.md evidence churn on --reverify (B22 known); the unlazy
    evidence rule (checked box without EVIDENCE = unmet) shaped the bench
    method and must be respected by the AFTER worktree filter too.