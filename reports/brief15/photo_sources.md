# Brief 15 photo sources (job 6: every photo's URL + owner recorded)

Decision 2 (Mayo, 2026-09-29): player photos allowed on cards; every photo
carries its source URL and owner here so a claim can be traced and swapped.

## Title card subject photo

| field | value |
|---|---|
| file | `sofascore_cache/yamal_photo_raw.jpg` (this repo) |
| cutout used by card | `yamal_cutout.png` (pod-side rembg, `tools/brief15_photo.py`) |
| subject | Lamine Yamal, Spain away shirt, #19 (2026-07-14, France v Spain) |
| source URL | https://commons.wikimedia.org/wiki/File:Lamine_Yamal_France_v_Spain_7.24.26-057.jpg |
| full-res direct | https://upload.wikimedia.org/wikipedia/commons/9/98/Lamine_Yamal_France_v_Spain_7.24.26-057.jpg |
| owner / author | Bryan Berlin (own work) |
| licence | CC BY-SA 4.0 |
| fetched | 2026-10-03 (replaced an earlier unrecorded-source raw the same day; the old raw stays in git history at commit 7f1fa6d) |
| note | cutout re-run on the pod after the swap; title card re-rendered |

## Results card opponent badges (5 rows, Sofascore)

Each badge is fetched from the Sofascore team image endpoint (owner
Sofascore, editorial use in a results list; cached raws in
`sofascore_cache/badges/`, spec rows carry the ids):

| badge | opponent (card row) | source URL |
|---|---|---|
| 2833.png | Sevilla | https://api.sofascore.app/api/v1/team/2833/image |
| 2835.png | Real Racing Club | https://api.sofascore.app/api/v1/team/2835/image |
| 2849.png | Levante UD | https://api.sofascore.app/api/v1/team/2849/image |
| 2959.png | Feyenoord | https://api.sofascore.app/api/v1/team/2959/image |
| 2828.png | Valencia | https://api.sofascore.app/api/v1/team/2828/image |

(Sofascore id of Barcelona: 2817; the results rows are Barcelona-first
scores, walked from the cached raw `barca_last_events.json` by
`tools/brief15_cards.py` and audited by `tools/brief15_check.py cards`.)