# close.md — brief 20r (the voiced ep02 rebuild)

Date: 2026-10-11 (UTC; Mayo's local 2026-10-10 evening).
Brief: briefs/brief20r.md (494 words, committed c0eb581 and fa0bec0 for
the gate rows).

## Shipped

- ep02_v2.mp4 (pass C): the voiced episode, 193.56 s (3:13.6),
  1920x1080 h264+aac, 12 voice-led rows; mean -22.9 dB / max -2.6 dB;
  assembled by spec_assemble.py (three passes: v1 assemble, +3.5 dB
  voice gain, S12 re-voice at pass C).
- Uploaded UNLISTED to YouTube: video_id nWjbw2GCP_A (pass C, the
  review copy), thumbnail set, pending_review. Supersede trail:
  nWjbw2GCP_A <- eUHPwhUOpTI (pass B v2: its description falsely said
  the fan clips carry the source's goal label on screen; the 75 percent
  crop cuts that corner out; audit blocker, caught by the session's
  own adversarial audit) <- 2L-Yj09gJ7M (the brief-20 silent 81 s
  review draft). All three stay unlisted; Mayo's review copy is v3.
- Narration v2: scripts/ep02_narration_v2.md (12 sections, 397 spoken
  words, 132 wpm measured, ElevenLabs Turbo at atempo 1.0).
- Claims table: scripts/ep02_claims.json (21 rows), every spoken number
  resolved from data/barcelona-getafe-20261010/sofascore_stats.json by
  path (b139, pass). The "six wins in a row" line was retired BEFORE
  upload: the file's own context fields contradict (21 points over 7
  matches forces 7 wins; the form string claims a 5-match run), so the
  streak is not spoken (DECISIONS 30i).
- Adversarial audit (four read-only lenses, 28 findings): one blocker
  (the false description sentence) fixed by the v3 upload; two table
  gaps fixed (rows for "from minute three onward" and "8 games");
  ledger and sync lenses found no defects.
- Benchmark study: one full Football Oculus episode (q_bkQib7Sk8, 8:21)
  downloaded and measured; notes + 12 frames + scene-cut scans
  committed under reports/brief20r/.
- Diagrams v2: 6 light-panel specs in diagram_specs_v2/ (Getafe labels
  neutral: the XI is not in the data file; d02/d04/d05 mirror fixed so
  Koundé attacks the right side, matching pass_map right=34%).
- Cards v2 + title card + final board: measured text (no overflow; the
  brief-20 card clipped text past the frame edge), one closing board.
- Thumbnail: assets/ep02_v2_thumbnail.png (also staged).
- Tools: diagram_gen.py spec-overridable style (legacy byte-identical,
  cmp-verified), stat_card_gen.py measured boxes + finalboard,
  spec_assemble.py row-audio model, tools/brief20r_check.py oracle
  (B137-B146). Census rebased: files=81 wired=54 handrun=27.

## Pending

- Mayo's review of nWjbw2GCP_A (pending_review). The earlier unlisted
  cuts stay up until v3 is approved; then they can be retired by hand.
- The benchmark channels' inset footage panels, burned quote subtitles
  and rights-cleared stills are not in our pipeline (noted in
  benchmark_notes.md "Not transferred").
- Ambience/music bed: the brief-20r cut is voice + slowed fan-clip
  audio only. If Mayo wants a bed under diagrams, it is one more mix
  pass (merge_voice.py's crowd lane exists).

## Broke (fixed in-run; none outstanding)

- mplsoccer's pitch_color paints nothing on this build (pixel-probed);
  the panel is painted by diagram_gen itself now.
- A 40pt title clipped at the canvas top (YLIM cap); big titles anchor
  top-down at y 109.7 now.
- spec_assemble refused card rows carrying PNG pieces (single-image
  inputs encode 0.04 s); the title/final boards are pre-encoded MP4
  pieces.
- The S04 Gordon row first pointed at the wrong webm (caught in review,
  fixed before assembly).
- ElevenLabs calls: none failed; three retries were armed and unused.

## Disk state

- DISK-GUARD-PASS at the assembly run: C: 17G free (>= 15), D: mounted
  and writable, no /mnt/f paths, ECC data 9.7 MB <= 10 MB.
- v1 render overwritten locally AFTER the B2 archive of the v1 draft
  was verified (2026-10-11 prefix, 21 artifacts).

## What I would fix next pass

- The two inset-card rows (106 s, 158 s) dim the diagram behind the
  wash more than the benchmark does; a lighter wash (alpha 120) or a
  smaller card keeps more of the diagram alive underneath.
- A 2-3 s cold-open still (the finalboard front-loaded) would let the
  voice start on a picture of the score. Benchmark does this with a
  photo; we have none rights-cleared yet.
- The Jesus and Koundé beats could use one more diagram each (a small
  passing-network for act 2) if a future brief wants 4+ minutes.