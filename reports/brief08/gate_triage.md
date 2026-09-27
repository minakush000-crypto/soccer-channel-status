# gate_triage.md — LIVE vs FROZEN decision for every gate (brief 08 job 2)

Decision rule (Mayo, brief 08 closed decision 2): LIVE = an invariant that
must stay true on every future commit and is re-runnable by `--reverify`.
FROZEN = a snapshot of a past state (durations, dates, copied counts,
one-time deletions/proofs) — moved verbatim with its evidence to
reports/<its brief>/GATES_frozen.md, headed "historical, not run".

Why the split exists: brief 06 reported "33/33 gates pass" from
`gate-check`'s default run, which only executes gates whose boxes are
unchecked; the 28 pre-checked gates were trusted from stored evidence. The
coordinator re-ran four by hand on 2026-09-27 and all four FAIL today
(G1 py=37 vs 35, G4 140.920000 vs 140.960000, G7 stamp 2026-09-22 vs docs
now 2026-09-27, B14 5 stamps vs 1). The brief-06 work itself checked out
where tested; the ledger was the failure.

## Brief 03 gates (G1-G12)

| Gate | Decision | Reason |
|---|---|---|
| G1 | FROZEN | tool-count snapshot (37 at brief 03; the live count invariant is census B20) |
| G2 | LIVE | no tool may reference a retired tool as live — true on every future commit; rewritten with a positive control |
| G3 | FROZEN | brief 03's momentum-board deployment artifact (volume + local paths of that era) |
| G4 | FROZEN | duration snapshot (v3 140.92s; v7 is 140.96s) |
| G5 | FROZEN | one-time CLAUDE.md replacement proof |
| G6 | FROZEN | one-time global CLAUDE.md rewrite proof (global files are .proposed-only now) |
| G7 | FROZEN | date-stamp snapshot (2026-09-22; FAILS today because the docs were honestly restamped) |
| G8 | FROZEN | one-time deletion proof; absence check with no positive control |
| G9 | FROZEN | global hook-count snapshot; breaks by design when hooks are added (removed from the live ledger per job 4) |
| G10 | LIVE | the two entry points must always parse (ast smoke) |
| G11 | FROZEN | checks brief 04's B2 archive folder, not current archives |
| G12 | FROZEN (replaced) | ran `git add -A` inside CHECK; replaced by B22 CLEAN-PUSHED which stages nothing |

## Brief 04 gates (B1-B15)

| Gate | Decision | Reason |
|---|---|---|
| B1 | FROZEN | one-time proof PNGs for the row-mirror fix |
| B2 | FROZEN | word-only grep for a comment; the retime behavior was proven by brief 04's dense frame scan |
| B3 | LIVE | one 2D renderer: renderer files exist, zero matplotlib imports — rewritten with a positive control |
| B4 | FROZEN | word-only wiring grep; the wiring runs on every pipeline build |
| B5 | FROZEN | word-only grep on push_status.sh; the scan is proven by brief 08 job 1's dry run + controls |
| B6 | LIVE | the source repo must stay private (gh, re-runnable) |
| B7 | FROZEN | word-only grep inside brief 04's own report text |
| B8 | LIVE | the episode slug must stay a CLI argument, never a module constant — rewritten with a positive control |
| B9 | LIVE | gate evidence logs must stay git-ignored — rewritten with a positive control (a tracked file is NOT ignored) |
| B10 | FROZEN | one-time script archive move |
| B11 | FROZEN | word-only grep for the judge's --runs flag |
| B12 | FROZEN | one-time proof frames for the reel t=90 resolution |
| B13 | FROZEN | one-time cleanup deletions (GEMINI.md, LANE_PLAN.md copy, q files) |
| B14 | FROZEN | date-stamp snapshot (2026-09-23; FAILS today for the same honest reason as G7) |
| B15 | FROZEN (replaced) | ran `git add -A` inside CHECK; replaced by B22 CLEAN-PUSHED |

## Brief 05 gate (B16)

| Gate | Decision | Reason |
|---|---|---|
| B16 | LIVE | the Madrid specs must keep matching the raw data (executable check, re-runnable) |

## Brief 06 gates (B17-B21)

| Gate | Decision | Reason |
|---|---|---|
| B17 | LIVE | Sofascore is the only facts source and match_data.py stays retired — true on every future commit |
| B18 | LIVE | BOTH episodes' specs must keep matching the raw data (executable, re-runnable) |
| B19 | LIVE (REWRITTEN) | word-only before (greps + log files); now RUNS the page check via tools/pagecheck_proof.sh: a deliberately mirrored input must FAIL, the real input must PASS |
| B20 | LIVE | the census must match tools/ on every future commit |
| B21 | FROZEN | snapshot of the v7 rebuild (a duration plus frame files) |

## New gate (brief 08 job 4)

| Gate | Decision | Reason |
|---|---|---|
| B22 | LIVE (new) | replaces G12+B15: work committed AND pushed, and the CHECK stages nothing |

## Totals (counted, not estimated)

$ ls reports/*/GATES_frozen.md
- brief03: 10 frozen, brief04: 11 frozen, brief05: (none), brief06: 1 frozen → 22 frozen
- live in the new GATES.md: G2, G10, B3, B6, B8, B9, B16, B17, B18, B19, B20, B22 → 12 live
- 22 frozen + 11 original live = 33 accounted for; B22 is the one replacement gate.