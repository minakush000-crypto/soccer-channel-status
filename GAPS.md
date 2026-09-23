# GAPS.md — what nobody has verified

> **Purpose:** registry of unverified claims and open uncertainties.
> **Reader:** every session.
> **Last verified against code:** 2026-09-22.

This file existing and being short is a warning sign, not a success. Each
entry is something that could be true or false and nobody has run the command
to find out. If you verify one, move it to STATUS.md or DECISIONS.md and date
it.

Rewritten 2026-09-22 (brief 03): the pre-flash gap list (3D renderer
unverified, Gemini CLI no-hooks exposure, old produce_v2 outputs) is closed
or moved to history. EP001 was built and judged through the flash pipeline;
the judge is the session model; the census lives in STATUS.md.

## Open gaps (current)

1. **Judge wobble is uncharacterized beyond one frame.** Measured ±1-2 on
   one frame (nine runs, 3-5). Whether wobble is worse on busy boards, on
   footage frames, or across judges is unknown. Load-bearing scores need
   repeat judging with the spread reported.
2. **reasoning_effort does not reach the model through Ollama.** Known
   (brief 03 "still open"); untested whether any gateway setting does.
3. **tpad holds on 4 of 12 EP001 footage sections** (0.1-1.1s, worst
   seg12 1.1s): verified present by duration math (voice-proportional
   allocation vs window length), not yet fixed. Whether viewers notice
   besides the owner is unknown.
4. **Unclaimed possible disallowed goal at reel t=90.** If confirmed, the
   goal3 narration and the goal3_pattern board need a correction pass.
5. **Public mirror content filter.** push_status.sh publishes vault
   markdown with no content filter (only files literally named .env are
   deleted). What else in the vault is publishable is an owner decision.
6. **migration phases 5-8** — unfinished, priority superseded by the
   benchmark-first approach.
7. **produce_v2's lane format vs the flash short format.** The 8-14 min
   lane format (validate_script word minimums) has not produced a
   benchmark-passing episode; EP001 (365 words, 140s) is the current
   format. Whether the long format is still wanted is an owner decision.

## Closed this pass (moved to STATUS/DECISIONS, dated)

- 3D renderer verification: closed — EP001's 3D boards rendered and judged
  through three_scene_v2.js on Modal (STATUS.md).
- Hooks exposure: closed — the harness is Claude Code again; hooks fire.
- B2 migration open items: closed — archive-before-delete ran (retired
  tools tarball, EP001 v3 final, both with manifests).
- parse_feed.py / player_mapper.py / sharpness_check.py orphans: closed —
  retired 2026-09-22 (rule 3 satisfied by deletion, not wiring).