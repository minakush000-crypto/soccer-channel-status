# GAPS.md — what nobody has verified

> **Purpose:** registry of unverified claims and open uncertainties.
> **Reader:** every session.
> **Last verified against code:** 2026-09-23 (brief 05).

This file existing and being short is a warning sign, not a success. Each
entry is something that could be true or false and nobody has run the command
to find out. If you verify one, move it to STATUS.md or DECISIONS.md and date
it.

Rewritten 2026-09-23 (brief 05): the brief-04 gap list closed (tpad holds were
a doc-level model — the real defect was window overrun, fixed; mirror filter
shipped; reasoning_effort drop demonstrated; migration phases 5-8 retired as
never-defined; reel t=90 resolved as a saved shot).

## Open gaps (current)

1. **Judge wobble is a model property, unmitigated beyond median-of-N.**
   Temperature 0 + fixed seed still gave 6,8,6,6,8 on one board (5 runs,
   2026-09-23). glm_judge.py --runs N reports the median; single-shot scores
   are not load-bearing. Whether any gateway setting reaches the cloud model
   is still unknown (○).
2. **Long-lane format future.** The 8-14 min lane format has not produced a
   benchmark episode; EP001's 140s format is current. Owner decision.
3. **Shotmap board polish:** the team-name side labels overlap shot dots near
   the goalmouths. Cosmetic.
4. **"3 POINTS" outro chip is a league-table fact** no match-data file can
   adjudicate; the data check verifies only goals chips and scorelines. If a
   league-table facts source ever matters, it needs its own file.

## Closed this pass (moved to STATUS/DECISIONS, dated)

- Stats rows mirrored (brief 05 job 1): rows drew away values under the home
  name; fixed home-left/away-right; every other board kind swept for the
  same inversion (bar, scorebug, momentum, xgflow, shotmap, avgpos, outro —
  correct; stat_card/possession shared the bug and were re-rendered).
- Momentum fill off the line (brief 05 job 2): clamped max/min(0,v) fill
  replaced with same-sign runs split at interpolated crossings; y-scale
  restored; key placed inside the plot; brief 03 label fix kept.
- No facts file (brief 05 job 3): match_data.json fetched from Sofascore
  (tracked, source_url + fetched_at) and tools/board_data_check.py added as
  produce_v2 step 1d + gate B16. The spec numbers all matched the real data.
- scripts/ untracked (brief 05 job 4): tracked, 17 files, no secrets.
- Legacy vault files in the mirror (brief 05 job 5): 16 files scanned clean,
  removed from HEAD, history kept.
- Spec namespace collisions (brief 05 job 8): per-slug /vol/specs and
  /vol/out folders + slug refusal in render2d/render3d/assemble.
- Two 3D engines (brief 05 job 7): three_scene.js + three_render3d.py
  retired (v1 needs a puppeteer install that exists nowhere); v2 kept.

## Closed in brief 04 (moved to STATUS/DECISIONS, dated)

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