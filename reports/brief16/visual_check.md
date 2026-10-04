# Brief 16: visual check (job 7, the artifact check)

Viewed directly (frames committed in this folder + /tmp review tiles):

- Job 0a title card: `keyframes/card_title_after_3.45s.jpg` (1080p) against
  brief 15's committed before (`../brief15/keyframes/card_title_final_3.45s.jpg`):
  the photo box grew to 900px = 46.9% of frame width (>= 45% audit passes)
  and the drawn cutout is 620px tall (57.4% of frame height, >= 45%);
  subject large, headline block clear at x=1000. Pair sheet:
  `ba_card_title.jpg`.
- Job 0b ellipses: `keyframes/sev_block_after_5.25s.jpg` against brief 15's
  restyle (`../brief15/keyframes/sev_block_after_5.25s.jpg`): the gold ring
  now carries a fat core (7px) plus a wider glow pass (16px, blur 48); the
  pair sheet `ba_overlay_clip.jpg` shows the before/after at 480p readability.
  The entity log proved the drawn stroke (7) on both audited specs
  (sev_block re-render + the new presstrap spec).
- Job 6 spotlight: `keyframes/presstrap_mid_5.4s.jpg` (freeze 21:42, SEV
  1-0 BAR): gold ellipse bold around the duel cluster, three pressing
  discs, the A. GORDON chip with its pointer; the broadcast's RAPHINHA
  MIN. 22' chip stays clear; vignette darkens the rest. FOOTAGE-CHECK OK
  at render time (identity vs the facts XI) and the local audit-only run
  printed OVERLAY15-OK with the brief16 stroke floor.
- Job 7 contact sheet: `contact_sheet.jpg` (640px, 6 clips): CL01 the
  Sevilla trap minute, CL02 Racing's box pressure at 4-1, CL03 Racing's
  goal, CL04 Romero through on the exposed keeper (1-3, 78:50), CL05
  Feyenoord's goal past the orange keeper, CL06 Cancelo's opener. Each
  panel matches its clip_list row.

Verdict: EVERY-RENDER-CHECKED for brief 16's rendered pieces (2 re-renders
+ 1 new overlay + 6 clips + 1 contact sheet).
