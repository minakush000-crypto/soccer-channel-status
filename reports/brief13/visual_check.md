# Brief 13 visual check (job 6) — 2026-09-28

Method: 8 keyframes per clip extracted from the PULLED MP4s (t = 1.0 pre-freeze,
2.1 freeze, 2.6 ellipses animating, 3.2 chips, 4.2 full overlay, 6.5 mid-hold,
8.2 fade-out, 9.5 play resumed), stamped with clip+id, tiled into two 4x4
sheets (/tmp/b13_check/vcheck_0.png, vcheck_1.png), viewed above.

## footage_rac_farpost (Raphinha far-post run, 42')
- every keyframe checked: pre-freeze play clean, ellipses land under
  Raphinha's feet (yellow) and the covering defender (red), RAPHINHA chip
  points at the yellow ellipse, BALL-WATCHING chip points at the red ellipse,
  YAMAL chip points at the bottom-right player, the arrow drops from the
  airborne ball to the run line, overlays fade out before play resumes.
- identity: chip names carry lineup:11 / lineup:10 sources; the defender has
  no name (number not visible) — editorial chip only. PASS.

## footage_rac_freekick (Raphinha free kick, 67')
- unit polygon + red discs sit ON the four-man wall; S. CANALES,
  I. SAINZ-MAZA, M. GUEYE chips point at wall shirts 20/6/14; the number 19
  wall shirt carries NO name (not in the XI — no evidence, no name); J. CANCELO
  chip points at the dark shirt fronting the wall; ball ellipse at the parked
  ball; shot-path arrow over the wall toward goal. Fade-out + resume clean. PASS.

## footage_sev_block (six-yard scramble, 63:32)
- unit polygon wraps the retreating white block; Y. FOFANA / A. SANGANTE /
  R. URE chips point at shirts 24/12/9; GK STRANDED editorial chip points at
  the yellow keeper (his number is not visible, so no name); the yellow
  ellipse marks the mid-air Barcelona attacker; the arrow runs from the
  airborne cutback to the attacker. Fade-out + resume clean. PASS.

## footage_sev_overload (2v1 at the box edge, 51:31)
- two yellow ellipses under the converging Barcelona pair, red ellipse under
  the single Sevilla presser, glowing link between the pair, 2v1 editorial
  chip above the trio, schematic arrow along the open lane into the box.
  No names anywhere (no readable numbers at this zoom) — evidence rule held.
  Fade-out + resume clean. PASS.

## Resized boards (job 1 re-render)
- after_sheet_formation.jpg / after_sheet_runners.jpg / after_sheet_move.jpg:
  card now 91.7% of frame width (was 79%), pitch fills the card, discs 84 px
  (was 60), labels 36 px (was 26), chips 52 px (was 34), runners board framed
  on the active right half. Before/after side-by-side: ba_sheet_*.jpg.
  All keyframe audits (minima + overlaps) pass inside the render. PASS.

## Drift control (job 6 positive control)
- brief13_check.py drift: drawn ellipse positions compared against the marked
  foot points in marks.json, all four moments clean; a doctored spec with one
  ellipse shifted 150 px was FLAGGED (DRIFT-CONTROL-OK).

Verdict: all four overlay clips and all three resized boards pass the
keyframe check on the first full render pass. No re-render needed.