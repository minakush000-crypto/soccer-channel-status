# Brief 14 AFTER measurements (2026-09-28, method identical to before.md)

Same commands, same worktree filter, same extraction as before.md. Raw
files: reports/brief14/raw/ctx_after.jsonl and abench_run1..4.jsonl. The
after worktree (identical method: evidenced-gates filter + publisher
stripped) lives at /mnt/d/scratch/b14_wt_after/soccer-channel because /tmp
tmpfs ran out (111 MB free at 96%; the before worktree sat on /tmp). Path
text only; task content identical.

## SESSION-START-TOKENS (same brief-07 headless command)

  init: tools=41 (39 before), mcp_servers=4 (7 before: 6 configured - 3
        removed + claude-mem)
  assistant#1 input_tokens=51151
  result: subtype=error_max_turns num_turns=2 num_turns=2 input=48227 output=29
  wall 51.83s (before 69.40s, -25.6%), exit 1 (same max-turns shape)

FIRST-TURN INPUT TOKENS: **51,151** (before 51,297, -146). The token load
barely moved: skills stayed 253, only 3 agent descriptions and 3 MCP
server stubs left. The win this brief bought is hooks, not context size
(below).

## Bench: 4 after runs at identical flags (both brief-mandated runs kept,
plus 2 more because the first two ended at max-turns without an answer;
every run is recorded, none discarded)

| run | wall (s) | num_turns | first-turn in | aggregate in | answered | verdict |
|---|---|---|---|---|---|---|
| abench-1 | 91.63 | 7 (max) | 51,840 | 77,372 | NO | max-turns; fact-gate round-trips + ledger/HEAD reconciliation curiosity; census data had been reached mid-run |
| abench-2 | 123.24 | 7 (max) | 51,639 | 86,454 | NO | same shape; fact-gate denial counts equal (1-2) in all runs |
| abench-3 | 137.04 | 6 | 51,644 | 90,666 | YES | "55 live gates... census reports 57 tool files, 36 wired and 21 hand-run, exit 0" — CORRECT |
| abench-4 | 167.06 | 7 | 51,640 | 52,953 | YES | "Live gates: 55. Census: 57 tool files, 36 WIRED + 21 HAND-RUN" — CORRECT |

Correctness: answered runs both CORRECT (55 = the bench worktree's filtered
ledger count; census line byte-equal). All four after runs carried 1-2
GateGuard fact-forcing denials, same as both before runs: the after harness
still enforces gates; the no-answer pair is turn-budget variance, not a
removed capability.

## Before/after table (the done-means rows)

| key | before | after |
|---|---|---|
| session_start_tokens | 51297 | 51151 |
| bench_wall_ms | 118370 / 244720 (answering runs; mean 181545) | 91630 / 123240 unanswered; 137040 / 167060 answered (answered mean 152050) |
| bench_input_tokens | first-turn 51007 / 51252 (mean 51129); aggregate 109455 / 90049 (mean 99752) | first-turn 51840 / 51679 / 51644 / 51640 (mean 51701); aggregate 77372 / 86454 / 90666 / 52953 (mean 76912, -23%) |
| skills | 253 | 253 |
| agents | 68 | 65 |
| stop_hooks | 25 | 7 |
| mcp_tools | 15 exposed headless (6 servers) | 17 exposed headless (3 servers; brave-search now connects: +2 brave tools) |

Row semantics: session_start_tokens = first-turn input_tokens of the 'Say
OK' headless run in the repo cwd. bench_input_tokens = per-run first-turn
and aggregate input. stop_hooks = Stop-event entries across all settings
sources plus the claude-mem plugin manifest. mcp_tools = tools with mcp__
prefix in the session init event.

## Hook end state (re-inventoried with the same script as before)

  global settings: 23 entries (was 75): Stop 4 (unlazy, plan-canvas-pending,
  ecc session-end, cost-tracker), PreToolUse 11, PreCompact 1,
  SessionStart 2, PostToolUse 2, PostToolUseFailure 2, SessionEnd 1.
  soccer 8, jiheeye 1, claude-mem plugin 8: unchanged.
  TOTAL: 40 entries (was 94); Stop-event entries: 7 (was 25).
  Per-turn-end hook cost: the removed chain took 16.3s of children and two
  duplicate passes; desktop-notify alone was 9.3s of it.

## Verdicts
  ● measured: all figures above (commands and raws kept)
  ◑ believed: per-turn latency tracks the removed hook spawns (the Stop
    chain 3x duplication and the 27-spawn PreToolUse fan-out are gone);
    hook-level runtimes were not re-timed individually after the change
  ○ unchecked: jiheeye-side behavior of the lean harness (its own hooks
    untouched; its disk-guard selftest runs in the quality bundle)
  ★ fragile: /tmp tmpfs full (111 MB free, 96%) blocked worktree creation;
    the AFTER worktree lives on /mnt/d/scratch (D:).