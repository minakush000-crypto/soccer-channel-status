# BRIEF 06 REPORT — one fact source, no unsourced numbers, check the rendered frame

Ran 2026-09-26/27 by glm-5.3-flash:cloud via Claude Code (tmux), repo
~/yt-digest/soccer-channel. Ten jobs in order, one commit per job. Answering
briefs/brief06.md (checkpoint 4f42245). Every visual claim points to a PNG
under reports/brief06/; every before/after pair has md5sum output below;
every path above was `ls`-verified before being cited (23/23 existing at
write time).

    ● verified   claim (command that proved it)
    ◑ believed   claim (why)
    ○ unchecked  what I did not examine
    ★ fragile    what breaks if touched

## Per-job verification

- ● **Job 1 (one fact source, Sofascore, every competition).**
  `produce_v2.py` step 1 now calls only `tools/sofascore_client.py`
  (`--event-id` or `--search`); `--date-range` removed. ESPN
  `match_data.py` → `retired/match_data.py` with a RETIREMENT_NOTES entry;
  `grep -rnE "^\s*(from|import) match_data\b|match_data\.py\"" tools/*.py`
  → no output (gate B17). Proof: fresh fetch to
  /tmp/brief06/madrid_match_data.json then a field-by-field diff against
  the committed facts file: **40 numeric fact fields equal** (all mapped
  stats, all raw stats, scores 2-1, formations 4-3-3/3-5-2, venue,
  competition, goal minutes 14/23/77, goal sides, goal scorers). Diffs that
  remain are non-numeric: goal text format, jersey int→string, avg
  positions now filtered to the XI unrounded (the committed file carried 14
  home + 16 away entries incl. 3 subs, rounded). Bournemouth: fresh event
  fetch says Bournemouth 2 - 2 Brentford = the committed facts and the
  boards; stat_card rows 4/4 MATCH + possession 56/44 MATCH against the
  tracked raw cache from the Sep-13 fetch. Two bugs caught by this proof:
  (1) the /statistics response carries ALL/1ST/2ND periods and the mapper
  let 2ND overwrite ALL (possession 36→42%, shots 16→10 = the second half
  posing as the match) — ALL is now preferred; (2) Sofascore RETIRES old
  events' sub-endpoints (16363640 empty 15 days post-match, while the
  older 16938768 serves everything) — the raw cache written at fetch time
  is the only durable copy, so `renders/<slug>/sofascore_raw.json` is now
  tracked in git. Search APIs return empty from this IP (measured); the
  client resolves events by --event-id or the team-id sweep
  (tools/sofascore_team_ids.json, accumulated: real madrid 2829, inter
  2697, bournemouth 60, brentford 50).
- ● **Job 2 (momentum from data).** `tools/momentum_board.py` rewritten to
  read the raw cache: **92 real graphPoints** (minutes 1-90.5, values
  -100..108, normalized), goal markers straight from raw incidents with
  post-goal running scores: `1-0 MBAPPÉ 14'`, `2-0 VALVERDE 23'`,
  `2-1 AUGUSTO 77'`. Sofascore HAS /graph for 16938768 (first fetched
  through api.sofascore.app when .com challenged) — the board stays and
  script line 74 is untouched. Spec stamps: source + fetched_at.
  Before/after md5 (boards/momentum.mp4):
  `e5d5ef76e23e70be0921087d9bf31d46` (before, invented data) →
  `bc1d8279e0db085591ceeb8af57a7c81` (after, real data). Frames:
  reports/brief06/board_momentum_before.png, board_momentum_after.png
  (both opened). **Who wrote the invented file:** session 50fa196e
  (glm-5.3-flash:cloud via Claude Code, the EP001 flash-rebuild session),
  2026-09-21T05:08:38Z, a bash heredoc
  `cat > renders/2026-09-08_real-madrid-inter/board_specs/momentum.json
  <<'EOF'` — minutes/momentum/goals typed by hand in the same batch that
  wrote stats_possession.json and a scoreline_outro stub (extracted from
  the session transcript's tool_use record; the file has no git history
  because board_specs/ was gitignored until job 3).
