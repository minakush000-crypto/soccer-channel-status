# Brief 12 — visual check (job 6)

Method: each proof MP4 pulled from the Modal volume, 1 frame per second
extracted with ffmpeg (`fps=1,scale=1280:-1`, frames in /tmp during the run),
every frame viewed in-session. Checks per frame: drawn names match the facts
XI (the per-keyframe audit already enforces name+jersey pairing; this is the
human pass), no label overlaps hide a name, animation order matches the spec.
Findings were clean on the first render pass — no re-render was needed.

## timeline_formation (Sevilla 1-3 Barcelona, 8.5 s, side=away)

| frame (s) | what is on screen | verdict |
|---|---|---|
| 0 | title + SHAPE kicker + "VS SEVILLA" caption, empty pitch | matches spec (caption step 0.2-0.5) |
| 1 | identical hold (GK reveal starts 0.8, not yet legible at 1 fps granularity) | ok |
| 2 | GK D. LIVAKOVIC (25) revealed alone, name label under disc | ok |
| 3 | GK + E. GARCIA (24), P. CUBARSI (5), A. CHRISTENSEN (15) + J. CANCELO (2) mid-fade | D-line reveal per spec |
| 4 | back four complete; F. LOPEZ (7), RODRI (16), PEDRI (8) revealed; L. YAMAL mid-fade (faint disc, no label yet — by design below alpha 0.5) | M-line per spec |
| 5 | all 11 visible incl. RAPHINHA (11), A. GORDON (17) | ok |
| 6 | D-line zone shaded in over 24/5/15/2 | zone step 5.4-5.8 per spec |
| 7 | hold, complete state | ok |
| 8 | hold, complete state | ok |

Names read clean, no label collisions (the emitter's label solver placed
F. LOPEZ/RODRI/PEDRI around the centre circle without overlap).

## timeline_runners (Barcelona 7-2 Real Racing Club, 9.0 s, side=home)

| frame (s) | what is on screen | verdict |
|---|---|---|
| 0 | title + "VS REAL RACING CLUB" | ok |
| 1 | hold | ok |
| 2 | five of the six revealed (L. YAMAL 10, RAPHINHA 11, D. OLMO 20, K. ADEYEMI 14, J. CANCELO 2 mid-fade) | reveal order per spec |
| 3 | hold | ok |
| 4 | PEDRI (8) revealed with the red carrier ring; no chips yet (chip 1 lands 3.8-4.2, at 1 fps granularity not yet legible) | ok |
| 5 | chip 1 on YAMAL + first yellow line to the carrier; chip 2 mid-fade | number+line sequence per spec |
| 6 | chips 1-3 + lines 1-3 | ok |
| 7 | all chips 1-5 + all five lines converge on PEDRI | complete count board |
| 8 | hold | ok |

Numbers 1-5 sit on visible players, no duplicate or missing chip; the lines
all terminate on the ringed carrier. Raphinha's label sits right of his disc
(solver choice) and clears chip 2.

## timeline_move (Barcelona 7-2 Real Racing Club, 10.0 s, side=home)

| frame (s) | what is on screen | verdict |
|---|---|---|
| 0 | title + both captions: "SCHEMATIC MOVE (NOT EVENT DATA)" bottom-left, "VS REAL RACING CLUB" bottom-right | schematic is labeled on the board itself |
| 1 | hold | ok |
| 2 | 4 of 5 revealed (J. GARCIA 1 GK, E. GARCIA 24, L. YAMAL 10, J. CANCELO 2; K. ADEYEMI 14 mid-fade) | ok |
| 3 | hold | ok |
| 4 | black ball dot mid-flight between GK and E. GARCIA; GK gold-ringed (arrival highlight) | ball_path + highlight per spec |
| 5 | ball between E. GARCIA and CANCELO; E. GARCIA gold-ringed | ok |
| 6 | ball between CANCELO and YAMAL; CANCELO gold-ringed | ok |
| 7 | ball arriving at YAMAL; YAMAL gold-ringed | ok |
| 8 | ball RESTING on ADEYEMI (final recipient), all five gold rings holding | end=rest per spec |
| 9 | hold | ok |

The zigzag (GK -> CB high -> Cancelo low -> Yamal high -> Adeyemi low) reads
as end-to-end switching; the board states in its own caption that the move is
schematic (no pass-sequence endpoint exists — reports/brief12/probes.txt).

## Verdict

All three boards clean: names match the facts XI by jersey, no label overlaps,
animation order matches the specs. No fixes, no re-render.