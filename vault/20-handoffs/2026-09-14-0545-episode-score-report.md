Answers: the brief at ~/claude/30-briefs/2026-09-14-0525-score-episode-against-spec.md

# Episode Score Against EPISODE_SPEC — 2026-09-14

## Stamps (reported before answering)
- STATUS.md: 2026-09-13
- DECISIONS.md: 2026-09-14

## Voice track: verified script or old narration?
**OLD NARRATION (◑ inferred).** voice_elevenlabs.mp3 mtime: 2026-09-13 02:16 (the first produce_v2 run). The script was fixed at 20:33 (later). The voice was TTS'd from the pre-fix 1911-word script (which contained the fabricated "Igor Thiago needed just one big chance to score"). The .script_verified gate was set at 20:33 — AFTER the voice was already generated. The re-runs reused the pre-fix voice (step6_voice: verified + voice exists → skip). The voice duration 719.1s matches the pre-fix 1911 words @ 159 WPM (the fixed 1751-word script would be ~660s). The voice carries the fabricated scorer. Cannot directly verify (no speech-to-text tool); inferred from the TTS process + timestamps.

## Words match pictures across 718s?
**NO — at the Trade-off section.** The voice says "Igor Thiago needed just one big chance to score" (the fabricated scorer from the pre-fix script). The stat_card board (shown at the Trade-off section, 137s) displays the real data (both Brentford goals were Kevin Schade's, per match_data.json). The footage windows (from the scoreboard scan) show the real goals (Schade 34/56). So the voice says the wrong scorer while the board + footage show the real goals. The words (voice) don't match the pictures (board/footage) at the Trade-off section. At other section boundaries, the stats narration matches the boards (both are from the same match_data). But the Trade-off mismatch is a credibility failure.

## Gemini frame-by-frame judgments (raw, pasted above in the session)

| Frame | Timestamp | Gemini Score | Key verdict |
|---|---|---|---|
| Opening | 2s | 6/10 | Dynamic aerial footage, NOT a static board. PASS hook rule. |
| Footage | 8s | 0/10 | "Graphic, not match footage, Google watermark." Possible false positive (branding screen at 5s PASSED). |
| Formation (3D) | 22s | 6/10 | Dark bg + direct labels. Standard sans-serif, no title-as-message. |
| Stat card | 137s | 4/10 | Bar lengths inaccurate. Generic title. |
| Possession | 180s | 5/10 | Lacks data percentages. Generic title. |
| xG flow | 208s | **10/10** | Flawless. Perfect title-as-message, direct labels, contrast, bold condensed. |
| Shotmap | 240s | 6/10 | Dark bg, bold text. Missing color key. |
| Momentum | 272s | 7/10 | Good contrast. Title conveys takeaway. (9/10 at 264s earlier — holds 7-9/10.) |
| Avg-positions | 304s | 5/10 | Color-mapping bug: Brentford label blue but nodes red. |

## EPISODE_SPEC section-by-section score

| § | Section | Score | Verdict |
|---|---|---|---|
| 1 | Runtime (8-14min) | 2/2 | 718s (12min). PASS. |
| 2 | Shot rhythm (40-80, 8-16s) | 2/2 | 45 segments (~54 scenedetect), 16s mean. PASS. |
| 3 | Content mix (graphics ≥60%, footage 5-20%) | 2/2 | 83% graphics, 17% footage. PASS. |
| 4 | Graphic types (MUST pitch/formation/stat) | 1.5/2 | All 3 MUSTs + 4 new. SHOULD arrows/lower-third absent. |
| 5 | Typography/colour | 1/2 | Inconsistent: 4 new 2D boards have Bebas/Barlow; 3D formation + 2D stat_card/possession don't. |
| 6 | Opening (hook in 5s, no static board) | 1.5/2 | Footage-led (PASS). Basic hook (6/10). Establishes match in 15s. |
| 7 | Audio (155-195 WPM) | 1.5/2 | 159 WPM (PASS). Crowd ambience. No music bed. Voice carries pre-fix narration. |
| 8 | Framing (1920x1080) | 2/2 | 1920x1080, no pillarbox. PASS. |
| 9 | Source-footage provenance | 1.5/2 | Screened 3 frames PASS. The 8s frame flagged "Google watermark" (possible false positive). |
| 10 | Narration source-of-truth (MUST) | **0/2** | **MUST FAIL.** The voice carries the pre-fix narration with the fabricated Igor Thiago scorer. The script is fixed but the voice wasn't re-TTS'd. |
| 11 | Board design properties | 1/2 | 4 new 2D boards have the 4 properties; 3D formation + 2D stat_card/possession don't. |
| **Total** | | **16/22 (7.27)** | But §10 is a MUST FAIL → disqualifying. |

## Overall score: **5.5/10**

Below the 7/10 gate. The freeze holds. The numeric total (7.27) is dragged below 7 by the §10 MUST failure: the voice carries the fabricated Igor Thiago scorer. A MUST failure is disqualifying regardless of the numeric total.

## Three biggest failures

1. **The voice carries the pre-fix narration with the fabricated Igor Thiago scorer (§10 MUST FAIL).** The script was fixed (Igor Thiago → Kevin Schade) but the voice wasn't re-TTS'd. The .script_verified gate was set retroactively (after the voice was already generated from the un-fixed script). A confident voice asserts a fabricated scorer about a real match — a credibility failure worse than any board scoring 4/10. ◑ inferred from the TTS process + timestamps (no speech-to-text to directly verify the audio content).

2. **The words don't match the pictures at the Trade-off section.** The voice says "Igor Thiago needed just one big chance to score" while the stat_card board (shown at the same section) displays data that shows both Brentford goals were Kevin Schade's. The footage windows (from the scoreboard scan) show the real goals (Schade 34/56). The viewer hears the wrong scorer while seeing the real goals.

3. **The stat_card board scores 4/10 (the lowest board).** Bar lengths don't reflect the data values, generic title, no design layer (Bebas Neue/Barlow Condensed). The 2D stat_card (tactical_boards.py) uses matplotlib defaults, not the shared design layer that the 4 new boards use.

## Recommendation: keep or retire the 3D formation board?

**RECOMMEND RETIRE (do not act).** The 3D formation board (scene_gen.py):
- Scores 6/10 in the episode — NOT the lowest (stat_card 4/10, avgpositions 5/10 are lower), but below the 4 new 2D boards (xg_flow 10, momentum 7-9, shotmap 6 — tied, avgpositions 5 — lower but fixable).
- Is NOT reliably reproducible (Stage 12: one run produced a broken render, 1/10 + 0/10, while the commit claimed reproducibility).
- Costs ~$0.09 + 9min per render (Modal T4), while the 2D boards are free + fast (local matplotlib).
- Does NOT use the shared design layer (Gemini: "standard sans-serif, not bold condensed") — the 4 new 2D boards do.
- The 2D avgpositions board (5/10, fixable to higher with the color bug fixed) is a 2D alternative for the formation visualization — 2D is the right medium for positional data (the extraction findings + Part 1 confirmed: 3D perspective distorts 2D positional data).

Retiring scene_gen.py + the 3D formation board, replacing it with the 2D avgpositions board (color bug fixed), would: close the PARTIAL 2D retirement debt, remove the reproducibility risk, eliminate the Modal cost, and unify all boards under the shared design layer (board_design.py). The 2D avgpositions (with the color fix) is the better formation board.

## Mining table

| Question | Answer |
|---|---|
| Episode's overall score | **5.5/10** (below 7; §10 MUST FAIL disqualifying) |
| Voice from verified script or old narration? | OLD narration (pre-fix, fabricated scorer). ◑ inferred. |
| Words match pictures? | NO at the Trade-off section (voice says Igor Thiago, board shows Schade). |
| Does the momentum board hold 9/10 inside an episode? | Holds 7-9/10 (7 at 272s, 9 at 264s — prompt/frame sensitivity). |
| Total distinct board seconds producible | ~597s (7 types × ~85s). Against 583s needed: PASS. |
| 3D formation: keep or retire? | RECOMMEND RETIRE (6/10, not reproducible, $0.09/render, no design layer, 2D avgpositions is the alternative). |