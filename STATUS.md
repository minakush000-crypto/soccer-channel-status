# STATUS.md — verified current state

> **Purpose:** verified current state of the pipeline (the status spine).
> **Reader:** every session (CLAUDE.md @STATUS.md).
> **Last verified against code:** 2026-09-28 (brief 12: census live —
> CENSUS OK files=53 wired=33 handrun=20; three timeline proof boards
> rendered on Modal, audited, archived to B2; gates B56-B61 written and
> passed).

Rewritten 2026-09-27 (brief 06) against the code on disk. Brief 03-05 history
lives in DECISIONS.md, PROGRESS.md, and retired/. Every claim below carries
the command that proved it.

## The two entry points

1. **tools/produce_v2.py** — the pipeline entry. `--flash` is the flash lane
   (what builds EP001): Sofascore facts → data check → board specs (emitters
   run only for script-referenced boards) → 2D/3D renders on Modal routed by
   spec shape (a `kind` field = render2d, none = render3d) → pod_build
   assemble + auto-pull. The long lane (default) keeps the local
   download/assemble/merge/shorts steps. Scripts are KEEP-IF-EXISTS: an
   existing tracked script is never regenerated (the old word-minimum
   regeneration overwrote EP001's fact-checked script once, 2026-09-26 —
   fixed before any render used it).
2. **tools/pod_build.py** — the Modal build (used by --flash and by hand).
   render2d/render3d take --slug: specs at /vol/specs/<slug>/, outputs at
   /vol/out/<slug>/ (final_video.mp4 and assemble_manifest.json included —
   the final was root-level before brief 06, letting two episodes'
   finals overwrite each other). render2d REQUIRES /vol/specs/<slug>/
   match_data.json and audits the drawn page against it before writing any
   frame (brief 06 job 4). assemble writes the ACTUAL cut list to
   /vol/out/<slug>/assemble_manifest.json for frame checks.

## FACTS: one source, one cache, two checks

- **tools/sofascore_client.py is the only fact source, every competition**
  (brief 06 job 1). ESPN match_data.py is retired (eng.1-only). The client
  maps stats to schema keys on every fetch, stamps fetched_at + source_url,
  and caches the RAW API responses to renders/<slug>/sofascore_raw.json
  (tracked). Event resolution: --event-id, or search routes (search API /
  day calendar / team-id sweep via tools/sofascore_team_ids.json — the
  first two return empty from this IP, measured 2026-09-26).
- **Sofascore retires old events' sub-endpoints** (event 16363640, played
  2026-09-12, serves empty bodies; 16938768, played 2026-09-08, serves
  everything). The raw cache filled at fetch time is the only durable copy.
- **/statistics carries ALL/1ST/2ND periods; 2ND must never overwrite ALL**
  (a naive loop turned possession 36%→42%, shots 16→10 on a refetch).
  Both the client and the checker prefer ALL.
- **tools/board_data_check.py** (produce_v2 step 1d, gates B16/B18) checks
  EVERY numeric leaf of EVERY spec against the RAW responses (fallbacks to
  the facts file are printed per endpoint; unverifiable parts are reported,
  not failed). Every spec must carry `source` + `fetched_at`. Board specs
  and the raw cache are tracked in git.

## The rendered-page audit (brief 06 job 4)

board_page.html records every fillText (text, x, y, colour);
board_html.js (3rd argv = facts file) renders the FINAL frame, reads the
drawn numbers back, and checks them against the facts BY TEAM SIDE:
possession bar columns, stat rows paired by drawn label, scorelines,
momentum goal labels (running score + scorer-or-team + minute), xgflow end
labels, outro chips, avgpos jerseys vs the facts XI. Any mismatch exits 1
and the render FAILS. pod_build render2d refuses an episode render with no
facts file. Proof: a mirrored spec (the brief-05 wrong-side bug) fails with
every swapped number named; the real boards pass.

## EP001 — 2026-09-08 Real Madrid 2-1 Inter

$ ffprobe -v error -show_entries format=duration,size -of csv=p=0 \
    renders/2026-09-08_real-madrid-inter/final_video.mp4
140.96s, 75.6MB (v7, brief 06 rebuild — ffprobe at the job 8 close)

