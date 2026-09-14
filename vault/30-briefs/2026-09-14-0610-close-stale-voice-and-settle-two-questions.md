# Close the stale-voice hole, then settle two open questions with evidence

**Brief date:** 2026-09-14
**Stamps reported:** STATUS.md 2026-09-13, DECISIONS.md 2026-09-14

## SCOPE
Three pieces of work, in this order, one change per run. (1) Re-render the
voice from the fixed script and prove the episode audio no longer names a
fabricated scorer. (2) Extract, from real practitioners, what medium is
actually used for football tactical boards and why, using the yt-digest
multi-platform research pipeline with transcript handling. (3) Run the
litmus head-to-head that the 5.5/10 score has now triggered: Gemini vs the
scoreboard scanner on mechanical output, judged against match_data.json.

Before anything: read STATUS.md and DECISIONS.md and report their stamp
dates in your first line. STATUS.md as fetched 2026-09-14 is stamped
2026-09-13 and contains no record of the episode score, the 5.5/10 verdict,
or the voice failure. Fix that under the STAMP RULE as part of piece 1.

## PIECE 1 — THE VOICE
Delete or move aside the existing voice artifact for
renders/2026-09-12_bournemouth-brentford. Re-run step6_voice from the
fixed, .script_verified script (1751 words). Then prove it:
 - paste the word count of the script file used
 - paste ffprobe duration of the new voice file and of the old one
 - paste the modification timestamps of script, .script_verified, and voice
   file, in order, showing the gate now precedes the voice
Then fix the skip logic itself: step6_voice must compare the script hash
recorded at voice-generation time against the current script hash, and
regenerate on mismatch. "Voice file exists" is not a valid skip condition
and caused this failure. Paste the diff.
Then re-assemble and re-score the episode against EPISODE_SPEC in full,
same method as the 5.5 run, pasting every raw Gemini response.

## PIECE 2 — WHAT PRACTITIONERS ACTUALLY USE
Question to answer: for football tactical analysis graphics, what medium do
the best practitioners use for each board type, and what reasons do they
give? Specifically: is any positional board (formation, average positions,
pass network) rendered in 3D by anyone credible, or is 3D confined to
broadcast pre-match sequences?

Sources and enumeration: start from the named benchmark channels (Football
Meta, Football Made Simple, DK FALCON, Mega Football, Ball Explained,
AlfonsoR10) plus Coaches' Voice. Then scour beyond them, cap at five extra,
report why each was chosen. Include tooling sources where practitioners
describe their own setup (studio breakdowns, mplsoccer / Tableau / After
Effects workflows, Opta and StatsBomb visual style guides).

Check ~/claude/40-lessons/viz-design-findings.md FIRST and say so if this is
already covered. "Nothing found" is a valid answer; skipping is not.

Mine for, in a table: board type | medium used (2D static, 2D animated, 3D)
| tool named if any | stated reason | source and timestamp.

Infer the principle. Never reproduce a design or a frame.

Then settle the internal contradiction on record. STATUS.md 2026-09-13 says
non-buggy 3D scored 6/10 against a 2D control at 4/10, three judges agreeing,
3D ahead by 2. The current handoff says 2D beats 3D because momentum (2D
timeline, 9/10) beats formation (3D, 6/10). Those are different board types
and do not compare. Report which claim survives, and state plainly whether
the same board with the same data has ever been rendered both ways. If it
has not, say so; do not construct the comparison from unlike boards.

## PIECE 3 — THE LITMUS HEAD-TO-HEAD
The trigger condition is met: the episode scored 5.5/10, below 7, built
under the mechanical rule. Run it.
Same episode, same footage. Gemini proposes cut timestamps for every goal
and every named passage. The scoreboard scanner proposes them. Both are
compared against match_data.json. Report per-goal offset in seconds, misses,
and phantoms, for each.
Extend beyond timestamps, which is the part never tested: have Gemini
produce pitch coordinates for one average-positions board and one cut
decision list for one segment. Compare the coordinates against the same
board's mechanical coordinates. Paste both outputs raw, side by side.
Note for context: the 2026-09-08 test found Gemini timestamps 1-2s early,
goal windows bloated, 2 of 3 goals found, 1 phantom invented, against the
scanner at 3 of 3, free. That was one clip on gemini-3.1-pro-preview.
Report the model version you use now. If the result reverses, say so.
Delegate to Gemini only where it wins on the pasted numbers.

## WHY THIS MATTERS NOW
The freeze turns on one number. The episode is at 5.5/10 and the single
disqualifier is a 719.1s audio file generated from a 1911-word pre-fix
script, while the fixed script is 1751 words. Piece 1 is the only work that
moves the number. Pieces 2 and 3 decide where the next twenty hours of
board work go, and both currently rest on claims that contradict each
other in the canonical docs.

## CONSTRAINTS
One change per run. No claim without pasted command output. Reading code and
concluding what it does is ◑, not ●. Report the age of every input file you
intend to use and never silently reuse an input older than the run.
Every brief and report is a file: this brief to ~/claude/30-briefs/, the
report to ~/claude/20-handoffs/, push_status.sh run explicitly after both,
every mid-task decision into DECISIONS.md with a date before work continues.

## DOCTRINE FOOTER
1. Nothing raw inside /home/muads. Staged on /mnt/f, shipped to the pod,
   deleted locally. No local GPU work. Everything produced by any task is
   archived to B2 with a provenance manifest.
2. No tasking without calling available tools, skills, connectors, web
   fetch, research or MCPs where they apply.
3. No orphaned, standalone or untested item may exist. Wired, tested, or
   retired.
4. Every brief carries this doctrine as a footer.
5. NO DEAD ENDS. Report the block, the alternatives considered, and which
   was chosen.
PRECEDENCE: code and filesystem first, DECISIONS.md second, all other
documents third.
LEARN FROM THE BEST BEFORE DESIGNING ANYTHING.