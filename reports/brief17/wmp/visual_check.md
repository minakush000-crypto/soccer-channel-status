# words-match-pictures — episode one (brief 17 job 7)

Method: brief17_stills.py extracted one drawtext-stamped mid-frame per
script row (77 frames: 25 row mids + 52 footage probes) from the final
render ON THE POD, using the real cut list assemble wrote
(renders/2026-10-06_flicks-high-line/assemble_manifest.json). Sheets:
wmp/sheet_wmp.jpg (640-wide overview) + sheet_wmp_640_1..5.jpg (five-row
sheets, 640x360 panels). The model viewed every row and verdicts it here.
Crops: wmp/crops/ — the B102 overlay chip crops (11 png, reused, native
400px at the anchors), 3 broadcast lower-third chips (native 1000x280
strips), 12 native board stills (every on-screen card/board number at
1920x1080).

## Verdicts

| row | t (final) | asset | verdict | note |
|---|---|---|---|---|
| R01 | 4.0 | CL01 (sev 58-66) | MATCH | the Sevilla keeper + shirts swarming; bug SEV 1-0 at 21:43; chip RAPHINHA MIN. 22' on screen |
| R02 | 13.2 | footage_presstrap overlay | MATCH | frozen box edge, disc + F. LOPEZ chip + ball mark (B102-verified crop) |
| R03 | 22.4 | card_title_yamal | MATCH | Yamal photo, decision-2 headline "FLICK'S HIGH LINE / WHAT IT WINS AND / WHAT IT COSTS" |
| R_CH1 | 29.1 | card_ch1_wins | MATCH | CHAPTER 01 WHAT THE LINE WINS |
| R04 | 38.4 | card_c1_platform | MATCH | THE PLATFORM 69/69/75/72 fully lit (hold resized) |
| R05 | 51.4 | avgpositions@sevilla | MATCH | both XIs drawn, keeper low on the axis; shape of the line visible |
| R06 | 64.7 | card_c10_line | MATCH | THE WALL: 44.26 / 50.20 / 2-4 / 11.40 / 14.71 all on screen |
| R07 | 76.6 | CL06 (rac 20-30) | FIXED | narration claimed "cancelo eight minutes in finishes the first"; the bug reads 1-0 at clock 21:43 (the 8' goal already on the board; the goal in this piece is the second, Raphinha's per incidents raw); narration rewritten to "raphinha's finish goes in past the keeper" |
| R08 | 84.6 | rac cut 63-69 | FIXED | narration said "the replay shows"; the package is the live 36' goal (chip JOÃO CANCELO MIN. 36' on screen through the cut); narration rewritten to "the chip names cancelo the goal came out of the lane one touch through and the finish" |
| R09 | 90.6 | CL02 (rac 104-110) | MATCH | box siege at BAR 4-1, bug 44:53 |
| R10 | 97.9 | timeline_runners | MATCH | carrier 8 Pedri + runner tags 1-5 (schematic by brief-12 design; counts are spoken-only) |
| R11 | 107.1 | footage_sev_overload | MATCH | 2V1 tag + lane out of pressure, chip RAPHINHA MIN. 52' |
| R12 | 116.3 | timeline_move | MATCH | honest SCHEMATIC label on board ("not event data") |
| R13 | 127.6 | card_c3_offside | MATCH | THE TRAP 19-9: 5/2/4/8 vs BARCA'S OWN 9 all on screen |
| R14 | 139.2 | footage_rac_freekick | MATCH | FK arrow to the goal-mouth mark; chip RAPHINHA J. CANCELO MIN. 67' |
| R15 | 149.2 | footage_rac_farpost | MATCH | BALL-WATCHING label, Raphinha far post, chip RAPHINHA MIN. 42' |
| R16 | 159.0 | footage_sev_block | FIXED | reel piece context re-verified: the 23-31 window was a 0-0 Barcelona chance at clock 07:20-35, not the goal; the sev pinball window re-verified on the same probe pass (GK STRANDED + the block discs) |
| R17 | 167.5 | card_ch2_cost | MATCH | CHAPTER 02 WHAT IT COSTS |
| R18 | 175.4 | sev cut 36-44 | FIXED | piece re-cut from sev 23-31 to 36-44: Fofana's 19' goal with the chip 24 YOUSSEF FOFANA MIN. 19' and the 1-0 flip (frame-probed 2026-10-07) |
| R19 | 186.6 | card_c7_errors | MATCH | THE GIVEAWAYS: LEV 4 / SEV opp 1, Barca 0 / RAC+FEY field not exposed (honest gap row) |
| R20 | 197.7 | CL04 (lev 116-124) | MATCH | ROMERO chip 9 IVÁN ROMERO MIN. 79'; bug flips 0-3 to 1-3 in the piece |
| R21 | 207.7 | CL05 (fey 86-98) | MATCH | Feyenoord's goal sequence (Steijn), bug 4-0 flips to 4-1 |
| R22 | 218.7 | CL03 (rac 130-142) | MATCH | racing's goal at 4-2 (bug 65:08 RAC 2); the few-entries claim's picture |
| R23 | 228.8 | card_c9_verdict | MATCH | THE TRADE: GOALS 19-6, XG 14.89-5.33 all on screen |
| R24 | 237.9 | card_title_yamal | MATCH | title reprise, same card |

## Mismatches found and fixed (all before this render)

1. R07: "cancelo eight minutes in finishes the first" — the rac highlights
   track starts at clock 21:18, so the 8' goal is not in any rac asset; the
   CL06 piece is the clock-21:43 goal (the second Barcelona goal, Raphinha
   per the incidents raw, minute listed 25 there but the broadcast clock
   says 21:4x). Narration rewritten; the voice re-generated (take 5).
2. R08: "the replay shows..." — the rac 62-70 package is the LIVE 36'
   Cancelo goal (chip JOÃO CANCELO MIN. 36' at clock 35:13-35:22), not a
   replay of the 8' goal. Narration rewritten; reel cut moved to 63-69 so
   the celebration close-up with the readable chip is inside the cut.
   Native crop: wmp/crops/chip_broadcast_cancelo36_R08.jpg.
3. R18's piece: the phase-A "sev23-31 window (Fofana's 19' + chip)" was
   wrong — that window is a 0-0 Barcelona chance at clock 07:20-35. The
   piece re-cut to sev 36-44 (the Fofana goal + the chip + the 1-0 flip).
   Native crop: wmp/crops/chip_broadcast_fofana19_R18.jpg. R20's crop:
   wmp/crops/chip_broadcast_romero79_R20.jpg.
4. R06's card printed 14.72 for the keeper at Racing; the raw is 14.71875
   and the claim's convention is TRUNCATE, never round up, so 14.71.
   brief17_cards.py's keeper_rows() now truncates. (The re-emit also
   clobbered the phase-A hand-tightened specs 921494c; the five restored
   from git before this render.)
5. Card hold sizing: the page fades every card line out at t0+hold
   (~6-6.6 s) regardless of duration_s, so 13-15 s slots played ~7 s of
   empty board (R04's mid-frame was the ghost of the card). The stage tool
   now sizes card hold to the slot; the 8 cards re-rendered.

## No unresolved mismatch

All 25 rows MATCH in this render. The card/board numbers were also audited
mechanically pod-side (CARD-CHECK on 9 cards, PAGE-CHECK on the avgpos and
timeline boards, all OK at the audit times).