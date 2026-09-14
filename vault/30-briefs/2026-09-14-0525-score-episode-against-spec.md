# New task — score the episode against EPISODE_SPEC

## Scope

The episode was never scored. Board scores are not an episode score. The production freeze lifts on an episode scoring 7/10 against EPISODE_SPEC, and that number has never once been produced.

## Sources and inputs

- `renders/2026-09-12_bournemouth-brentford/final_video.mp4`
- `EPISODE_SPEC.md` — every numbered section, not only the measurable ones
- `~/claude/40-lessons/viz-design-findings.md` — the four design properties
- `STATUS.md` and `DECISIONS.md` — stamps reported before answering

## What to produce

Score the episode through Gemini as authoritative:

- Extract a frame at 2s, and at the midpoint of every board segment and every footage segment
- Judge the opening 5 seconds against the hook rule and the never-open-on-a-static-board rule
- Score each numbered section of EPISODE_SPEC
- One overall score out of 10, with the three biggest failures named
- Paste every raw Gemini response

Also report:
- Whether the voice track was rendered from the verified script or carries the old narration
- Whether the words match the pictures across the 718 seconds, sampled at each section boundary

If it scores 7 or above, the freeze decision is Mayo's. Do not upload.

**Recommend, do not act:** the 3D formation board is the lowest-scoring board in the pool at 6/10, on a renderer Stage 12 proved is not reliably reproducible. Keep or retire?

## STANDING RULE — learn from the best before designing anything

Before executing any task that involves a design, a format, a structure or a method, first study how the people who do it best actually do it. Do not design from first principles when practitioners have already solved it.

Sequence, every time, before any building:

1. **Check the extraction already done.** `~/claude/40-lessons/viz-design-findings.md` holds the reference-visualisation study (20+ chart types, narrative ranking, four design properties). If the task is covered there, use it and say so. Do not re-derive what is already filed.

2. **If it is not covered, extract.** Use the yt-digest multi-platform research pipeline with transcript handling. Pull transcripts and frames from the best practitioners in that specific area. Enumerate what they do.

3. **Scour beyond the given sources.** Web search, fetch, connectors, MCPs, research tools, whatever applies. Find related work on that exact task. Cap at five additional sources and report why each was chosen.

4. **Infer the principle, not the artefact.** Extract what they chose to show and why, what they left out, what makes it readable. Never reproduce a design, a layout or a frame. Principles travel; copies do not.

5. **Report before building.** State what you learned, which practitioners it came from, and how it changes the approach. Then build.

This applies to visual design, script structure, pacing, sourcing, tooling choices and workflow design. Not only to charts.

If nothing relevant exists to learn from, say so explicitly and proceed. Saying "nothing found" is a valid answer; skipping the step is not.

**Record this rule in DECISIONS.md and in CLAUDE.md as binding, so it survives this session.**

## Why it matters right now

The freeze has held since the spec was written. Every episode has been judged on parts (runtime, boards, shot count) and never as a whole. Seven boards scoring well is not an episode scoring 7.

## Rules

One change per run. Paste real command output, never a summary. If you do not know, say unknown. Check the artifact, not the report.

## Carry-forward uncertainty set

**★ fragile**
- `scene_gen.py` is not reliably reproducible; the 3D board is the lowest of the seven.
- Two mirror allowlists must stay in agreement.
- The stamp rule depends on being remembered; `doc_stamp_check.py` is the only backstop.
- ext4.vhdx stays at 36.82 GB until compaction.

**◑ believed**
- The episode metrics are accurate (ffprobe, not independently re-run).
- Design, not dimension, is the cap. Supported by artifacts.

**○ unchecked**
- The episode's overall score.
- Whether the voice carries the verified script.
- Whether words match pictures across 718s.
- Total spend across RunPod, Vast, Modal, ElevenLabs.
- Whether any of the 127 skills has ever fired.

## Doctrine footer

1. Nothing raw inside `/home/muads`. Staged on `/mnt/f`, shipped to the pod, deleted locally. No local GPU work. Extended: everything produced by any task is archived to B2 with a provenance manifest.
2. No tasking without calling the available tools, skills, connectors, web fetch, research or MCPs where they apply.
3. No orphaned, standalone or untested item may exist. Everything is wired, tested, or retired. Everything is wired, tested, or retired. This governs what happens after a tool exists; it never forbids building one the work needs.
4. Every brief carries this doctrine as a footer.
5. NO DEAD ENDS. Report the block, the alternatives considered, and which was chosen.

**Gemini is the authoritative visual judge.** It judges what it can see. It does not generate timestamps, pitch coordinates, or cut decisions. Those stay mechanical.

**CONDITIONAL — the litmus test.** If an episode built under this rule still fails 7/10, run the Gemini-vs-scanner head-to-head on cut timestamps against `match_data.json` before delegating. Delegate only if Gemini wins.

**PRECEDENCE.** Code and filesystem first, DECISIONS.md second, all other documents third.

**STAMP RULE.** Any task editing a canonical document updates its stamp having verified it, or reports which claims could not be verified and leaves the stamp unchanged.

**SCRIPT RULE.** No voice render without `.script_verified`. Every factual claim traces to `match_data.json` or `sources.json`.