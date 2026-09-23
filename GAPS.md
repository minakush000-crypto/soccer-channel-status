# GAPS.md — what nobody has verified

> **Purpose:** registry of unverified claims and open uncertainties.
> **Reader:** every session.
> **Last verified against code:** 2026-09-23.

This file existing and being short is a warning sign, not a success. Each
entry is something that could be true or false and nobody has run the command
to find out. If you verify one, move it to STATUS.md or DECISIONS.md and date
it.

Rewritten 2026-09-23 (brief 04): the brief-03 gap list closed (tpad holds were
a doc-level model — the real defect was window overrun, fixed; mirror filter
shipped; reasoning_effort drop demonstrated; migration phases 5-8 retired as
never-defined; reel t=90 resolved as a saved shot).

## Open gaps (current)

1. **Judge wobble is a model property, unmitigated beyond median-of-N.**
   Temperature 0 + fixed seed still gave 6,8,6,6,8 on one board (5 runs,
   2026-09-23). glm_judge.py --runs N reports the median; single-shot scores
   are not load-bearing. Whether any gateway setting reaches the cloud model
   is still unknown (○).
2. **/vol/specs is a single-episode namespace.** Same-named specs from two
   episodes collide (it happened 2026-09-23: a Bournemouth momentum board
   rode into an EP001 assemble). pod_build needs per-slug spec paths.
3. **Two 3D scene engines coexist.** three_scene.js (produce_v2 formation via
   three_render3d.py) vs three_scene_v2.js (flash named boards). Both wired,
   overlapping capability. Consolidation unstarted.
4. **scripts/ is gitignored** (soccer-channel/.gitignore:4). Episode scripts
   exist only locally + on the Modal volume; the input of record is not in
   git. Owner decision: track scripts/ or accept the risk.
5. **Long-lane format future (was #7).** The 8-14 min lane format has not
   produced a benchmark episode; EP001's 140s format is current. Still an
   owner decision.
6. **Legacy vault files in the public mirror** (17 files published before the
   brief 04 lock). Removal from the mirror HEAD is Mayo's call; history
   rewrite is explicitly off the table without his decision.
7. **Shotmap board polish:** the team-name side labels overlap shot dots near
   the goalmouths (visible in board_shotmap_bournemouth_new.png). Cosmetic.

## Closed this pass (moved to STATUS/DECISIONS, dated)

- tpad holds (was #3): the tpad filter was DEAD CODE in this invocation; the
  real defect (footage overrunning verified windows by 0.02-1.13s) is fixed
  by the retime pass (brief 04 job 2, verified by dense frame scan).
- reasoning_effort drop (was #2): demonstrated — Ollama's OpenAI shim accepts
  the parameter and silently discards it (200 OK, no effect, 2026-09-23).
  Mitigation shipped: judge --seed + --runs N with the median reported.
- Public mirror content filter (was #5): allowlist + secret scan shipped and
  block-tested (brief 04 job 4).
- Migration phases 5-8 (was #6): RETIRED — the plan is not defined in any
  recoverable source (live docs, full git history, retired tarball, claude-mem
  all searched 2026-09-23); the only named phase ("Phase 5 zero-Gemini") is
  complete. Superseded by the flash pipeline.
- Reel t=90 possible disallowed goal (was #4): resolved as a SAVED Inter shot
  (score bug 0-0 throughout, ball never over the line, no referee signal;
  frames reports/brief04/goal90_*). The pipeline counts 3 goals — correct.
  DESIGN_FLASH.md corrected 2026-09-23.