# artifacts/scoreboard/ — what is in here and how to recheck it

Each `clip_<id>.scoreboard.json` is the raw output of `tools/scoreboard_scan.py`:
it samples the top-left score bug every 3s, reads the scoreline with a vision
model (gemma4:cloud), carries the last-known score across bug-absent (NONE)
frames, and records both the per-frame `timeline` and a derived `changes` list.

## How to recheck the goal count (IMPORTANT)

Do NOT count goals from `len(changes)`. Count them from `timeline[]`.

`changes[]` records a transition between two consecutive *readable* (non-PRE)
bug frames. It misses a goal when the bug was absent before that goal's first
appearance, because there is no prior score to transition from.

On clip_PrCW_geeRAU (Arsenal-Chelsea), `changes[]` has 6 entries: 2 real goals
plus 4 single-frame OCR blips (ARS 1-1 BUE, ARS 2-4 CHE and their reverts). The
blips revert within ~9s and are dropped by the scanner's persistence filter
(MIN_PERSIST=9s).

The 3 real goals are the first occurrence of each new scoreline in `timeline[]`:
- t=102s  first "ARS 0-1 CHE"  Rogers (Chelsea)   [bug absent until ~99s; PRE
          before, so NOT in changes[]]
- t=171s  first "ARS 1-1 CHE"  Havertz (Arsenal)
- t=261s  first "ARS 2-1 CHE"  Odegaard (Arsenal)

These match match_data.json (Rogers 2', Havertz 25', Odegaard 50').

So: recheck by scanning `timeline[]` for the first frame where `effective`
becomes a new scoreline, cross-checked against match_data. `changes[]` is a
convenience summary with a known gap on the first goal.

## Bug-burned-in caveat (Part 2)

The score bug is burned in by the uploader across ALL footage types in this
reel, including fan-shot phone footage. So bug *presence* does NOT split
broadcast action from filler. `effective == "PRE"` (bug absent) is not a clean
"broadcast vs filler" signal here — see STATUS.md Part 2 and the main
CONTEXT.md. Goal-finding is unaffected; only the broadcast/filler split is.