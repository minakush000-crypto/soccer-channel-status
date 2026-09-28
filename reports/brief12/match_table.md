# Brief 12 — job 1 data: Barcelona team id and its 4 most recent completed matches

Fetched 2026-09-28 (05:10-05:14 UTC), endpoint `/api/v1/team/2817/events/last/0`
(30 events, page 0; the search and calendar endpoints stayed IP-blocked —
measured, see probes below). The id 2817 is verified against the payload:
every returned event features FC Barcelona. Raw responses cached at
`renders/<slug>/sofascore_raw.json`, facts at `renders/<slug>/match_data.json`,
every fetch through tools/sofascore_client.py (which gained a per-round
Chromium fetch fallback this brief: Akamai challenged every curl_cffi
impersonation for 40+ min, measured).

| date | competition | match | score | slug | endpoints returned |
|---|---|---|---|---|---|
| 2026-09-19 | LaLiga | Sevilla v FC Barcelona | 1-3 | 2026-09-19_sevilla-fc-barcelona | event, lineups, average_positions, incidents, statistics |
| 2026-09-16 | LaLiga | FC Barcelona v Real Racing Club | 7-2 | 2026-09-16_fc-barcelona-real-racing-club | event, lineups, average_positions, incidents, statistics |
| 2026-09-13 | LaLiga | Levante UD v FC Barcelona | 2-4 | 2026-09-13_levante-ud-fc-barcelona | event, incidents, statistics (lineups + average_positions retired) |
| 2026-09-09 | UEFA Champions League | FC Barcelona v Feyenoord | 5-1 | 2026-09-09_fc-barcelona-feyenoord | event, incidents, statistics (lineups + average_positions retired) |

2/4 matches carry lineups + average positions — above the brief's stop rule
("fewer than 2: stop and report"), so the job proceeded. The proof boards use
the two data-complete matches (formation from the Sevilla match, runners and
move from the Racing Club match).

## Pass/possession-sequence probe (uncertainty set)

Seven plausible sequence endpoints probed against event 16938768 (a known-good
event), all returning no JSON payload — evidence in `probes.txt`. Conclusion:
Sofascore exposes NO pass or possession sequences, so a traveling ball is
SCHEMATIC and is labeled as such on the board itself
("SCHEMATIC MOVE (NOT EVENT DATA)") and in the spec (`source: "schematic"`).

## Frame-accurate capture method (uncertainty set)

Resolved to JS animation stepped per frame, the existing renderer's own
contract: board_html.js calls `window.render(t)` once per frame
(t = i/frames, 30 fps), the page state is a pure function of (spec, t), and
ffmpeg encodes the PNG frames to H.264 1920x1080 on Modal. Evidence: the
smoke render produced a 30 fps H.264 MP4 whose ffprobe duration equals the
spec duration exactly (8.500000 s, 255 frames); no screencast involved.

## Render times and pod cost (uncertainty set + job 7)

| board | duration | frames | Modal wall time |
|---|---|---|---|
| timeline_formation | 8.5 s | 255 | 108.0 s |
| timeline_runners | 9.0 s | 270 | 54.2 s |
| timeline_move | 10.0 s | 300 | 87.7 s |

Modal cost for the brief-12 window (2026-09-28 UTC interval, `modal billing
report --start 2026-09-27T00:00 --end 2026-09-29T00:00 --json`, summing the
8 app entries of that interval, which are all this brief's runs — two smoke
renders, the egress probe, and the three proof renders): **$0.0182**. The
prior-day interval ($0.0968) predates this brief's first Modal call and
belongs to earlier briefs' work.