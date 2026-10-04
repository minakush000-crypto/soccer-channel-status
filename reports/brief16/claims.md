# Brief 16: claims.md — the thesis and its evidence

## Thesis

**"Flick's press: how Barcelona win the ball high, and what it costs them."**

Both phases: (1) the press (in-possession lost, then the regain) and (2)
the cost (space behind the high line, chances conceded). The honest
counter-argument beat is the cost side. Every number below is re-checked by
the oracle (B94/B95) against the cached raws named in press_facts.md; the
JSON sidecar `claims.json` is the single source of truth for ids/numbers.

## Accepted claims

| id | side | claim (wording the episode uses) | traced numbers (fact keys) |
|---|---|---|---|
| C1 | press | The platform is the ball: Barcelona held 69, 69, 75 and 72 percent of possession across the four matches (the press always starts with Barcelona already in the game's leading share). | 16416349-01 (69%), 16416346-01 (69%), 16416329-01 (75%), 16938784-01 (72%) |
| C2 | press | Opponents cannot build through it: their final-third pass phase is 37 to 63 percent (Racing 37, Levante 52, Sevilla 55, Feyenoord 63) while Barcelona's runs 79 to 84 (Sevilla 84, Racing 83, Levante 79, Feyenoord 84). | the four -14 cells, quoted as their percents (all part_of raw cells) |
| C3 | press | The high line wins the offside battle 19-9: opponents caught 5, 2, 4 and 8 times (Sevilla, Racing, Levante, Feyenoord) against Barcelona's 2, 3, 2, 2. | 16416349-28 (5), 16416346-27 (2), 16416329-28 (4), 16938784-27 (8) |
| C4 | press | Territory, whatever the phase: 314 final-third entries to 133 (78, 82, 72, 82 against 51, 15, 50, 17) and 171 touches in the box against 60 (28, 64, 33, 46 against 12, 10, 27, 11). | the four -13 cells and the four -12 cells (each quoted value inside its raw cell) |
| C5 | press | The press ends in the box: 25 big chances in four games (6, 13, 3, 3) and 19 goals (3, 7, 4, 5). | the four -02 cells and the four g1 rows |
| C6 | cost | The cost is real: 11 big chances conceded (1, 3, 5, 2), 5.33 expected goals against (0.40, 1.98, 2.42, 0.53) and 6 goals conceded (1, 2, 2, 1). | -02 opp cells (1/3/5/2), -05 opp cells, g2 rows (trace via each raw cell) |
| C7 | cost | One raw field names the mechanism: in Levante's match, 4 Barcelona errors led straight to a shot (Sevilla's match carries 1 such error for the opponent and 0 for Barcelona; Racing's and Feyenoord's raws carry no such field), so the giveaway cost is direct. | 16416329-27 (4) |
| C8 | cost | The few entries still bite: from 15 final-third entries Racing built 1.98 expected goals and scored 2, against Barcelona's 82 entries building 6.74. | rac -13 (15), rac -05 opp (1.98), rac g2 (2), rac -05 barca (6.74) |
| C9 | cost | The trade still pays: 19 goals for, 6 against, and 14.89 expected goals against 5.33 (3.47+6.74+2.58+2.10 for; 0.40+1.98+2.42+0.53 against). | the four -05 cells both sides + the g1/g2 rows |

## Adversarial verification (brief 16, ultracode pass)

Nine skeptic agents (one per claim) tried to refute every claim against the
JSON sidecar + the raws. Three wordings died and were fixed before this
file was committed:

- C1's original "so every press starts on the front foot, not as a
  scramble": a causal universal the possession cells cannot support.
  Softened to the leading-share wording above.
- C4's original "Territory from the press": the entries count is
  origin-blind (no provenance field), so the causal "from the press" was
  replaced with "whatever the phase".
- C8's original "Every entry is a threat": refuted by its own cells (15
  entries, only 5 shots, 3 big chances): replaced with "The few entries
  still bite" and plain numbers.

The six holdouts (C2, C3, C5, C6, C7, C9) kept their wording; every number
in the kept claims re-verified against the raw cells.

## Dropped (the brief says: mark claims the data cannot support)

- **"Possession won in the final third"** (the headline press field):
  "label absent from the ALL-period statistics of all four raws; the
  vocabulary was printed and it (or any per-third recovery count) does not
  exist. Not invented."
- **"Tackles/interceptions by zone"**: "Sofascore's team statistics carry
  totals only (Tackles, Interceptions, Recoveries), no zone split, in all
  four raws."
- **"PPDA / press-resistance per build-up"**: "not exposed in the raw
  statistics; computing it needs events data Sofascore does not serve on
  these endpoints."
- **"Every conceded goal came from a counter"**: "cannot be traced: the
  raws carry no build-up provenance per goal; the video clips (CL03-CL05)
  show counter moves, but a general claim would outrun the data."

## Clip coverage

CL01 (Sevilla 58-66s, freeze at 60.0) covers C2; CL02 (Racing 104-110)
covers C4; CL03 (Racing 130-142) covers C8; CL04 (Levante 116-124) and
CL05 (Feyenoord 86-98) cover C6. C1, C3, C7 and C9 are card claims
(no-clip reasons in clip_list.json, echoed in the clip list report).

## Input ages (doctrine 5)

- All four per-match raws: fetched 2026-09-28 (the brief-13/15 episode
  fetches), stamped inside each JSON's `fetched_at`.
- The last-events list cache: fetched 2026-09-29T06:14:14Z. The re-fetch
  was attempted and got Akamai 403 challenges every time this session
  (log: reports/brief16/logs/fetch_attempts.log); the latest-completed set
  matches the discovery sweep's independent sources (no Barcelona match
  between 2026-09-25 and 2026-10-03; next scheduled 2026-10-10 v Getafe).
- The highlight videos: metadata probed 2026-10-03 (two downloaded fresh,
  two adopted from the brief-13 pod copies that were downloaded
  2026-09-28).