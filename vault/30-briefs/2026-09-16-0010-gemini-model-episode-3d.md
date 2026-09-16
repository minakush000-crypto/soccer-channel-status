TITLE: Best available Gemini, a real Gemini-built episode, and 3D as the standard

SCOPE
Three pieces. (1) Find out what Gemini models this key can actually reach and
move off the preview endpoint. (2) Build a complete episode end to end from
Gemini's own decisions and put the file on screen next to the mechanical one.
(3) Treat 3D as the decided direction for positional boards and report what it
takes to get it to the same design standard as the 2D boards.

Read STATUS.md and DECISIONS.md first and report their stamp dates.

PIECE 1 — WHICH MODEL
List every model the GEMINI_API_KEY can actually reach. Paste the raw list, not a
summary. Report for each candidate: exact model string, stable or preview,
whether it accepts video input, and price per million tokens.
Then say plainly which is the most capable available to this key for
(a) video understanding and (b) image judging, and whether they are the same
model. gemini-3.1-pro-preview is a PREVIEW endpoint; move to the stable
equivalent if one exists and say what changed.
Re-run the 9 board judgments on the chosen model, unchanged boards, and paste
every raw response. Then run the SAME 9 judgments THREE times and paste all
three sets. The same board files scored 10/10 at 05:45 and 7/10 at 06:10 with
no pixel changed. Report the spread per board. If the spread is more than 2
points, a single score is not a measurement and the freeze gate needs a median
of three, not one number.

PIECE 2 — THE GEMINI-BUILT EPISODE
Build a complete, watchable episode where every mechanical decision comes from
Gemini and nothing is corrected by the mechanical tools.
- Gemini picks the goal timestamps.
- Gemini picks the cut windows.
- Gemini supplies the coordinates for any positional board.
- Gemini writes nothing factual that is not its own: do NOT substitute
  match_data.json values where Gemini is wrong. Let the errors through.
- Reuse the same voice and the same board renderer so the only variable is
  the decision source.
Render it to a separate folder. Do NOT overwrite the existing episode.
Produce: the file path, ffprobe output, and a second-by-second list of where
the Gemini episode and the mechanical episode differ.
Then score BOTH against EPISODE_SPEC with the model chosen in Piece 1, and
paste every raw response.
Expect it to narrate a goal that did not happen: Gemini named Mbeumo, who is
not in this match, and placed a goal at 97s where the scoreline never changed.
That is the point of building it. Do not clean it up.
Report the cost.

PIECE 3 — 3D IS THE DIRECTION
Decision taken by Mayo, not open: positional boards go 3D. Do not re-argue it.
Report what it costs to make the 3D formation board meet the same standard the
2D boards already meet.
- It scores 4-6/10 and the stated faults are: no title, no explanatory text,
  standard sans-serif instead of Bebas Neue / Barlow Condensed, flat tokens,
  compressed layout, no shared design layer.
- board_design.py is the shared design layer the 4 high-scoring 2D boards use.
  Report whether it can be applied to a Blender render, and if not, what would
  have to exist.
- scene_gen.py produced a broken render from unchanged code at Stage 12
  (camera inside the scene, zero players). Report the cause or report that it
  is unknown. A renderer that silently breaks cannot be the standard.
- Report per-render cost and wall time on Modal, and what the 3D lane costs
  per episode if every positional board goes 3D.
Then recommend: fix scene_gen, or replace it with a different 3D path
(Three.js in headless Chromium is one, name others). One recommendation, with
the reason, with numbers.

CONSTRAINTS
One change per run. No claim without pasted command output. Report the age of
every input file. Brief to ~/claude/30-briefs/, report to ~/claude/20-handoffs/,
push_status.sh run explicitly after both, decisions into DECISIONS.md.

CARRY-FORWARD UNCERTAINTY SET
★ fragile
- Gemini board scores moved up to 3 points on unchanged files between two runs
  five hours apart. The freeze gate rests on one such score.
- gemini-3.1-pro-preview is a preview endpoint subject to deprecation.
- scene_gen.py is not reliably reproducible and is still wired into
  step2_boards.
- The new voice hash gate depends on .voice_script_hash surviving.
- Two mirror allowlists must agree or files vanish silently.
- ext4.vhdx 36.82 GB, grows and never shrinks; compaction written, not run.
- Doctrine rule 1 violated by the pipeline; rule 3 by 4 dead and ~18 untested
  tools. 127 unaudited skill folders.
- STATUS.md was stamped 2026-09-13 with no episode score in it until this
  session's update.

◑ believed
- The audio says Schade not Thiago. Hash proves the source text, not the sound.
  No speech-to-text exists in this pipeline.
- 137 WPM is a real regression and not a measurement artifact.
- gemini-3.1-pro is the most capable Gemini available to this key. Not
  enumerated. Piece 1 settles it.

○ unchecked
- Nobody has listened to the 794s voice track end to end.
- Nobody has read the 1819-word script end to end.
- Why the voice got 75s longer on fewer words.
- Why the Shorts crop failed in the re-assembly (marked non-fatal, unexamined).
- EPISODE_SPEC §2 was scored down for "longest 27s (below 30s min)", which
  reads like a spec misreading. Check it.
- Total spend across RunPod, Vast, Modal, ElevenLabs.
- Whether /mnt/f's EINVAL is resolved.
- Whether a MUST failure disqualifies. §10 failing dropped 7.27 to 5.5; §7
  failing left 7.5 standing. Same rule, two outcomes. Settle it in
  DECISIONS.md before any score is trusted.

DOCTRINE FOOTER
1. Nothing raw inside /home/muads. Staged on /mnt/f, shipped to the pod,
   deleted locally. No local GPU work. Everything produced is archived to B2
   with a provenance manifest.
2. No tasking without calling the available tools, skills, connectors, web
   fetch, research or MCPs where they apply.
3. No orphaned, standalone or untested item may exist. Wired, tested, or
   retired. Missing capability is a reason to build, not a reason to stop.
4. Every brief carries this doctrine as a footer.
5. NO DEAD ENDS. Report the block, the alternatives considered, and which was
   chosen.
PRECEDENCE: code and filesystem first, DECISIONS.md second, all other
documents third.
LEARN FROM THE BEST BEFORE DESIGNING ANYTHING.