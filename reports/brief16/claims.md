# Brief 16: claims.md — the thesis and its evidence

## Thesis

**"Flick's high line: what it wins and what it costs."**

Re-sided 2026-10-06 (brief 17 job 1) to the decision-2 wording:
the high line is the frame, C3 and C6-C9 are the spine, and the
new C10 carries the line-height measurements from the average-
position raws (two of the four matches carry that data). Both
phases stay: what the line wins (C1-C5, C10) and what it costs
(C6-C9). Every number below is re-checked by the oracle (B94/B95)
against the cached raws named in press_facts.md; the JSON sidecar
`claims.json` is the single source of truth for ids/numbers.

## Accepted claims

| id | side | claim (wording the episode uses) | traced numbers (fact keys) |
|---|---|---|---|
| C1 | press | The platform is the ball behind the high line: Barcelona held 69, 69, 75 and 72 percent of possession across the four matches (the line plays its game from the leading share). | 16416349-01 (69%); 16416346-01 (69%); 16416329-01 (75%); 16938784-01 (72%) |
| C2 | press | In the opposition's final third the gap is stark: opponents' final-third pass phase ran 37 to 63 percent (Racing 37, Levante 52, Sevilla 55, Feyenoord 63) while Barcelona's ran 79 to 84 (Sevilla 84, Racing 83, Levante 79, Feyenoord 84). | 16416349-14 (55%/84%), 16416346-14 (37%/83%), 16416329-14 (52%/79%), 16938784-14 (63%/84%) |
| C3 | press | The high line wins the offside battle 19-9: opponents caught 5, 2, 4 and 8 times (Sevilla, Racing, Levante, Feyenoord) against Barcelona's 2, 3, 2, 2. | 16416349-28 (5); 16416346-27 (2); 16416329-28 (4); 16938784-27 (8) |
| C4 | press | Territory, whatever the phase: 314 final-third entries to 133 (78, 82, 72, 82 against 51, 15, 50, 17) and 171 touches in the box against 60 (28, 64, 33, 46 against 12, 10, 27, 11). | 16416349-13 (78); 16416346-13 (82); 16416329-13 (72); 16938784-13 (82); 16416349-12 (28); 16416346-12 (64); 16416329-12 (33); 16938784-12 (46) |
| C5 | press | The wins are counted in the box: Barcelona created 25 big chances in four games (6, 13, 3, 3) and scored 19 goals (3, 7, 4, 5). | the four -02 cells and the four g1 rows |
| C6 | cost | The cost is real: 11 big chances conceded (1, 3, 5, 2), 5.33 expected goals against (0.40, 1.98, 2.42, 0.53) and 6 goals conceded (1, 2, 2, 1). | 16416349-02 (1); 16416346-02 (3); 16416329-02 (5); 16938784-02 (2); 16416349-05 (0.40); 16416346-05 (1.98); 16416329-05 (2.42); 16938784-05 (0.53); 16416349-g2 (1); 16416346-g2 (2); 16416329-g2 (2); 16938784-g2 (1) |
| C7 | cost | One raw field names the mechanism: in Levante's match, 4 Barcelona errors led straight to a shot (Sevilla's match carries 1 such error for the opponent and 0 for Barcelona; Racing's and Feyenoord's raws carry no such field), so the giveaway cost is direct. | 16416329-27 (4) |
| C8 | cost | The few entries still bite: from 15 final-third entries Racing built 1.98 expected goals and scored 2, against Barcelona's 82 entries building 6.74. | 16416346-13 (15); 16416346-05 (1.98); 16416346-g2 (2); 16416346-05 (6.74) |
| C9 | cost | The trade still pays: 19 goals for, 6 against, and 14.89 expected goals against 5.33 (3.47+6.74+2.58+2.10 for; 0.40+1.98+2.42+0.53 against). | 16416349-05 (3.47); 16416346-05 (6.74); 16416329-05 (2.58); 16938784-05 (2.10); 16416349-05 (0.40); 16416346-05 (1.98); 16416329-05 (2.42); 16938784-05 (0.53); 16416349-g1 (3); 16416346-g1 (7); 16416329-g1 (4); 16938784-g1 (5); 16416349-g2 (1); 16416346-g2 (2); 16416329-g2 (2); 16938784-g2 (1) |
| C10 | press | How high the wall stands: Barcelona's four defenders' average x reads 45.69, 41.62, 41.41 and 48.30 in Sevilla's match and 52.62, 43.45, 44.37 and 60.36 in Racing's (means 44.26 and 50.20), against the keepers' 11.40 and 14.72 on the same per-team attack axis (0 is the own goal) - position data exists in two of the four matches; the other two say nothing. | 16416349-avg-876214 (45.69); 16416349-avg-1402913 (41.62); 16416349-avg-186795 (41.41); 16416349-avg-138892 (48.30); 16416346-avg-876214 (52.62); 16416346-avg-186795 (43.45); 16416346-avg-1094827 (44.37); 16416346-avg-138892 (60.36); 16416349-avg-190419 (11.40); 16416346-avg-930267 (14.72) |

