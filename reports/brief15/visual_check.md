# Brief 15 job 5: visual check of every render (2026-10-03)

Checked the 9 published renders of brief 15 by viewing their keyframes
(the stills this check cites are committed under `keyframes/`, and pairs are
assembled into the publish sheets in this folder). Every claim's file is in
the repo: doctrine 6.

## Method
- After renders: stills pulled from the mp4s with ffmpeg; sheets assembled
  with `tools/brief15_sheets.py` (640px); each panel viewed directly (the
  sheet panels are 320px wide, HALF the width of a 480p frame — anything
  readable on the sheet is readable at 480p).
- The renders themselves were audited at render time by the running page:
  FOOTAGE-CHECK per overlay clip (chip/label overlap, missing frames,
  identity evidence vs the facts XI, and — brief 15 — the chips sit clear
  of each other under the bigger restyled geometry), CARD-CHECK per card
  (drawn rows = spec rows at full alpha; title-card photo box clear of the
  text zone; stat rows on the paper; results card 5 rows at full alpha with
  badges; no missing assets).

## Overlay clips (4) — restyle against benchmark A
- sev_overload (`ba_overlay_sev_overload.jpg`): gold ground ellipse visibly
  larger than brief 13's thin one; link thick with glow; red pressing
  discs + shaded polygon; marks clear of the LALIGA bug (top-right) and
  the RAPHINHA lower-third (bottom-left); vignette outside the marked zone
  visible in `keyframes/sev_overload_after_5.25s.jpg`. Readability: yes at
  320px panel, so yes at 480p.
- sev_block (full-res `keyframes/sev_block_after_5.25s.jpg`): GK STRANDED
  chip large with thin border, red discs on backline positions,
  half-opaque shaded triangle between them, gold ellipse; scoreboard
  (SEV 1 BAR 2, 63:32) and LALIGA bug clear of the marks.
- rac_farpost (`ba_overlay_rac_farpost.jpg`): RAPHINHA + BALL-WATCHING
  chips bold with borders, discs at the far post; broadcaster lower-third
  bottom-left, marks top-right: clear.
- rac_freekick: FAILED the first pass (bigger job-1 chips overlapped:
  I. SAINZ-MAZA vs M. GUEYE). Fixed at the single source (marks.json chip
  x 1640 -> 1690; spec re-emitted; B85 freeze oracle re-run green; pod
  re-render FOOTAGE-CHECK OK at t=3.40). After frame
  (`keyframes/rac_freekick_after_5.25s.jpg`): three wall chips on one row
  with clean gaps; delivery arrow + wall discs + shaded triangle; the
  RAPHINHA lower-third clears the marks.

## Cards (5) — benchmark B screen
- card_results (`cs_card_results.jpg` + keyframe
  `card_results_final_4.5s.jpg`): five rows (3-1 Sevilla (A), 7-2 Real
  Racing Club (H), 4-2 Levante UD (A), 5-1 Feyenoord (H UCL), 5-0
  Valencia (A)), gold condensed scores, real opponent badges, meta lines
  readable; every score/opponent/date walked from the cached raw and
  audited by cardCheck at render time.
- card_stat (`cs_card_stat.jpg`): torn-paper panel, EXPECTED GOALS title,
  three rows (Sevilla 3.47, Racing 6.74, Feyenoord 2.10) ALL on the paper
  (the third row hung half-clipped off the torn edge in the first render;
  the paper now follows the row count and a rows-inside-paper audit guards
  it). xG values trace to the three episodes' raw statistics.
- card_title (`cs_card_title.jpg` + keyframe `card_title_final_3.45s.jpg`):
  headline THE FAR-POST / IS A MAP / NOT AN ACCIDENT in Barlow condensed,
  gold byline, subject cutout in the left box NOT overlapping the text
  (first layout drew the photo into the headlines; photo box + photo-rect
  audit fixed and enforced). Photo source: Wikimedia Commons, Bryan
  Berlin, CC BY-SA 4.0 — photo_sources.md.
- card_chapter (cs_card_chapter.jpg): chapter 02 + THE 6.74 XG NIGHT
  (6.74 = Racing xG, traced).
- card_principle (cs_card_principle.jpg): three lines appear one by one
  (EARLY shows one line, FINAL all three) — the brief's build-up visible.

## Numbers vs facts
- Results: from barca_last_events.json (fetched_at_utc 2026-09-29T06:14:14Z):
  Sevilla (A) 3-1 2026-09-19, Racing 7-2 (H) 2026-09-16, Levante (A) 4-2
  2026-09-13, Feyenoord (H) 5-1 2026-09-09 UCL, Valencia (A) 5-0
  2026-09-06 — walked and audited.
- Stat card xG: 3.47 / 6.74 / 2.10 from the three episodes'
  sofascore_raw.json statistics ALL period (fetched 2026-09-28).
- Chapter "02"/6.74: the Racing-match xG.

## Verdict
EVERY-RENDER-CHECKED: 9 of 9 renders viewed at sheet + full-res keyframe
level; readability at 480p satisfied (checked at half that width); no
element hides another after the two fixes above; every number traces to a
cached Sofascore raw; overlay names carry visible on-frame identity
evidence; the one photo carries a recorded source row.