- ● **Job 3 (every spec names its source; every number checked).**
  board_data_check.py rewritten: source + fetched_at REQUIRED on every
  spec; every numeric leaf must be covered by a raw-data check or a
  documented render-parameter exemption (numeric-leaf walk); 3D customs
  check goal identity/pre+post scores/clocks/players-vs-lineups
  (substitutions in scope after their minute — goal3_pattern's
  Stones/Sučić/Zieliński/Bonny are real raw substitutions at 46'/60')/
  formations; layout coordinates structurally bounded 0-100 and documented
  as drawing instructions. Sources: the RAW cache, with printed fallbacks.
  Results:
  - Madrid: **DATA-CHECK OK (7 specs + script)** — counter_map PASS,
    formation_clash PASS, goal3_pattern PASS, valverde_strike PASS,
    momentum PASS (curve regenerated from raw within 0.0015), 
    stats_possession PASS (after fixing possession 35.6/64.4 → 36.0/64.0 —
    the fetched values; 35.6 was invented precision the old ±0.6 tolerance
    let pass), scoreline_outro PASS.
  - Bournemouth: **DATA-CHECK OK (6 specs + script)** with 3 UNVERIFIABLE
    rows (momentum curve, shotmap positions, xgflow series — the raw
    endpoints are retired for this event; reported, not failed).
  - The brief-05 "14 specs" is explained and fixed: the old checker had
    `checked += 1` twice per spec (double count); Madrid has 7.
  - Bournemouth previously FAILED the check silently (10 mismatches: the
    checker read a `minute` key only Madrid's file had, and the scoreline
    regex false-positived on "4-2-3-1" formations) — both fixed.
  - Specs + raw cache tracked in git (both .gitignore files updated;
    `git check-ignore -v` shows the negation rules).
- ● **Job 4 (check the rendered page).** board_page.html records every
  fillText (text, x, y, colour); board_html.js takes the facts file as a
  3rd argument, renders the FINAL frame, reads the drawn numbers back and
  checks them against the facts BY TEAM SIDE (possession bar columns, stat
  rows paired by drawn label, scorelines, momentum goal labels with
  running score + scorer-or-team + minute, xgflow end labels, outro chips,
  avgpos jerseys vs the facts XI). pod_build render2d REQUIRES the facts
  file for slug renders and refuses an unauditable render. Proof:
  reintroducing the brief-05 mirror bug in a scratch spec
  (`render2d mirror_check --slug mirror-check`) → exit 1,
  `PAGE-CHECK FAIL for mirror_check (8)` naming every swapped number
  (reports/brief06/pagecheck_mirror_fail.log); the real stats_possession
  render → `PAGE-CHECK OK stats_possession: every drawn number matches the
  facts file by team side (19 draws audited)`
  (reports/brief06/pagecheck_real_pass.log + pagecheck_real_pass.png,
  opened: 36% REAL MADRID left, 64% INTER right, rows 16/21, 8/7, 4/11).
  Gate B19. Two operational catches: the pod runs /vol/tools copies, so
  the renderer files must be volume-put after edits (the first mirror test
  silently ran the stale renderer and "passed"); and the audit's own
  goal-label check initially compared labels to the FINAL score instead of
  the running score — caught when the momentum render failed, fixed, then
  it passed (19 draws audited).
- ● **Job 5 (census by script).** tools/census.py: lists tools/ (USAGE.md
  exempt as the folder README), parses every TOOLS.md row, fails on a file
  with no row, a row naming a missing file, a classless row, or class
  totals ≠ file count. First run FAILED with exactly the brief's
  complaints + more (no rows: census.py, sofascore_client.py,
  sofascore_team_ids.json; missing file: match_data.py; totals 25+13=38 ≠
  40; and the brief-05 census never counted board_page.html at all — 41
  files on disk, not 38). TOOLS.md fixed until: **`CENSUS OK files=40
  wired=26 handrun=14`** (full table in the pass output; gate B20).
