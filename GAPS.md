# GAPS.md — what nobody has verified

> **Purpose:** registry of unverified claims and open uncertainties.
> **Reader:** every session.
> **Last verified against code:** 2026-09-27 (brief 06).

This file existing and being short is a warning sign, not a success. Each
entry is something that could be true or false and nobody has run the command
to find out. If you verify one, move it to STATUS.md or DECISIONS.md and date
it.

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
4. **Bournemouth's raw graph/shotmap/average-positions are gone forever**
   (Sofascore retired the event's sub-endpoints, measured 2026-09-26): the
   momentum curve, shot positions and xG series on its boards can never be
   re-verified against raw data (checker reports them UNVERIFIABLE). Any
   Bournemouth re-render must reuse the existing specs; the emitters will
   refuse to regenerate them without raw data.
5. **The Sofascore event-id resolver is IP-limited.** Search + calendar APIs
   return empty from this machine; new episodes resolve through the
   accumulated team-id sweep or need --event-id by hand. Whether the search
   API works from a different network is untested (○).
6. **3D board label collisions** (formation_clash/valverde_strike name
   labels overlap in the opened frames, 2026-09-27). Cosmetic design debt,
   same class as the shotmap one.

## Closed this pass (moved to STATUS/DECISIONS, dated)

- Invented momentum board (brief 06 job 2): the EP001 momentum spec was
  hand-typed in a bash heredoc (session 50fa196e, 2026-09-21T05:08:38Z) —
  25 invented points, goal at min 13 labelled "MBAPPE 14'". Regenerated from
  the real Sofascore /graph (92 points) with goal markers from raw incidents.
- Unverifiable spec numbers (brief 06 job 3): every numeric leaf of every
  spec now checks against the RAW responses; specs without source +
  fetched_at fail; specs and the raw cache are tracked in git.
- Right-numbers-wrong-side passing checks (brief 06 job 4): the renderer
  now audits the drawn page against the facts file before writing frames
  (mirror proof fails, real renders pass).
- Hand-written TOOLS.md drift (brief 06 job 5): census is a script + gate
  (B20); the drift it caught included a missing sofascore_client row and a
  never-counted board_page.html.
- The "14 specs" data-check count (brief 06 job 7d): the old checker counted
  every spec twice (checked += 1 twice in its loop); Madrid has 7.
- ESPN as the fact source (brief 06 job 1): eng.1-only, could not fetch a
  Champions League match; Sofascore wired for every competition.