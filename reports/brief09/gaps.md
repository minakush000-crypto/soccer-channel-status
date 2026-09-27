# Brief 09 — gap list (job 7)

Each line: something the benchmark videos do, whether our pipeline has a tool
for it today, and the evidence. Facts only; the pipeline direction is the
coordinator's to write. Evidence markers: frame references are
`<video> t=<sec>` (frames_<L>.csv row or grid cell viewed this brief);
tool facts come from tools/ TOOLS.md rows as of 2026-09-27 (census 43 files:
28 WIRED + 15 HAND-RUN).

## What the benchmarks do that we have no tool for

1. **Tactical overlays drawn on top of match footage** (arrows, rings, zones,
   labels, connection lines that follow real players). The benchmark's single
   most common device: A t=30 (yellow discs under players), A t=334-336
   (multi-color glow ellipses + connection lines), A t=444-448 (red under-rings
   + yellow pressing polygon), B t=108-110 (red rings + shaded triangles +
   position labels inside a rounded card), B t=288-292 (press triggers on
   footage), C t=0-2 (MESSI label + red triangle), C t=90-92 (white tracking
   arrow + green badge). Our pipeline renders boards as separate segments and
   cuts them against footage (`assemble_manifest.json` sections are typed
   `board` or `footage`; pod_build.py assemble concatenates); no tool composes
   a drawn overlay onto a footage frame. Closest: board_html.js/board_page.html
   (canvas drawing, but on its own page, not over a video frame).

2. **Overlay animation that tracks play** (arrows fade in/out, rings appear
   player by player, shapes morph). A t=150-158 (red press rings appear
   sequentially on the teal board), A t=324-332 (yellow shape morphs
   circle->rectangle->pentagon->triangle while players hold), C t=90-94
   (arrow follows the player then fades), B t=0-12 (quote text builds word by
   word). board_html.js renders one frame per spec state; a timeline of
   overlay states per second does not exist in any spec format today.

3. **Animated match-replay boards** (a ball dot travels across a static
   formation while labels stay put). B t=360-376, an entire sequence captioned
   "VS RAYO VALLECANO" where the ball moves possession to possession with blue
   dashed pass options appearing. No board kind in board_page.html implements
   a moving ball or a pass sequence; specs are static per-frame states.

4. **3D demo scenes with figures** (mannequin-style players acting out a duel
   or a pressing trigger, camera close-ups). A t=224-232 (close 3D renders of
   two players contesting, "?!!" callout), A t=368-376 (full 3D rendered match
   replay with marker ellipses). three_scene_v2.js renders 3D pitch boards
   (dots/spheres on a pitch); there is no 3D player-figure asset or scene
   grammar for body animation.

5. **Chapter/text cards with staged reveals** (title cards, thesis cards,
   principle cards that fade in line by line). A t=54 ("The Complete
   Discipline" chapter card), B t=74-76 ("PRESS HIGH / DEFEND HIGH / ATTACK
   VERTICALLY" appearing one line at a time), B t=116-118 ("DIRECT VERTICAL
   RUNNERS" over a blurred photo). Our outro board is a static frame; no
   text-card spec with staged animation exists.

6. **Cutout photography segments** (player cutout cards that accumulate,
   polaroid stacks of coach-player pairs, name cards with giant type). B
   t=296-304 and t=442-448 (cutouts accumulate), C t=272-282 (photo stack of
   five coach-player pairs), B t=450 ("BASTONI" giant-name card). No tool
   composites photographic assets; our renders are canvas/SVG only.

7. **Coach and player audio as punctuation** (interview clips with burned-in
   captions carrying the section's argument: A t=44-46, 98-116, "Credit: The
   PFA"). Our audio path is generate_voice.py (narration) + ambience; there is
   no tool to source, license-check, cut or mix third-party clip audio.

8. **Sourcing footage from many matches per episode.** The closed decision
   (Mayo, brief 09) is 2-4 matches per episode. runpod_download.py takes ONE
   url or query per run and returns cut excerpts; there is no multi-match
   planner or per-claim footage index. (YouTube blocks datacenter IPs; home
   connection only — brief 09 job 1 note.)

9. **Game-engine simulation segments** (Football Manager UI deep-dives). A
   t=200, 432-440 are full FM screen recordings used as evidence. No tool
   records or assembles sim footage; three_scene_v2.js is our only 3D asset
   and it renders boards, not UI.

10. **A visible style system for information density.** The benchmarks run a
    consistent brand per video: A = red/black/teal boards with face-badge
    tokens; B = white rounded card on a dark blue grid; C = minimal white
    marks over raw footage. Our boards (stats/momentum/xgflow/shotmap/avgpos)
    share one visual language per episode but have no per-video brand system
    and no evidence in EPISODE_SPEC.md of a style pass. (Fact about
    EPISODE_SPEC.md content as of its 2026-09-13 stamp; the file is flagged
    stale by doc_stamp_check and was NOT re-verified this brief.)

## What we already have that the benchmarks also have

- A numbers-on-screen evidence layer (B's stat callouts) ~ our stats board
  (stats_spec.py + board_page.html), though B's are hand-made creator claims
  while ours check against raw Sofascore data (gate B16/B18).
- A one-fact-source discipline the benchmarks do NOT have: B shows season
  stats with no source line (tactical-episode SKILL.md, quality check 3); our
  pipeline refuses numbers without a raw-data check. Keep ours.
- Long-form structure: hook -> thesis question -> context -> mechanism ->
  limit -> recap -> next-video hook (tactical-episode SKILL.md, verified
  against the three transcripts this brief).

## Measurement anchors (from job 4, reports/brief09/measurements.md)

- Shot rhythm: A cuts hard every ~6.9s (74 detected cuts at threshold 0.3);
  C ~8.9s in its catalog region; B has almost no hard cuts at all (7 detected;
  its transitions are gradual dim/fade morphs, a deliberate style choice).
- The footage/diagram/overlay mix per video is in measurements.md section 1;
  the per-minute overlay rate is section 3.