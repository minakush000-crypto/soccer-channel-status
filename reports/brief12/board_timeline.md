> Snapshot of docs/board_timeline.md (canonical, tracked) at brief 12 close, 2026-09-28.

# Board timeline format (brief 12, job 2)

A **board timeline spec** is a JSON file the one 2D renderer
(`tools/board_page.html` + `tools/board_html.js`) plays as a deterministic
animation. It describes a single team's board on one pitch: who is on it,
what appears when, what is emphasised, and what moves. The page state at any
time T is a pure function of `(spec, T)` — no wall clock, no randomness, so
the same spec always renders the same video.

## File location and naming

`renders/<slug>/board_specs/timeline_<name>.json`, name matching the file
stem (`timeline_formation.json` → name `timeline_formation`). Facts live
beside it at `renders/<slug>/match_data.json`, raw responses at
`renders/<slug>/sofascore_raw.json`.

## Top level

```json
{
  "slug": "<episode slug>",
  "kind": "timeline",
  "name": "timeline_formation",
  "duration_s": 10,
  "fps": 30,
  "kicker": "SHAPE",
  "title": "BARCELONA: HOW THE BUILD-UP WORKS",
  "teams": {"home": {"name": "BARCELONA", "color": "a50044"},
            "away": {"name": "ATLETICO MADRID", "color": "20715f"}},
  "side": "home",
  "players": [{"jersey": "8", "name": "Pedri", "x": 61.4, "y": 41.2}],
  "steps": [],
  "source": "sofascore /average-positions + /lineups via renders/<slug>/sofascore_raw.json",
  "fetched_at": "2026-09-28T05:12:33Z"
}
```

- `kind` is always `"timeline"` (rendered by the same page, never a second
  renderer).
- `side` is `"home"` or `"away"`: whose XI the board shows. `teams` still
  carries BOTH sides for the audit (the caption names the opponent).
- `players` positions are the RAW Sofascore average-positions values,
  unmodified: for every team the keeper sits near x=0 and the forwards near
  x=80 (per-team attack-normalized, verified 2026-09-28 against
  `renders/2026-09-08_real-madrid-inter/sofascore_raw.json`: Courtois x=8.6,
  Diomande x=80.4; Martinez x=11.2, Thuram x=77.3). A single-team board draws
  raw x,y directly, so the board's team always attacks RIGHT (x=100).
- `duration_s` is total seconds; inside the renderer,
  `seconds = t * duration_s`.
- `source` and `fetched_at` are mandatory (board_data_check refuses a spec
  with no source: a spec with no source is a fact nobody fetched).

## Targets

Every step targets players by `"jersey:<n>"` (the jersey number is unique in
a match XI). A step whose target is not in `players` fails validation.

## Steps

Ordered list of `{t_start, duration, action, targets, params, source}`.
Times are seconds from board start.

| action | effect | params |
|---|---|---|
| `reveal` | players fade/scale in, one by one (`params.stagger` seconds apart) or together | `stagger` (s), default 0.12 |
| `highlight` | ring around each target's disc; persists until hidden or re-highlighted | `color` |
| `number` | black chip `1..N` on the targets, in target order, staggered; persists | `stagger` (s), `start` (first number, default 1) |
| `line` | line from target[0] to target[1] (chain through the list if longer), drawn progressively | `color`, `style` (`solid`\|`dashed`), `width` |
| `zone` | shaded polygon through the targets' positions | `color`, `alpha` (0-1) |
| `ball_path` | ball dot moves along `params.path` (list of `[x, y]` pitch points), eased | `path`, `color`, `end` (`rest` = hold at last point, `fade` = fade out) |
| `caption` | small text in a card corner | `text`, `corner` (`top-left`\|`bottom-left`\|`bottom-right`) |
| `hide` | targets fade out | — |

Easing is smooth in/out (smoothstep) with 250-400 ms per element by default;
`duration` may be longer for movement (ball_path, line draw).

## The source rule (facts vs schematic)

Every step carries `"source"`:

- `"sofascore:<endpoint>"` — the step asserts something the named endpoint's
  cached response (`renders/<slug>/sofascore_raw.json`) actually contains:
  player identities/positions (`sofascore:average-positions`), a score
  (`sofascore:event`), a goal (`sofascore:incidents`).
- `"schematic"` — the step is editorial or illustrative: numbering order,
  highlight emphasis, connection lines, zones, ball movement. A schematic
  ball_path is illustrative movement that serves the narration; it is never
  presented as event data. Specs mark it in the params (`"note":
  "schematic move for illustration"`) and the board's caption spells it out.

Rule: a schematic step may not be the only support for a factual claim. If
the narration says "Barcelona built with five players", the five players must
be real (data); only the way they are counted (numbers 1-5) is schematic.

## Rendering

30 fps, H.264 1920x1080 on Modal (`pod_build.py render2d <name> --slug <slug>`).
Duration accuracy: the MP4 carries `duration_s * fps` frames, so its duration
is within one frame (1/30 s) of `duration_s`.

## Audit (per keyframe)

At the END of every step the renderer freezes that exact time, reads back
what the page drew (player discs, name labels, jersey numbers, number chips,
lines, zones, captions), and checks it against `match_data.json` by team
side: every drawn name must match that side's XI in the facts (token-prefix
match, so "Lamine" matches "Lamine Yamal"), every jersey must be in the XI,
number chips must sit on visible players without gaps or duplicates, and
line/zone endpoints must be visible players. Any mismatch fails the render
(exit 1) before a single frame is encoded. A spec whose player name does not
exist in the facts XI fails by construction — that is the swapped-name
positive control.