- ● **Job 6 (record the 3D decision).** md5sum re-verified:
  `2ef1613ec604ad90e500aeb593423ce8` for BOTH
  reports/brief05/3d_formation_clash.png and 3d_formation_clash_v2.png —
  one file under two names, no comparison ever happened.
  DECISIONS.md 2026-09-26: "v1 retired because it cannot run on Modal; no
  side-by-side comparison was done." The duplicate PNG deleted
  (git rm); 3d_formation_clash_v2.png remains the only artifact.
- ● **Job 7 (brief 05 loose ends, with commands).**
  a. LANE_PLAN.md history: 22 commits touch it, 21 distinct blobs scanned
     with the push_status.sh secret patterns → **0 token/keyassign/phone/
     email hits; 9 money-pattern lines in the 3 latest blobs** (spend-table
     rows like "~$1.50-2.60" + "nothing left running"). No credentials in
     any version.
  b. `tut` is a MINIFIED identifier in claude-mem's bundled
     worker-service.cjs, and it is build-specific: in the ACTIVE
     marketplace copy
     (~/.claude/plugins/marketplaces/thedotmack/plugin/scripts/
     worker-service.cjs) `var tut=["https://beacon.claude-ai.staging.
     ant.dev","https://claude.fedstart.com",
     "https://claude-staging.fedstart.com"]` — the approved-values
     allowlist for CLAUDE_CODE_CUSTOM_OAUTH_URL, used only by
     `if(!tut.includes(n)) throw Error("...not an approved endpoint.")`
     (env var unset here → never used). In the cache copies the same name
     means unrelated things (13.24.1: the CLAUDE_SNIP env var; 13.24.23:
     the CLAUDE_CODE_ARTIFACT_TYPES flag; 13.25.2: a path helper). Brief
     05's claim was about the marketplace copy and holds; the minified
     name itself is not stable across builds.
  c. The 8 board kinds swept in brief 05 job 1: (1) stats rows,
     (2) possession bar, (3) scorebug, (4) momentum fills/tags,
     (5) xgflow series+end labels, (6) shotmap, (7) avgpos, (8) outro
     (verbatim from reports/brief05/REPORT.md job 1).
  d. There were never 14 specs: the old checker counted Madrid's 7 twice
     (`checked += 1` twice in its loop). The real Madrid set:
     renders/2026-09-08_real-madrid-inter/board_specs/{counter_map,
     formation_clash,goal3_pattern,momentum,scoreline_outro,
     stats_possession,valverde_strike}.json (slug
     2026-09-08_real-madrid-inter). Bournemouth has its own 6
     (slug 2026-09-12_bournemouth-brentford), so the two episodes total 13
     specs — the "14" was an arithmetic bug, now fixed.
  e. reports/brief05/board_momentum_before.png shows the **Real Madrid v
     Inter (EP001)** momentum board (opened: "MOMENTUM OF THE NIGHT
     (+ MADRID / - INTER)", MBAPPE 14'/VALVERDE 23'/AUGUSTO 77' labels) —
     the same board as board_momentum_after_ep001.png, before/after of the
     brief-05 job-2 fill fix. No `board_momentum_before_ep001.png` ever
     existed; the report's `{before,after}_ep001` brace notation was
     misleading, not a missing file. md5 of the three momentum PNGs:
     before c153ad1f..., after 58e94548..., after_ep001 8c666f38... —
     three distinct files, no image under two names.
- ● **Job 8 (re-render and check the whole video).** One wired command:
  `produce_v2.py 2026-09-08_real-madrid-inter --flash --event-id 16938768
  --query "Real Madrid Inter"` — facts refetched through step 1
  (fetched_at 2026-09-27T01:56:32Z), script kept (see the fix below), data
  check passed, script-referenced boards only (stats_spec/xg_flow/shotmap/
  avgpositions emitters skipped: their boards are not in the script), all
  three 2D boards re-rendered THROUGH the page audit (scoreline_outro 10
  draws, stats_possession 19, momentum 19 — all PAGE-CHECK OK,
  reports/brief06/flash_run_checks.log), 4 3D boards reused (specs
  unchanged since brief 05), assembled on Modal, pulled home.
  `ffprobe -v error -show_entries format=duration,size -of csv=p=0
  renders/2026-09-08_real-madrid-inter/final_video.mp4` →
  **140.960000,75581383** (v7). The manifest written by assemble
  (assemble_manifest.json, now pulled home) gives the REAL segment times;
  one frame from the middle of every board segment extracted and ALL
  SEVEN OPENED: reports/brief06/final_boards/{formation_clash,counter_map,
  valverde_strike,stats_possession,goal3_pattern,momentum,
  scoreline_outro}.png — all correct (4-3-3 v 3-5-2 with both real XIs;
  GOAL 1/2/3 boards with the right clocks and scorebugs; 36/64 bar;
  the real momentum curve; the 2-1 outro). The stats_possession and
  momentum mid-frames catch the boards' count-up/sweep animation mid-loop
  (a 10.75s/9.16s segment cut from a looping 12s/9s board) — their
  completed states are the audited ones (pagecheck_real_pass.png, job-2
  after frame). Final data check after the rebuild: **DATA-CHECK OK
  (7 specs + script)**. Three pipeline bugs fixed because the run hit
  them: (1) the flash run REGENERATED EP001's fact-checked script (365
  words < the long-lane 1400 minimum) — caught before any render, script
  restored from git, step1b is keep-if-exists and --flash skips script
  gen; (2) fetch_all was called without the slug so every pipeline fetch
  CLOBBERED the raw cache (graph vanished mid-run) — the merge is
  slug-keyed now; (3) the assemble's final output was root-level on the
  volume (two episodes' finals would collide) and the pull block pulled
  twice — both fixed, the re-assemble pulled from
  out/<slug>/final_video.mp4.
- ● **Job 9 (canonical sync).** STATUS.md, CONTEXT.md, DECISIONS.md,
  TOOLS.md, GAPS.md all stamped 2026-09-27 (brief 06). DECISIONS.md
  records: Sofascore the only fact source + raw cache tracked; ALL/2ND
  statistics rule; specs require source + fetched_at with the numeric-leaf
  walk; rendered-page audit added; census gate added; 3D v1 retired
  without comparison; the momentum board regenerated from real data; the
  flash-lane bugs. GAPS.md: two new open gaps (Bournemouth's retired raw
  endpoints; the IP-limited event resolver) + one cosmetic (3D label
  collisions seen in the opened frames); the "3 POINTS" chip gap closed
  (now rule-checked: 3 iff non-draw).
- ● **Job 10 (cost).** `modal billing report --start 2026-09-23 --end
  2026-09-27 --json` → 43 entries, **$0.3063, all on 2026-09-23** (brief
  05's session ran into that day; no other Modal use between then and
  brief 06). `modal billing report --start 2026-09-26 --end 2026-09-28
  --json` → 15 entries, **$0.0769, all on 2026-09-27** (brief 06: 3
  mirror-test renders + 2 stats renders + momentum renders + 2
  assembles). `modal billing summary` → billed $0.00 (starter credits
  cover the ~$2.06 month metered). Not yet billable; the numbers are
  metered cost, not charges.

## Most important thing learned

The invented momentum board was not a one-off: it was the NORM for episode
specs written outside the pipeline (hand-typed heredocs, no source, no
history), and the old checks could not see it because they compared specs
to the very file the specs were copied from. The fix is structural, not a
one-off correction: facts flow one way (Sofascore → raw cache → mapped
facts → specs → drawn page), every hop carries a timestamp and a source,
and two independent checks (spec-vs-raw, page-vs-facts) can each fail the
build. The page audit then immediately caught a bug in ITSELF (final score
vs running score), which is the system working: nothing is above being
checked against the data.

## Cost of this session

Modal: $0.0769 metered across 15 runs (billed $0.00, credits); brief 05's
measured spend recorded for comparison ($0.3063 / 43 runs). Wall time:
~3.5 hours. Local disk: renders/ working files only (raw download
unchanged; boards + final are volume outputs pulled home). B2: v7 final +
brief06 proof set archived with manifests (soccer-channel/2026-09-27/).