## Adversarial verification (brief 16 pass + the brief-17 re-side pass)

Brief 16's nine skeptics (one per claim) ran against the JSON sidecar + the raws in brief 16; three wordings died and were fixed there (C1's causal universal, C4's causal 'from the press', C8's universal). The brief-17 re-side pass (2026-10-06, skeptic agents one per claim) re-ran on every changed or new wording; the results land below once the pass is recorded.
The re-side pass's verdicts (2026-10-06, ten skeptic agents, one per
claim, against the cached raws only):

| id | verdict | what the skeptic found |
|---|---|---|
| C1 | held | all four possession rows trace and recompute from the raws; the wording stays inside what the cells carry |
| C2 | held-with-fix | the stat is a final-third PASS-PHASE share, not a "through the line" measure; opponents also entered (51-50 entries at Sevilla and Levante), so the causal "cannot build through the line" died; the wording now states the phase-gap numbers only |
| C3 | held | 19-9 recomputed from the raws, ALL-period, no home/away flip; no causal claim asserted |
| C4 | held | all eight cells trace; sums recomputed (314/133, 171/60); the origin-blind caveat from brief 16 still binds |
| C5 | held-with-fix | the mechanism clause ("the press ends in the box") is not carried by the cells; the wording now counts the box outcomes only |
| C6 | held | all 12 cost cells trace and recompute; the wording carries only them |
| C7 | held | both errors rows trace to the raws (Levante 4/1, Sevilla 0/1); the two raws without that field stay listed as absent |
| C8 | held | the trade-off wording carries its cells; the "every entry" universal stays dead from brief 16 |
| C9 | held | 19-6 and 14.89-5.33 recomputed; the verdict wording carries the sums |
| C10 | held | all ten averageX rows recomputed to the digit; the keeper-axis argument confirmed; the two-match limit stated in the wording |

## Dropped (the brief says: mark claims the data cannot support)

- **Possession won in the final third (the headline press field)**: "label absent from the ALL-period statistics of all four raws; the vocabulary was printed and it (or any per-third recovery count) does not exist. Not invented."
- **Tackles/interceptions by zone**: "Sofascore's team statistics carry totals only (Tackles, Interceptions, Recoveries), no zone split, in all four raws."
- **PPDA / press-resistance per build-up**: "not exposed in the raw statistics; computing it needs events data Sofascore does not serve on these endpoints."
- **Every conceded goal came from a counter**: "cannot be traced: the raws carry no build-up provenance per goal; the video clips (CL03-CL05) show counters, but a general claim would outrun the data."

## Clip coverage

CL01 (Sevilla 58-66s, freeze at 60.0) covers C2; CL02 (Racing 104-110) covers C4; CL03 (Racing 130-142) covers C8; CL04 (Levante 116-124) and CL05 (Feyenoord 86-98) cover C6. C1, C3, C7, C9 and C10 are card claims (no-clip reasons in clip_list.json, echoed in the clip list report).

## Input ages (doctrine 5)

- All four per-match raws: fetched 2026-09-28 (the brief-13/15 episode fetches), stamped inside each JSON's `fetched_at`.
- The last-events list cache: fetched 2026-09-29T06:14:14Z; the re-fetch is Akamai-blocked (log: reports/brief16/logs/fetch_attempts.log); treated as likely-permanent 2026-10-06 (STATUS.md).
- The average-position rows behind C10: the same 2026-09-28 raws (the values re-read and re-computed 2026-10-06 by brief17_check.py).

## Side notes

- The x axis: average_positions values are per-team normalized along each team's own attacking direction (0 = the own goal, 100 = the opponent goal): both keepers in both fixtures read low (Livakovic 11.40 / Vlachodimos 6.61; the Racing fixture's Joan Garcia 14.72 / Agirrezabala 7.81, recomputed 2026-10-06 from the raws). The coordinator's premise (one fixture's keepers at 11.4/14.7) was wrong: those two are Barcelona's two keepers across the two fixtures with position data. The axis direction stands anyway, proven from four keeper readings.
- The coordinator's one-decimal defender list rounds the raws' two-decimal values; claims and cards quote the raw values.