- Seven builds: flash v1 → v6 (brief 05) → **v7 (brief 06)**: momentum board
  regenerated from real Sofascore /graph (92 points; the v6 board was
  hand-typed), stats_possession possession fixed to the fetched 36/64, all
  three 2D boards re-rendered THROUGH the page audit, re-assembled from
  slug-namespaced outputs. Voice unchanged (155.5 WPM, ElevenLabs, Sep 20).
- 19 sections, 365 words: 12 footage + 7 board. Boards: 4 render3d
  (formation_clash, counter_map, goal3_pattern, valverde_strike — specs
  unchanged since brief 05, renders reused), 3 render2d audited
  (stats_possession, momentum, scoreline_outro). Mid-segment frames of all
  7 board segments opened: reports/brief06/final_boards/.
- Facts file: renders/2026-09-08_real-madrid-inter/match_data.json
  (tracked, Sofascore event 16938768, refetched 2026-09-27 through the
  pipeline's step 1; 40/40 numeric fields identical to the brief-05 fetch).

## Tool census (enforced by tools/census.py, gate B20)

$ ls -p tools/ | grep -v / | wc -l  →  57  (USAGE.md exempt)

40 .py + 2 .js + 7 .sh + board_page.html + watchlist.yaml +
sofascore_team_ids.json = 56 rows: 35 WIRED + 21 HAND-RUN (census.py
2026-09-28, brief 13: CENSUS OK files=56 wired=35 handrun=21; brief 13 added
brief13_check.py WIRED, brief13_overlays.py WIRED, brief13_frames.py HAND-RUN).
TOOLS.md is now command-enforced: a file
without a row, a row without a file, or wrong class totals fails gate B20.

## What is broken, worst first

1. **Judge wobble is a model property** (unchanged from brief 05): ±2 across
   5 runs at temperature 0 + seed 0; median-of-N only.
2. **Bournemouth's raw graph/shotmap/avgpos endpoints are retired by
   Sofascore** (measured 2026-09-26): its momentum curve, shot positions and
   xG series can never be re-verified against raw data — the specs are
   stamped UNVERIFIABLE by the check. Any Bournemouth re-render must not
   regenerate those specs (the emitters will refuse without raw data).
3. **Long-lane format future** (unchanged): no benchmark episode for the
   8-14 min lane.
4. **F: RETIRED (brief 11, 2026-09-28)**: the Memorex stick (Disk 2)
   disconnects under sustained write (brief 07 evidence: NTFS event 140,
   1.6 MB/s, 147 MB of a 1.7 GB tarball). All caches + staging now live at
   /mnt/d/scratch/ (SD card, Disk 1, NTFS, ~12 MB/s write, ~124 MB/s read;
   brief 11 job 3); disk_guard FAILS on any path on the retired F: stick
   (retired_f check, selftest control 5); f_mount.sh is a stub; the f-mount
   sudoers rule is removed; nothing on F: needed rescuing (keys .env.gpg
   already on B2, verified 1==1 in brief 07; leftovers listed read-only in
   reports/brief11/f_leftovers.txt, ~586M regenerable debris + Mayo's two
   own files). Proofs: cache_resolution.txt, ytdlp_download.txt,
   modal_smoke.txt (volume round trip staged through D:), controls.txt.
   Job 7 closed on the SECOND restart check (2026-09-28): after Mayo's
   `wsl --shutdown` the session-start hook auto-remounted D: unattended
   (hook line: "guard reported a failure; remount attempted via
   scratch_mount.sh --fix (passwordless d-remount), then re-checked");
   scratch_mount --check printed D-MOUNT-OK, disk_guard passed 5/5, and
   the scratch tree survived intact (reports/brief11/restart_check2.txt).
   Job 8 closed: reverify.txt ends ALL MET and reports/brief11/ is
   allowlisted on the mirror.
5. **ECC INSTALLED at v2.2.1** (brief 07 jobs 5-8 done, APPROVED ECC
   2026-09-27): 814 files + 24 hooks merged (unlazy Stop hook preserved,
   hooks now Stop 8 / PreToolUse 11), doctor clean, context cost +10815
   tokens (39837 to 50652 first-turn input), ECC data capped by disk-guard
   check 4 at 10 MB (1.0 MB used), rollback documented (uninstall dry-run
   would remove 816; B2 pre-ECC backup verified). Proof: reports/brief07/
   reverify.txt ALL MET 34/34 at H=b43434a. GateGuard fact-forcing gates are
   LIVE on Bash and first-touch Edit/Write: answer the requested facts, do
   not disable them.

## Hand-run command reference

| Task | Command |
|------|---------|
| Full flash episode | ~/yt-digest/.venv/bin/python tools/produce_v2.py <slug> --flash --event-id <id> [--query "..."] |
| Facts only | ~/yt-digest/.venv/bin/python tools/sofascore_client.py --event-id <id> --slug <slug> (or --search "...") |
| Render a 3D board (Modal) | ~/yt-digest/.venv/bin/python tools/pod_build.py render3d <spec> --slug <slug> |
| Render a 2D board (Modal, audited) | ~/yt-digest/.venv/bin/python tools/pod_build.py render2d <spec> --slug <slug> |
| Assemble + pull home | ~/yt-digest/.venv/bin/python tools/pod_build.py assemble --slug <slug> |
| Pull the final home | ~/yt-digest/.venv/bin/python tools/pod_build.py pull <slug> |
| Judge a frame (stable) | ~/yt-digest/.venv/bin/python tools/glm_judge.py --image <file> --runs 5 "<prompt>" (use the median) |
| Emit board specs from live data | tools/stats_spec.py / momentum_board.py / xg_flow_board.py / shotmap_board.py / avgpositions_board.py --slug <slug> |
| Data check (specs vs raw) | ~/yt-digest/.venv/bin/python tools/board_data_check.py --slug <slug> |
| Tool census | ~/yt-digest/.venv/bin/python tools/census.py |
| Long-lane pipeline | ~/yt-digest/.venv/bin/python tools/produce_v2.py <slug> --query "<match>" |
| Archive to B2 with manifest | ~/yt-digest/.venv/bin/python tools/b2_archive.py --file <f> --project soccer-channel --task "<t>" --produced-by "<model> via Claude Code" |
| Upload to YouTube (private) | ~/yt-digest/.venv/bin/python tools/youtube_upload.py |
| Topic research | tools/viral_angle.py, tools/agent_reach_research.py, tools/fresh_fetch.py |
| Goal ground truth | tools/scoreboard_scan.py |

## Facts that were measured, do not re-derive

- Output is 1920x1080. ffprobe verified every build.
- Sofascore shotmap playerCoordinates: x measures distance from the goal the
  team ATTACKS (bournemouth event: goals at x=11-15, per-side means ~13,
  measured 2026-09-23). The board page maps home to (100-x) [right goal] and
  away to x [left goal].
- Scoreboard scanner beats LLM video reading on timestamps (Stage 15 litmus).
  Use the scanner for WHEN; the session model for WHAT.
- Modal costs (measured from `modal billing report --start D --end D --json`,
  summed over entries): brief 05 (2026-09-23) = $0.3063 across 43 runs;
  brief 06 (2026-09-27) = $0.0769 across 15 runs. Month summary: metered
  ~$2.06 of credits, billed $0.00 (starter credits cover it).
- The B2 bucket is the archive of record: rclone remote "b2", bucket
  "mendymax-archive", per-project prefix, manifests written by
  tools/b2_archive.py at archive time.
- The public mirror is allowlist+scan locked (brief 04 job 4): 11 project
  docs, empty vault list, secret scan blocks the push on any hit.
- Sofascore average-positions averageX/averageY are 0-100 already (do NOT
  scale); lineups give formation + 11 starters per side.
- The Modal pod runs /vol/tools/board_html.js + board_page.html from the
  VOLUME: after editing either, `modal volume put` both to /tools/ or the
  pod silently runs the stale renderer (the first page-audit mirror test
  "passed" because of exactly this, 2026-09-26).
## Animated boards (brief 12, 2026-09-28)

- The ONE 2D renderer (board_page.html + board_html.js) gained kind
  "timeline": a base board (white rounded card, thin dark pitch, player
  discs with name labels, dark blue grid surround — benchmark-B look per
  decision 1) plus an ordered step list (reveal / highlight / number / line
  / zone / ball_path / caption / hide). The page state at time T is a pure
  function of (spec, T): frame-accurate per-frame capture, 30 fps, H.264
  1920x1080 on Modal. Format: docs/board_timeline.md; emitter:
  tools/brief12_timeline.py; gate oracle: tools/brief12_check.py (GATES
  B56-B61, all passing).
- The rendered-page audit now runs at EVERY STEP END (not just the final
  frame): drawn names must match the facts XI by jersey+name pair, number
  chips must be a contiguous 1..N on visible players, line/zone endpoints
  visible, name labels must not overlap, the "VS <opponent>" caption must
  name the facts opponent. Positive control: a swapped name FAILS the audit
  (B59). A planted fake position FAILS the data check (B61).
- Label placement is solved in the emitter (global knowledge of all boxes,
  label_dir per player); the page only draws what the spec says. A page-side
  greedy fallback exhausted its candidates and shipped a BELLINGHAM/MBAPPE
  label collision in the first smoke render — the solve lives with the
  emitter and the audit enforces the result.
- Sofascore exposes NO pass/possession sequences (7 candidate endpoints
  probed, all empty — reports/brief12/probes.txt): a traveling ball is
  SCHEMATIC, labeled on the board itself and in the spec.
- Proof boards (full-res MP4s archived to B2, contact sheets + 640px GIFs +
  visual_check.md in reports/brief12/): timeline_formation (Sevilla 1-3
  Barcelona, 8.5 s), timeline_runners + timeline_move (Barcelona 7-2 Real
  Racing Club, 9 s / 10 s). Render wall times 108/54/88 s; brief-12 Modal
  window cost $0.0182 (billing report).
- Fetch reliability: Akamai held a 403 challenge against every curl_cffi
  impersonation for 40+ min (residential AND Modal egress). The real cached
  Chromium answers: sofascore_client falls back per round to
  tools/sofascore_chrome_fetch.js. modal volume put remote paths are
  RELATIVE to the volume root ("/vol/..." creates an unmounted vol/ tree).

## Board sizing + freeze-frame overlays (brief 13, 2026-09-28)

- Boards fill the frame now (the brief-12 coordinator score was 5/10 for
  look): card 1760 px wide = 91.7% of 1920 (floor 88%), pitch fills the card
  (34 px margins), and the element minima are AUDITED PER KEYFRAME at
  1080p: disc r>=42 (84 px disc), number chip r>=26, name label 36 px —
  the 480p readability floor (1080p/480p = 2.25). Chip-vs-label and
  label-vs-disc overlaps are audited too (a chip on a label hides text).
- Half-pitch framing: spec field "frame" full|left-half|right-half|auto
  (default auto) — auto counts only ACTIVE players (jersey-targeted by any
  step) plus schematic ball paths; a board that reveals 6 of 11 players
  frames the half they occupy. The runners board now frames the right half.
- Freeze-frame overlays on footage (benchmark-A signature, decision 2): new
  kind "footage_overlay" in the SAME renderer. base {clip, freeze_at} +
  image-space steps (pixel coords on the 1920x1080 frozen frame):
  ground_ellipse / chip / link / unit / arrow. Clip play-in, freeze, overlay
  animate-in, hold, fade-out, resume — state still a pure function of
  (spec, T): pod-side ffmpeg pre-extracts the play frames, the page draws
  them (no <video>, no wall clock). pod_build.py renderfootage drives it.
- Identity evidence rule (brief 13 job 3): a chip that names a player must
  carry source "lineup:<jersey>" and that jersey must be in a facts XI
  (either team — footage frames are mixed; a jersey can exist in BOTH XIs,
  14 = Adeyemi AND Gueye, matched by name too). No evidence, no name:
  editorial chip or unlabeled marker. Lived examples in the proof: the
  number 19 wall shirt (a substitute) and the stranded keeper carry no name.
- Marking (job 4): frames extracted ON THE POD, stamped with ids before
  viewing (vision batch rule), foot points read off 100px/50px grids,
  reports/brief13/marks.json holds marks + evidence + plans; the specs are
  EMITTED from marks (brief13_overlays.py), never hand-typed.
- Drift control (job 6): the rendered ellipse positions are compared
  against the marked foot points; a doctored spec with one ellipse shifted
  150 px FAILS (brief13_check.py drift, gate B65).
- Proof: 3 re-sized boards + 4 overlay clips (all 10.5 s, 1080p H.264),
  audits clean inside every render; visual_check.md PASS; 7 MP4s + manifests
  archived to B2 under soccer-channel/2026-09-28/; render times + ~$0.09
  brief-13 pod cost (billing report, minus brief-12's known $0.0182).
- Gates B62-B68 written BEFORE implementation (unlazy); reverify ALL MET
  55/55 after the last code commit (reports/brief13/reverify.txt).
