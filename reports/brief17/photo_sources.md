# Brief 17: photo sources for the episode-one title card

Two versions rendered for Mayo's pick (brief 17 job 0b). Every row carries
its licence, the owner and the fetch date. The raws are archived to B2
under brief17/ (job 8's batch).

## Version A (default, kept per the brief): the Yamal photo

- file: `reports/brief15/sofascore_cache/yamal_photo_raw.jpg`
  (5000x3333: Lamine Yamal, Spain away shirt no. 19, 2026-07-14 France v
  Spain)
- cutout: `reports/brief16/sizing/assets/yamal_cutout.png` (brief-16
  copy); the waist-up crop used here:
  `reports/brief17/cards/assets/yamal_waist.png` (1826x2620, cut at the
  hip line, 40px alpha pad)
- source: https://commons.wikimedia.org/wiki/File:Lamine_Yamal_France_v_Spain_7.24.jpg
- owner: Bryan Berlin, licence CC BY-SA 4.0 (the row brief 16 used,
  reports/brief15/photo_sources.md)
- raw sha256: recompute at archive time (the brief-15 row's own sha is
  the record for the unchanged raw)

## Version B: Hansi Flick, Barcelona press desk (APRIL 2025) — NOT USED

- source page: https://commons.wikimedia.org/wiki/File:Hans-Dieter_Flick_during_a_press_conference_in_April_2025_at_FC_Barcelona.jpg (2048x1536)
- owner: JaflaumS05 (Wikimedia Commons user), licence CC BY-SA 4.0
- fetched 2026-10-06; raw sha256
  220f4ca3eb3832c2324ed95badf10ce542695be4f2710a2eb09bbe2f858a4c9e
- cutout 630x550 (rembg on the full raw): carries the desk, a laptop with
  a brand logo and a bottle; a brand logo at title size is unwanted, and
  the figure is small in the wide frame. Not rendered; recorded here for
  the pick log.

## Version B RENDERED: Hansi Flick 2022 portrait

- source page: https://commons.wikimedia.org/wiki/File:2022_Hansi_Flick_(cropped).jpg (1400x2063, chest-up portrait)
- owner: User:Stepro (Wikimedia Commons user), licence CC BY-SA 4.0
- fetched 2026-10-06; raw sha256
  a5373e86c5b94a01c406d83df173f0dca50b93db2576edef3fb4545fcf65a869
- cutout: `reports/brief17/cards/assets/flick_cutout.png` (1400x2024:
  rembg + alpha trim, pod-side brief15_photo.py on in/brief16/
  flick2022_raw.jpg -> assets/flick2022_cutout.png, CUTOUT-OK 2026-10-06)
- rendered: reports/brief17/ba_card_title_flick.jpg (the before/after
  sheet Mayo picks from)

Note (licence): both Flick sources come from Wikimedia Commons with
recordable licences (CC BY-SA 4.0, named owners) as brief 17 job 0b
requires. The Yamal row keeps its brief-15 licence record unchanged.