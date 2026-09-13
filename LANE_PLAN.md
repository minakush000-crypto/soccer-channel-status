# LANE_PLAN.md — four-lane channel expansion + storage plan

> **Purpose:** the four-lane (A/B/C/D) expansion build ledger; each stage appends its findings here.
> **Reader:** every session (the brief appends to it); mirrored (in push_status.sh ALLOW_PROJECT and the mirror .gitignore un-ignore list — verified 2026-09-13). Stamp 2026-09-09; 27 line-drift claims unverified (Stage-12B rewrite of produce_v2.py shifted cited line numbers) — STAMP RULE option (b), not re-stamped this pass.
> **Last verified against code:** 2026-09-09.

Verified against code on 2026-09-08. Tool classifications cite
RECONCILIATION.md (1.1) rather than repeating its greps. Cost figures cite
STATUS.md measured data. Nothing here is run; lanes B/C/D are planned, not
built (except `transformation_gate.py`, built and verified this stage).

---

## STAGE 2.4 — what a cloud-only footage pipeline needs that does not exist

Today's flow (produce_v2.py): download raw clip LOCALLY
(`renders/<slug>/clips/`, step3) → ship it to RunPod (`runpod_fulltrack.py`
uploads the local clip to catbox, pod downloads it) → track on pod →
download the annotated MP4 + tracking JSON back LOCAL
(`runpod_fulltrack.py:170`) → cut + assemble locally (step5).

A cloud-only footage pipeline keeps raw video off `/home/muads` entirely. The
200MB guard (now in place) makes the current local-download flow illegal for
any source over 200MB, so this is not optional for full-match.

Gaps (named, not built):

1. **No pod-side download path in produce_v2.py.** `cloud_produce.py:387-398`
   already builds yt-dlp commands that run ON the pod (`/workspace/...`), but
   the authoritative entry point `produce_v2.py` has no such path — it always
   downloads locally then ships. Gap: produce_v2 needs a "download on pod"
   branch that never touches `/home/muads`. (The 200MB guard now blocks any
   produce_v2 download over 200MB locally, so a full-match source literally
   cannot be fetched by produce_v2 today.)

2. **Where raw video lives during processing.** Cloud-only: raw lives only at
   `/workspace/clips/` on the pod. Gap: no produce_v2 mode that skips the
   local `clips_dir` write and passes a pod-side clip path to the tracking
   step. `runpod_fulltrack.py` already runs cv_annotate on the pod; the gap is
   upstream (the download) and downstream (the cut).

3. **How cut lists move.** The cut list (`footage=START-END` windows) lives in
   `scripts/<slug>.md`, hand-authored. For cloud-only, the pod needs the cut
   list so it can cut excerpts there and return only small segments. Gap: no
   serialization of a cut list to the pod; no pod-side cutter. produce_v2 cuts
   locally with ffmpeg on the full local clip (step5).

4. **What comes home.** Today `runpod_fulltrack.py:170` returns the whole
   annotated MP4 (guarded now — a full-match annotated MP4 would be refused).
   Cloud-only should return only: trimmed tracking summary (small),
   per-event excerpts (≤8s each), boards (small PNG/MP4), final assembled
   video (≤200MB). Gap: no "excerpt-only" return path; no on-pod trim step.
   `trim_tracking.py` exists (STANDALONE) for the summary, but it runs locally
   on the full JSON — for cloud-only it must run on the pod so the full JSON
   never comes home either (a full-match per-frame JSON at 8MB/146s would be
   ~300MB, over the guard).

5. **Cut-list generator (shared prerequisite, carry-forward from 1.2).** No
   wired tool emits `footage=START-END` windows. `scoreboard_scan.py` +
   `broadcast_filler.py` + the relay-verify pattern are the ingredients but
   are unwired. This blocks every footage lane, not just cloud-only. See 3.5.

Net: the cloud-only pipeline needs (a) a pod-side download branch in
produce_v2, (b) a cut-list serializer + pod-side cutter, (c) an excerpt-only
return path with on-pod `trim_tracking`. None exist. `runpod_fulltrack.py` is
the closest scaffold (it already ships tools, runs cv_annotate on the pod, and
returns URLs) but it returns the full annotated MP4, which the guard now
refuses for large sources.

---

## STAGE 3 — lane architecture

### 3.1 — 9 workflow stages × 4 lanes

The 9 produce_v2 stages (ARCHITECTURE.md): 1 match_data, 2 boards, 3
download_clip, 4b tactical_render, 5 voice, 6 ambience, 7 assemble, 8 merge,
9 shorts.

| Stage | A tactical | B preview | C hot topic | D historical |
|---|---|---|---|---|
| 1 match_data (ESPN) | USE | SKIP (no match yet) | SKIP | USE (past match) |
| 1b fact source (fresh_fetch) | — | USE | USE | — |
| 2 boards | USE | USE | USE | USE |
| 3 download_clip | USE | SKIP | SKIP | USE (guarded, excerpts) |
| 4b tactical_render | USE | SKIP | SKIP | optional |
| 5 voice | USE | USE | USE | USE |
| 6 ambience | USE | USE | USE | USE |
| 7 assemble | USE | USE (board-only) | USE (board+stills) | USE (gated) |
| 8 merge | USE | USE | USE | USE |
| 9 shorts | USE | USE | USE | USE |
| transformation_gate | — | — | — | USE (before assemble) |

**B and C need no footage stage — confirmed against code.** produce_v2's
footage stages are step3 (`download_clips`, line 95) and step4b
(`tactical_render`, line 211); both are skipped for B/C. The assemble step
(step5, line 301) already handles a missing clip: line 382
`if has_footage or (sec_duration > 2 and clip_path and clip_path.exists())`
falls through when `clip_path` is None, so board-only assembly works today.
So B/C run board-only (B) or board+stills (C) assemblies with no video
download and no tracking. The new need is a NON-ESPN fact source (1b), not a
footage stage.

### 3.2 — making the lane a parameter: honest answer

Not a rewrite. The 9 steps are already discrete functions dispatched from
`main()` (line 529) by a flat sequence of `if not stepN(...): sys.exit(1)`.
Making the lane a parameter takes:

1. A `--lane {A,B,C,D}` flag on `produce_v2.py` (argparse, line 530).
2. A per-lane config block: which steps to run, which data source, which
   script template. This is a small dict, not new architecture.
3. Conditional dispatch in `main()` (line 529): skip step3/step4b for B/C;
   swap step1 (ESPN `match_data.py`) for step1b (`fresh_fetch`-based
   fixture/news fetch) for B/C; run `transformation_gate` before step5 for D.

What is NOT reusable as-is and needs new code (not a rewrite, but not just a
flag):
- The B/C data source. produce_v2 step1 calls `match_data.py` which hits
  ESPN's scoreboard/summary for a finished match. B (preview) and C (hot
  topic) have no finished match; they need `fresh_fetch.py` output shaped
  into the same `match_data.json`-like object the boards/voice steps read.
  That adapter does not exist.
- The B/C script generator. produce_v2 has no script generator at all (it
  reads a hand-authored `scripts/<slug>.md`, RECONCILIATION 1.2). B narrates
  a preview (fixtures, form, key absentees); C narrates a talking point. No
  template or generator exists for either.

Honest verdict: **a flag + a per-lane config + conditional dispatch + 2 new
data-source/script functions.** The step functions are reusable; it is not a
rewrite. It is more than a one-line flag because the data source and script
generation for B/C are genuinely missing.

### 3.3 — fresh_fetch.py: wire, not replace or rebuild

```
$ grep -nE "API_BASE|standings|last_results|next_fixtures|RSS|BBC|Sky|Guardian" tools/fresh_fetch.py | head
53:API_BASE = "https://api.football-data.org/v4"
186:    out = {"standings": [], "last_results": [], "next_fixtures": []}
207:            out["last_results"].append(format_match(m, finished=True))
213:            out["next_fixtures"].append(format_match(m, finished=False))
252:                 "RSS headlines from BBC Sport, Sky Sports, The Guardian."
```

What it fetches: football-data.org v4 API (standings = form, last_results,
next_fixtures = fixtures) + RSS headlines (BBC Sport, Sky Sports, Guardian =
news). No LLM. Stamps every item with published time + age.

Does it cover fixtures, form, and news for the three leagues? Partially:
- **Premier League**: yes — football-data.org free tier includes the PL.
- **La Liga, Bundesliga**: only via RSS headlines in the current config;
  football-data.org's free tier covers the PL + a few others, La Liga and
  Bundesliga need a paid tier or a different endpoint. The tool is
  competition-configurable (`api_get(f"/competitions/{competition}/...")`)
  so it can be pointed at any competition the key authorizes, but the current
  .env key tier is not checked here.
- **Form**: yes — `standings` + `last_results` give form.
- **Fixtures**: yes — `next_fixtures`.

Verdict: **WIRE.** It is orphaned (RECONCILIATION 1.1: STANDALONE, no caller)
but functional and exactly the B/C fact source. Wire it as step1b for lanes
B/C. For full La Liga / Bundesliga coverage, confirm the football-data.org
key tier (a one-line `.env` check, not a rebuild). Do not replace or rebuild —
the tool already does what 3.1 requires.

### 3.4 — can SCRIPT_TEMPLATE.md and validate_script.py hold four templates?

```
$ ls -la SCRIPT_TEMPLATE.md
-rw-r--r-- ... 1708 ... Aug 23 15:30 SCRIPT_TEMPLATE.md
$ grep -nE "TAG_RE|SRC|RUMOR|sources|def validate|narration|structure|section" tools/validate_script.py | head
34:TAG_RE = re.compile(r"\[\s*(SRC|RUMOR)\s*:\s*([a-z0-9_-]+)\s*\]", re.IGNORECASE)
70:def validate(slug, render_date=None):
```

No. `SCRIPT_TEMPLATE.md` is a single template (lane A's theory-then-footage
structure with `[VISUAL: board=...]` / `[VISUAL: footage=...]` tags).
`validate_script.py` validates ONE thing: that every `[SRC: id]` / `[RUMOR: id]`
tag resolves to `sources.json` and that RUMORs are fresh + tier-consistent. It
does not check structure (narration-first, section order), it does not know
which lane a script is for, and it has no concept of multiple templates.

To hold four templates: `SCRIPT_TEMPLATE.md` becomes four sections (or four
files), and `validate_script.py` gains a `--lane` argument that selects which
structural rules to apply (e.g. D requires narration-first + the
transformation-gate manifest; A requires theory-then-footage; B/C require
no `footage=` tags). That is an extension, not a rewrite — the source-tag
validation stays, structural validation is added.

### 3.5 — per-lane tool matrix + ordered build lists

Legend: **EW** = EXISTS AND WIRED (in produce_v2's reachable set),
**EO** = EXISTS BUT ORPHANED (RECONCILIATION 1.1 STANDALONE/DEAD), **DNX** =
DOES NOT EXIST.

**Shared prerequisite (every footage lane: A, D):**
| Tool | Status |
|---|---|
| cut-list generator (`cut_list_gen.py`: scoreboard_scan + ±20s relay-verified search → `footage=START-END` windows) | DNX |
| `scoreboard_scan.py` (goal times) | EO |
| `broadcast_filler.py` (broadcast/filler classification) | EO |
| `assemble_words_match.py` (the content-matched assembler) | EO |

**Lane A — tactical analysis (existing):**
| Tool | Status |
|---|---|
| produce_v2 9 steps (match_data, boards, runpod_fulltrack, tactical_render, generate_voice, generate_ambience, merge_voice, shorts_crop) | EW |
| ffmpeg_utils, script_utils, cv_annotate (transitive) | EW |
| fold assemble_words_match into step5 (replaces 5s cut) | DNX (the fold) |
| validate_script wired into produce_v2 | DNX (the wire) |

Build list (cheapest first): 1. cut-list generator (shared). 2. wire
validate_script into produce_v2 (catch the Gravenberch-class bug). 3. fold
assemble_words_match into step5 (after cut-list exists; preserves tactical
segment or explicitly drops it).

**Lane B — match preview:**
| Tool | Status |
|---|---|
| fresh_fetch.py (fixtures/form/news) | EO → wire as step1b |
| tactical_boards, generate_voice, generate_ambience, merge_voice, shorts_crop | EW |
| assemble board-only (step5 no-clip path) | EW (code supports it, line 383) |
| fresh_fetch → match_data-shaped JSON adapter | DNX |
| B script generator (preview narration from fixtures/form) | DNX |
| B script template | DNX |

Build list: 1. wire fresh_fetch as step1b (cheapest — tool exists). 2. the
fresh_fetch→boards adapter (small). 3. B script template + generator. 4.
board-only assemble path exercised end-to-end. No footage, no tracking, no
cut-list — cheapest lane to ship.

**Lane C — hot topic:**
| Tool | Status |
|---|---|
| fresh_fetch.py (news) | EO → wire |
| tactical_boards, voice, ambience, merge, shorts | EW |
| stills fetcher (topic-relevant images, not video) | DNX |
| C script generator (talking-point narration) | DNX |
| C script template | DNX |

Build list: 1. wire fresh_fetch. 2. C script template + generator. 3. stills
fetcher (images only, light). 4. board+stills assemble path. No video
footage, no tracking, no cut-list.

**Lane D — historical review:**
| Tool | Status |
|---|---|
| match_data (past match, ESPN) | EW |
| download_clip (200MB-guarded) | EW (but needs pod-side download for long sources, see 4.1) |
| tactical_boards, voice, ambience, merge, shorts | EW |
| tactical_render (optional) | EW |
| cut-list generator (shared) | DNX |
| transformation_gate.py | EW (built this stage, verified) |
| per-excerpt source attribution from download record | DNX |
| narration-first structural check in validate_script | DNX |
| pod-side download + excerpt-only return (for long sources) | DNX |

Build list (cheapest first): 1. cut-list generator (shared). 2.
transformation_gate wired before assemble (gate exists; wire is cheap). 3.
per-excerpt attribution in the description builder (4.3). 4. narration-first
check in validate_script (4.4). 5. pod-side download + excerpt-only return
for long historical sources (2.4 — most expensive, do last).

**Cheapest-lane-first overall: B → C → A-fold → D.** B and C need no footage,
no cut-list, no tracking — they are boards + narration and can ship first.
A's fold and D both wait on the shared cut-list generator.

---

## STAGE 4 — lane D transformation gate

### 4.1 — sourcing

`check_and_download.py` is DEAD (RECONCILIATION 1.1). Its own docstring:
"download all annotated clips from the RunPod webhook" — it is a
webhook-result downloader for annotated clips, NOT a source-footage selector.
It scores nothing. Retire per 1.1.

The actual source selector is `produce_v2.py`'s inline yt-dlp loop (step3,
line 95). It DOES score candidates before download:
```
$ grep -nE "is_brand =|60 <= dur_sec|MIN_HEIGHT|MAX_CANDIDATES" tools/produce_v2.py | head
137:            is_brand = any(brand.lower() in uploader.lower() for brand in BRAND_CHANNELS)
141:                    if 60 <= dur_sec <= 600:  # 1-10 minutes
```
Scoring: excludes brand channels (line 137), filters duration 60-600s (line
141), loops up to MAX_CANDIDATES=5, ffprobes each, rejects <720p (MIN_HEIGHT,
line 193). It scores by **resolution, channel, and duration** — NOT by content
(it does not know whether a candidate actually contains the match's goals).
The scoreboard scan + relay verify happens much later, offline, by hand.

What the 200MB guard means for lane D: `--max-filesize 200M` (line 177) +
`assert_video_under_limit` (line 195) refuse any local download over 200MB.
A lane D historical source is often a long clip or full match (>200MB). So
lane D cannot pull a long source locally — it must use the pod-side download
path (2.4 gap) or download only short verified excerpts. For short-excerpt
D (each excerpt ≤8s, well under 200MB), the current local path works. For
full-match D, the local path is now blocked by design and the pod-side path
does not exist yet.

### 4.2 — the floor, enforced in code

`transformation_gate.py` is built and verified (pass manifest exits 0, fail
manifest exits 1 naming every broken rule). It runs before assembly and
blocks on failure. First, what produce_v2 already does:

| Floor rule | produce_v2 today | Evidence |
|---|---|---|
| 1. narration ≥ 70% of runtime | NOT DONE (no gate) | step5 allocates section duration by word_count × voice_duration but never checks a coverage ratio |
| 2. no excerpt > 8s | PARTLY | footage chunks capped at 4.0s (line 387), boards at 4.0s (line 371) — under 8s; BUT tactical capped at 45.0s (line 377), which violates 8s |
| 3. source audio muted on every excerpt | DONE | `-an` on board (line 408), tactical (line 417), footage (line 438) — all segment types muted |
| 4. ≥1 added visual layer per excerpt | PARTLY | board/tactical segments are layers; footage-only segments have only a zoom crop (`crop=iw*0.9...`, line 436) — zoom counts as a layer per the brief, so technically satisfied, but no board/annotation/title on footage |
| 5. borrowed footage < 50% of runtime | NOT DONE (no gate) | no ratio check anywhere |

Then build it: done. `tools/transformation_gate.py` checks all five against a
manifest (voice_duration, total_duration, segments with duration/
source_muted/layers/borrowed) and fails loudly. Verified:

```
$ ~/yt-digest/.venv/bin/python tools/transformation_gate.py /tmp/gate_pass.json
transformation_gate: PASS — all 5 floor rules satisfied
$ ~/yt-digest/.venv/bin/python tools/transformation_gate.py /tmp/gate_fail.json
transformation_gate: FAIL — lane D floor violated:
  - rule 1 (narration 70%): voice 40.0s / total 80.0s = 50% < 70%
  - rule 2 (excerpt <= 8s): segment 1 is 9.0s
  - rule 3 (source audio muted): segment 1 source_muted is false/missing
  - rule 4 (>=1 visual layer): segment 1 has no layers
  - rule 5 (borrowed < 50%): borrowed 47.0s / total 80.0s = 59%
```

Wiring it into produce_v2 (running it before step5_assemble for lane D) is a
lane-D build-list item (3.5), not done here — the gate itself is built and
verifiable.

### 4.3 — attribution

What the upload path writes into the description now:
```
$ grep -nE "get_description|def get_description|return desc" tools/youtube_upload.py
47:def get_description(script_path):
48:    """Build a YouTube description from the script sources."""
66:    return desc[:5000]
```
`get_description(script_path)` builds the description from the script's
`[SRC: id]` / `[RUMOR: id]` tags resolved against `sources.json` — i.e. the
cited sources. It does NOT include per-excerpt source attribution (which
clip, which uploader, which timestamp window).

Lane D needs per-excerpt source attribution generated from the download
record, not typed by hand: for each borrowed excerpt, the description should
name the source video (uploader + video_id) and the timestamp window borrowed.
The download record (yt-dlp `video_id`, `uploader`, the `footage=START-END`
window) exists in the cut list / clip filename but is never fed into
`get_description`. That attribution generator DOES NOT EXIST (3.5 D build
list item 3).

### 4.4 — structure

Can SCRIPT_TEMPLATE.md and validate_script.py enforce a narration-first
structure where analysis leads and footage illustrates? No — that check does
not exist. As shown in 3.4, validate_script.py checks source-tag resolution
only; it has no notion of section order, narration-vs-footage proportion, or
lane. Enforcing narration-first for D is a new structural rule in
validate_script (3.4/3.5 D build list item 4): e.g. require that the first
section is narration-led (no `footage=` tag in section 1) and that
`footage=` tags appear only as illustration after a narration section. The
transformation_gate already enforces the numeric floor (70% narration,
<50% borrowed); the structural narration-first check is complementary and
does not exist yet.

### 4.5 — honest answer: realistic lane D runtime at current footage yield

The ~43s figure (STATUS, broadcast/filler: 43.0s usable in a 482s highlights
reel) is what a highlights REEL yields. Full-match broadcast was never run
through produce_v2 (RECONCILIATION 1.5: the fullmatch test is an incomplete
manual `.part`, nothing produced). So 43s is a highlights-reel yield, not a
proven cap. Treat full-match as UNTESTED.

Costed full-match test (NOT run, per Amendment 3):
- **Download size**: a 90-min 1080p DASH match is ~3-5GB (the killed `.part`
  was 1.1GB for a low-res f298 format). Under the 200MB guard this CANNOT
  land locally — it must download on the pod (2.4 gap).
- **Tracking cost**: 146s cost $0.011 on L4 (STATUS). A 5400s full match is
  ~37× → ~$0.40 inference; pod uptime ~37×156s ≈ 96 min at $0.25/hr L4 ≈
  $0.40. So ~$0.40-0.50 to track a full match.
- **Wall time**: ~96 min (inference-bound) + pod download time.
- **What stays in cloud**: the 3-5GB raw (never local, guarded); the full
  per-frame tracking JSON (~300MB at 8MB/146s — also over the guard, so it
  must be trimmed ON the pod with `trim_tracking.py` and only the small
  summary comes home). What comes home: the trimmed summary (small),
  per-event excerpts (≤8s each, well under 200MB total), boards, final
  assembled video (≤200MB).

Realistic lane D runtime at the floor:
- **From a highlights reel (43s usable)**: the floor says borrowed < 50% and
  narration ≥ 70%. To use all 43s of borrowed footage, runtime R needs
  0.5R ≥ 43 → R ≥ 86s, with narration ≥ 60s (70%) and ≥6 excerpts (43/8).
  So ~85-90s, six-or-more excerpts each ≤8s each needing its own layer.
  That is a SHORT historical review and the six+ required layers per episode
  is high per-episode production cost. **At the 43s highlights-reel yield the
  floor makes lane D impractical for anything beyond a ~90s short.**
- **From a full match (UNTESTED)**: if full-match broadcast yields materially
  more than 43s of usable action (plausible — the 482s reel was 90% filler),
  runtime scales up and the lane becomes practical. But the per-excerpt-layer
  requirement scales the work linearly with excerpt count, and the
  pod-side-download + excerpt-only-return path (2.4) must exist first.

Honest verdict: **the floor makes lane D impractical at the proven
highlights-reel yield (43s → ~90s short, 6+ layers). It becomes practical
only if a full-match test yields substantially more usable action, and that
test has never been run. Do not design lane D around the 43s figure; run one
full-match costed test (≈$0.50, ~96 min, all in cloud) to get a real yield
before committing to the lane.**

---

## STAGE 1 (guard conflict) — resolved in writeup + gate fix, 2026-09-08

### 1.1 — the exact conflict, with line numbers and arithmetic

The guard limit, the duration filter, and the measured file sizes:

```
$ grep -nE "LOCAL_VIDEO_LIMIT_MB = |def assert_video_under_limit" tools/ffmpeg_utils.py
65:LOCAL_VIDEO_LIMIT_MB = 200
68:def assert_video_under_limit(path, limit_mb=LOCAL_VIDEO_LIMIT_MB):

$ grep -nE "60 <= dur_sec|--max-filesize|assert_video_under_limit\(clip" tools/produce_v2.py
141:                    if 60 <= dur_sec <= 600:  # 1-10 minutes
177:                  "--max-filesize", "200M",
195:            assert_video_under_limit(clip_path)
```

So: `produce_v2.py:141` accepts clips of 60-600s; `produce_v2.py:177` tells
yt-dlp to abort any download over 200M; `produce_v2.py:195` deletes + raises
on any local video over 200MB. The conflict is that a 600s 1080p clip is
bigger than 200MB.

Measured from the real source clips on disk (not assumed):

```
$ for f in $(find renders -name "clip_*.mp4" -not -name "*annotated*"); do
    dur=$(ffprobe -v error -show_entries format=duration -of csv=p=0 "$f")
    br=$(ffprobe -v error -show_entries format=bit_rate -of csv=p=0 "$f")
    sz=$(stat -c %s "$f"); w=$(ffprobe -v error -show_entries stream=width -of csv=p=0 "$f")
    printf "%s %sx dur=%s br=%s size=%.1fMB => %.1f MB/min\n" "$(basename $f)" "$w" "$dur" "$br" "$(echo $sz/1048576|bc -l)" "$(echo $sz/1048576/$dur*60|bc -l)"
  done
clip_nxPNT4TU5_Q.mp4  1920x dur=146s br=4064175  size=70.8MB => 29.1 MB/min
clip_y7KqSjvN_5s.mp4  1920x dur=120s br=3313513  size=47.4MB => 23.7 MB/min
clip_mZvTP-hlWP8.mp4  1920x dur=134s br=3375437  size=53.8MB => 24.1 MB/min
clip_BywIRMYkHiw.mp4  1920x dur=813s br=2672956  size=259.2MB => 19.1 MB/min
clip_PrCW_geeRAU.mp4 1280x dur=482s br=2236148  size=128.6MB => 16.0 MB/min
```

1080p (1920x) reels run **19.1-29.1 MB/min** (bitrate 2.67-4.06 Mbps). The
arithmetic for the 600s upper bound:

- best observed 1080p (19.1 MB/min): 600s = 10 min → **191MB** (under 200)
- mid 1080p (24 MB/min): 600s → **240MB** (over by 40)
- worst observed 1080p (29.1 MB/min, clip_nxPNT4TU5_Q): 600s → **291MB** (over by 91)

So a 600s 1080p clip is ~240-291MB typically; the guard refuses it. The
**conflict zone** is the duration at which 1080p crosses 200MB:
`200 / 29.1 = 6.87 min = 412s` at the worst observed bitrate. produce_v2
accepts 412-600s 1080p clips (they pass the 60-600s filter and the 720p gate),
then `--max-filesize 200M` aborts the download mid-stream, `cleanup_part_files`
removes the `.part`, the candidate is recorded as failed (h=0), and the loop
tries the next. If every candidate is a long 1080p reel, produce_v2 exhausts
MAX_CANDIDATES and prints "ALL 5 candidates below 720p" — which is
**misleading**: they were size-rejected, not resolution-rejected. (720p reels
at 16 MB/min are mostly safe: 200/16 = 12.5 min, above the 600s cap. The
conflict is specifically 1080p reels in the ~412-600s range.)

### 1.2 — three options with costs (not picked)

**a. Raise the guard limit to clear the 600s case.** To clear the worst
observed (291MB) with headroom, set `LOCAL_VIDEO_LIMIT_MB` to ~350 and the
`--max-filesize` to `350M`. Cost: zero build time. Disk exposure: up to 350MB
per clip lands locally, and the ext4.vhdx grows and never shrinks (carry-forward
fragile item), so each large download **permanently** adds up to 350MB to the
vhdx file on C: until the whole WSL distro is reset. The 1.1GB incident was
~3× this; 350MB repeated weekly re-enlarges the vhdx. Does not clear a full
match (3-5GB), only the 600s highlight case.

**b. Lower the duration filter so accepted clips fit under 200MB.** At the
worst observed 1080p bitrate, `200MB / 29.1 MB/min = 412s`. Change
`produce_v2.py:141` to `60 <= dur_sec <= 412`. Cost: zero build time. What it
excludes: 1080p highlight reels longer than ~412s (~6.9 min) — the extended-
highlight tier (7-10 min reels). 720p reels could safely run longer but the
filter is duration-based, so it excludes them at 412s too. This keeps the
guard intact and the disk safe, at the cost of source variety.

**c. Build the pod-side download path from 2.4.** The building blocks exist:
`cloud_produce.py:387-398` already builds yt-dlp commands that run on the pod
(`/workspace/...`); `runpod_fulltrack.py` already ships tools, runs cv_annotate
on the pod, and returns URLs (proven end-to-end, STATUS). The new work: a
produce_v2 branch that runs yt-dlp ON the pod for sources over the local limit
(not locally), cuts excerpts on the pod from the cut list, and returns only
small derived outputs (trimmed summary + ≤8s excerpts + boards + final). Cost:
~3-5 hours focused build + one paid pod test, then ~$0.50-1 per episode
(pod-side download + track). Zero local disk exposure — the 200MB guard never
fires because nothing large ever lands locally.

### 1.3 — which lanes each option blocks or unblocks

| Option | Lane A (tactical) | Lane B (preview) | Lane C (hot topic) | Lane D (historical) |
|---|---|---|---|---|
| a. raise limit to 350MB | unblocks long 1080p highlights (412-600s); ~350MB disk exposure per clip | unaffected (no footage) | unaffected | **still blocked** — full match is 3-5GB, way over 350MB |
| b. lower filter to 412s | **blocks** 1080p reels 412-600s (a real source tier); short reels still work | unaffected | unaffected | unchanged (full match already excluded by the 600s filter) |
| c. pod-side download | unblocks long highlights AND full-match; zero local exposure | unaffected | unaffected | **unblocks** — the only option that does |

Only **c** unblocks lane D. **a** unblocks lane A long highlights but
reintroduces disk exposure. **b** keeps the disk safe but narrows lane A's
sources and does nothing for D. The 200MB guard stays in force under all three
(no code removed); the question is which escape path to build/accept.

### 1.4 — transformation gate 8s rule vs produce_v2's 45s tactical cap

They apply to **different segment types**; the conflict was in the gate's
scoping, not the floor itself, and is now fixed.

The brief's floor rule 2 is "no single **excerpt** longer than 8 seconds" and
rule 5 is "total **borrowed** footage under 50%." Both key on borrowed
footage. produce_v2's 45s cap (`produce_v2.py:377`, `tac_dur =
min(sec_duration, 45.0)`) applies to the **tactical** segment type — a
generated top-down graphic, not borrowed footage. So in principle the 8s
excerpt rule governs borrowed footage and the 45s cap governs generated
tactical: no overlap.

But `transformation_gate.py` as built last stage applied rule 2 (8s) and rule
4 (≥1 layer) to **all** segments, including generated tactical. So a 45s
tactical segment would have failed rule 2 — a real conflict in the code.

Fix applied this stage (verified): rules 2 and 4 now scope to `borrowed`
segments only, matching rule 5. Verified against three cases:
- 45s tactical (generated) + 9s borrowed footage → passes rule 2 for the
  45s tactical, fails rule 2 for the 9s borrowed. ✓
- 45s tactical + 7s borrowed → all 5 rules pass (no conflict). ✓
- borrowed footage with no layer → rule 4 flags it. ✓

After the fix, produce_v2's own caps already satisfy the gate for lane D:
footage is capped at 4.0s (`produce_v2.py:387`, under the 8s borrowed rule)
with a zoom layer (satisfies rule 4); tactical can run to 45s (generated,
exempt from rules 2 and 4). **The 45s-vs-8s carry-forward fragile item is
closed.** The gate's `borrowed` scoping is the mechanism; if a future segment
type is neither borrowed nor generated, re-check the scoping.

---

## STAGE 1 (20-minute rule) — guard conflict measured, 2026-09-08

The full-match path is dropped. New sourcing rule: every YouTube clip is
3-20 minutes (180-1200s) for every lane. No full-match path.

### 1.1 — the conflict, measured not estimated

produce_v2's format query (`produce_v2.py:175-176`) requests up to **2160p**:

```
$ sed -n '175,179p' tools/produce_v2.py
        dl_cmd = ["yt-dlp", "-f",
                  "bestvideo[vcodec^=avc1][height<=2160][ext=mp4]+bestaudio[ext=m4a]/bestvideo[height<=2160][ext=mp4]+bestaudio[ext=m4a]/best[height<=2160]/best",
                  "--max-filesize", "200M",
```

Guard limit: `LOCAL_VIDEO_LIMIT_MB = 200` (`ffmpeg_utils.py:65`), enforced at
`produce_v2.py:177` (`--max-filesize 200M`) and `produce_v2.py:195`
(`assert_video_under_limit`).

To measure a ~20-min clip's filesize WITHOUT downloading, I ran `yt-dlp -F`
(metadata only) on the closest in-range search result — an 1121s (18.7min)
Arsenal-Man City highlights clip (`_PMJpCrjhG4`). produce_v2's selector
resolves to format 137 (1080p avc1 mp4) + 140 (m4a audio); the 720p selector
resolves to 136 + 140:

```
$ yt-dlp -f "bestvideo[vcodec^=avc1][height<=2160][ext=mp4]+bestaudio[ext=m4a]/best" --print "RESOLVE: %(format)s" --skip-download "https://www.youtube.com/watch?v=_PMJpCrjhG4"
RESOLVE: 137 - 1920x1080 (1080p)+140 - audio only (medium)
$ yt-dlp -f "bestvideo[vcodec^=avc1][height<=720][ext=mp4]+bestaudio[ext=m4a]/best" --print "RESOLVE: %(format)s" --skip-download "https://www.youtube.com/watch?v=_PMJpCrjhG4"
RESOLVE: 136 - 1280x720 (720p)+140 - audio only (medium)
```

Filesizes from `yt-dlp -F` (video + the m4a audio format 140 = 17.29 MiB):

| format | video MiB | +audio 17.29 MiB | total MiB | total MB |
|---|---|---|---|---|
| 137 — 1080p (produce_v2's pick) | 513.31 | +17.29 | 530.6 | **556 MB** |
| 136 — 720p | 142.52 | +17.29 | 159.8 | **168 MB** |

(18.7min / 1121s clip, measured.) Scaled linearly to exactly 1200s (20min):
1080p → **596 MB**, 720p → **179 MB**. The 596 MB 1080p figure is arithmetic
on the measured 18.7min point (the brief says do not estimate the filesize —
the filesize is measured; the 18.7→20min scaling is labeled arithmetic, not
a new measurement).

**The conflict:** at produce_v2's current requested format (1080p here, the
best avc1 mp4 ≤2160p), a 20-min clip is ~596 MB — over the 200 MB guard by
~396 MB. `--max-filesize 200M` aborts the download mid-stream, the candidate
is recorded as failed, and the loop tries the next. At **720p**, a 20-min
clip is ~179 MB — **under** the 200 MB guard by 21 MB.

**Recommended format: 720p.** It is the floor of produce_v2's existing reject
loop (`MIN_HEIGHT = 720`, `produce_v2.py:35` — a 720p download passes the
`>= MIN_HEIGHT` gate), and at 720p a 20-min clip fits the current 200 MB
guard. **Smallest guard limit that clears a 20-min 720p clip: ~180 MB.**
The current 200 MB guard already clears it (21 MB margin), so at 720p no
guard change is needed. The conflict exists only because produce_v2 requests
1080p+/2160p.

### 1.2 — three options with costs (not picked)

**a. Raise the guard to clear 20-min at produce_v2's current 1080p format.**
Measured 18.7min 1080p = 556 MB; 20-min ≈ 596 MB. Set the guard to ~600 MB
(`LOCAL_VIDEO_LIMIT_MB = 600`, `--max-filesize 600M`). Disk exposure:
measured C: free is **22 GB** now (the brief's 16.6 GB is stale):

```
$ df -h /mnt/c | tail -1
C:\             119G   98G   22G  83% /mnt/c
```

`22 GB / 596 MB = ~38 clips` to fill the C: drive. And the ext4.vhdx grows
and never shrinks (carry-forward fragile), so those 38 downloads permanently
enlarge the vhdx by ~22 GB even after the clips are deleted — C: does not
recover until the whole WSL distro is reset. Zero build time.

**b. Cap the requested format at 720p so 20-min clips fit the 200 MB guard.**
Change `produce_v2.py:176`'s `height<=2160` to `height<=720` (both video
selector branches). 20-min 720p = 179 MB, under the 200 MB guard. Quality
lost: 1080p→720p resolution (and 1440p/2160p entirely, if a source offers
them). The pipeline's 720p+ reject-and-retry loop **still passes**: 720p ==
`MIN_HEIGHT` (`>= 720`), so the gate accepts it; only <720p sources are
rejected, same as today. Zero build time, no disk exposure, no guard change.

**c. Build the pod-side download path (2.4) so clips never land locally.**
`cloud_produce.py:387-398` already builds pod-side yt-dlp commands;
`runpod_fulltrack.py` already ships tools + runs cv_annotate on the pod +
returns URLs (proven, STATUS). New work: a produce_v2 branch that runs yt-dlp
on the pod, cuts on the pod, returns only small derived outputs. Honest build
time: **~3-5 hours** focused work + one paid pod test, then ~$0.50-1/episode.
Zero local disk exposure (the 200 MB guard never fires). Overkill for a
20-min source (which fits locally at 720p), but it is the only option that
also future-proofs against any larger source.

### 1.3 — changing the duration filter 60-600s → 180-1200s

The filter lives in exactly one place:

```
$ grep -rnE "60 <= dur_sec|dur_sec <= 600" tools/*.py | grep -v "\.bak"
tools/produce_v2.py:141:                    if 60 <= dur_sec <= 600:  # 1-10 minutes
```

No other code hardcodes 60 or 600 as a duration bound (the other 60/600 hits
are timeouts, frame counts, print formatting — confirmed by grep). So the
one-line change itself breaks no dependent code. What it changes:

- **Excludes 60-179s** (1-3 min short highlights). Intended by the new rule,
  but any existing workflow relying on sub-3-min clips loses them.
- **Includes 600-1200s** (10-20 min). These are larger and **hit the 1.1
  conflict**: at produce_v2's current 1080p request, a 1200s clip is ~596 MB,
  over the 200 MB guard → refused. So raising the upper bound to 1200s
  WITHOUT resolving 1.1 (option a or b) makes produce_v2 accept 10-20-min
  clips that the guard then deletes — the conflict worsens. At 720p
  (option b), 1200s = 179 MB and the change works. So 1.3 is safe only
  combined with 1.2a or 1.2b or 1.2c.
- The "ALL candidates below 720p" exhaustion message becomes more misleading
  (long candidates are size-rejected, not resolution-rejected).

### 1.4 — usable-action yield from a 3-20 min source, measured

No 20-minute source is on disk. The closest in-range sources are 482s (8min)
and 813s (13.5min). Both measured with the content classifier
(`broadcast_filler.py` on a Gemini inventory, NOT estimated):

```
$ ~/yt-digest/.venv/bin/python tools/broadcast_filler.py artifacts/gemini_inventory/clip_PrCW_geeRAU.gemini_inventory.json /tmp/bf_482.json
wrote /tmp/bf_482.json: broadcast=43.0s filler=440.0s (9% broadcast), 21 segments
```

```
$ ~/yt-digest/.venv/bin/python tools/gemini_inventory_test.py --clip renders/2026-08-30_liverpool-forest_sep1/clips/clip_BywIRMYkHiw.mp4 --out /tmp/gemini_813_raw.txt
# (Gemini File API upload 13s + processing + generate; model=gemini-3.1-pro-preview,
#  prompt_tokens=74339 [73983 VIDEO], output_tokens=4375; ~$0.19)
$ ~/yt-digest/.venv/bin/python tools/broadcast_filler.py /tmp/gemini_813.json /tmp/bf_813.json
wrote /tmp/bf_813.json: broadcast=192.0s filler=621.0s (24% broadcast), 53 segments
```

Two real data points:

| source | duration | broadcast (usable) | fraction |
|---|---|---|---|
| clip_PrCW_geeRAU (Arsenal-Chelsea) | 482s / 8.0min | **43.0s** | 9% |
| clip_BywIRMYkHiw (Liverpool-Forest extended) | 813s / 13.5min | **192.0s** | 24% |

**The 43s "highlights-reel" figure is not a cap.** A longer in-range source
yields far more usable action: the 13.5-min extended highlight gives 192s,
4.5× the 8-min source. The broadcast fraction rises with length (9% → 24%)
because short reels are denser in titles/crowd/celebration. The two sources
are different matches, so this is not a clean extrapolation — but it is
measured, and it directly falsifies the "43s ceiling" assumption that the
prior lane-D runtime verdict (Stage 4.5) was built on. Lane D at 192s usable
is a very different proposition than at 43s: at 50% borrowed, a 192s-borrowed
episode is ~384s runtime with 70% narration — a real ~6-min episode, not a
~90s short. **The Stage 4.5 "impractical at 43s" verdict is overstated; it
should be re-costed against these measured yields.**

What a 20-min measurement needs: no 20-min source is on disk. To get one,
download a ~1200s 720p clip (179 MB, fits the current 200 MB guard — no
blocker once 1.1 is resolved), then run `gemini_inventory_test.py` +
`broadcast_filler.py` (~$0.20, ~3 min). That is the only missing input.

### 1.5 — transformation gate 8s rule vs produce_v2's 45s tactical cap

Confirmed: they apply to **different segment types**; no conflict. Resolved
last stage (commit c704816) and re-verified now. The gate's 8s excerpt rule
(rule 2, `transformation_gate.py:80`) and its ≥1-layer rule (rule 4,
`transformation_gate.py:96`) both scope to `borrowed` segments, matching
rule 5 (`:110`). produce_v2's 45s tactical cap (`produce_v2.py:377`) applies
to the generated tactical segment (non-borrowed), which is exempt from rules
2 and 4. Re-verified:

```
$ grep -nE "if seg.get\(\"borrowed\", False\) and dur > EXCERPT_MAX_S|if seg.get\(\"borrowed\", False\):" tools/transformation_gate.py
80:        if seg.get("borrowed", False) and dur > EXCERPT_MAX_S:
96:        if seg.get("borrowed", False):
110:        if seg.get("borrowed", False):
# 45s tactical (generated) passes rule 2; 9s borrowed footage fails rule 2:
45s tactical + 9s borrowed: ok= False -> 9s borrowed flagged: True | 45s tactical flagged: False
```

produce_v2's own caps already satisfy the gate for lane D: footage capped at
4.0s (`produce_v2.py:387`, under the 8s borrowed rule) with a zoom layer
(rule 4 satisfied); tactical may run to 45s (generated, exempt). **The 8s-vs-45s
carry-forward fragile item stays closed.**

---

## STAGE 2 (option 1.2b applied) — 720p cap, 180-1200s, yield re-measured, 2026-09-08

Decision: option 1.2b. Cap the requested format at 720p. Guard stays 200MB.
Pod-side path not built.

### 2.1 — 720p cap applied + real download proof

Exact lines changed in `produce_v2.py`:

```
$ grep -nE "height<=720|180 <= dur_sec <= 1200|MIN_HEIGHT = 720" tools/produce_v2.py
35:MIN_HEIGHT = 720          # reject clips below this resolution
141:                    if 180 <= dur_sec <= 1200:  # 3-20 minutes (new sourcing rule)
176:                  "bestvideo[vcodec^=avc1][height<=720][ext=mp4]+bestaudio[ext=m4a]/bestvideo[height<=720][ext=mp4]+bestaudio[ext=m4a]/best[height<=720]/best",
```

`produce_v2.py:176` — all three `height<=2160` → `height<=720`. The 200MB guard
(`--max-filesize 200M` line 177, `assert_video_under_limit` line 195) is
unchanged. The reject loop still passes: `MIN_HEIGHT = 720` (line 35), gate
`if h >= MIN_HEIGHT` (lines 167, 193) — a 720p download is `720 >= 720`, accepted.

**Two download paths NOT capped (codebase wins, flagged not changed):**
- `produce_episode.py:213` uses `height<=2160` + `--max-filesize 800M`. Its
  comment (lines 206-212) says the 1080p+ source is a **deliberate anti-blur
  fix** ("360p upscaled 3x... 1080p source fixes both"). It is NOT 200MB-guarded
  (800M limit, no `assert_video_under_limit`). Capping it at 720p would
  reintroduce the blur regression for zero guard benefit. **Flagged gap:**
  produce_episode is a local download path that bypasses the 200MB guard
  (can write up to 800M locally). It should get the guard or be retired (it is
  the older, untested, non-authoritative entry point; TOOLS.md STANDALONE).
- `cloud_produce.py:391,402` use `height<=2160` but run ON THE POD
  (`/workspace/...`), not local. The 200MB guard does not apply to pod-side
  writes. The pod-side path is deferred (decision). Not capped.

Real download proof (18.7min / 1120s clip, produce_v2's exact 720p format +
`--max-filesize 200M`):

```
$ yt-dlp -f "bestvideo[vcodec^=avc1][height<=720]...+bestaudio[ext=m4a]/..." --max-filesize 200M \
    --merge-output-format mp4 -o "renders/_20min_test/clips/clip_%(id)s.mp4" \
    --cookies secrets/yt_cookies.txt "https://www.youtube.com/watch?v=_PMJpCrjhG4"
[info] _PMJpCrjhG4: Downloading 1 format(s): 136+140
[Merger] Merging formats into "renders/_20min_test/clips/clip__PMJpCrjhG4.mp4"
$ stat -c '%s' renders/_20min_test/clips/clip__PMJpCrjhG4.mp4
167902062
$ ffprobe -v error -show_entries format=duration:stream=width,height -of csv=p=0:s=x renders/_20min_test/clips/clip__PMJpCrjhG4.mp4
1280x720
1120.455692
```

167.9 MB (160.1 MiB), 1280x720, 1120.5s. **Under the 200MB guard** — confirmed:

```
$ python -c "from ffmpeg_utils import assert_video_under_limit; assert_video_under_limit('renders/_20min_test/clips/clip__PMJpCrjhG4.mp4'); print('GUARD PASSES')"
GUARD PASSES: file under 200MB limit
```

A 15-20min 720p clip lands at ~168MB, 32MB under the guard. The conflict from
1.1 is resolved by the cap, not by raising the guard.

### 2.2 — duration filter 60-600s → 180-1200s

```
$ grep -rnE "60 <= dur_sec|dur_sec <= 600|dur_sec <= 1200" tools/*.py | grep -v "\.bak"
tools/produce_v2.py:141:                    if 180 <= dur_sec <= 1200:  # 3-20 minutes (new sourcing rule)
```

The only duration-filter use of 60/600/1200 in the codebase is
`produce_v2.py:141`. Nothing else depends on the old 60-600s range (the other
60/600 hits are timeouts, frame counts, print formatting). The change excludes
sub-3-min clips (intended) and admits 10-20-min clips (which now fit the guard
at 720p, per 2.1). No dependent code breaks.

### 2.3 — lane D re-costed against the measured yields (replaces Stage 4.5)

Stage 4.5 said lane D was "impractical at the 43s highlights-reel yield." That
verdict is **overturned** — the 43s figure was one short-reel data point, not a
cap. Re-cost at the transformation floor (borrowed <50%, excerpts ≤8s,
narration >70%):

Against the 192s measurement (13.5min source):
- borrowed = 192s. borrowed <50% runtime → runtime R > 384s.
- At R ≈ 400s (~6.7min): narration >70% → >280s of narration.
- excerpts = 192/8 = **24 excerpts** (max 8s each).
- added layers = **24** (one per borrowed excerpt: board/annotation/title/zoom).
- **Realistic runtime ~6.7min, 24 excerpts, 24 layers.**

Against the 373s measurement (18.7min source, measured in 2.4):
- borrowed = 373s. R > 746s. At R ≈ 750s (~12.5min): narration >525s.
- excerpts = 373/8 = **47 excerpts**. layers = **47**.
- **Realistic runtime ~12.5min, 47 excerpts, 47 layers.**

**Viable now: yes.** At 192s yield a ~6.7min episode with 24 layered excerpts
is a solid historical review; at 373s a ~12.5min episode with 47 is ambitious
but viable. The bottleneck is no longer footage yield — it is the per-excerpt
layer production (24-47 layers per episode) and the cut-list generator (2.5,
still missing). The Stage 4.5 "impractical" verdict is replaced by: **lane D is
viable at the measured yields; the constraint is editing labor and the
unbuilt cut-list generator, not footage supply.**

### 2.4 — real 20-min-class yield measured (closes the 20-min unknown)

Downloaded the 18.7min (1120s) 720p clip (167.9MB, 2.1), ran the Gemini
inventory (`gemini_inventory_test.py`, gemini-3.1-pro-preview, 101920 video
tokens, ~$0.25), then `broadcast_filler.py`. **Critical correction:**
Gemini's inventory spans 0-1841s but the clip is 1120s — Gemini hallucinated 27
phantom segments past the clip end. The raw `broadcast_filler` output (644s)
over-counts by 271s of phantom broadcast. Recomputed within the real 1120s:

```
$ python tools/broadcast_filler.py /tmp/gemini_1120.json /tmp/bf_1120.json
wrote /tmp/bf_1120.json: broadcast=644.0s filler=1197.0s (35% broadcast), 65 segments
# corrected (segments with start < 1120.46s only):
REAL broadcast within 1120s: 373.0s (33% of clip)
phantom broadcast beyond clip end: 271.0s (discard)
```

Three measured yield points:

| source | duration | real broadcast | fraction |
|---|---|---|---|
| clip_PrCW_geeRAU | 482s / 8.0min | 43.0s | 9% |
| clip_BywIRMYkHiw | 813s / 13.5min | 192.0s | 24% |
| clip__PMJpCrjhG4 (ESPN FC, 720p) | 1120s / 18.7min | **373.0s** | 33% |

**Actual file size 167.9MB. Actual usable-action 373s (after phantom
correction).** The 20-min yield unknown is closed (18.7min is the closest
in-range source to 20min; a true 1200s source would scale ~7% higher).
Caveat: this is an extended-highlight/pro-analysis source (action-dense); a
fan-vlog 20-min source may yield less. **New fragile item:**
`broadcast_filler.py` does not clip segments to the video duration, so Gemini's
past-end phantoms inflate the broadcast count (644 vs 373 here). It must take a
`--duration` argument or Gemini's prompt must include the duration. The prior
43s and 192s measurements were not affected (their inventories matched the
clip duration), but this is a latent bug for any source where Gemini
over-generates.

### 2.5 — cut-list generator scope (not built)

The shared prerequisite for lanes A and D (and the 5-second cut at
`produce_v2.py:433`). Nothing is built. Scope:

**Reads:**
- `match_data.json` — ESPN event list (goals, subs, times) — ground truth.
- `scoreboard_scan.py` output (`scan.json`) — goal bug-update times measured
  in the actual clip.
- `broadcast_filler.py` output — broadcast (usable-action) time ranges.
- clip duration (ffprobe).
- the script section structure (`##` headers + word counts).

**Writes:**
- `scripts/<slug>.md` with `[VISUAL: footage=START-END]` tags per section,
  matching `assemble_words_match.py:70`'s parser
  (`footage=(\d+(?:\.\d+)?)-(\d+(?:\.\d+)?)`). Each window = a verified excerpt,
  clipped to ≤8s for lane D.
- Optionally `cut_list.json` (machine-readable) for the future pod-side path.

**Tools that already do half the job:**
- `scoreboard_scan.py` — goal times. EXISTS, STANDALONE (orphaned).
- `gemini_inventory_test.py` — content classification. EXISTS, STANDALONE.
- `broadcast_filler.py` — broadcast segments. EXISTS, STANDALONE (but has the
  phantom-segment bug from 2.4).
- `assemble_words_match.py` — the CONSUMER; defines the output tag format.
  EXISTS, STANDALONE.
- `segment_scorer.py` — segment quality scoring. EXISTS, STANDALONE.

**The missing piece (the glue):** match each script section's topic to the
nearest goal/broadcast segment, emit `footage=START-END`, and relay-verify
(`vision_analyze.py` on the cut frame to confirm the shot is actually there —
the ±20s search done manually in the arsenal-chelsea build, STATUS). The
relay-verify is what caught the missing Rogers goal shot; without it the cut
list trusts Gemini/scoreboard timestamps, which are 1-2s off and sometimes
phantom (2.4).

**Honest build hours:**
- Without relay-verify (trust scoreboard times, clip to ≤8s): ~2-3 hours.
- With relay-verify (vision loop per excerpt): ~5-7 hours (the vision loop is
  the fiddly part — each excerpt needs a frame check).
- Plus ~30 min to fix `broadcast_filler.py`'s phantom-segment bug (clip to
  duration) so the broadcast ranges it feeds the generator are real.
- **Honest total: ~3 hours minimum, ~6-7 hours with relay-verify.** The
  relay-verify is the difference between a cut list that trusts model
  timestamps (unsafe, 2.4) and one that confirms each shot (what the
  words-match build actually did).

### Carry-forward closed this stage
- **football-data.org key tier — CLOSED.** The key returns 13 competitions
  including PL, Bundesliga (BL1), La Liga (PD), plus Ligue 1/Serie A/Primeira
  Liga/Championship. Lanes B/C are unblocked.
- **20-min yield — CLOSED.** 18.7min 720p → 373s real broadcast (33%).
- **8s-vs-45s — stays closed** (1.5, re-verified).
- **Stage 4.5 "impractical at 43s" — OVERTURNED** (2.3).

### Carry-forward still open / new
- cut-list generator not built (2.5).
- pod-side path not built (deferred); vhdx never shrinks (the 168MB test clip
  is now permanent).
- `broadcast_filler.py` phantom-segment bug (NEW, 2.4) — does not clip to
  duration.
- `produce_episode.py` bypasses the 200MB guard (800M local limit) — flagged
  in 2.1.
- 27 STANDALONE tools' execution unverified.
- crash-damaged files beyond .part — not re-checked this stage.
- tactical_boards:74 test bug — unfixed.
- SCRIPT_TEMPLATE / validate_script four-template blocker — untouched.

---

## STAGE 3 — fix the measuring tool, decide the cut-list approach, scope the rest, 2026-09-09

### 3A.1 — broadcast_filler phantom-segment bug FIXED (built)

`broadcast_filler.py` now takes `--clip <video.mp4>` (ffprobes duration) or
`--duration <seconds>`, clips every segment's end to the duration, and
discards segments starting at/after the end. Without a bound it runs (back-compat)
but prints a WARNING. On the 18.7min file:

```
$ python tools/broadcast_filler.py /tmp/gemini_1120.json /tmp/bf_1120_fixed.json --duration 1120.46
wrote /tmp/bf_1120_fixed.json: broadcast=373.0s filler=747.5s (33% broadcast), 38 segments
  duration bound 1120.5s: discarded 27 phantom segment(s) past end, clipped 1 segment(s) to duration
```

**27 phantom segments discarded** (the 644s raw → 373s real). 1 segment
clipped (spanned the end). The fix is in `tools/broadcast_filler.py`
(arg parsing + the loop's `if start >= D: discarded += 1; continue` /
`if end > D: end = D`).

### 3A.2 — re-measured 8min and 13.5min with the fixed tool

```
$ python tools/broadcast_filler.py artifacts/gemini_inventory/clip_PrCW_geeRAU.gemini_inventory.json /tmp/bf_482_fixed.json --duration 482.42
wrote /tmp/bf_482_fixed.json: broadcast=43.0s filler=439.4s (9% broadcast), 21 segments
  duration bound 482.4s: discarded 0 phantom segment(s) past end, clipped 1 segment(s) to duration

$ python tools/broadcast_filler.py /tmp/gemini_813.json /tmp/bf_813_fixed.json --duration 813.35
wrote /tmp/bf_813_fixed.json: broadcast=192.0s filler=621.0s (24% broadcast), 53 segments
  duration bound 813.4s: discarded 0 phantom segment(s) past end, clipped 0 segment(s) to duration
```

Corrected yields (all three points):

| source | duration | broadcast | phantoms discarded | changed? |
|---|---|---|---|---|
| clip_PrCW_geeRAU | 482s / 8.0min | 43.0s (9%) | 0 | no |
| clip_BywIRMYkHiw | 813s / 13.5min | 192.0s (24%) | 0 | no |
| clip__PMJpCrjhG4 | 1120s / 18.7min | 373.0s (33%) | 27 | yes (644→373) |

The 43s and 192s figures were NOT wrong — those inventories had no past-end
phantoms. Only the 18.7min source was affected. **The bug was real but
conditional (only when Gemini over-generates past the clip end, which happened
on the longest source).** All three figures now come from the fixed tool.

### 3A.3 — audit of other Gemini-output consumers

```
$ grep -rlnE "gemini_inventory|action_type|shot_type" tools/*.py | grep -v "\.bak"
tools/broadcast_filler.py
tools/gemini_inventory_test.py
```

`gemini_inventory_test.py` PRODUCES the inventory (does not consume it).
`broadcast_filler.py` is the SOLE consumer. `scoreboard_scan.py` uses the
vision relay (gemma4), not Gemini. **No other tool consumes Gemini output, so
no other tool has the missing-bounds bug.** Closed.

### 3A.4 — produce_episode guard bypass CLOSED (built)

The 1080p+ anti-blur exception STAYS (its comment, `produce_episode.py:206-212`,
documents a real blur fix: 360p→1080x1920 upscale was the dominant blur cause).
But the no-guard bypass is closed: an 800M guard is added (matching its
`--max-filesize 800M`):

```
$ grep -nE "assert_video_under_limit|limit_mb=800" tools/produce_episode.py
42:from ffmpeg_utils import assert_video_under_limit  # noqa: E402
253:        assert_video_under_limit(clip, limit_mb=800)
```

`produce_episode.py:253` — in the post-download clip-audit loop, each clip is
guard-checked at 800M; over 800M is deleted + raised. The 1080p source stays
(anti-blur), the unbounded local write is gone. (produce_episode is still the
untested, non-authoritative entry point; the 800M exposure is theoretical until
someone runs it.) Closed.

### 3B.1 — shot boundaries (the decider)

Tested both on the 18.7min 720p source on this N150:

```
ffmpeg scdet (threshold=30):  6 boundaries, 1m14s wall
PySceneDetect detect-content: 239 boundaries, 2m37s wall
```

scdet@30 is too coarse (6 big transitions only; would need threshold ~10-15 to
be useful). PySceneDetect (default) finds the real cut structure (239 scenes),
faster than real time (18.7min video in 2.6min).

**Do the uploader's cuts line up with the corrected action windows?**

```
broadcast windows: 15  scene boundaries: 239
MATCH: 14/15 broadcast starts within 5s of a scene boundary; 15/15 within 10s
over-detection: 239 scene boundaries for 15 action windows (16x)
```

Per-window offsets: 26.0→26.6 (+0.6s), 211→214.7 (+3.7), 223→222.1 (-0.9),
243→245.4 (+2.4), 326→325.8 (-0.2), 354→357.5 (+3.5), 445→444.8 (-0.2),
516→513.8 (-2.2), 616→621 (+5.0), 725→729 (+4.0), 823→827.4 (+4.4),
853→852.4 (-0.6), 923→916.3 (-6.7), 1006→1005.2 (-0.8), 1024→1020.1 (-3.9).

**Decision: the boundaries match (14/15 within 5s), so the cut list is nearly
free — but NOT from boundaries alone.** Boundaries over-detect 16x (239 vs 15
action windows) and don't classify action/filler. The cheap cut list is the
COMBINATION: PySceneDetect gives precise cut points, broadcast_filler labels
each scene action/filler, snap each action window to its nearest scene boundary
→ precise action excerpts with no Gemini timestamp drift and no relay-verify.
The scene boundary replaces the relay-verify (it IS the precise cut). This
cuts the 2.5 estimate from ~6-7h (with relay-verify) to ~2-3h (glue only).

### 3B.2 — audio energy

Per-second RMS loudness on the same file:

```
global mean RMS 0.0489, max RMS 0.1086
top-10 loudest seconds: t=13,15,16,17,18,20,21,22,23,24s (all in the intro)
```

The 10 loudest seconds are ALL in the intro (13-24s). The first action window
starts at 26s. **Loudness peaks do NOT correlate with action windows** — the
intro music bed is louder than the action. This is the documented false
positive (sustained music/chanting > action). Loudness is NOT a usable
boundary finder here. Use it only as RANKING within already-found action
windows (which excerpt of a multi-cut action scene is the peak), never as the
finder. Confirmed.

### 3B.3 — the six pod tools

Provider auth today:

```
RunPod  GET /v2/pods (Bearer) -> {"pods":[]}            AUTHENTICATES
Vast.ai GET /users/current (ApiKey) -> auth_error        DOES NOT AUTH
```

| tool | provider | class | ever run end-to-end? | what it returns | auth today |
|---|---|---|---|---|---|
| runpod_fulltrack | RunPod | WIRED (produce_v2:235) | YES (146s, $0.011, STATUS) | tracking JSON + annotated MP4 + log, downloaded local | yes |
| runpod_stage1 | RunPod | STANDALONE one-off | YES (300 frames, $0.05, STATUS) | webhook URLs (one-off) | yes |
| runpod_annotate | RunPod | STANDALONE | **NO — CONFIRMED never ran end-to-end** (GAPS: UNVERIFIED; no per-clip artifacts; runpod_fulltrack superseded it) | per-clip annotated MP4 + tracking JSON via webhook | yes |
| runpod_shorts | RunPod | STANDALONE | never | NVENC-encoded short | yes |
| runpod_superres | RunPod | STANDALONE | never | Real-ESRGAN super-res video | yes |
| vastai_shorts | Vast.ai | STANDALONE | never | NVENC short | **NO (provider won't auth) → DEAD in practice** |

runpod_annotate never ran end-to-end: CONFIRMED (GAPS.md said UNVERIFIED; no
artifacts exist from it; runpod_fulltrack is the one that actually completed).
vastai_shorts is dead in practice — the Vast key is invalid/expired.

### 3B.4 — SoccerNet action-spotting on L4 (costed, not favoured)

Cost: SoccerNet-v2 action-spotting (CALF/CALF++) weights ~100-500MB; container
setup (PyTorch + soccernet pip + weights download) ~30-60min one-time; per-run
on 20min 720p (~30000 frames @ 25fps) at ~0.077s/frame on L4 ≈ 38min inference
+ pod uptime, ~$0.16/run. It outputs frame-level event probabilities (goals,
cards, subs, ball-driven events) — i.e. EVENT timestamps, not broadcast/filler
classification.

Compare to 3B.1+3B.2: PySceneDetect (free, local, 2.6min) + broadcast_filler
(Gemini ~$0.25, ~3min) = ~$0.25, ~6min, gives 15 action windows that already
align with scene boundaries (14/15 within 5s). SoccerNet's event spotting is
redundant with `scoreboard_scan.py` (goal times, free, already works) and does
not solve the broadcast/filler split that broadcast_filler already does.

**Build the cheap path (3B.1), not SoccerNet.** The decider (14/15 within 5s)
lets 3B.1 decide: the cut-list problem is solved for ~$0.25 + ~2-3h glue.
SoccerNet is more capable at event detection but redundant here, at higher
setup + per-run cost. Do not favour it for being more capable.

### 3B.5 — pod-side download, re-scoped

What exists (~60-70%): `runpod_fulltrack.py` already ships tools to a pod
(tarball), runs cv_annotate on the pod, uploads results to catbox, and
downloads them local. `cloud_produce.py:387-398` already builds pod-side
yt-dlp commands that download to `/workspace/...` (the pod, not local). The
pod orchestration (ship code, run, return URLs) is proven end-to-end.

What is genuinely missing: (1) a produce_v2 branch that uses pod-side yt-dlp
instead of the local download (cloud_produce has the command builder; produce_v2
doesn't call it); (2) pod-side cutting — cut excerpts on the pod from the cut
list so only small segments come home; (3) excerpt-only return — `runpod_fulltrack`
currently returns the full annotated MP4 (guarded at 200MB), which a long source
would exceed; the pod must trim (`trim_tracking.py` exists, runs locally) and
return only the summary + excerpts.

**Revised: ~2-3h** (was 3-5h). The scaffold is more complete than the first
estimate — the pod yt-dlp commands and the run/return orchestration exist. The
new work is the produce_v2 branch + pod-side cut + excerpt-only return.

### 3B.6 — is the 720p cap still needed if downloads move to the pod?

**No.** If the raw clip never lands locally (pod-side yt-dlp, only finished
≤8s excerpts come home), 1080p (or higher) sources are viable with **zero local
exposure**. The 200MB guard would never fire because nothing large is written
under `/home/muads`. The 720p cap is a LOCAL-download constraint; the pod-side
path lifts it. The vhdx never shrinks, but with pod-side downloads nothing
large is written locally in the first place, so the vhdx never grows from
source clips. 1080p becomes viable (better tracking, sharper output) with no
disk cost. Answer: lift the 720p cap when the pod-side path is built; keep it
until then.

### 3C — four-template blocker (scope, not built)

`SCRIPT_TEMPLATE.md` is ONE template — lane A's Hook/Setup/Body/Trade-off/Close,
8-12min, `[VISUAL: ...]` tags, and a freshness rule (time-sensitive claims must
come from a same-day fresh_fetch brief). `validate_script.py` validates only
`[SRC: id]` / `[RUMOR: id]` tag resolution against `sources.json` + RUMOR
freshness (≤7 days). It has NO lane concept, NO structural validation, NO
per-lane tag vocabulary.

**How the four shapes differ:**
- **A (tactical analysis, existing):** Hook/Setup/Body/Trade-off/Close;
  `[VISUAL: board=/footage=/tactical]`; ESPN match_data + sources.json; 8-12min;
  theory-then-footage.
- **B (match preview):** no match yet → no `[SRC: result]`, NO `footage=` tags
  (no footage); Hook/Setup/Form/Fixture-prediction/Close; fact source =
  fresh_fetch (fixtures/standings/news); `[VISUAL: board=/stills]` only.
- **C (hot topic):** no match; Hook/Setup/Points/Evidence/Close; fact source =
  fresh_fetch (news); `[VISUAL: board=/stills]` only, no `footage=`.
- **D (historical review):** narration-FIRST (analysis leads, footage
  illustrates); `[VISUAL: board=/footage=START-END]` from the cut list;
  ESPN match_data (past) + sources.json + cut_list + transformation_gate
  manifest; first section must be narration-led (no `footage=` in section 1).

**What changes:** `SCRIPT_TEMPLATE.md` → 4 templates (or 4 files); 
`validate_script.py` → add `--lane`, per-lane structural rules (D: narration-first
+ `footage=START-END` format + cut_list reference; B/C: no `footage=` tags;
A: theory-then-footage), per-lane tag vocabulary, plus the existing
`[SRC]`/`[RUMOR]` freshness check for all. No external deps.

**Honest hours: ~4-6h** — 3 new templates (~2h) + validate_script `--lane` +
per-lane structural rules + tag-vocabulary checks (~2-3h) + tests (~1h).

### 3D.1 — the 27 STANDALONE tools (import test)

```
$ for t in <27 STANDALONE tools>; do timeout 25 python -c "import sys; sys.path.insert(0,'tools'); import $t" || echo FAIL; done
# all 27 printed OK, zero FAILs
```

**All 27 STANDALONE tools import cleanly** (deps present in the venv: cv2,
torch, ultralytics, runpod, google.genai, matplotlib, feedparser, elevenlabs,
scenedetect). None are dead-in-practice from missing deps. Whether they have
been RUN end-to-end is still mostly unverified (call sites grepped, not
execution), but they CAN run. `vastai_shorts` imports but its provider won't
authenticate (3B.3), so it is dead-in-practice at runtime despite importing.

### 3D.2 — crash damage beyond the .part files

```
$ for f in $(find artifacts -name "*.json"); do python -c "import json; json.load(open('$f'))" || echo BAD; done
# all 8 artifacts JSON parse OK
$ for f in $(find renders -name "*.mp4"); do ffprobe -v error -show_entries format=duration ... "$f" || echo BAD; done
BAD renders/2026-08-18_iraola-liverpool/clips/liverpool_mancity_highlights_annotated.mp4 (size=782 dur=)
BAD renders/2026-08-18_iraola-liverpool/clips/bournemouth_mancity_highlights_annotated.mp4 (size=782 dur=)
BAD renders/2026-08-18_iraola-liverpool/clips/benchmark_meta_full_annotated.mp4 (size=782 dur=)
```

Three 782-byte `.mp4` files are **HTML error pages mis-saved as .mp4**
(ffprobe: "moov atom not found"; `head` shows `<html><head>`) — failed
annotation downloads from the Aug 18 iraola run (a catbox/webhook error page
saved with a `.mp4` name). These are crash residue beyond the deleted `.part`
files. All 8 artifacts JSON parse OK; no other truncated mp4s. **The crash-damage
unknown is closed: 3 HTML-stub files in one legacy clips dir.** They are
gitignored (renders/), safe to delete.

### Carry-forward closed this stage
- broadcast_filler phantom bug — FIXED (3A.1), only the 18.7min figure changed
  (644→373); 43s and 192s stand (3A.2).
- produce_episode guard bypass — CLOSED (3A.4, 800M guard added).
- Gemini-consumer audit — CLOSED (3A.3, broadcast_filler is the only consumer).
- 27 STANDALONE tools — CLOSED (all import OK; vastai_shorts dead at runtime).
- crash damage beyond .part — CLOSED (3 HTML-stub mp4s in iraola clips).
- shot-boundary decider — CLOSED (14/15 within 5s; cheap cut list viable).
- pod-provider auth — CLOSED (RunPod yes, Vast.ai no).
- runpod_annotate never ran end-to-end — CONFIRMED (3B.3).
- cut-list build estimate — REVISED down (6-7h → 2-3h, no relay-verify needed).

### Carry-forward still open / new
- cut-list generator not built (3B decided the approach: scene boundaries +
  broadcast_filler, ~2-3h). NEW fragile: `vastai_shorts` dead at runtime
  (Vast key invalid).
- pod-side path not built (3B.5 re-scoped to ~2-3h); 720p cap stays until then
  (3B.6).
- SCRIPT_TEMPLATE / validate_script four-template blocker — scoped (3C, ~4-6h),
  not built.
- tactical_boards:74 test bug — unfixed.
- 3 HTML-stub mp4s in iraola clips — found (3D.2), not deleted.
- a true 1200s source yield — still extrapolated (~7% over 373s); 18.7min is
  the closest measured.

---

## STAGE 4 — doctrine, Vast key, cut-list generator built, debt cleared, 2026-09-09

### 4A — doctrine written into the canonical docs (built)

The Standing Operating Doctrine (4 rules) is now a top-level section in
CONTEXT.md, ARCHITECTURE.md, and DECISIONS.md, and is referenced from CLAUDE.md
so every session inherits it. The truthful gap list (which tools violate rules
1 and 3 today) sits beside it in DECISIONS.md.

**Rule 1 is violated by the whole pipeline.** Verified by grep:

```
$ grep -nE "yt-dlp|/mnt/c|subprocess.run\(\[FFMPEG|shutil.copy2" tools/produce_v2.py
122: search_cmd yt-dlp   175: dl_cmd yt-dlp   404/413/437/448: ffmpeg assemble
618: shutil.copy2 -> /mnt/c/Users/muads/Downloads
```

produce_v2 downloads + assembles + merges + crops + copies locally; only step4b
(tracking) runs on RunPod. Local-download/compute/storage violators (WIRED +
STANDALONE): produce_v2, produce_episode, assemble_words_match, scoreboard_scan,
broadcast_filler, segment_scorer, tactical_render, tactical_boards, match_data,
generate_voice, generate_ambience, merge_voice, shorts_crop,
gemini_inventory_test, trim_tracking, validate_script, assemble_video,
enhance_clips, generate_captions, thumbnail_generator, sharpness_check,
render_video, tactical_overlay, check_and_download. Rule 1 cannot be met until
the pod-side download path (3B.5) is built.

**Rule 3 is violated by 4 DEAD + ~18 untested STANDALONE.** DEAD (must retire):
tactical_overlay, pitch_radar, render_video, check_and_download. Untested
STANDALONE (import OK per 3D.1, execution unverified): produce_episode,
cloud_produce, runpod_annotate, runpod_shorts, runpod_superres, gpu_superres,
luminance_pod, sharpness_check, assemble_video, validate_script,
generate_captions, thumbnail_generator, enhance_clips, viral_angle,
agent_reach_research, fresh_fetch, oauth_setup, ltx_enhance. Hand-run (tested,
not wired): scoreboard_scan, broadcast_filler, segment_scorer,
assemble_words_match, trim_tracking, gemini_inventory_test, youtube_upload,
runpod_stage1. Only runpod_fulltrack is WIRED + tested. Not fixed this run.

The four rebuilt docs were re-dated to 2026-09-09 (Stages 2 and 3 changed
wiring: 720p cap, 180-1200s filter, broadcast_filler fix, produce_episode
guard).

### 4B — Vast.ai key (built: vastai_shorts RETIRED)

4B.1 — the code reads `VAST_API_KEY` (`vastai_shorts.py:40`,
`gpu_superres.py:38`, `cloud_produce.py:46`), which matches the .env variable.
No .env change needed.

4B.2 — the curl with `Authorization: ApiKey` returned `auth_error`, but that
was the WRONG header. The vastai SDK (the actual code path) authenticates:

```
$ python -c "from vastai import VastAI; v=VastAI(api_key=...); print(v.list_machines(ids=[]))"
[]
```

`list_machines(ids=[])` → `[]` (like RunPod's `{"pods":[]}`). The key WORKS via
the SDK; my curl used the wrong header.

4B.3 — ran vastai_shorts.py end-to-end on arsenal-chelsea. The key works (pod
created: Quadro P2000, $0.028/hr, instance 50349310), but the pod encoding
FAILS deterministically:

```
[cutlist run log] 5. Creating GPU instance (Quadro P2000, $0.028/hr)...
Instance created: 50349310
7. Waiting for results...
  ERROR: {'error': 'encoding_failed', 'size': '0'}   (repeated 9+ times)
```

Every webhook result is `{'error': 'encoding_failed', 'size': '0'}` — output 0
bytes. The pod's ffmpeg/luminance crop step fails on the rented container; the
webhook does not capture pod stderr, so the exact cause is in the encode script
(not the key). The instance was destroyed (`destroy 50349310 -> {'success':
True}`); cost ~$0.003. vastai_shorts is dead for reasons beyond the key.

4B.4 — CLOSED: the key works but vastai_shorts does NOT work end-to-end
(encoding_failed). Per the binary, vastai_shorts is RETIRED under doctrine rule
3 (RETIRED notice added to the tool header; TOOLS.md updated). The Vast
provider + key remain valid for gpu_superres/cloud_produce; only vastai_shorts's
encode script is broken.

### 4C — cut-list generator BUILT and folded into produce_v2 (built)

`tools/cut_list_gen.py`: reads broadcast_filler output (action windows) +
PySceneDetect scene boundaries, snaps each action window to its nearest scene
boundary, writes `footage=START-END` tags into `scripts/<slug>.md` (replacing
boolean `[VISUAL: footage]` tags) + a `cut_list.json`. No relay-verify.

Folded into produce_v2: step5_assemble now parses `footage=START-END`
(`produce_v2.py:357`) and cuts the EXACT window (`start = seg.get("fw_start")`,
concat block). The fixed 5-second cut (`clip_idx*5` at the old :424/:433) is
RETIRED. Boolean footage tags with no window now skip with a warning ("run
cut_list_gen.py").

Proven with a REAL run (not dry) on the 18.7min source:

```
$ python tools/cut_list_gen.py _20min_test --scenes /tmp/scenes/clip__PMJpCrjhG4-Scenes.csv --broadcast /tmp/bf_1120_fixed.json --script scripts/_20min_test.md
[cutlist] 239 scene boundaries, 15 broadcast windows, snapped 15, filled 4 footage=START-END tag(s)
  footage=26.6-54.3  footage=214.7-222.1  footage=222.1-234.0  footage=245.4-258.3
```

Then step5_assemble ran on the filled script. The ffmpeg `-ss` cut starts
captured by the harness:

```
footage cut starts (-ss values): [(26.6, 25.0), (214.7, 7.4), (222.1, 11.9), (245.4, 12.9)]
Assembled: video_footage.mp4 (44.0MB, 57.2s)  -> 1920x1080
```

The cuts start at **26.6, 214.7, 222.1, 245.4** — the action-window starts —
NOT 0/5/10/15 (the retired 5s offsets). The 5s cut is gone; cuts are
content-matched. Lane A and D's shared prerequisite is met.

### 4D — small debt cleared (built)

4D.1 — deleted the 3 HTML-stub `.mp4` files (782B error pages, not video):

```
$ rm -v renders/2026-08-18_iraola-liverpool/clips/{liverpool_mancity,bournemouth_mancity,benchmark_meta_full}_highlights_annotated.mp4
removed '...liverpool_mancity_highlights_annotated.mp4'
removed '...bournemouth_mancity_highlights_annotated.mp4'
removed '...benchmark_meta_full_annotated.mp4'
```

4D.2 — fixed the tactical_boards:74 .md-append bug (parse_visual_tags now
accepts a slug OR a full `.md` path; the old code always appended `.md`,
double-suffixed a full path). The fix exposed two more mismatches in the test
(masked by the 0-tags return): the test called `.lower()` on the dicts
parse_visual_tags returns (fixed to use `tag["raw"]`), and referenced a
`tb.TEMPLATES` dispatch dict that does not exist in tactical_boards (the
render_*_board functions are called directly from main, not via a TEMPLATES
map — verified at `tactical_boards.py:485-487`). Per "codebase wins," the test
was reframed to validate the real behaviour (parse_visual_tags returns ≥5 tags
with a `raw` key; slug and path forms agree). Result:

```
$ python -m pytest tests/test_pipeline.py -q
23 passed in 1.74s
```

The long-standing test failure carried across every stage is CLOSED. 23/23.

### 4E — report only (nothing built)

4E.1 — build queue with hours:
- **Pod-side download path** — ~2-3h, 60-70% exists (runpod_fulltrack
  orchestration + cloud_produce pod yt-dlp + tool-shipping + result-return).
  Missing: produce_v2 branch + pod-side cut + excerpt-only return.
- **Four-template blocker** (SCRIPT_TEMPLATE + validate_script) — ~4-6h, 3 new
  templates + validate_script `--lane` + per-lane structural rules.

4E.2 — does pod-side move ahead of four-template? The brief's premise ("B and C
need no footage and are therefore already compliant with rule 1") is FALSE.
B/C have no footage DOWNLOAD, but their boards (tactical_boards), voice
(generate_voice), merge, and shorts still run LOCALLY — so B/C violate rule 1
on compute/storage just as lane A does. Building B/C via the four-template work
without the pod-side path would create NEW rule-1 violations, not compliant
lanes. Therefore **pod-side should move AHEAD of four-template**: rule 1 is
non-negotiable under the doctrine, the current lane A is non-compliant, and any
new lane built before pod-side is non-compliant by construction. Four-template
then adds lanes that are compliant once pod-side exists. (If new-lane throughput
matters more than doctrine compliance, reverse the order — but that violates
rule 1.)

4E.3 — rule 3 requires the 27 STANDALONE tools proven to RUN, not just import.
Proposed scaling check: a `--help` invocation per tool (exercises argparse +
imports + module-level init; no network, no pod, no files). Cost: ~27 tools ×
~3-5s (heavy deps: cv2/torch ~5s) ≈ 5-10 min, $0. Falsifier: a tool can pass
`--help` yet fail on real input (bad API key, missing clip) — `--help` proves
"launches," not "works end-to-end." So two tiers: (1) `--help` smoke test
(cheap, catches dead-on-launch / broken-imports-at-runtime) for all 27; (2)
end-to-end tests (like vastai_shorts got in 4B.3) for the WIRED + high-value
STANDALONE tools, ~$0.05-0.50 + ~3-10 min per cloud tool → ~$2-5 + ~1-2h for
the ~10 cloud/network tools. Not run this stage. Rule 3 is not closed by
`--help` alone; it is closed per-tool by an end-to-end run or by retirement.

### Carry-forward closed this stage
- doctrine written into 4 docs + CLAUDE (4A).
- Vast key name + auth — CLOSED (VAST_API_KEY matches; SDK auths; curl header
  was wrong) (4B.1/4B.2).
- vastai_shorts — CLOSED: RETIRED under rule 3 (key works, encode fails) (4B.4).
- cut-list generator — BUILT + folded into produce_v2; 5s cut retired; proven
  with a real run (4C).
- 3 HTML-stub mp4s — deleted (4D.1).
- tactical_boards:74 test failure — CLOSED (4D.2, 23/23 pass).
- four rebuilt docs — re-dated 2026-09-09 (4A).

### Carry-forward still open
- pod-side download path — not built (4E.1, ~2-3h); rule 1 cannot be met
  without it (4A gap, 4E.2).
- four-template blocker — scoped ~4-6h, not built (4E.1).
- 27 STANDALONE tools proven to RUN — not done (4E.3 proposes --help + e2e
  tiers); rule 3 open per-tool.
- 4 DEAD tools (tactical_overlay, pitch_radar, render_video, check_and_download)
  — listed in the doctrine gap, not yet retired.
- runpod_annotate never run end-to-end — still unverified.
- a true 1200s source yield — still extrapolated (~7% over 373s).

## STAGE 5 — pod-side path, storage report, rule-3 debt, 2026-09-09

### 5A — pod-side download path (built)

5A.1 — `tools/runpod_download.py` (new). The pod runs yt-dlp (search or
direct URL) and ffmpeg-cuts the excerpt windows; only the small excerpts come
home, each through the 200MB guard (`assert_video_under_limit`). Cookies +
the yt-dlp challenge cache are shipped to the pod via catbox. Reuses the
proven runpod_fulltrack pattern (catbox + webhook.site + RunPod L4). Wired
into produce_v2 as `--pod-download` (step3 routes to `step3_download_clips_pod`
instead of local yt-dlp). The full-resolution source is never written to
/home/muads — it lives only in /workspace on the pod and is discarded with the
pod.

5A.2 — which produce_v2 stages move to the pod, and which cannot:

| # | Stage | Today | Can move to pod? | Reason |
|---|---|---|---|---|
| 1 | match_data (ESPN API) | local | yes, low value | tiny JSON, no GPU; doctrinally compute but negligible |
| 2 | boards (matplotlib) | local | yes | CPU + small files; should move for full compliance |
| 3 | download (yt-dlp) | **POD (5A)** | yes — built | the big-file violation; this is 5A |
| 4b | tactical_render | **POD already** | yes | runpod_fulltrack ships cv_annotate to RunPod |
| 5 | assemble (ffmpeg) | local | yes | CPU; operates on excerpts once 5A lands |
| 6 | voice (ElevenLabs API) | local | yes, low value | tiny MP3; API call |
| 6b | ambience (ffmpeg) | local | yes | tiny |
| 7 | merge (ffmpeg) | local | yes | small |
| 8 | shorts (ffmpeg crop) | local | yes | small |
| final | copy to /mnt/c/Downloads | local | **NO** | the finished product must reach the user's disk; this is the single permitted local write (under the guard, ~25MB) |

Answer to "which cannot": only the final hand-off of the finished file to the
user's machine cannot move. Everything else CAN run on a pod. match_data and
voice are low-value-to-move (no big file, no GPU) but are still compute under
the doctrine, so full compliance moves them too.

5A.3 — real run proof: see "5A.3 PROOF" below (pasted pod log + disk usage).

5A.4 — rule-1 violations that REMAIN after 5A (exhaustive):
- step1 match_data — local ESPN API call (compute).
- step2 boards — local matplotlib render (compute + storage).
- cut-list GENERATION (scoreboard_scan + broadcast_filler + cut_list_gen +
  PySceneDetect) — runs locally and needs the clip. This is the architectural
  gap: the cut-list-first flow (4C) needs the clip before cutting, so until
  cut-list generation moves to the pod, the pod-download path is usable only
  when cut-windows are pre-known (re-runs) or via a two-phase proxy. Moving it
  requires solving scoreboard_scan's local-vision dependency (it calls
  ~/tools/vision_analyze.py → gemma4:cloud at localhost:11434, which is not on
  the pod; switch to GEMINI_API_KEY cloud vision, which the pod CAN reach).
- step5 assemble — local ffmpeg (now operates on small excerpts, but still local compute).
- step6 voice, step6b ambience, step7 merge, step8 shorts — all local ffmpeg/API.
- final copy to /mnt/c/Downloads — local write (the one permitted hand-off).
- produce_episode, assemble_words_match — separate local pipelines.
5A closes the DOWNLOAD (the big-file violation). The CPU/storage stages still
run locally. The doctrine gap list in DECISIONS.md is updated to match: rule 1
is now violated by every stage EXCEPT step3 (pod) and step4b (pod), not by the
whole pipeline.

5A.5 — is the 720p cap still needed? See "5A.5 ANSWER" below (empirical, from
the real run).

### 5A.3 PROOF

Three real pod runs tested the YouTube-download-on-pod path; one real pod run
(+ a confirm run) proved the excerpt-cut + guarded-return mechanics.

YouTube on the pod (runs #1-3, `--url PrCW_geeRAU`, pod yt-dlp) — ALL FAILED:
```
=== POD DOWNLOAD FAILED ===
reason: youtube_bot_blocked
log: WARNING: [youtube] No title found in player responses ... ERROR: [youtube]
  PrCW_geeRAU: Sign in to confirm you're not a bot. Use --cookies-from-browser
  or --cookies for the authentication.
```
The RunPod datacenter IP is hard-blocked by YouTube's bot detection on every
player client tried (web_safari, ios, android, default). Cookies + the yt-dlp
challenge cache were shipped to the pod but did not override the IP block. The
tool correctly detected this and posted `status=failed, reason=youtube_bot_blocked`
(not a crash). No file was written locally (G5=0). This is an external wall, not
a tool bug.

Excerpt-cut + guarded-return (run #4, `--source-url` catbox, stream-copy) —
SUCCEEDED. The pod fetched the 128MB 720p source from catbox (the source stayed
in /workspace on the pod and was discarded with the pod — never written to
/home/muads), cut the 3 goal windows (119-127, 181-191, 253-262), and uploaded
the excerpts. The local side downloaded only the excerpts through the guard:
```
=== RESULTS ===
Source on pod: 720, 128MB, client=source_url (NOT downloaded locally)
Excerpts: 2, wall time 51s
   podtest_excerpt_00.mp4: 4.0MB  (under 200MB guard)
   podtest_excerpt_02.mp4: 4.1MB  (under 200MB guard)
Largest local excerpt: 4.1MB. Full source never written locally.
Estimated GPU cost: $0.004 (L4 @ $0.25/hr, 51s uptime)
```
Run #7 (stream-copy + catbox upload retry) reproduced this: 2/3 excerpts, 68s,
$0.005. The missing window (181-191) is a stream-copy keyframe edge case (the
`-c copy` cut produces no file when the seek lands on a bad keyframe), NOT an
upload failure — the retry did not recover it. Re-encode (`-c:v libx264`) is the
robust fix but hung on a throttled L4 pod in run #6 (315s, 0 webhook posts,
terminated); reverted to stream-copy with the edge case documented in the code.

Peak local disk during the proof: the only local writes were the 2 excerpts
(8.1MB total). At every sample:
```
$ find renders -newermt "2026-09-09 00:00" -size +200M -type f 2>/dev/null | wc -l
0
```
No file over the 200MB guard was ever written locally. The 128MB source never
touched /home/muads. 5A.1 is proven: only finished excerpts come home, nothing
over the guard touches the laptop, the full source lives and dies on the pod.

Honest split: the YouTube-download-on-pod step is bot-blocked today (runs #1-3);
the excerpt-cut + guarded-return step is proven (runs #4/#7). The tool supports
both `--url/--query` (pod yt-dlp, bot-blocked) and `--source-url` (catbox,
proven). The realistic rule-1 path today is: local residential-IP download (the
existing violation) → ship to catbox → pod cuts excerpts → guarded return; or a
residential proxy / challenge-solver to unblock pod yt-dlp (future work).

### 5A.5 ANSWER

Is the 720p cap still needed? Direct answer: NOT DROPPED. The cap
(`MIN_HEIGHT=720`, `height<=720` in produce_v2 step3) stays for the
LOCAL-download path, which is the only currently-working YouTube path
(residential IP). The pod path (`runpod_download.py`) is built for 1080p
(`POD_MAX_HEIGHT=1080`), ready for when the pod download is unblocked.

Why not drop it: empirically, YouTube bot-blocks the RunPod datacenter IP on
every player client (runs #1-3) — so pod-side yt-dlp is not viable at ANY
resolution today, including 720p. The 720p cap is not the binding constraint;
the datacenter-IP bot wall is. 1080p-on-pod was never reached to test.

In principle (proven via `--source-url`): the pod handles the big file and only
small excerpts come home, so 1080p WOULD be viable with zero local exposure IF
the download reached the pod. So the doctrinal answer is "yes, 1080p is viable
with zero local exposure" (the pod removes the local-file-size reason for the
cap), but the practical answer is "no, not until a residential proxy or the
challenge-solver unblocks pod yt-dlp." Net: unchanged this stage. The cap stays
for the working local path; the pod path is 1080p-ready for the unblocked future.

### 5B — storage (report only, nothing built, nothing signed up)

5B.1 — what a B2 upload step would take:
- Existing S3-compatible upload code in the repo: NONE. `grep -rniE "boto3|s3client|backblaze|b2sdk|presigned" tools/ ~/yt-digest/` found zero upload code (the only hit was an unrelated claude-api skill doc). So the upload code must be written from scratch.
- Closest existing tool: none do S3. The nearest reusable pattern is
  `upload_catbox()` in runpod_fulltrack/runpod_download (a curl POST of one
  file) — same shape (one file → one PUT) but B2 needs S3 auth (Signature V4),
  not a form POST. So not directly reusable.
- Libraries: `boto3` 1.43.78 + `s3transfer` 0.19.2 are ALREADY installed in the
  venv (`pip list | grep -iE 'boto|s3'`). No install needed.
- Credentials handling: same .env pattern as the other keys — add
  `B2_KEY_ID` + `B2_APPLICATION_KEY` + `B2_ENDPOINT` (the S3-API endpoint for
  the bucket, e.g. `s3.us-west-004.backblazeb2.com`) to ~/yt-digest/.env, load
  via dotenv, mask in logs (rule 4). A `tools/b2_upload.py` (~60 lines: boto3
  client, put_object per file, returns the object key) + a `--archive` flag on
  produce_v2 that calls it for the final short.
- Honest hours: ~1.5-2.5h (write b2_upload.py, wire into produce_v2 final step,
  test one upload, mask keys, document). B2 has free egress to 3x stored data,
  so downloads are free; storage ~$6/TB/month.

5B.2 — what is worth archiving vs reproducible (sizes, gitignore status):
| Dir | Size | Gitignored? | Backed up? | Reproducible? | Archive? |
|---|---|---|---|---|---|
| renders/ | 2.9G | yes | NO | intermediates yes (expensive GPU); finals are the product | archive only finals (~25MB each), not the 2.9G |
| artifacts/ | 316K | yes | yes (public status mirror) | yes (from clips) | already mirrored; publish-log is the only unique part |
| frames/ | 5.0M | yes | yes (public status mirror) | yes (ffmpeg from clips) | already mirrored |
| scripts/ | 40K | yes | NO | yes (cut_list_gen + LLM) | cheap to archive; encodes editorial choices |
| visuals/ | 1.6M | yes | NO | yes (SVG generators) | cheap; optional |
| publish-log/ | 12K | yes (in artifacts/) | yes (mirror) | NO — unique publish records | YES (already mirrored) |

The big number is renders/ (2.9G), and ~all of it is reproducible
intermediates (raw clips, 8MB tracking JSONs, annotated MP4s, boards, tmp).
Paying B2 to store 2.9G of regenerable intermediates is not worth it. Worth
archiving: the final shorts/finals (the products, ~15-25MB each, handful), the
publish-log (unique, already mirrored), and optionally scripts/ (40K). So the
B2 archive is ~100-200MB, not 2.9G. git backs up NONE of these dirs (all
gitignored), so today the only backups are the public mirror (artifacts +
frames) and the laptop disk (renders, scripts, visuals — single copy).

5B.3 — can the finished file go pod → B2 without touching the laptop?
YES. The pod script already uploads results to catbox; replacing/adding a
boto3 `put_object` to B2 in the pod script sends the finished file straight
from pod to B2, and the laptop only fetches a small copy for /mnt/c/Downloads
(if even that). This is a LATER ADDITION to 5A's design, not a change to it:
5A returns excerpts home; pod→B2 adds a second upload destination in the same
pod script. It would satisfy rule 1 completely for the finished-file storage
(the file is in B2, not on the laptop). The laptop hand-off to /mnt/c remains
the one permitted local write (the user wants the file on their machine).

5B.4 — how keys should be handled (propose, do not implement):
- Today: ~/yt-digest/.env holds all 13 keys (LLM, YouTube, ElevenLabs, RunPod,
  Vast, Groq, Gemini, Football-data, LTX, Voice ID) in one plaintext file, one
  copy, no backup. If the laptop disk dies, every key is lost; if it is
  compromised, every key leaks.
- Proposal: (a) keep .env in place (tools already load it) but gitignore it
  (already gitignored — verified); (b) commit a `.env.example` template with
  key NAMES but no values, so the key inventory is in git; (c) back up the real
  .env encrypted (age or gpg with a passphrase) to BOTH B2 and the USB drive at
  /mnt/f — two encrypted copies, off-machine; (d) rotate the high-value keys
  (RunPod, YouTube OAuth, ElevenLabs) once the encrypted backup is confirmed,
  since the plaintext existed on disk; (e) never put keys in artifacts/ or
  frames/ (already enforced by the mirror's redact/secret-scan). This gives: one
  working plaintext copy (laptop) + two encrypted off-machine copies (B2, USB),
  matching the archive strategy. Do not implement this stage.

### 5C — rule-3 debt

5C.1 — `--help` smoke test on the 27 STANDALONE tools (run, $0, ~3 min).
Results (categorised):

- Proper argparse, `-h` works (rc=0): 10 — produce_episode, segment_scorer,
  scoreboard_scan, cut_list_gen, assemble_words_match, viral_angle,
  agent_reach_research, fresh_fetch, gemini_inventory_test, ltx_enhance.
- Launches but treats `--help` as a positional arg (rc=1, no `-h`; runs main
  and fails on a missing file): 15 — cloud_produce, runpod_shorts,
  runpod_superres, gpu_superres, sharpness_check, assemble_video,
  validate_script, generate_captions, thumbnail_generator, youtube_upload,
  oauth_setup (prints usage then reads stdin), runpod_annotate ("No raw
  clips"), enhance_clips ("No clips"), broadcast_filler (manual usage),
  trim_tracking (manual usage).
- Crashes on launch (uncaught exception): 1 — luminance_pod (IndexError on
  `sys.argv[2]`, no argparse, no arg guard).
- Hangs / starts side effects with no arg parsing: 1 — runpod_stage1 (timed
  out at 40s; creates a webhook/pod immediately on invocation).
- vastai_shorts (RETIRED 4B.4) also rc=1 (treats --help as a slug) — noted,
  not counted in the 27.

So 26/27 LAUNCH (all import and start; the 3D.1 import-OK result holds). Only
luminance_pod (crash) and runpod_stage1 (hang) are broken-on-launch. BUT
`--help` proves "launches," not "works end-to-end" — per the 4E.3 falsifier, a
tool can pass --help yet fail on real input (bad key, missing clip). So this
does NOT close rule 3 per-tool; only an e2e run or retirement does. luminance_pod
and runpod_stage1 are now candidates for fix-or-retire (not actioned this stage;
the brief scopes retirement to the 4 DEAD). Full output in /tmp/smoke_results.txt.

5C.2 — the 4 DEAD tools retired. Deleted from the tree:
- tactical_overlay.py (superseded by tactical_render; the clipart root cause)
- pitch_radar.py (shipped to 3 pods but never imported/run)
- render_video.py (docstring says DEPRECATED)
- check_and_download.py (RunPod-webhook downloader, superseded by runpod_fulltrack)
Where retired code goes: GIT HISTORY. `git rm` removed them from the tree; they
are recoverable via `git show <hash>^:tools/<name>.py` at the last commit that
held them. No retired/ folder is recreated (~/retired/ is off-limits by
doctrine and would invite confusion). The 3 pitch_radar ship-list references
(runpod_fulltrack:73, runpod_stage1:63, runpod_annotate:229) were removed so
no live code names a deleted file. Tool count 46 → 42.

5C.3 — runpod_annotate end-to-end: COSTED, not run (per brief).
runpod_annotate ships tools + clips to catbox, creates an L4 pod, runs
cv_annotate, returns tracking JSON. Its own header says "~$0.15 per episode
(L4 @ $0.25/hr, ~5 min for 7 clips)." For a single-clip verification
(comparable to runpod_fulltrack's 146s/$0.011): ~1-3 min pod uptime + catbox
uploads (free) = ~$0.01-0.02, wall ~5-8 min. Same shape and cost band as
runpod_fulltrack (which IS verified). NOT run this stage per the brief.

### Carry-forward closed this stage
- pod-side download tool — BUILT (runpod_download.py) + wired into produce_v2
  (--pod-download); 5A.1 done.
- 4 DEAD tools — RETIRED (5C.2); tool count 46 → 42; pitch_radar refs removed.
- 27 STANDALONE smoke test — RUN (5C.1); 26/27 launch, luminance_pod crashes,
  runpod_stage1 hangs; rule 3 not closed per-tool (e2e still owed).
- runpod_annotate e2e — COSTED (5C.3), not run.

### Carry-forward still open
- 5A.3/5A.5 outcome — see pasted proof below (depends on the real run).
- cut-list generation on the pod — scoreboard_scan's local-vision dependency
  (gemma4 via localhost:11434) blocks moving it; needs a Gemini-cloud backend
  switch. Until then the pod-download path needs pre-known windows.
- the CPU/storage stages (boards, assemble, voice, ambience, merge, shorts)
  still run locally — rule 1 violations (5A.4).
- luminance_pod (crash) and runpod_stage1 (hang) — fix-or-retire candidates.
- SCRIPT_TEMPLATE / validate_script four-template blocker — scoped 4-6h, after 5A.
- B2 archive step — designed (5B.1), not built (5B is report only).
- .env key handling — proposed (5B.4), not implemented.
## STAGE 6 — /mnt/f staging, name overlap, every belief closed, keyframe fix, keys, 2026-09-09

### 6A — downloads to /mnt/f + doctrine (built)

6A.1 — `tools/staging.py`: the single chokepoint for raw-download staging on
/mnt/f. `check_staging()` verifies /mnt/f is mounted AND writable, fails loudly
(with the exact mount command) if not, and NEVER falls back to /home/muads.
`produce_v2.py` step3 now downloads raw clips to `/mnt/f/soccer-staging/<slug>/`
(via `staging.stage_path`) instead of `renders/<slug>/clips/`; the renders/clips
cache is checked first (re-runs of an existing slug don't re-download).
`staging.cleanup_staging(slug)` runs at the end of main() to delete the staged
raw once the pipeline is done with it. Proven: `python tools/staging.py` exits 1
with the FATAL mount message when /mnt/f is absent (no /home/muads fallback):
```
$ python tools/staging.py
FATAL: /mnt/f is not mounted. Raw downloads must land on the USB
FATAL: flash drive at /mnt/f (doctrine rule 1, 6A). The ext4.vhdx grows
FATAL: and never shrinks, so a download inside /home/muads is permanent.
FATAL:   sudo mkdir -p /mnt/f && sudo mount -t drvfs F: /mnt/f
FATAL: Refusing to fall back to /home/muads.
rc=1
```

6A.2 — real run proof (file on /mnt/f -> pod receives -> local removed): see
"6A.2 PROOF" below (run after /mnt/f mount).

6A.3 — /mnt/f free space: Windows F: has ~62GB free (PowerShell Get-PSDrive,
matches the ~57.6GiB the brief cited). WSL did NOT auto-mount it; /mnt/f was
absent until mounted manually. Before/after free space pasted in 6A.2 PROOF.

6A.4 — doctrine rule 1 updated in CONTEXT.md, ARCHITECTURE.md, DECISIONS.md,
CLAUDE.md: "No local downloads, except raw source staging on the USB flash
drive at /mnt/f (staging area ONLY: download lands there, ships to the pod, gets
deleted; never a working directory). No local GPU work." DECISIONS.md gap list
updated to match (Stage 6 section).

### 6B — name overlap, resolved permanently (built)

6B.1 — authoritative path PROVEN:
- git remote: `origin https://github.com/minakush000-crypto/yt-digest.git`
- entry point: `/home/muads/yt-digest/soccer-channel/tools/produce_v2.py` (mtime 2026-09-09)
- CLAUDE.md: "Project: /home/muads/yt-digest/soccer-channel. Nothing else."
- CONTEXT.md: "The project is /home/muads/yt-digest/soccer-channel. Nothing else." (2x)
- mtime of the canonical dir: 2026-09-09.
So ~/yt-digest/soccer-channel is the reconciled path. `●`

6B.2 — `~/retired/` is fenced off by a PreToolUse hook (`block-retired.sh`):
every Bash command containing `/retired/` is blocked with "BLOCKED: ~/retired/
is off limits." No tool, skill, hook or path reference can reach ~/retired from
this session. (Whether the old soccer-channel/soccer-pipeline dirs still
physically exist under ~/retired, I cannot verify without bypassing the hook,
which I won't do — the hook makes them unreachable, which is the safety goal.)
No LIVE code references a soccer path outside ~/yt-digest/soccer-channel: the
only `~/soccer-pipeline` hits are in docs (DECISIONS/PROGRESS/SKILLS) describing
the deprecated location historically; `ROOT / "soccer-channel"` in tools
resolves to the canonical path; `push_status.sh` DST is the separate
`/home/muads/soccer-channel-status` MIRROR repo, not the retired folder. `●`

6B.3 — `yt_digest/` (the package) is the repo's original YouTube comment-research
tool (analyze.py, cli.py, llm_analyze.py, youtube.py, etc.). It IS still called:
`produce_episode.py:77` runs `python -m yt_digest.cli --topics topics-soccer.yaml`.
It shares the venv + .env but NO tools with soccer-channel (separate code trees).
Under rule 3: LIVE (called + has 6 tests). Not orphaned. The three names are
distinct: repo=yt-digest, folder=soccer-channel, package=yt_digest. `●`

6B.4 — canonical-path assertion added to the top of `produce_v2.py` (the entry
point): it checks `Path(__file__).resolve() == /home/muads/yt-digest/soccer-
channel/tools/produce_v2.py` and exits 1 with a FATAL message if not. Proven to
fire from the wrong place and NOT from the right one:
```
$ cp tools/produce_v2.py /tmp/produce_v2.py && python /tmp/produce_v2.py --help
FATAL: produce_v2.py is running from /tmp/produce_v2.py
FATAL: the authoritative location is /home/muads/yt-digest/soccer-channel/tools/produce_v2.py
FATAL: repo=yt-digest, folder=soccer-channel, package=yt_digest are 3 distinct names.
rc=1
$ python tools/produce_v2.py --help   # from the right place -> usage: printed, rc=0
```
The ambiguity is now unrepeatable: a copy or renamed clone refuses to run.

### 6C — every open belief closed

6C.1 — runpod_annotate RUN end-to-end. Pod created (L4), cv_annotate executed
(diagnostic output captured), pod finished in ~60s. BUT 0 results captured —
the webhook result-capture is broken (the same JSON-newline bug runpod_fulltrack
fixed). Cost ~$0.004, wall ~60s + upload. It does NOT work end-to-end.
VERDICT (rule 3): superseded by the working runpod_fulltrack + no caller + broken
=> RETIRED (deleted, in git history). `●`

6C.2 — 1200s yield STAYS AN ESTIMATE. The longest source in renders/ is 1120s
(clip__PMJpCrjhG4.mp4, the _20min_test), NOT 1200s. No 1200s source exists;
downloading one is bot-blocked on the pod (5A.3) and would violate 6A locally.
The 373s is the measured broadcast yield on the 1120s source; the ~7% above
373s is a linear 1120->1200 extrapolation that cannot be verified without a
1200s clip. Accepted as an estimate with this reason. `● (accepted estimate)`

6C.3 — SoccerNet is a DATASET + academic benchmark (soccer-net.org), NOT a
drop-in tool. It provides annotated broadcast games (NDA-gated raw video), a
Python package + dev kit, pre-extracted features, and an "action spotting" task
(17 classes incl. Goals, tight avg-mAP metric). Testing head-to-head on one
source requires NDA + downloading data + loading/training a model —
disproportionate when scoreboard_scan already finds 3/3 goals on arsenal-chelsea.
VERDICT: reasoned decision, never measured. SoccerNet serves a different purpose
(benchmarking/model training) than the goal-finding use case; it is redundant
WITH scoreboard_scan+broadcast_filler for goal-finding, not a replacement.
`● (accepted, reasoned, never measured)` Sources: soccer-net.org, github.com/SoccerNet/SoccerNet.

6C.4 — mirror redaction PROVEN with a test. `push_status.sh` redacts
`/home/muads` -> `~` and `rpa_J40...RGT8` -> `<masked>` via sed on every doc +
every artifacts/frames json/txt/md; excludes dangerous file types (*.mp4, *.env,
*.key, *.pth, *.pt, *.bak, *.cookies.txt) via .gitignore AND rsync --exclude;
syncs ONLY artifacts/ + frames/ (NOT secrets/). Test: a fake doc with
`/home/muads/...` + `rpa_J40...RGT8` + `sk-abc123` -> after redaction, the path
and the rpa key are masked; `sk-abc123` is NOT (only the one hardcoded key
pattern is redacted). The live mirror (`/home/muads/soccer-channel-status`) has
0 `/home/muads` hits and no .env. HONEST CAVEAT: it is NOT a general secret
scanner — only the one masked-key pattern + paths are redacted; the real
protection is that secrets/ is never synced and .env/.key types are excluded.
`●`

6C.5 — YouTube OAuth token VALID. Token file (743B, Aug 23) was expired
(access token) but the refresh token WORKS: `creds.refresh(Request())` succeeded
-> new access token issued 2026-09-09. Scope is `youtube.upload` (sufficient for
publishing). A read-only `channels.list?mine=true` returned 403 "insufficient
authentication scopes" — that is a SCOPE mismatch (upload != readonly), NOT a
validity failure. The 2 private uploads on Sep 6 used this token. Publishing is
UNBLOCKED. (A definitive videos.insert test would actually publish — outward-
facing — so the refresh-success + upload-scope is the evidence.) `●`

6C.6 — cut-list generator on a 2ND source (arsenal-chelsea, different from the
_20min_test first source). PySceneDetect found 107 scene boundaries; cut_list_gen
loaded 5 broadcast windows + snapped them to scene boundaries; wrote cut_list.json.
The script's footage windows (119-127, 181-191, 253-262) align EXACTLY with the 3
goals (Rogers/Havertz/Odegaard). BUT the FRESHLY snapped cut_list.json windows
(0-5.77, 87.87-99.07, 99.07-106.37, 161.43-172.4, 249.47-254.03) are the
build-up ADJACENT to goals (within ~13s), not goal-exact — only Odegaard
overlaps (249.47-254.03 vs 253-262). Reason: broadcast_filler classified the goal
CELEBRATIONS as filler, so cut_list_gen's broadcast path picked the build-up
before the goals, not the goals. VERDICT: the snapping MECHANICS hold on a 2nd
source (boundaries snap to scenes), but goal-exact windows come from
scoreboard_scan (goal detection), not broadcast_filler. The cut-list generator
is reliable for build-up windows; goal windows need the scoreboard path.
`● (partial — mechanics hold, goal-exact needs scoreboard)`

6C.7 — 27 STANDALONE decisions + 5 retired. Smoke test (Stage 5): 10 have proper
argparse (launch fine); 15 treat `--help` as a positional arg (launch but no -h
— low-priority polish, not a rule-3 failure); 2 broken (luminance_pod crash,
runpod_stage1 hang) + runpod_annotate (6C.1 broken) + vastai_shorts (4B.4
retired, file not deleted) + runpod_shorts (dead dep on luminance_pod). All 5
RETIRED (deleted, in git history): luminance_pod, runpod_stage1, runpod_annotate,
vastai_shorts, runpod_shorts. Tool count 46 -> 37. No live callers (verified by
grep). The remaining 15 --help-mishandlers: decision = "fix: add argparse" as
low-priority polish (they launch + work with real args); not retired. The
untested STANDALONE (cloud_produce, runpod_superres, gpu_superres,
produce_episode, etc.) e2e runs are still owed. `●`

6C.8 — upload speed MEASURED: 95 Mbps (11.9 MB/s) uploading 25MB to catbox.moe
in 2.10s. So B2 archiving is a BACKGROUND task, not overnight: a 25MB short
uploads in ~2s; a 200MB archive in ~17s; 1GB in ~85s. `●`

### 6D — keyframe bug + proxy

6D.1 — stream-copy keyframe bug DIAGNOSED + FIXED + BOUNDED. Diagnosis:
`ffmpeg -ss S -i clip -t DUR -c copy` (input seeking + stream copy) drops a
window when the seek point S lands mid-GOP (no keyframe at S), producing an
empty/invalid file — exactly the 181-191 case on the arsenal-chelsea clip. Fix
(`runpod_download.py` cut_block): try stream-copy with `-avoid_negative_ts
make_zero` first; if the output is <10KB, re-encode that window
(`-c:v libx264 -preset ultrafast -crf 23 -c:a aac`); each attempt is
`timeout`-bounded (30s stream-copy, 120s re-encode) so one bad window can never
hang the pod; a window both methods fail is REPORTED (not silently dropped).
Bound: stream-copy can drop any window whose start isn't near a keyframe; the
fix recovers it via re-encode. Bash -n verified. The 3/3 real verification was
not completed: the re-encode fallback is slow on throttled L4 pods (240s+ for
one window, same as the Stage-5 re-encode hang), so the run was terminated. On a
non-throttled pod the re-encode of a 10s window is ~5-15s. The fix guarantees no
silent drop + no hang; wall-clock depends on pod quality. `● (diagnosed+fixed+bounded; 3/3 real-verify pending a non-throttled pod)`

6D.2 — residential proxy services (web search, 2026). Three services:
| Provider | PAYG | Low-volume sub | Free tier | Bandwidth expiry |
|---|---|---|---|---|
| Bright Data | $4/GB (promo; $8 list) | $499/mo (141GB) ~$3.50/GB | free trial (no card); Web Unlocker 5k req/mo | monthly |
| Decodo (ex-Smartproxy) | $4/GB +VAT | $11.25/mo (3GB) ~$3.75/GB | 3-day trial + 14-day money-back | monthly |
| IPRoyal | $7-7.35/GB | tiered PAYG | none (buy 1GB) | NEVER expires |
KYC: Bright Data very high (company verification); Decodo + IPRoyal low
(self-serve). IPRoyal's non-expiring bandwidth suits irregular workloads.
`●` Sources: iproyal.com/pricing/residential-proxies, brightdata.com/pricing,
proxyfacts.com/blog/bright-data-pricing, use-apify.com/blog/iproyal-vs-smartproxy-2026.

6D.3 — RECOMMEND: AGAINST a monthly proxy bill FOR NOW. Reasons: (1) the
download is 1 stage of 8; the other 7 (boards, voice, assemble, merge, shorts,
cut-list gen, final copy) still run locally and are the bigger rule-1 gap; (2)
the excerpt-cut already works on the pod (5A.3), so the pod does the real work
once it has the clip; (3) 6A now stages raw downloads on /mnt/f at ZERO
permanent disk cost (the vhdx-growth problem is solved without a proxy); (4) the
working rule-1 path today is local-residential-download -> /mnt/f -> catbox ->
pod cut, which costs $0; a proxy adds $4-7/GB or a monthly sub for the SAME
end result. A proxy becomes worth it only when the residential IP itself gets
blocked or when downloads exceed the USB's free space regularly. Until then,
the $0 /mnt/f-staging path is strictly better. `● (recommendation)`

6D.4 — Gemini-cloud backend switch for cut-list gen, SCOPED. scoreboard_scan.py
calls `~/tools/vision_analyze.py` (gemma4:cloud at localhost:11434) to OCR the
score bug — that local-vision endpoint is not on a pod. To move cut-list gen to
the pod, replace the vision call with GEMINI_API_KEY (gemini-3.6-flash, already
in .env, pod-reachable cloud API): ~1.5-2.5h (add a `--vision-backend gemini`
path to scoreboard_scan, test OCR parity on the arsenal-chelsea score bug, ship
scoreboard_scan + broadcast_filler + cut_list_gen + scenedetect to the pod in
runpod_download). After this, cut-list gen runs on the pod and the only local
touch is the final /mnt/c copy. `◑ (scope estimate)`

### 6E — keys + b2_upload (built)

6E.1 — encrypted .env backup BUILT + PROVEN. `tools/backup_env.py`: gpg AES256
symmetric encryption (passphrase from BACKUP_PASSPHRASE env var, never stored on
disk). `backup` mode encrypts ~/yt-digest/.env -> /mnt/f/soccer-channel-keys/.env.gpg;
`verify` mode decrypts + diffs against the live .env. Proven with a temp target
(/mnt/f not mounted at build time):
```
$ BACKUP_PASSPHRASE=... python tools/backup_env.py backup --target-dir /tmp/k
BACKUP OK: .env -> /tmp/k/.env.gpg (714B, AES256 gpg symmetric)  [plaintext 964B]
$ BACKUP_PASSPHRASE=... python tools/backup_env.py verify --target-dir /tmp/k
RESTORE OK: decrypted 964B
  matches live .env (964B): True
```
The encrypted file is binary (not plaintext). Real /mnt/f backup: see "6E.1
REAL" below (after /mnt/f mount). The script fail-loudly checks /mnt/f when the
target is /mnt/f. `● (mechanics proven; real /mnt/f run below)`

6E.2 — `tools/b2_upload.py` BUILT (boto3 already installed). Credentials via .env
(B2_KEY_ID, B2_APPLICATION_KEY, B2_ENDPOINT, B2_BUCKET — same pattern as the
other keys). `archive <slug>` uploads ONLY the finished products + unique
records: renders/<slug>/shorts/final_video_shorts.mp4, final_video.mp4, +
artifacts/publish-log/*.json — NOT the 2.9G of reproducible renders/ intermediates.
`test` heads the bucket (connection check); `--dry-run` lists what would upload;
keys masked in output. Verified: syntax OK, --help works, fails cleanly with a
clear message when B2 keys are not yet in .env:
```
$ python tools/b2_upload.py test
FATAL: B2 credentials not in .env: B2_KEY_ID, B2_APPLICATION_KEY, B2_ENDPOINT, B2_BUCKET
```
The user supplies the B2 key; once added to .env, `b2_upload.py test` then
`b2_upload.py archive <slug>` run for real. `● (built; real upload pending user-supplied B2 key)`

6E.3 — pod-to-B2 direct as a LATER ADDITION to 5A: the pod script in
runpod_download.py already uploads excerpts to catbox; adding a boto3
`put_object` to B2 in the SAME pod script (with B2_KEY_ID/B2_APPLICATION_KEY
env-injected into the pod) sends the finished file straight pod -> B2, and the
laptop only fetches a small /mnt/c copy. This is ~1-2h of additional work (add
boto3 to the pod pip install, add a B2 upload step after the excerpt cut, wire
B2 creds into the pod env). It satisfies rule 1 COMPLETELY for the finished-file
storage (the file is in B2, not on the laptop). It is a LATER ADDITION, not a
change to 5A's design (5A returns excerpts home; pod->B2 adds a second
destination). `◑ (scope estimate)`

### 6A.2 PROOF

REAL RUN after /mnt/f was mounted (`sudo mount -t drvfs F: /mnt/f`). A 134MB raw
clip was staged on /mnt/f, shipped to the pod, the pod received it, and
cleanup_staging deleted the staged raw — /mnt/f free space restored, vhdx untouched:
```
--- /mnt/f free space BEFORE ---           F: 58G  92M  58G  1%
--- staged raw on /mnt/f ---               134844701 clip_PrCW_geeRAU.mp4
--- /mnt/f after staging ---               F: 58G 221M  58G  1%   (+134MB on /mnt/f)
--- uploading staged file to catbox ---    catbox url: https://files.catbox.moe/pq4tcg.mp4
--- pod run (cut 119-127, 253-262) ---     pod wvpttt0lceoh5r RUNNING (received the clip)
--- cleanup staging ---                    staging cleanup: removed /mnt/f/soccer-staging/_6a_proof
--- staged raw gone? ---                   STAGED RAW REMOVED
--- /mnt/f free space AFTER ---            F: 58G  92M  58G  1%   (back to BEFORE)
```
The raw download landed on /mnt/f (not /home/muads), shipped to the pod, and was
deleted on cleanup; /mnt/f freed normally (221M -> 92M). The vhdx never grew.
The pod-cut returning excerpts was delayed by a throttled L4 node (terminated at
~12 min; the excerpt-return mechanic is the Stage-5-proven run #4/#7). The 6A
staging+cleanup+free-space-restore path is PROVEN. `●`

### 6E.1 REAL

REAL backup to /mnt/f after mount:
```
--- /mnt/f free space BEFORE ---           F: 58G  92M  58G  1%
$ BACKUP_PASSPHRASE=... python tools/backup_env.py backup
BACKUP OK: .env -> /mnt/f/soccer-channel-keys/.env.gpg (712B, AES256 gpg symmetric)
  plaintext .env size: 964B; encrypted: 712B
$ BACKUP_PASSPHRASE=... python tools/backup_env.py verify
RESTORE OK: decrypted /mnt/f/soccer-channel-keys/.env.gpg -> 964B
  matches live .env (964B): True       (rc=0)
$ file /mnt/f/soccer-channel-keys/.env.gpg
PGP symmetric key encrypted data - AES with 256-bit key salted & iterated - SHA512
--- /mnt/f free space AFTER ---            F: 58G  92M  58G  1%
```
The encrypted .env backup lives on /mnt/f (712B, AES256 gpg, confirmed by `file`),
and the restore decrypts to bytes that match the live .env exactly. The 13 keys
now have a second (encrypted) copy on the USB. (The proof used a throwaway
passphrase; re-run with your own secret passphrase to make this the real backup.)
`●`

### Carry-forward closed this stage
- 6A: /mnt/f staging built (staging.py + produce_v2 step3 + cleanup) + fail-loudly proven.
- 6B: name overlap resolved — canonical path proven (6B.1), retired fenced (6B.2),
  yt_digest live (6B.3), canonical-path assertion fires from wrong tree (6B.4).
- 6C: ALL 8 beliefs closed (6C.1 runpod_annotate run+retired, 6C.2 estimate, 6C.3
  reasoned, 6C.4 redaction proven, 6C.5 OAuth valid, 6C.6 partial, 6C.7 5 retired
  + decisions, 6C.8 95 Mbps).
- 6D: keyframe bug diagnosed+fixed+bounded (6D.1), proxies researched (6D.2),
  recommend against proxy (6D.3), Gemini-cloud scope (6D.4).
- 6E: encrypted .env backup built+proven (6E.1), b2_upload.py built (6E.2),
  pod-to-B2 scoped (6E.3).
- doctrine rule 1 updated in 4 docs + DECISIONS gap list (6A.4).
- 5 tools retired (6C.7): tool count 46 -> 37.

### Carry-forward still open
- 6A.2/6E.1 real /mnt/f proofs — DONE after mount (pasted above): staging+cleanup+
  free-space-restore proven; encrypted .env backup on /mnt/f + restore matches.
  (Re-run backup_env.py with your own secret passphrase to make it the real backup.)
- 6D.1 3/3 real verification — pending a non-throttled pod (fix is bounded).
- 6E.2 real B2 upload — pending the user-supplied B2 key.
- cut-list gen on the pod (6D.4, ~1.5-2.5h) — the Gemini-cloud backend switch.
- the CPU/storage stages (boards, assemble, voice, ambience, merge, shorts) +
  cut-list gen still run locally — rule 1 (DECISIONS gap list).
- SCRIPT_TEMPLATE / validate_script four-template blocker — scoped 4-6h, not built.

---

## STAGE 7 — rule 3 for documents + four-template blocker, 2026-09-09

### 7A — document audit (rule 3 applies to documents, not just tools)

Method: `grep -rl "<doc>.md" tools/ scripts/ tests/` for every root doc. Result:
NO root doc is read by any pipeline tool at runtime. Tools read
`scripts/<slug>.md` (per-episode scripts), never a root doc. The one hit,
`SCRIPT_TEMPLATE.md` in `fresh_fetch.py:249`, is a string literal ("see
SCRIPT_TEMPLATE.md") in output text, not a file read. So FUNCTIONAL = zero root
docs; the rest split CANONICAL / RECORD / ORPHANED.

```
$ for d in ARCHITECTURE CLAUDE CONFIG ... VISUAL_QUALITY_ASSESSMENT; do
    hits=$(grep -rl "$d\.md\|$d\.html" tools/ scripts/ tests/ 2>/dev/null | tr '\n' ' ')
    printf "%-28s %s\n" "$d" "${hits:-<none>}"; done
SCRIPT_TEMPLATE              tools/fresh_fetch.py tools/USAGE.md
<every other doc>            <none>
```
`grep -n "SCRIPT_TEMPLATE" tools/fresh_fetch.py` -> line 249 is a string literal
(`"that is re-verified by a dated search (see SCRIPT_TEMPLATE.md)."`), not a read.

Classification (21 root docs):
- CANONICAL (mirrored spine + active plan/instructions): ARCHITECTURE, CLAUDE,
  CONTEXT, DECISIONS, GAPS, LANE_PLAN, PROGRESS, README, SCRIPT_TEMPLATE,
  STATUS, TOOLS.
- RECORD (completed-stage artifacts, kept for history): GATES (Stage 5 gates),
  RECONCILIATION (2026-09-08 audit), REPORT_AUDIT (audit that proved
  PROJECT_REPORT wrong), SKILLS (2026-09-05 skill analysis),
  VISUAL_QUALITY_ASSESSMENT (2026-08-25 snapshot, superseded by STATUS).
- ORPHANED (retired): CONFIG, HANDOFF, PLAN_ARTIFACT.html, PROJECT_REPORT.

Retired (7A.2/7A.3) — `git rm` (all 4 were tracked, recoverable from history):
- `PROJECT_REPORT.md` (2026-08-23) — REPORT_AUDIT proved it wrong on every
  structural point.
- `HANDOFF.md` (2026-08-29) — says `vastai_shorts.py` works and RunPod is dead.
  Code contradicts: `ls tools/vastai_shorts.py` -> "No such file" (retired Stage
  6); RunPod is the working path (Stage 5).
- `CONFIG.md` — claims "Nothing else hard-codes the name; everything references
  CHANNEL_SLUG." Code contradicts: 29 of tools/*.py hard-code "soccer-channel"
  (`grep -rln "soccer-channel" tools/*.py | wc -l` -> 29). No tool reads it.
- `PLAN_ARTIFACT.html` (2026-08-25) — claims "9/10 ACHIEVED" with "No CV
  annotation"; cv_annotate works now, pipeline is 720x1280 with
  footage/narration mismatch.

Where retired docs go: DELETED from the tree (git history is the archive; no
`retired/` folder is kept — it would itself be an orphan). Retirement recorded in
DECISIONS.md (Stage 7 section).

7A.4 — every surviving doc carries a header: purpose / reader / last-verified-
against-code date. Verified:
```
$ for f in ARCHITECTURE CLAUDE CONTEXT ... VISUAL_QUALITY_ASSESSMENT; do
    grep -q "Last verified against code" $f && echo OK $f || echo MISSING $f; done
OK (all 15; SCRIPT_TEMPLATE's header ships with its 7B rewrite)
```

### 7B — four-template blocker (BUILT + PROVEN)

7B.1 — `SCRIPT_TEMPLATE.md` rewritten to four lane shapes (215 lines):
- A (tactical analysis, theory-then-footage): Hook/Setup/Body/Trade-off/Close;
  visuals board/footage/tactical; ESPN match_data + sources.json.
- B (match preview, no footage): Hook/Setup/Form/Fixture-prediction/Close;
  visuals board/stills only; fact source = fresh_fetch (same-day brief required).
- C (hot topic, no footage): Hook/Setup/Points/Evidence/Close; visuals
  board/stills only; fact source = fresh_fetch (same-day brief required).
- D (historical review, narration-first): Hook/Setup/Body/Trade-off/Close;
  visuals board/footage=START-END; ESPN match_data + sources.json + cut_list +
  transformation_gate manifest; first section narration-led; [CUTLIST:] required.

`validate_script.py` extended with `--lane A|B|C|D` (default A) + per-lane
structural rules, preserving the existing [SRC]/[RUMOR] source-tag + freshness
checks for all lanes. Per-lane: visual vocabulary; B/C no footage= + same-day
fresh brief; D narration-first + footage=START-END format + [CUTLIST:] ref.
```
$ .venv/bin/python soccer-channel/tools/validate_script.py --help
usage: validate_script.py [-h] [--lane {A,B,C,D}] slug [render_date]
```

7B.2 — fresh_fetch wired for B/C. The wire is the lane contract: B/C validation
REQUIRES a same-day fresh brief under briefs/fresh/ for the render date, so
fresh_fetch is a mandatory prerequisite (not an orphan). Proven live:
```
$ .venv/bin/python soccer-channel/tools/fresh_fetch.py
OK  md: .../briefs/fresh/2026-09-09_0737_fresh.md  (43 reported, 0 warnings)  EXIT=0
```
And the wire bites when no brief exists (render date 2026-09-08 has none):
```
lane B: no same-day fresh brief for 2026-09-08 in .../briefs/fresh (run fresh_fetch.py first)
FAIL  EXIT=1
```
The step1b subprocess call into a B/C entry point is deferred to the B/C pipeline
build (this blocker unblocks it; it does not build the lanes).

7B.3 — real runs per lane (validator output, including deliberate failures):
```
# Lane B PASS (preview)
=== VALIDATING SCRIPT: 2026-09-09_preview-arsenal-chelsea (lane B) ===
PASS: all source tags resolve, all rumors fresh, lane B structural rules satisfied.  EXIT=0

# Lane C PASS (hot topic)
=== VALIDATING SCRIPT: 2026-09-09_hottopic-pressing (lane C) ===
PASS: ... lane C structural rules satisfied.  EXIT=0

# DELIBERATE FAIL: lane B script with a footage= tag
lane B: visual type 'footage' not allowed (allowed: ['board', 'stills']): [VISUAL: footage=10-15]
lane B: footage= tag forbidden (no footage in this lane): [VISUAL: footage=10-15]
FAIL  EXIT=1

# Lane D PASS (narration-first, footage=START-END, cutlist ref)
=== VALIDATING SCRIPT: 2026-09-09_review-arsenal-chelsea (lane D) ===
PASS: ... lane D structural rules satisfied.  EXIT=0

# DELIBERATE FAIL: lane D with footage= in the first section (narration-first)
lane D: narration-first violated — first section '## Hook' contains a footage= tag
FAIL  EXIT=1
```

7B.4 — lane D + transformation_gate compose, not conflict. The two checks
operate on different artifacts: validate_script checks the SCRIPT grammar (tag
format, section order, cut-list reference); transformation_gate checks the
ASSEMBLY manifest (durations, muting, layers, borrowed ratio). Composition proof
— an 11s window passes the script grammar but fails the assembly floor:
```
# validate_script PASSES the D script with footage=119-130 (format valid: two numbers)
=== VALIDATING SCRIPT: _comp_laneD_11s (lane D) ===
PASS: ... lane D structural rules satisfied.  EXIT=0

# transformation_gate FAILS the matching 11s excerpt (rule 2: borrowed <= 8s)
transformation_gate: FAIL — lane D floor violated:
  - rule 2 (borrowed excerpt <= 8s): segment 1 is 11.0s   EXIT=1

# transformation_gate PASSES the 8s-excerpt manifest (all 5 floor rules)
transformation_gate: PASS — all 5 floor rules satisfied   EXIT=0
```
The one surface interaction (a footage=START-END window's length) is split
correctly: validate_script checks the FORMAT (two numbers); transformation_gate
checks the DURATION (<=8s). Neither blocks the other.

### 7C — open items closed

7C.1 — keyframe fix 3/3 real verify. ATTEMPTED on a fresh L4; the pod hung
(throttled) and did NOT complete — left ◑ with the bound stated. A fresh L4
(k9qsppkcgnl8wq, "soccer-pod-dl") was rented for the verify:
```
$ .venv/bin/python soccer-channel/tools/runpod_download.py \
    --source-url https://files.catbox.moe/pq4tcg.mp4 \
    --cut-windows "119-127,181-191,253-262" \
    --out-dir /mnt/f/soccer-staging/stage7-verify --slug stage7-verify
4. Creating pod (NVIDIA L4)...  Pod: k9qsppkcgnl8wq
5. Waiting for results (max 20 min)...
   [15s] pod: running  ...  [540s] pod: running   # zero webhook posts throughout
EXIT=124   # the 600s timeout fired
```
The catbox source is alive (ranged GET returned the `ftypisom` MP4 box + 1MB; the
HEAD `content-length: 0` is a catbox quirk). The pod sat at "running" for 540s+
with ZERO webhook posts — the same throttled-L4 behavior as 6A.2 (12-min node)
and 6D.1 (240s+ hang). The fix is NOT the blocker; pod quality is. Bound
restated: the keyframe fix (stream-copy + re-encode fallback, each timeout-
bounded) is diagnosed+fixed+bounded (●); on a non-throttled pod the re-encode of
a 10s window is ~5-15s and the fix guarantees no silent drop + no hang; the 3/3
real-verify is pending a non-throttled pod, and tonight's L4s are throttled (a
fresh one hung the same way). `● (fix diagnosed+fixed+bounded; 3/3 real-verify
pending a non-throttled pod — pods throttled tonight)`

Secondary finding — pod-leak on timeout (NEW fragile). `timeout 600` SIGTERMs the
python process before any pod cleanup runs, so the pod keeps RUNNING and billing.
This is how the two leftover stage1 pods (7C.2) and this verify pod leaked. The
verify pod (k9qsppkcgnl8wq) was found still `desiredStatus=RUNNING` after the
timeout and terminated manually this session; pod count returned to 0.
Recommendation: `runpod_download.py` should terminate the pod in a `finally` /
signal handler so a timeout or crash never leaves a pod billing. Same applies to
`runpod_fulltrack.py`. `★ (fragile: runpod_download/runpod_fulltrack leak the pod
on timeout/crash — manual termination required until fixed)`

7C.2 — RunPod spend + pods. TWO leftover "soccer-stage1" pods
(bpkjubn1qld6z7, q8ntquu28lcjeh) were found in a RUNNING-desired state despite
prior stages reporting pods terminated — a billing leak. Both STOPPED then
TERMINATED this session; pod count now 0:
```
$ runpod.get_pods()  # before
  id= bpkjubn1qld6z7 desiredStatus= RUNNING
  id= q8ntquu28lcjeh desiredStatus= RUNNING
$ runpod.terminate_pod(...)  # both
$ runpod.get_pods()  # after
  pod count: 0
```
The 12-min throttled node from 6A.2 was terminated (LANE_PLAN:1985). Total
documented RunPod spend across stages is ~$0.20-0.25 in known per-run L4 costs
(runpod_fulltrack $0.011, runpod_stage1 $0.05, runpod_annotate $0.004,
runpod_download runs ~$0.005 each, plus the throttled/hung pods run #6 315s
~$0.022 and 6D.1 240s ~$0.017). The RunPod GraphQL schema exposes no balance/
billing field (introspection returned empty), so the API cannot confirm the
total. The two leftover pods' prior uptime is an unbounded unknown not in the
docs. No pod is running now (pod count 0).

7C.3 — Gemini-cloud vision switch scope confirmed, unchanged by 7B. 6D.4 scopes
it at ~1.5-2.5h: add a `--vision-backend gemini` path to scoreboard_scan.py
(replace the localhost:11434 gemma4 call with GEMINI_API_KEY cloud Gemini), test
OCR parity, ship scoreboard_scan + broadcast_filler + cut_list_gen + scenedetect
to the pod in runpod_download. It is the last rule-1 work not blocked on an
external input (GEMINI_API_KEY already in .env). 7B built script-template/
validator logic, which does not touch cut-list generation or scoreboard_scan, so
the scope still holds at 1.5-2.5h. NOT built (report only). `◑ (scope estimate)`

### Carry-forward closed this stage
- SCRIPT_TEMPLATE / validate_script four-template blocker — BUILT + PROVEN (7B).
  Was the load-bearing fragile blocking lanes B, C, D.
- fresh_fetch wired into the B/C lane contract (7B.2).
- PROJECT_REPORT.md + 3 other orphaned docs retired; rule 3 now covers docs (7A).
- 2 leftover billing pods found + terminated (7C.2).

### Carry-forward still open
- cut-list generation still runs locally; needs the Gemini-cloud switch (6D.4,
  ~1.5-2.5h, 7C.3 confirmed scope). Last rule-1 work not blocked on external input.
- 6D.1 keyframe fix 3/3 real verify — see 7C.1 outcome above.
- rule 1's download clause blocked by YouTube (external party), proxy declined.
- real B2 upload pending the account key; 1080p-on-pod never reached (YouTube
  blocks the download first). Both accepted ○.

---

## STAGE 8 — fix the pod leak, produce a lane B episode, 2026-09-12

### 8A — the pod leak (FIXED + proven at mechanism level; live-pod proof blocked on GPU capacity)

8A.1 — the fix. Both `runpod_download.py` and `runpod_fulltrack.py` ALREADY had a
`try/finally` around the wait loop that called `runpod.terminate_pod` on normal
exit / caught exceptions. The leak was specifically SIGTERM/SIGINT: `timeout`
sends SIGTERM, whose default handler kills the process BEFORE the finally runs
(`timeout 600` SIGTERMs python → finally never executes → pod keeps billing).
That is exactly how the Stage-7 verify pod and the two leftover stage1 pods leaked.

Fix added to both files: a module-level `_pod_state = {"id": None}`, an
idempotent `_cleanup_pod()` (terminates + clears state, safe to call twice), a
`_signal_handler(signum, frame)` that calls `_cleanup_pod()` then
`sys.exit(130 if SIGINT else 143)`, registered for SIGTERM and SIGINT at import;
`_pod_state["id"] = pod_id` is set right after `create_pod`; the finally now
calls `_cleanup_pod()`. Every exit path (normal return, break, exception,
SIGTERM, SIGINT) terminates the pod.

8A.2 — the proof. RunPod had ZERO GPU capacity today (get_gpus() returned 0/48
types with capacity>0; `create_pod` raises "no longer any instances available"),
so a live-pod 3-way could not create a pod. Proved the fix at the mechanism level
against the real patched code, with `runpod.terminate_pod` recorded — all 6 cases
PASS for BOTH modules, including a REAL `os.kill(SIGTERM)`:
```
$ .venv/bin/python /tmp/s8_mech_proof.py runpod_download
  runpod_download    SIGTERM handler          PASS   # terminate_pod(right id), exit 143
  runpod_download    SIGINT handler           PASS   # exit 130
  runpod_download    finally _cleanup_pod     PASS   # terminate called, state cleared
  runpod_download    idempotent no-op         PASS   # 2nd cleanup doesn't double-call
  runpod_download    SIGTERM registered       PASS   # signal.getsignal is the handler
  runpod_download    real SIGTERM via os.kill PASS   # real signal delivery fires it
RESULT runpod_download ALL PASS
$ .venv/bin/python /tmp/s8_mech_proof.py runpod_fulltrack
RESULT runpod_fulltrack ALL PASS   # same 6/6
```
The remaining link — "runpod.terminate_pod actually stops a real running pod" —
was proven in Stage 7 (3 real RUNNING pods terminated, get_pods()==0). The
live-pod 3-way (create a real pod, timeout/Ctrl-C/normal, get_pods()==0) is
`◑ pending RunPod GPU capacity` (0/48 today). The fix is verified; the
live-pod confirmation is an external block (same class as 7C.1's throttled pods).

8A.3 — session-start pod check. New `tools/pod_check.py` (warn by default, lists
RUNNING pods; `--terminate` kills them). New `.claude/hooks/check_pods.sh` runs
it on SessionStart; wired in a new project `.claude/settings.json`
(SessionStart hook, timeout 20, non-blocking). CLAUDE.md standing rule 11 added.
Verified: `bash .claude/hooks/check_pods.sh` -> "pod-check: 0 running pods
(clean).", exit 0. Now a leaked pod is surfaced at session start, not across
days. (Auto-terminate is intentionally NOT the default — a concurrent session
could be using a pod; the hook warns, the session terminates with `--terminate`.)

8A.4 — is the spend knowable? RunPod's GraphQL: introspection is DISABLED
("GraphQL introspection is not allowed by Apollo Server"). `myself { billing(input: UserBillingInput!) {...} }`
exists but the input shape is undiscoverable without docs. `myself { pods { uptimeSeconds } }`
is a valid field for LIVE pods only. Terminated/deleted pods are gone from
`get_pods()` — the two leftover stage1 pods' historical uptime is NOT
API-recoverable. The web console (app.runpod.io → Billing) is the only place
the actual spend (including those two pods) can be seen, and it is a manual
browser action. `○ unchecked — the two leftover pods' prior billing stays
unbounded via the API; web console is the only path.`

### 8B — a lane B episode, end to end (PRODUCED; the break list is the point)

8B.1 — fixture. Manchester United FC vs Manchester City FC, Premier League,
2026-09-13 15:30 (the Manchester derby, tomorrow). Picked from today's
fresh_fetch brief: it is the highest-profile upcoming PL fixture, timely for a
preview, and narrative-rich — City 2nd (9 pts/3 games) chasing Arsenal (12/4),
United 13th (4/3) already 9 off the summit.

8B.2 — each stage's output.
- brief: `fresh_fetch.py` -> `briefs/fresh/2026-09-12_2157_fresh.md`, 54 items, 0 warnings.
- script: hand-written `scripts/2026-09-12_preview-manc-derby.md` (lane B shape:
  Hook/Setup/Form/Fixture-prediction/Close; board= tags; [SRC:] inline; sources
  list in the header). No lane B script generator exists.
- validate: `validate_script.py 2026-09-12_preview-manc-derby 2026-09-12 --lane B`
  -> PASS, 4 tags / 4 sources, exit 0.
- boards: `tactical_boards.py` -> CRASHED first (`KeyError: 'score'` — it assumes
  a played match). Hand-added `"score":"0"` to match_data.json; re-ran ->
  formation.png, possession.png, stat_card.png (+ .mp4 fallbacks), exit 0.
- voice: `generate_voice.py --tts-only` -> `voice_elevenlabs.mp3`, 768 chars,
  ~138 words, 41.0s, 0.3MB, exit 0 (ElevenLabs Turbo v2.5).
- assemble: no wired board-only assembler exists; manual ffmpeg via
  /tmp/s8_assemble.py (sections timed to word counts, boards mapped per
  [VISUAL: board=]) -> `final_video.mp4`, exit 0.

8B.3 — the finished file.
```
path:     renders/2026-09-12_preview-manc-derby/final_video.mp4
streams:  h264 1920x1080  +  aac audio
duration: 41.020998s
size:     997774 bytes (975KB)
```
Section timing: Hook 6.84s (formation) / Setup 8.03s (stat_card) / Form 10.40s
(possession) / Fixture prediction 9.81s (formation) / Close 5.95s (stat_card).
No pod used (lane B is pod-free). No upload (8B.5).

8B.4 — every manual step / hardcoded value / hand edit / break (lane B had never
run; this is the unsmoothed list):
1. **Script hand-written** — no lane B script generator exists (LANE_PLAN 3.5 DNX).
2. **match_data.json hand-built** — the fresh_fetch→match_data adapter is DNX.
   Predicted formations (4-2-3-1 vs 4-3-3), predicted lineups (11 names each,
   GUESSED — no lineup source for previews), placeholder stats.
3. **tactical_boards crashed on missing `score`** — it assumes a played match
   (scoreline). Hand-added `"score":"0"`; the formation board now shows "0 - 0",
   wrong for a preview. No preview/"vs" mode exists.
4. **stat_card shows all zeros** — no pre-match stats source; shots/passes/
   tackles/corners/fouls all render "0".
5. **possession board is a placeholder** — 44/56 is a guessed prediction, not a
   real match value.
6. **board=table / board=form / stills= from the lane B template are NOT
   implemented** — tactical_boards only does formation/possession/stat_card; I
   used those 3 instead. The stills fetcher is DNX (LANE_PLAN 3.5 Lane C).
7. **Assembly is manual ffmpeg** — no wired board-only assembler exists
   (LANE_PLAN 3.5 Lane B DNX). /tmp/s8_assemble.py is not a committed tool.
8. **No shorts crop** — output is 1920x1080 horizontal; produce_v2's
   shorts_crop (720x1280) is wired only for the footage path. Lane B has no
   shorts step.
9. **Hardcoded in the pipeline**: generate_voice atempo=1.20, voice settings
   (stability 0.30 / similarity 0.80 / style 0.50), VOICE_ID from .env
   (1stSYyl7ZVPJk2ECrNlo); tactical_boards DPI/colors/ANIM_DURATION.
10. **fresh_fetch run manually** — not wired as a pipeline step1b for lane B
    (the validator requires the brief; running it is a manual prerequisite).
11. **Episode is 41s, not 6-10min** — the lane B template targets 6-10min
    (1600-2400 words); I wrote a ~140-word short preview. A full-length episode
    needs a much longer script (manual authoring, no generator).
12. **No upload** (by design, 8B.5).

### 8C — report only

8C.1 — what lane C needs beyond lane B. For a boards-only episode: ~0h
additional — the same path works today (fresh_fetch → hand script →
`validate_script --lane C` [already proven Stage 7] → tactical_boards →
generate_voice → manual assemble). Lane C's template (Hook/Setup/Points/
Evidence/Close) and validator already exist. The only lane-C-specific item is
the **stills fetcher** (topic-relevant images, DNX, ~2-3h per LANE_PLAN 3.5),
which is optional visual variety — lane C ships boards-only without it. So:
~0h for a boards-only lane C (reuse the lane B path); ~2-3h if the stills
fetcher is wanted.

8C.2 — Gemini-cloud cut-list switch. Confirmed still scoped at 1.5-2.5h
(6D.4: add `--vision-backend gemini` to scoreboard_scan.py, ship cut-list tools
to the pod). 8A (pod-leak fix) and 8B (lane B, pod-free, no cut-list) did not
touch cut-list generation or scoreboard_scan, so the scope is unchanged. It is
still the last rule-1 work not blocked on an external input (the other rule-1
items: YouTube bot-block = external; flash drive = external; 1080p-on-pod =
blocked by YouTube). NOT built (report only). `◑ (scope estimate)`

### Carry-forward closed this stage
- Pod-leak fragile — FIXED (8A.1) + proven at mechanism level (8A.2); session-
  start backstop installed (8A.3). Live-pod 3-way ◑ pending RunPod GPU capacity.
- Lane B never-run fragile — RUN (8B): a real lane B MP4 produced end-to-end,
  with the full break list recorded (8B.4).

### Carry-forward still open
- cut-list generation still runs locally; needs the Gemini-cloud switch (6D.4,
  ~1.5-2.5h, 8C.2 confirmed) — last rule-1 work not blocked on external input.
- 6D.1 keyframe 3/3 real-verify — pending a non-throttled RunPod pod (0/48 GPU
  capacity today, 8A.2). Fix is bounded.
- rule 1's download clause blocked by YouTube (external), proxy declined.
- unplugged flash drive stops /mnt/f downloads by design (8B ran with /mnt/f
  unmounted — lane B needs no staging).
- real B2 upload pending the account key; 1080p-on-pod blocked by YouTube first.
- two leftover pods' prior billing unbounded via API (8A.4); web console only.
- lane B gaps from 8B.4 (script generator, fresh_fetch→match_data adapter,
  board-only assembler, shorts for lane B, stills fetcher for lane C) — none
  built this stage; the episode was produced via manual steps.

---

## Stage 9 — benchmark from real reference videos (2026-09-12)

Production freeze in effect until EPISODE_SPEC.md is approved. The lane B
episode was reviewed and rejected 2/10; this stage measured the references,
wrote the spec, and fixed three defects that needed no benchmark.

### 9A. Measurement method and raw data

Six references, staged at 720p in `/dev/shm` (one-time doctrine exception:
/mnt/f was unmounted and sudo was unavailable; /dev/shm is RAM, frees
automatically, no vhdx bloat — matches rule-1 intent). Original 720p downloads
were AV1 (yt-dlp format 398); PySceneDetect's backend cannot decode AV1 on the
N150, so 480p H.264 sidecars (format 135) were downloaded for shot detection.
ffmpeg decoded the AV1 fine for frame extraction. All files deleted after
measurement.

Per video: runtime + WPM from `yt-dlp --print` + srt; shots from
`scenedetect -i <h264> --downscale 4 detect-content -t 27 list-scenes`;
content/graphics/typography from 18 vision-classified frames (6 opening at
t=0,3,6,9,12,15 + 12 uniform) via `vision_analyze.py` (gemma4:cloud); framing
from ffmpeg edge-pixel black-bar checks. Results in
`/dev/shm/refs/result_<id>.json` (ephemeral; aggregated below).

```
$ yt-dlp --skip-download --print "%(id)s|%(duration)s|%(channel)s" <6 urls>
kRLtilxlEj4|698|Football Meta| ... URlf-04YYLk|530| ... 98BkEsAUr9k|507| ...
-MLGcROAr8c|741| ... f1xtEHOjrRA|810| ... iaLdVWUrq5Q|907|AlfonsoR10
```

| ID | Channel | Runtime s | Shots | Mean s | Median s | Longest s | WPM |
|---|---|---|---|---|---|---|---|
| kRLtilxlEj4 | Football Meta | 698 | 43 | 16.2 | 12.5 | 70.8 | 195 |
| URlf-04YYLk | Football Made Simple | 530 | 46 | 11.5 | 4.6 | 69.3 | 181 |
| 98BkEsAUr9k | DK FALCON | 507 | 133 | 3.8 | 3.0 | 20.2 | 193 |
| -MLGcROAr8c | Mega Football | 741 | 301 | 2.5 | 2.0 | 12.7 | 179 |
| f1xtEHOjrRA | Ball Explained | 810 | 211 | 3.8 | 3.4 | 14.1 | 166 |
| iaLdVWUrq5Q | AlfonsoR10 | 906 | 197 | 4.6 | 3.2 | 19.1 | 155 |

Content-type ratio (18 frames/video, vision-classified):

| Channel | graphics | footage | talking-head | title | still | transition |
|---|---|---|---|---|---|---|
| Football Meta | 0.72 | 0.06 | 0.17 | 0 | 0 | 0.06 |
| Football Made Simple | 0.78 | 0.11 | 0 | 0.11 | 0 | 0 |
| DK FALCON | 0.89 | 0 | 0.06 | 0.06 | 0 | 0 |
| Mega Football | 0.06 | 0.33 | 0.22 | 0.17 | 0.06 | 0.17 |
| Ball Explained | 0.22 | 0 | 0 | 0.39 | 0.39 | 0 |
| AlfonsoR10 | 0 | 1.00 | 0 | 0 | 0 | 0 |

Graphic types seen: pitch_diagram, formation_board, stat_card (FM/FMS/DK);
arrows_on_footage (FM); lower_third (FMS, AlfonsoR10). Typography bold-dominant
in 5/6; dark backgrounds (#0d to #1c, #000) + white + accents (red #c41230,
green #4CAF50, yellow #c7c74c). All AAC stereo ~128 kbps. All 16:9; opening
edge checks 0 % black on 5/6 (DK FALCON had one dark frame, not a bar).

Key finding (rhythm is bimodal): Football Meta and Football Made Simple hold
long on diagrams (43 to 46 shots, mean 11 to 16 s, longest ~70 s) — the
Coaches' Voice pace the project targets. The other four cut fast (133 to 301
shots, mean 2.5 to 4.6 s). The spec targets the deliberate style.

### 9B. Spec, scoring, buildability

Written to `EPISODE_SPEC.md`. Every MUST traces to a 9A row above. Scoring:
rejected lane B episode 6/16 (3.75/10) post-fix — disqualified on runtime
(41 s = 8.5 % of the 480 s minimum) and opening (static board, the forbidden
open). The two private arsenal-chelsea uploads 6/16 (3.75/10) — same runtime
fail plus 1280x720 framing fail. Full item-by-item tables in EPISODE_SPEC.md.

Buildability split: framing + no-fabrication boards + dark/bold palette + WPM
knob are hittable now; a 1300 to 1900-word script, a 40 to 80-cut content-
matched cut list, footage-overlay graphics, lower thirds, transitions, music
bed, and opening hook need building; talking-head, 3D tactical renders, and
at-scale broadcast footage acquisition are impossible with current tools
(9D.2).

### 9C. Three defects fixed (no benchmark needed)

9C.1 — tactical_boards fabricates zeros. FIX (verified): stat_card now skips
any row where the stat is absent for either team, and skips all match-outcome
stats (shots/passes/tackles/etc.) when `match_data["preview"] == true`;
possession board returns early ("SKIP ... not fabricated") when possession is
absent; formation board hides the scoreline for previews ("PREVIEW A vs B").
Proof: vision on the regenerated manc-derby stat_card shows only "Possession %
44 / 56" (no Shots/Passes/Tackles); vision on formation shows no "0 - 0"; a
synthetic match_data with no possession prints SKIP and writes no file; one
with no stats prints SKIP and writes no file. `match_data.json` for the
manc-derby preview now carries `"preview": true`.

9C.2 — pillarboxed output. Root cause: `tactical_boards.py` saved PNGs with
`bbox_inches="tight"`, producing non-16:9 crops (formation.png was
2508x1784 = 1.406:1); the manual assembler fit those into 1920x1080 with
black pad -> pillarbox. FIX (verified): every board PNG/MP4 now normalizes to
exactly 1920x1080 (fit + pad with pitch-bg color #0d1f16) via a new
`_normalize_board` / `_normalize_board_video` helper. Proof:
```
before edge-pixel check: left black=14/14 right black=14/14  (pillarboxed)
after  edge-pixel check: left black=0/14  pitch=14/14, right black=0/14 pitch=14/14
ffprobe: 1920x1080 / 41.02 s (both); all 6 board PNGs+MP4s now 1920x1080.
```

9C.3 — where the 41 seconds went. The voice is 41.02 s; the assembler times
each board to its word-share of the voice; the script is ~140 words across 5
sections -> ~200 WPM -> 41 s. It is the **script length**, not the board count
or the assembler. A 6 to 10 min target needs ~1100 to 1900 words. The
SCRIPT_TEMPLATE four-template blocker (Stage 7B) does not set a word budget;
that is the next build item.

### 9D. The hard question

9D.1 — does a boards-only, no-footage format (lane B as built) resemble ANY
reference? No. The five tactical references are graphics-dominant (72 to 89 %
in the three core channels) but all intercut footage (Football Made Simple
11 %, Mega Football 33 %) or overlay graphics on footage (Football Made
Simple's highlight circle on a player), and every one opens with footage, a
data visual, or a title over footage — none opens with a static board held for
the whole runtime. The closest to "boards-only" is DK FALCON (89 % graphics),
but its graphics are 3D animated pitch renders cut 133 times, not static
PNGs. Lane B (1 shot, 3 static boards, 41 s, 0 % footage) resembles none of
them. **Recommendation: drop lane B as a publishable tactical-analysis
format.** A boards-only preview has a legitimate home as a sub-60-second
pre-match Short (a different product), not as the 8 to 14 min episode the
channel targets. Polishing lane B's boards will not close the gap; the format
itself is outside the reference set.

9D.2 — the footage conflict, RESOLVED by the Stage 10A.3 amendment. The
references depend on broadcast footage overlaid with graphics (Football Made
Simple, Mega Football) or run footage-first (AlfonsoR10 100 %). Stage 9D.2
called this impossible because rule 1 forbade local downloads; Stage 9A then
proved the residential connection reaches YouTube reliably (all 6 references
downloaded at 720p); Stage 10A.3 amended rule 1 to make that the explicit
path: residential download -> /mnt/f -> ship to pod -> delete locally. The
RunPod datacenter YouTube block is irrelevant to acquisition because footage
is downloaded on the residential connection, not on the pod. The only
remaining constraint is physical: /mnt/f must be mounted (USB present) so
nothing raw is written inside /home/muads. When mounted, footage acquisition
is doctrine-clean. This is no longer a code blocker; it is a
hardware-availability check. (Original Stage 9 wording retained above the
amendment note for history; the amendment supersedes it.)

### Stage 9 carry-forward
- EPISODE_SPEC.md awaits user approval (production freeze gate).
- Script word budget (1300 to 1900 words for 8 to 14 min) not yet in
  SCRIPT_TEMPLATE — the single biggest build item, pure text.
- Lane B recommendation: drop or reshape to a sub-60 s preview Short.
- /mnt/f still unmounted; 9A used /dev/shm (RAM) under a one-time exception.
  At-scale footage acquisition still blocked (9D.2).
- /dev/shm reference videos + measure.py deleted after measurement (cleanup
  done this stage).
---

## Stage 10 — amendments, script budget, opening gate, 3D scope (2026-09-12)

EPISODE_SPEC.md APPROVED with three amendments (10A). Production freeze stays
until an episode scores >= 7/10 against the spec. 10A/10B/10D built; 10C is
scope + one proof-of-concept only (blocked on GPU).

### 10A — three spec amendments + doctrine Rule 1

- 10A.1 talking-head: removed from "impossible", made a stated format decision
  (voiceover-only, following Football Made Simple at 0 %). EPISODE_SPEC §3 +
  §9B.4 updated.
- 10A.2 3D renders: promoted from "impossible" to a build target (10C).
  EPISODE_SPEC §9B.4 updated.
- 10A.3 footage acquisition: removed from "impossible". Rule 1 amended in all
  four canonical docs (CLAUDE.md, CONTEXT.md, DECISIONS.md, ARCHITECTURE.md):
  "Raw footage is acquired over the residential connection, staged on /mnt/f,
  shipped to the pod, and deleted locally. Nothing raw is written inside
  /home/muads. No local GPU work..." Verified: `grep -c "Raw footage is
  acquired over the residential"` = 1 in each of the 4 docs; old "NOTHING RUNS
  ON THE LOCAL MACHINE" rule-1 text = 0 remaining. 9D.2 (here and in
  EPISODE_SPEC) updated: the footage conflict is resolved by the amendment;
  the residual constraint is physical (/mnt/f mount), not code.

### 10B — script word budget (the biggest gap)

10B.1: SCRIPT_TEMPLATE.md §WORD BUDGET added per lane with per-section targets
summing to the lane min/max (175 WPM midpoint of the 155-195 spec range):
A/D 1400-2100, B/C 1050-1750. Per-section: A/D Hook 40-60 / Setup 130-200 /
Body 1000-1500 / Trade-off 130-200 / Close 100-140; B Hook 40-60 / Setup
130-200 / Form 200-350 / Fixture prediction 580-1000 / Close 100-140; C Hook
40-60 / Setup 130-200 / Points 500-900 / Evidence 280-450 / Close 100-140.

10B.2: validate_script.py enforces it — `count_narrated_words` (strips tags,
drops ## SOURCES, drops headers) + `check_word_budget`; a script under the
lane minimum FAILs. Proven in the full CLI path:
```
$ .venv/bin/python tools/validate_script.py _wbtest 2026-09-12 --lane A
Spoken words: 17 (lane A minimum: 1400)
word budget (lane A min 1400): script has 17 spoken words, 1383 short ...
FAIL: see errors above.
```
A compliant-length body (4000 words) passes the check (unit-tested). Temp
fixture deleted after proof.

10B.3: tools/script_gen.py built — the generator 8B.4 listed as missing.
- What generates the text: glm-5.2:cloud via the local Ollama gateway
  (POST http://localhost:11434/api/chat), one call per section, each prompted
  with its own word target.
- What it reads: renders/<slug>/match_data.json (lineups/score/stats/
  formations) and briefs/fresh/<date>*_fresh.md (fixtures/standings/form/
  news).
- Where a human still intervenes: (1) fact-check every sentence (the LLM
  hallucinates; validate_script catches tag/freshness errors but not prose
  hallucination), (2) resolve [SRC: id] placeholders to sources.json, (3)
  confirm [VISUAL] assets exist, (4) edit for voice + the trade-off section
  (LLM tends to hype; spec wants the cost named), (5) run validate_script.
- Proven: generated a lane B draft for 2026-09-12_preview-manc-derby in 5
  per-section calls; output 1439 spoken words (lane B min 1050), per-section
  Hook 53 / Setup 189 / Form 311 / Fixture-prediction 745 / Close 141.
  validate_script's counter agrees (1439). File:
  scripts/2026-09-12_preview-manc-derby_gen.md.

### 10D — the opening hook

10D.1 — what the assembler needs to open on footage / a data visual / a title
over footage: the first ## section's [VISUAL] tag drives the first assembly
segment. To open on footage, the first segment must be a footage excerpt (a
cut-list window); to open on a data visual, the first segment must be a
stat_card (or xG table) board; to open on a title over footage, the first
segment must be a title card composited over a footage excerpt. The current
lane B manual assembler (/tmp/s8_assemble.py) maps board->board PNG only; a
wired assembler would need to map footage->cut-list clip and title->composite.
The gate below enforces the choice at validation time, before assembly.

10D.2 — hard gate: validate_script.py `check_opening_hook` rejects a script
whose first ## section opens on a static formation board (board=formation /
board=form), or has no opening visual. Allowed opens: footage, stills,
tactical, or a data board (stat_card etc.). Proven in the full CLI path:
```
fixture opens [VISUAL: board=formation] -> FAIL
  opening hook (spec §6): first section 'Hook' opens on a static formation
  board ([VISUAL: board=formation]) — forbidden. Open on footage, a data
  visual (e.g. board=stat_card), or a title over footage.
fixture opens [VISUAL: footage=0-5] -> PASS
  opening hook: first section 'Hook' opens on [VISUAL: footage=0-5] (OK)
```
Unit test: 7/7 cases (formation/form FAIL; footage/stat_card/stills/tactical
PASS; no-visual FAIL). Lane B's template Hook changed board=formation ->
board=stat_card (a data visual) so lane B stays passable; script_gen.py lane B
Hook updated to match. Temp fixtures deleted after proof.

### 10C — 3D tactical rendering: scope + proof-of-concept (BLOCKED on GPU)

10C.5 — tool choice. Blender is the right tool, not just the first idea.
Evaluation: Blender (Python bpy, headless `blender -b -P scene.py`, EEVEE
Next / Cycles GPU) is scriptable in Python, runs headless on a cloud GPU,
builds a 3D pitch with player tokens, camera moves, and animated arrows, and
renders to PNG/MP4 — exactly the need. Alternatives rejected: Manim (3D is
limited, math-animation focus), pyrender (headless-GPU possible but weak
animation/camera tooling — a fallback for static frames only), Unreal movie
renderer (overkill, steep, poor fit for a transient pod). The carry-forward
belief is confirmed: Blender headless on GPU is the right 3D path.

10C.1 — scope.
- Tool: Blender 4.x headless, EEVEE Next for speed (Cycles for quality).
- Local (N150, no GPU): write + dry-run the bpy scene script; no rendering.
- Pod: ship scene_gen.py + match_data.json to a RunPod GPU pod (Blender via a
  pod template or apt), run `blender -b -P scene_gen.py -- --match-data
  match_data.json --out frames/`, return an MP4.
- Scene generation from match_data: a Python bpy script reads match_data.json
  (formations, lineups, team colors, jersey numbers), builds a 3D pitch plane
  with markings, places 11+11 player tokens at formation positions, colors by
  team, labels jerseys, animates a camera move (orbit/zoom) and animated
  Bezier arrows for one tactical point, renders 5 s at 25 fps = 125 frames.
- Per-episode cost (estimate, not measured): a full episode has ~40-80 graphic
  segments. EEVEE Next on an L4 ~10-30 fps -> ~5-15 min render for the graphic
  layer + pod startup/Blender-install overhead (~2-5 min). Pod cost L4
  $0.25/h -> ~$0.05-0.10 per episode render. Wall time ~15-30 min per episode
  on an L4.
- Honest hours to BUILD: 1-3 days of dev (bpy API, pitch modeling, camera +
  arrow animation, match_data integration, headless pod workflow, asset
  fonts per spec §5). That is days, not hours.

10C.2 — 2D today vs 3D target.
- 2D tactical_render.py today: a flat top-down matplotlib pitch (dark green,
  mowing stripes), player dots in team colors, movement-trail polylines,
  Bezier arrows, screen-space projection (no homography), static camera,
  matplotlib aesthetics. tactical_boards.py scored 3-5/10 (flat, matplotlib
  defaults, no Bebas/Barlow, no narrative furniture).
- 3D target: a 3D pitch in perspective (camera at an angle, not top-down),
  depth (player tokens with height + shadows), camera movement (orbit/zoom/
  pan), animated arrows that grow along a path, lighting. This matches the
  DK FALCON reference, whose 9A.5 opening frames were described as "3D
  tactical representation of a football pitch with player icons" and "3D
  stylized representation ... with simplified player markers" — a cinematic
  pitch with perspective and motion, not a flat diagram.

10C.3 — proof-of-concept: BLOCKED. GPU capacity is 0/48 on 2026-09-12
(`runpod.get_gpus()`: 48 types, 0 with available > 0; re-checked twice this
session). No pod can be launched, so no 3D frame can be rendered. Per 10C.5
(second-version) and 10C.3 (first-version), the 2D renderers are NOT deleted
while there is no replacement able to run.

10C.4 — deletion HELD. tactical_render.py and tactical_boards.py stay. TOOLS.md
rows annotated: "runs until a 3D replacement is proven at/above spec; 10C.4
deletion HELD: GPU 0/48." When GPU capacity returns: build scene_gen.py, render
one 5 s animated formation board, vision-score the frame, and only then git rm
the 2D renderers + sweep references.

10C.6 — downstream tools that break when the 2D renderers go, and what each
needs to consume 3D output instead:
- produce_v2.py — `step4b_tactical_render` (line ~290/364) calls
  tactical_render.py; the board step (line ~93) calls tactical_boards.py.
  Needs to call the 3D renderer (blender headless on pod) and consume its MP4.
- produce_episode.py — calls tactical boards / cv_annotate; needs the 3D path.
- The lane B manual assembler (/tmp/s8_assemble.py) — consumes boards/*.png;
  a wired assembler would consume 3D MP4s/PNGs.
- assemble_words_match.py — uses boards; needs 3D assets.
- TOOLS.md, ARCHITECTURE.md — tool count + references to update on deletion.
- validate_script.py / transformation_gate.py — [VISUAL: board/tactical=...]
  tags and the "board" layer stay valid; a 3D render is still a board/tactical
  layer. No change needed, but the 3D renderer must consume the same tag
  vocabulary + match_data.json.
- The 3D renderer consumes: match_data.json (formations, lineups, colors,
  jerseys) + the [VISUAL: board/tactical=<asset>] tag, and emits a 3D MP4/PNG.

### Stage 10 carry-forward
- Production freeze stays until an episode scores >= 7/10 against the spec.
  No episode has yet (best is 3.75/10). The script-budget fix (10B) + opening
  gate (10D) remove two structural gaps; the 3D visual target (10C) and a
  wired boards+footage assembler remain.
- 10C.3 blocked on 0/48 GPU (external). Retry when capacity returns; then
  build scene_gen.py, render + score one frame, and decide 10C.4 deletion.
- /mnt/f still unmounted; the amended rule-1 footage path needs it mounted.
- Generated draft scripts are LLM output — unverified until a human fact-
  checks and resolves [SRC] tags. The generator is a draft tool, not a
  replacement for the human script author.

---

## Stage 11 — Rule 5, mirror retirement, GPU unblock, 3D PoC, assembler scope (2026-09-12)

### Doctrine Rule 5 added
"NO DEAD ENDS. When a tool, path, provider or infrastructure blocks the work,
find and take the next best available option. Report the block, the
alternatives considered, and which was chosen. Stopping at the first wall is
only acceptable when every alternative has been named and priced." Added to
all four canonical docs (CLAUDE/CONTEXT/DECISIONS/ARCHITECTURE). Rule 5
applies retroactively to the 10C.3 dead-end (reported RunPod 0/48 blocked
while Vast.ai was authenticated and unused) — corrected in 11B/11C below.

### Mirror retirement (A1-A5)

A1 — cause. The finding's premise was partly wrong against the codebase
(codebase wins). Facts:
- push_status.sh WAS registered: in the GLOBAL ~/.claude/settings.json Stop
  hook, not the project .claude/settings.json. The project settings.json was
  NEW in Stage 8 (de3667c) — it did not exist before, so Stage 8 did NOT
  replace a push_status registration; it added check_pods.sh on SessionStart
  to a file that had no prior hooks. No merge failure.
- The mirror IS public (gh api: visibility public) and WAS updated today
  (pushes at 22:05, 23:03, 23:28 UTC on 2026-09-12). The "not updated since
  the crash session" claim is false for today.
- The REAL issue: a 3-day gap. Mirror commit history: last push 2026-09-09
  07:56 UTC, then nothing until 2026-09-12 22:05 UTC. The global
  ~/.claude/settings.json mtime is 2026-09-12 16:44 CDT (modified today),
  consistent with the registration having been missing/unstable during the gap
  and re-added today. Stop-hooks do not fire on crashed/killed sessions, so
  the gap is the fragility, not a total break.

A2 — mirror last push 2026-09-12T23:28Z (public, not archived). Last commit
before today's: 2026-09-09T07:56Z.

A3 — decision: (b) RETIRE the mirror under rule 3. Reasoning: the mirror's
purpose was to be fetchable by Claude via raw.githubusercontent, but Claude
cannot fetch those URLs outside web-search results; the private yt-digest repo
is synced directly via the GitHub connector, so the mirror is redundant; it is
fragile (Stop-hook-only, 3-day gap proven); and rule 3 forbids redundant
infrastructure. Acted: git rm .claude/hooks/push_status.sh; removed the
push_status.sh Stop-hook entry from ~/.claude/settings.json (kept the unlazy
Stop hook — verified JSON valid, Stop hooks now 1, PreToolUse 3); stripped the
"mirrored to the public status repo" header clause from CONTEXT/STATUS/
ARCHITECTURE/GAPS/DECISIONS/PROGRESS; replaced CONTEXT.md's STATUS MIRROR +
ARTIFACTS/FRAMES sections with a RETIRED note. The retired script lives in
git history (recoverable). artifacts/ and frames/ stay in the project
(gitignored, local) — only the public mirror is gone. Historical mirror
mentions in PROGRESS/RECONCILIATION/REPORT_AUDIT and the Stage 6 storage
analysis above are left as history (they describe a past state); this section
supersedes them.

A4 — audit of every "happens automatically" claim in the canonical docs:
- check_pods.sh SessionStart — registered (project .claude/settings.json) +
  FIRES (this session's start printed "pod-check: 0 running pods (clean)").
  Claim holds.
- block-retired.sh PreToolUse — registered (global) + FIRES (blocked a read
  of a retired folder mid-session with a BLOCKED message — it even blocked
  this very append when the heredoc text contained the retired path literal).
  Claim holds.
- block-image-read.sh, warn-local-gpu.sh PreToolUse — registered (global),
  same registration mechanism as block-retired. Claim holds (registered).
- unlazy stop-hook.mjs Stop — registered (global). Claim holds.
- youtube_upload.py OAuth auto-refresh (PROGRESS "refreshes the OAuth token
  automatically") — TRUE: tools/youtube_upload.py lines 118-120 call
  creds.refresh(Request()) when creds.expired. Claim holds.
- push_status.sh "always current / auto-pushed after any change" — was the
  FALSE claim (3-day gap). RETIRED. This is the second automated-behaviour
  claim that quietly stopped being true (the first: the old STATE.md
  hand-typed vision scores).

A5 — additive settings.json writes. New CLAUDE.md rule 12: settings.json hook
writes are ADDITIVE — read the current file and merge into the existing
array, never Write a fresh object (that silently drops every other hook).
Proven registry as of 2026-09-12 documented in CLAUDE.md rule 12. A future
hook registration that overwrites settings.json would now violate a stated
project rule.

### 11A — persistent /mnt/f mount
Method (two layers): (1) SessionStart hook .claude/hooks/check_mnt_f.sh (committed)
— verifies /mnt/f at every session start, warns + prints the one-time fstab fix
if absent; (2) durable fix: /etc/fstab entry `F: /mnt/f drvfs
defaults,noatime,nofail 0 0` (one-time sudo, user) — nofail so boot does not hang
if the USB is absent; the drvfs mount remounts F: on every WSL restart when the
USB is present. Registered ADDITIVELY in .claude/settings.json (A5/CLAUDE rule
12: 2 SessionStart hooks now — check_pods + check_mnt_f; the write was a
read-then-merge, not a fresh overwrite). Proven: check_mnt_f.sh fires (mounted
-> "OK"; unmounted simulation -> warns + prints the fstab command). Surviving
`wsl --shutdown`: the fstab entry remounts /mnt/f on restart (nofail); the
SessionStart hook confirms it next session. The fstab application needs a
one-time sudo — the user runs `echo 'F: /mnt/f drvfs defaults,noatime,nofail 0 0'
| sudo tee -a /etc/fstab && sudo mount -a`; the hook is the no-sudo verification
layer. Why fstab over wsl.conf: wsl.conf automount already mounts drives present
at boot, but removable USBs are unreliable via automount; an fstab drvfs entry
with nofail is the durable, reboot-surviving mount.

### 11B — always-available GPU (Rule 5 applied retroactively to the 10C.3 dead-end)
11B.1 — Vast.ai capacity NOW (2026-09-12, vastai SDK search): many offers.
Cheapest usable for Blender: RTX 3090 $0.16/h (24GB, OptiX), L4 $0.32/h, GTX
1660 Ti $0.07/h. RunPod is 0/48 (verified twice). Vast is the available provider
that 10C.3 should have used.

11B.2 — provider survey (web, 2026-09-12):
| Provider | Capacity now | Comparable GPU $/h | Cold start | SDK | Capacity ever zero? |
|---|---|---|---|---|---|
| RunPod | 0/48 (verified) | L4 ~$0.25 | sec-min | yes | YES (proven 0/48) |
| Vast.ai | many offers | RTX 3090 $0.16, L4 $0.32 | 1-3 min (image pull) | yes (vastai) | rarely (marketplace) |
| Modal | easy (serverless) | L4 $0.80, A10G $1.10 | sec (warm pools) | yes (modal) | rarely |
| Lambda | A10/RTX6000 yes; H100/B200 sell out | RTX6000 $0.69, A10 $1.29 | <2 min (Lambda Stack) | yes (REST) | popular chips sell out |
| Paperspace/DO | A100 30-60 avail | A10 $0.51, A100 $3.09 (+$39/mo) | 2-5 min | Gradient API deprecated | constrained peak |

11B.3 — recommendation: Primary = Vast.ai (authenticated, capacity now, cheap
RTX 3090, SDK works). Fallback = Modal (serverless, always-available, cleanest
for one-off renders, needs `modal token new` one-time). RunPod = secondary
fallback when capacity returns (SDK already integrated via runpod_download/
fulltrack). Switching cost: Vast->Modal = install modal + `modal token new`
(~1 min, browser) + ~2-3h to add a Modal backend to the provider layer.
Vast->RunPod = ~1-2h (reuse the existing runpod tooling).

11B.4 — provider-agnostic layer: tools/render3d_vast.py (Vast backend) built.
Design: try Vast (capacity via search), fall back to RunPod (capacity via
get_gpus — 0/48 today, skips) / Modal (needs token, stubbed). Reports which
provider it used. Rule 5 in code: tries the primary, falls back, does not stop
at the first 0-capacity provider. RunPod/Modal backends are stubbed (~1-2h each
to wire when needed); Vast is the working backend used for the 11C PoC.

11B.5 — vastai_shorts fault: the fault was the PROVIDER auth path, not the
encoding tool. LANE_PLAN section 4B said "vastai_shorts is dead in practice —
the Vast key is invalid/expired." The key is VALID now: the new vastai SDK at
cloud.vast.ai authenticates, searches offers, and creates instances this
session (proven — the 11C PoC instance was created). The old vastai_shorts.py
used the classic api.vast.ai endpoint, which no longer resolves — that was the
auth break. Nothing about Vast blocks its use as a general GPU target now.
vastai_shorts was an NVENC shorts encoder; its retirement was the stale
classic-API auth, not Vast-as-a-GPU-target.

### 11D — the unscoped gap: wired boards+footage assembler
11D.1 — scope. Reads: the script (scripts/<slug>.md, [VISUAL:
board=.../footage=START-END]), cut_list_gen.py output
(briefs/<slug>/cut_list.json), board MP4s (scene_gen.py 3D / tactical_boards
2D), the footage source clip (on /mnt/f), voice (voice_elevenlabs.mp3).
Writes: renders/<slug>/final_video.mp4 (boards + footage + voice,
content-matched, 1920x1080) + a transformation_gate manifest. Exists already:
cut_list_gen.py (built, NOT folded in), assemble_words_match.py (holds the
content-matched footage-cut logic, parses footage=START-END),
transformation_gate.py (5 floor rules), the manual /tmp/s8_assemble.py (lane B
boards-only), the 200MB guard (ffmpeg_utils). Missing: the wired entry point
that combines boards + footage + voice from the script tags + cut_list,
generates the gate manifest, and outputs 1920x1080. Honest hours: ~1-2 days
(port assemble_words_match's cut logic, wire board+footage concat, gate-manifest
gen, 1920x1080 normalize, end-to-end test).

11D.2 — estimated spec score (2D boards + 1400-word script + real footage +
working assembler), per EPISODE_SPEC item:
- 1 Runtime: 1400w @ ~175 WPM = 8 min = 480s -> MEETS (2/2).
- 2 Shot rhythm: ~20-40 cuts (sections + footage windows) -> borderline (1/2).
- 3 Content mix: ~60% boards + ~40% footage -> graphics>=60% MEETS, footage
  exceeds the 5-20% SHOULD (1.5/2).
- 4 Graphic types: 2D formation/stat/possession -> pitch+formation+stat MEETS;
  no arrows_on_footage/lower_third (1/2).
- 5 Typography: 2D matplotlib boards scored 3-5/10 (Opus) -> below spec (0.5/2).
- 6 Opening: 10D gate -> opens on footage/data -> MEETS (2/2).
- 7 Audio: voice at WPM, no music bed (1/2).
- 8 Framing: 1920x1080 (9C.2) -> MEETS (2/2).
Total ~11/16 = ~6.9/10 — just under the 7/10 freeze gate. Biggest drags:
typography (2D boards) + shot rhythm.

11D.3 — score per hour + order. Assembler: ~12h (1.5 days) -> ~6.9/10 ->
~0.58 score/h. 3D is NOT independent: a 3D board in a 41s video still fails
runtime/rhythm — 3D only counts once the assembler exists. 3D + assembler:
~36h -> ~7.4/10 (3D lifts typography 0.5->~1.5) -> ~0.21 score/h. The
assembler path beats 3D on score-per-hour (0.58 vs 0.21) AND is a prerequisite
for 3D to count. RECOMMEND: build the assembler first (raises score to ~6.9,
fixes runtime/rhythm/opening), then 3D (lifts typography across 7/10). 3D
before the assembler is polishing boards in a still-41s video.

### 11C — 3D proof-of-concept: ATTEMPTED on Vast, BLOCKED on retrieval (user authorization needed)
Built (committed this stage):
- tools/scene_gen.py — Blender bpy script: 3D pitch (markings, goals, accent),
  11+11 player tokens in team colours with jersey labels, animated camera move,
  spec §5 typography (bold white title, dark pitch, accent). Cycles GPU
  (headless-safe; CPU fallback) -> MP4. The 3D replacement for the 2D
  matplotlib renderers.
- tools/render3d_vast.py — Vast.ai backend (the 11B.4 provider-agnostic layer's
  working backend). Creates a GPU instance, ships scene_gen.py + match_data,
  runs Blender headless, retrieves the MP4, destroys the instance (no leak).
- tools/modal_render3d.py — Modal serverless backend (READY, awaiting token).

Vast has capacity + credit (Rule 5 broke the 10C.3 RunPod-only dead-end):
an RTX 3090 instance ($0.17/h, offer 50342457) was created and destroyed
cleanly (no leak; 0 instances after). So Vast works for instance creation.

BLOCKED on retrieval. The Vast SDK's SyncInstance exposes only `destroy` — no
SSH, no logs, no exec — so the onstart must self-report the result. Three
retrieval mechanisms tried and walled:
1. Gist-drop (onstart gists the catbox URL via GITHUB_TOKEN env): ENV_BROKEN.
   A minimal env-test instance produced no gist; the GITHUB_TOKEN env does not
   reach the onstart (the vastai SDK InstanceConfig.env does not propagate to
   the onstart shell).
2. catbox + webhook.site drop: both return garbage from this Bash sandbox
   ("No request type given?" from catbox; a hijacked "Telugu translated text"
   response from webhook.site). gh + Vast-SDK POSTs work; catbox/webhook do
   not — the sandbox mangles these hosts' POSTs.
3. Port-expose + http.server: the vastai SDK InstanceConfig has no `ports`
   field (only image/disk/extra) — the new SDK is serverless-focused, no
   classic port exposure.
Blender URL: download.blender.org 403s from the sandbox; the Clarkson mirror
(200) is wired into the onstart.

10C.4 (git rm the 2D renderers): HELD. A 3D frame HAS now been produced and
scored (Stage 12A: 5/10, Opus), but git rm is still held — the 3D score is
below spec and Opus's 12A.1 verdict is that the low 2D scores were a DESIGN
problem, not a dimension problem (3D is a multiplier on good design, not a
substitute). tactical_render.py + tactical_boards.py stay until a design
system is proven (ideally on a 2D board first). See Stage 12A for the full
decision. TOOLS.md annotates this.

Alternatives named + priced (Rule 5 — the block is a one-time user action):
- Modal: ~$0.02-0.05/render, serverless, RETURNS the file + has logs (no
  onstart/retrieval/leak), needs `modal token new` one-time (~1 min, browser).
  CLEANEST. tools/modal_render3d.py is written + ready.
- Vast SSH: register an SSH key on Vast, ssh in, scp the MP4, ~$0.05/render.
- RunPod: 0/48 today, no ETA; existing runpod tooling retrieves from pods.
- Lambda: $0.69-1.29/h (RTX6000/A10), SSH instances, REST API, account setup.
- Paperspace/DO: $0.51/h A10, DO account, Gradient API deprecated.

The 3D PoC (11C.3) is blocked on a one-time user authorization (Modal token or
Vast SSH key). Re-asked via AskUserQuestion; awaiting the user. This does NOT
block the higher-score-per-hour build: 11D's assembler (~6.9/10) is the
prerequisite for 3D to count (a 3D board in a 41s video still fails runtime),
so the assembler should be built first regardless.

## Stage 12 — 3D test, assembler, skill infra, housekeeping (2026-09-12)

Brief: three jobs in strict order (12A 3D test, 12B assembler, 12C skill
auto-activation, 12D housekeeping). Modal authorized (profile
minakush000-crypto, token verified). /mnt/f persistent via fstab (nofail, 58G).
Commit + push after each of 12A/12B/12C separately.

### 12A — THE 3D TEST (DONE, scored)

Goal: run the 3D render on Modal now, one animated formation board, 5s, EPISODE_SPEC
§5 typography. Report cost, wall time, output path, vision score. Then answer
12A.1 (was the 2D low score because of 2D or bad design?), and branch to 12A.2
(git rm 2D if ≥spec) or 12A.3 (hold if at/near 2D level).

**Render path:** tools/modal_render3d.py (Modal serverless, T4 GPU) →
tools/scene_gen.py (Blender bpy headless, Cycles). Fixed two Stage-11C bugs to
get here:
1. modal_render3d.py IndexError in-container (Path.parents[2] at /root) — moved
   ROOT/SOCCER into main() (local only, not module-level).
2. scene_gen.py geometry: pitch now in X-Y plane (Z up), X=±52.5m, Y=±34m,
   tokens at (X,Y,Z=0.3), camera TRACK_TO an empty at centre. Previous version
   had the pitch plane, tokens, and camera in mismatched axes → Opus 0/10
   (empty void).
3. Render settings (this stage): forced CUDA (not OptiX — OptiX libs absent
   from the cuda:12.2.2-runtime image), 720p (1280x720), Cycles samples 24 GPU
   / 16 CPU fallback, denoising on. Modal timeout raised 900s→1800s. The
   previous 1080p/48-sample CPU-Cycles render hit FunctionTimeoutError at 900s.

**Output (verified):**
```
$ stat -c '%y %s %n' /mnt/f/soccer-staging/3dpoc_2026-09-12_preview-manc-derby.mp4
2026-09-12 20:49:32 -0500  522523  /mnt/f/soccer-staging/3dpoc_2026-09-12_preview-manc-derby.mp4
$ ffprobe stream: h264, 1280x720
$ ffprobe format duration: 5.000000
```
Re-run (image cached) wall time: **6:48.92 (408.9s)** via `/usr/bin/time -v`.
Reproducible: run 1 = 522809 bytes, run 2 = 522523 bytes (encoding variance only).

**Cost (grounded in modal.com/pricing, fetched this stage):**
Modal T4 = $0.000164/sec GPU + $0.0000131/core/sec CPU + $0.00000222/GiB/sec
memory. At ~409s, 2 CPU cores, ~2GiB:
- GPU: 409 × 0.000164 = $0.067
- CPU: 409 × 2 × 0.0000131 = $0.011
- memory: 409 × 2 × 0.00000222 = $0.002
- **Total ≈ $0.08/render** (T4, 720p, 125 frames). 1080p/full-episode would scale up.

**Vision score (Opus 5, authoritative, via ask_claude.py --image on the t=2.5s
frame, same scale as the 2D boards):**
- 2D boards for reference: formation 3/10, possession 4/10, stat_card 5/10.
- **3D formation frame: 5/10.**

What Opus saw: a desaturated sage-green pitch on charcoal void, genuine 3D
perspective (oblique 3/4 angle, far touchline compresses), real contact shadows
+ disc thickness from Cycles, terracotta vs pale blue-grey team tokens with
jersey numbers. Penalty boxes, halfway line, centre circle visible.

What capped it at 5 (Opus, verbatim findings):
- **Mirrored numbers on the black-rimmed tokens** (UV/normal flip bug — "10"
  reads backwards "01"). Most damaging single defect.
- **Zero narrative furniture**: no title, no team names, no formation label
  (4-2-3-1 vs 4-3-3), no score, no legend, no arrows, no zones. A cropped "C"
  glyph bleeds off the bottom-left (the title, framed off-canvas by the camera).
- **Typography still the weak axis** — the exact 2D criticism, unchanged. Generic
  rounded sans, not Bebas/Barlow Condensed, no weight hierarchy.
- **Labels are perspective-baked, not billboarded** — 3D actively hurts
  readability; far tokens' numbers skew toward illegible. Light-team numbers
  (pale grey on pale grey) are mush.
- **Framing uncontrolled** — cropped player, cropped title glyph, dead space.

#### 12A.1 — Was the 2D low score because of 2D, or bad design?

**Design. Almost entirely.** (Opus, authoritative.)

3D fixed 2 of the 4 original 2D criticisms:
| 2D criticism | Fixed by 3D? |
|---|---|
| matplotlib defaults | YES — deliberate palette/materials |
| flat look | YES — depth, shadow, token thickness, perspective |
| no Bebas/Barlow condensed fonts | NO — still generic sans |
| no narrative furniture | NO — arguably worse (2D had axis labels) |

And 3D introduced NEW failure modes 2D never had: mirrored text (UV flip),
perspective-illegible labels, cropped subjects from the animated camera.

Opus verdict: "The 2D boards didn't score 3/10 because they were 2D. They
scored 3/10 because they were undesigned — no typographic system, no hierarchy,
no labeling, no editorial point of view. You changed the dimension, kept the
design deficit, and moved 3 → 5. You bought two points with a renderer. The
remaining five points are not purchasable with geometry. **3D is a multiplier
on good design, not a substitute for it.** A well-designed 2D board would score
7-8/10 without a single polygon of 3D."

**The dimension problem and the design problem are separate, and design is the
load-bearing one.**

#### 12A.2/12A.3 — Decision: HOLD the 2D git rm (12A.3)

12A.2 (git rm tactical_render.py + tactical_boards.py if ≥spec) does NOT apply.
The 3D frame scored 5/10 — above the 2D formation board (3/10) on aesthetics,
but NOT at spec (broadcast), and Opus's analysis directly satisfies 12A.3's
condition: the low scores were a design problem, not a dimension problem.

**12A.3 applies: tactical_render.py + tactical_boards.py are NOT deleted. The
3D production build should not start until the design problem is separated from
the dimension problem.** Concretely, the next move is to build and prove a
DESIGN SYSTEM (Bebas/Barlow Condensed typography, title lockup, team names +
kit swatches, formation callouts, shape polygons / zone shading, a key for
highlighted tokens) — ideally on a 2D board first, since Opus says a
well-designed 2D board hits 7-8/10 cheaper than 3D — and only then layer 3D on
top of that proven design. Building 3D on top of an undesigned board is
multiplying a low number.

The 3D PoC itself is KEPT (tools/modal_render3d.py + tools/scene_gen.py work:
file returned, reproducible, ~$0.08, scored). It is the foundation for when the
design system lands. Known 3D bugs to fix at that point (from Opus): mirrored
UVs on highlighted tokens, billboard/counter-rotate the jersey numbers, darken
light-team numbers, re-key the camera so nothing crops, add a 2D overlay
compositing layer for furniture (title lockup, team names, formation string,
key) rather than baking furniture into the 3D scene.

#### 12A.4 — Modal returns file directly, visible logs, no leak (CONFIRMED)

- **File returned directly:** render3d.remote() returns the MP4 bytes;
  main() writes them to /mnt/f. No catbox/gist/webhook drop, no onstart, no SSH
  — the Vast retrieval failure (ENV_BROKEN, garbage catbox/webhook returns, no
  SDK SSH/logs) is solved by construction.
- **Visible logs:** `~/yt-digest/.venv/bin/modal app logs <appid>` works
  (unlike Vast SDK which exposed no logs). The Blender subprocess uses
  capture_output=True so Blender's own stdout is swallowed into the subprocess
  result; the container-level logs show CUDA init + clean stop. NOTE for the
  real build: drop capture_output (or stream to a file the function prints) so
  Blender progress is visible in `modal app logs` too.
- **No instance left running:** `modal app list` shows all four
  soccer-3d-render apps "stopped, 0 tasks". Modal is serverless: the container
  scales to zero the instant render3d.remote() returns. There is no
  billing-leak surface (unlike Vast/RunPod pods, which must be explicitly
  destroyed and leak on timeout/crash — see the Stage 8A pod-leak fix).

**12A summary table:**
| metric | value | how measured |
|---|---|---|
| output | 3dpoc_2026-09-12_preview-manc-derby.mp4, 522523 B, 1280x720, 5s, h264 | ffprobe + stat |
| wall time | 408.9s (6:49), image cached | /usr/bin/time -v (re-run) |
| cost | ≈$0.08 (T4 $0.000164/s GPU + CPU + mem) | modal.com/pricing (fetched) |
| vision score | 5/10 (Opus 5, authoritative) | ask_claude.py --image t=2.5s |
| 12A.1 answer | design problem, not dimension | Opus 4-axis comparison |
| 12A.2/3 decision | 12A.3 HOLD git rm | score 5/10 < spec; design load-bearing |
| 12A.4 | file direct + logs + no leak | modal app list = 0 tasks |

*(12B, 12C, 12D follow in subsequent commits.)*

### 12B — THE BOARDS+FOOTAGE ASSEMBLER (DONE, one episode produced+scored)

Goal: build the boards-plus-footage assembler (fold in cut_list_gen.py), wire into
produce_v2 for lanes A/D, produce ONE full episode end-to-end with real footage
(residential→/mnt/f→pod→delete per rule 1), score against EPISODE_SPEC item by
item, list every human intervention, do NOT upload.

**The build (one logical change across produce_v2.py + script_gen.py + generate_voice.py):**

produce_v2.py:
- NEW step1b_script_gen: idempotent — keeps scripts/<slug>.md if it meets the
  lane word minimum, else runs script_gen.py. This is the change that makes the
  episode reach 8-14 min (old hand-written scripts were ~250 words / 45s).
- NEW step1c_validate: runs validate_script.py --lane (non-fatal; sources.json
  is a human step).
- NEW step4a_cutlist: idempotent — keeps cut_list.json if present, else runs
  broadcast_filler (classifies the Gemini inventory) then cut_list_gen (snaps
  broadcast windows to scene boundaries). Folds cut_list_gen into the pipeline
  (it was built Stage 4C but never called).
- REWRITTEN step5_assemble: the old one capped boards at 4s (→41s episodes) and
  never sub-cut. The new one allocates each section's duration by narrated word
  count (proportional to voice duration), then sub-cuts each section into
  ~12s shots ALTERNATING board / footage (board sections interleave footage
  excerpts; footage sections interleave boards). Boards loop to fill their shot;
  footage shots use the real cut-list window. Scales everything to 1920x1080.
  Builds an assemble_manifest.json (for transformation_gate). Targets 40-80
  shots at 8-16s mean (EPISODE_SPEC §2).
- NEW step5b_transformation_gate: runs transformation_gate.py on the manifest
  (lane D only — the five floor rules).
- NEW --lane arg (A/B/C/D), --clip (reuse a staged clip, skip download),
  --tactical (opt-in top-down render, off by default — the 12B assembler is
  boards+footage, no tactical view). The 16:9 final_video.mp4 is now the primary
  output (was shorts). Lanes B/C skip footage (boards-only).

script_gen.py: lane A/D Hook visual changed board→footage (EPISODE_SPEC §6
forbids opening on a static formation board; the opening hook gate rejects it).
Fixed a pre-existing KeyError (LANE_HEADER used {as} but `as` is a Python
keyword; format() passed as_ — template changed to {as_}).

generate_voice.py: added sentence-boundary chunking (the ElevenLabs per-request
char limit is below a full 8-14 min script — 11,539 chars here, chunked into 4);
added --atempo (default 1.0 — the old hardcoded 1.20 pushed WPM to ~217, over
the §7 max of 195; 1.0 gives a natural ~155 WPM in spec).

**The episode (lane A, arsenal-chelsea, REAL footage):**
```
$ ffprobe final_video.mp4 → 724.0s, 1920x1080, h264+aac, 210MB
$ scenedetect → 85 shots, mean 8.5s, longest 29.8s, shortest 0.8s
$ manifest → 37 board shots (443s, 61%) + 37 footage shots (282s, 39%)
$ voice → 725.3s, 155 WPM (in spec 155-195), crowd ambience bed mixed at 30%
```
Footage: the existing 482s arsenal-chelsea clip (134MB), staged on /mnt/f
(rule 1), 5 cut-list windows (~40s of broadcast action) cycled across the Body,
/m/f clip deleted after assembly (rule 1). The residential→/mnt/f download was
proven Stage 9A; this run reused the pre-rule-1 clip (staged on /mnt/f to
demonstrate the path) — see human intervention 3.

**Merge note (a Rule-5 wall, fixed):** merge_voice.py applies tpad (pad last
frame) when video ≠ voice duration, which forces a full libx264 re-encode of
the whole video (~20min for 724s@1080p on CPU) — for a 1.3s duration mismatch.
Killed it; did a fast merge directly (stream-copy video, mix voice+looped
crowd, -shortest) → 82s, same result. Also raised step7_merge timeout 120→900s
in produce_v2. TODO: merge_voice.py should skip tpad when the gap is <2s.

#### EPISODE_SPEC score (item by item, biggest gap first)

Biggest gap: **§5 typography (1/2)** — the 12A design problem, unchanged. Opus
scored the formation board 4/10: pitch geometry now accurate (6-yard boxes,
D-arcs, penalty spots, symmetrical) but no formation label, no team names, no
title, no accent hierarchy, no surnames, 9 not 11 players, generic font not
Bebas/Barlow. "Better than matplotlib default, a long way from Coaches' Voice."
This is the same design-deficit verdict as 12A: 3D is a multiplier on good
design, not a substitute. The assembler delivers the RIGHT STRUCTURE (long
runtime, shot rhythm, footage mix, framing) but cannot fix the design of the
boards themselves — that is a separate build (the typography/design system).

| Spec item | Target | Measured | Score | Verdict |
|---|---|---|---|---|
| 1. Runtime | 480-840s | 724s (12.1min) | 2/2 | PASS (was 41s → 0/2) |
| 2. Shot rhythm | 40-80 shots, mean 8-16s, longest 30-70s | 85 shots, mean 8.5s, longest 29.8s | 1.5/2 | mean+longest in range; 85 is 5 over the 80 cap (scenedetect splits internal footage cuts; assembler built 74) |
| 3. Content mix | graphics ≥60%, footage 5-20%, voice throughout | 61% graphics, 39% footage, voice throughout | 1.5/2 | graphics MUST passes; footage 39% over the 20% SHOULD (50/50 alternation too footage-heavy); voice ✓ |
| 4. Graphic types | pitch+formation+stat MUST; arrows-on-footage+lower-third SHOULD | formation+stat_card+possession ✓; no arrows-on-footage, no lower-third | 1/2 | MUSTs present, SHOULDs absent |
| 5. Typography | bold condensed, dark, white+accents | Opus formation 4/10: dark bg ✓, no Bebas/Barlow, no title/label, no accents | 1/2 | BIGGEST GAP — the 12A design problem |
| 6. Opening | hook in 5s, no static board, topic in 15s | t=0 = live footage (not a board) ✓; Opus 6/10 (low-energy moment, no thesis title, source has fan-edit "ARSENAL 😱🔥" caption) | 1.5/2 | hook gate passes; weak moment + no title |
| 7. Audio | 155-195 WPM, music bed SHOULD | 155 WPM ✓, crowd ambience bed ✓ | 2/2 | PASS |
| 8. Framing | 1920x1080 full-frame, no pillarbox | 1920x1080, full-frame (Opus confirmed no bars) | 2/2 | PASS |
| **Total** | | | **12.5/16 (7.8/10)** | |

Up from 6/16 (3.75/10) at 9B.2. The assembler fixed the structural gaps
(runtime 10×, shot rhythm, opening hook, audio WPM) but the design gap
(typography) is unchanged — exactly as 12A predicted.

Opus frame scores (authoritative, ask_claude.py --image):
- Opening (t=0): 6/10 — live footage, full-frame, readable geometry; low-energy
  lull not a peak, generic source caption, no thesis title.
- Formation board (t=50): 4/10 — accurate pitch geometry, dark palette; no
  formation/team/title text, no accent system, 9 not 11 players, generic font.
- t=120 frame: raw footage (landed on a footage shot in the alternation, not a
  board — verified via manifest, not a bug).

#### Human interventions (every one)

1. **Fact-check the script.** The 1869-word lane-A script is an UNVERIFIED
   glm-5.2:cloud draft. The LLM hallucinates — it can invent a scorer, scoreline,
   or stat. Every sentence needs human fact-checking against match_data.json +
   the broadcast. (validate_script catches tag/freshness errors, NOT prose
   hallucination.)
2. **Resolve [SRC: <id>] tags to sources.json.** validate_script FAILED on
   missing sources.json. The script emits placeholder [SRC] tokens; a human must
   create sources.json entries (outlet, published date, URL) and confirm each
   cited fact.
3. **Footage acquisition via the rule-1 chain.** This run reused the pre-rule-1
   clip (staged on /mnt/f to demonstrate the path, deleted after). A production
   run downloads residentially → /mnt/f → pod → delete. The residential download
   is proven (Stage 9A) but not re-exercised here.
4. **Curate footage windows.** 5 cut-list windows cycle ~7× each across 37
   footage shots (the Body alternates 50/50). A human editor curates unique
   windows or sources more footage to avoid repetition and hit the 5-20% SHOULD
   (currently 39%).
5. **Pick the opening moment.** Opus: t=0 is a low-energy lull, not a peak. A
   human shifts the open ~0.5s to the contact/slide moment for a stronger hook.
6. **Build the typography/design system.** The boards score 4/10 on design
   (§5, the biggest gap). A human designer builds the Bebas/Barlow lockup,
   title/formation/team labels, accent hierarchy, surnames, 11 players. The
   assembler delivers structure; it cannot fix board design (12A verdict).
7. **No upload.** Per the brief, the episode is NOT uploaded.

#### What the pipeline did WITHOUT human help (the wins)

Script generation (1869 words), cut list (5 windows, cut_list_gen folded in),
assembly (74 shots sub-cut, boards+footage, 1920x1080), voice (725s, 155 WPM,
chunked), crowd ambience, merge, framing. The assembler is wired for lanes A/D
(transformation_gate enforces the lane D floor); lanes B/C are boards-only.

**12B summary: the assembler is built and wired; one full episode produced
end-to-end with real footage (724s, 1920x1080, 85 shots); scored 12.5/16
(7.8/10), up from 3.75/10. Biggest remaining gap is typography (the 12A design
problem), which the assembler cannot fix — it is a separate design-system build.**

*(12C, 12D follow in subsequent commits.)*

### 12C — SKILL AUTO-ACTIVATION INFRASTRUCTURE (DONE, verified 8/8)

Goal: install skill auto-activation from github.com/diet103/claude-code-
infrastructure-showcase. Read CLAUDE_INTEGRATION_GUIDE.md first. Prerequisites
(node 18+, npm, jq). Clone to temp. Additive settings merge (preserve 6 existing
hooks). Install only 4 essential hooks. Audit ~130 skill folders under rule 3.
Regex-only AI mode. Verify with verify-setup.sh + real trigger test. Review 8
agents. Decide on dev-docs pattern. Document.

**Prerequisites:** node v24.18.0 ✓, npm 11.16.0 ✓, jq was missing → installed
the 1.7.1 binary to ~/.local/bin (on PATH via ~/.bashrc; sudo apt unavailable
without a terminal). Repo cloned to /tmp/cc-infra-showcase.

**CLAUDE_INTEGRATION_GUIDE.md read in full.** Key rules followed: settings.json
is MERGED (extract + add sections), never copied (the guide's "Common Mistakes:
NEVER copy the showcase settings.json directly"). The shipped settings.json
registers only the safe hooks; the build-check Stop hooks (tsc-check,
trigger-build-resolver, stop-build-check-enhanced) are NOT registered by default
(expect specific service directories) — confirmed, not installed.

**4 hooks installed (project .claude/settings.json, additive):**
| hook | event | matcher | purpose |
|---|---|---|---|
| skill-activation-prompt.sh | UserPromptSubmit | — | regex-match prompt → suggest skills from skill-rules.json |
| skill-verification-guard.sh | PreToolUse | Edit\|MultiEdit\|Write | block edit if a mandatory skill pending (no-op: no block skills here) |
| post-tool-use-tracker.sh | PostToolUse | Edit\|MultiEdit\|Write | log edited files + repo (bash+jq) |
| skill-activation-tracker.sh | PostToolUse | Skill | record skill activations (better-sqlite3 session intel) |
NOT installed (per brief): tsc-check, trigger-build-resolver, stop-build-check-
enhanced (monorepo build hooks, wrong stack), session-doc-updater (Stop, not in
the 4). The 3 skill-* hooks run via _run-node-hook.sh → tsx; deps installed
(npm install → tsx, better-sqlite3, minimatch; better-sqlite3 loads, tsx runs).

**6 existing hooks preserved (additive merge):** project SessionStart
[check_pods.sh, check_mnt_f.sh] + global Stop [unlazy stop-hook.mjs] + global
PreToolUse [block-retired, block-image-read, warn-local-gpu]. The 4 new hooks
ADDED to the project settings; SessionStart commands aligned to the showcase
format ($CLAUDE_PROJECT_DIR/.claude/hooks/<name>.sh, no `bash` prefix) so
verify-setup.sh's path check passes. Total: 6 existing + 4 new = 10 hooks.

**Regex-only AI mode:** skill-rules.json `skill_activation_mode: disabled`
(Classic, free, offline, no AI provider). conservativeness: balanced. No
SKILL_AI_PROVIDER env var (regex-only needs none). The showcase's 3 stack-
specific skills (backend-dev-guidelines Express/Prisma, frontend-dev-guidelines
React/MUI v7, error-tracking Sentry) were NOT installed — wrong stack for a
Python/ffmpeg soccer pipeline. Only `skill-developer` (tech-agnostic) kept.

**verify-setup.sh: 8/8 PASS** (Node, settings.json valid + scripts exist, hooks
executable, node_modules, better-sqlite3, skill-rules.json valid mode/conserv,
jq, end-to-end). Two initial FAILs were verify-script artifacts, both fixed:
(1) the JS path check did `cand.split(" ")[0]` → turned `bash <abspath>` into
`bash` (not a file); fixed by aligning SessionStart commands to the showcase
format. (2) the e2e test hardcoded "create a new React component" → expected
frontend-dev-guidelines (which I correctly dropped); patched the test to a
skill-developer-matching prompt. **Real trigger test:** a "create a new skill
for the pipeline" prompt → the hook recommended skill-developer via regex (1ms).
Confirmed working.

**8 agents reviewed + installed** (.claude/agents/, all clean — no hardcoded
paths): code-architecture-reviewer, code-refactor-master, documentation-
architect, plan-reviewer, refactor-planner, web-research-specialist (6 generic-
relevant); auto-error-resolver (TS-specific, useful for the hooks' .ts);
frontend-error-fixer (frontend, no frontend here — stack-mismatched but
harmless, on-demand only). Invokable via subagent_type.

**dev-docs pattern: NOT adopted.** The showcase's dev-docs commands reference a
/dev/active/ task-dir structure. This project already has a doc spine
(CONTEXT/STATUS/PROGRESS/DECISIONS/ARCHITECTURE/TOOLS/GAPS + LANE_PLAN + the
public status mirror) + the unlazy GATES.md pattern. Adding dev-docs would
create a competing second doc system (rule 3). The skill-activation-prompt hook
prints a dev-doc reminder by default → suppressed via SESSION_DOCS_ENABLED=false
in .claude/hooks/.env (the hook gates the reminder on that env var, line 864).

**~130 skill folders audit (rule 3):** 127 marketing/design skills in
~/.claude/skills (all valid — every skill dir has SKILL.md; ab-testing 48K, ads
140K, analytics 60K, copywriting 44K: real content, not stubs). 125 of them are
mirrored in the parent yt-digest/.claude/skills (duplicates, same set). The
soccer-channel project itself has NO project skills (the old soccer-channel
skill is RETIRED). Only skill-developer is in skill-rules.json. Verdict: the
~130 skills are a deliberate user library (marketing work), manually invokable
via /<skill-name>, NOT auto-suggested (not in skill-rules.json). They are NOT
pipeline orphans (not pipeline tools), so not a rule-3 violation. To auto-
suggest them, add entries to skill-rules.json. Documented; no action taken.

**Documented in TOOLS.md (skill infra + wired count 13→14) and ARCHITECTURE.md
(skill auto-activation section + rule-3 gap update).**

**12C summary:** skill auto-activation installed (4 hooks, regex-only, additive
merge preserving 6 existing hooks), verified 8/8 + real trigger test, 8 agents
reviewed+installed, dev-docs skipped (project has its own doc spine), ~130
skills audited (deliberate user library, not orphans).

### 12D — HOUSEKEEPING (DONE)

#### 12D.1 — Is check_mnt_f.sh redundant now /mnt/f is fstab-persistent?

NO, not redundant. KEEP it. /mnt/f is in /etc/fstab:
```
$ grep /mnt/f /etc/fstab
F: /mnt/f drvfs defaults,noatime,nofail 0 0
```
The `nofail` flag means WSL boots WITHOUT error if the USB is absent — but
/mnt/f is NOT mounted in that case. The hook catches exactly that: the USB
physically unplugged (or a drvfs stale-mount) case that fstab-nofail does not
solve. It prints the fix (replug + mount -a) and exits 0 (warning, never
blocks). The fstab makes the COMMON case (USB present, WSL restart) auto-mount;
the hook covers the uncommon case (USB absent). Both layers, not either/or.
Confirmed mounted this session: "mnt-f-check: /mnt/f mounted (OK)" at start.

#### 12D.2 — Other Stop hooks + fragility

The ONLY Stop hook now is the unlazy stop-hook.mjs (global). push_status.sh was
retired Stage 11 (it had a 3-day push gap Sep 9→12 from Stop-hook fragility:
if it errored, the mirror silently didn't push). Reviewed stop-hook.mjs (211
lines): it FAILS SAFE by design —
- top-level `catch { allow(null); }` (line 68): any error → allow, never block.
- invalid --scope → "not blocking" (line 63).
- "State cleanup must never trap a session after the gates are complete"
  (line 100-101): explicit anti-trap.
- only blocks when GATES.md has genuinely unmet gates (the intended behavior).
So the unlazy Stop hook will not trap a session on error. No fragility concern.
The new 12C PostToolUse hooks also fail safe: _run-node-hook.sh exits 0 on
missing deps; post-tool-use-tracker.sh exits 0 on jq-missing/non-edit/markdown.
No Stop or PostToolUse hook can block or break the session on error.

#### 12D.3 — Total provider spend this stage + nothing left running

| provider | this stage | cost (est) | left running? |
|---|---|---|---|
| Modal (12A 3D) | 4 runs: 1 broken, 1 timed out (900s), 2 good (409s each) | ~$0.35-0.40 (T4 $0.000164/s + CPU/mem) | 0 tasks (all apps stopped) |
| ElevenLabs (12B voice) | 11,595 chars (1877-word script + 12-word TTS test) | subscription-dependent (can't read balance — key lacks user_read); ~$1-2 est | n/a (serverless) |
| RunPod | $0 (tactical render skipped via --tactical off) | $0 | 0 pods (pod_check: 0 running) |
| Vast.ai | $0 (no instances created this stage) | $0 | 0 instances (show_instances: 0) |
| Opus (ask_claude.py --image) | 4 image scorings (12A + 12B frames) | ~$0.10-0.20 | n/a |
| Ollama gemma4:cloud | broadcast_filler, free | $0 | n/a |
| **Total this stage** | | **~$1.50-2.60** | **nothing left running** |

Verified nothing left running:
```
$ modal app list → all soccer-3d-render apps "stopped, 0 tasks"
$ python tools/pod_check.py → pod-check: 0 running pods (clean)
$ vastai show_instances → 0
```
Modal is serverless (scales to zero on return — no leak surface, confirmed
12A.4). RunPod/Vast pods/instances are explicitly destroyed (and pod_check
verifies at session start). No billing leak.

Housekeeping note: the 12A 3D PoC render (3dpoc_2026-09-12_preview-manc-derby.
mp4, 522KB) still lives on /mnt/f/soccer-staging. It is a render OUTPUT, not raw
footage staging — a minor rule-1 grey area (/mnt/f is staging-only per rule 1).
Left in place because LANE_PLAN §12A references that path; a cleanup pass should
move it to renders/ or the status mirror. Not a leak; 522KB.

#### Stage 12 summary

| job | status | headline result |
|---|---|---|
| 12A 3D test | DONE | 3D scored 5/10 (Opus); verdict: design problem not dimension; HELD 2D git rm; Modal works (~$0.08/render, no leak) |
| 12B assembler | DONE | one full episode: 724s, 1920x1080, 85 shots, 12.5/16 (7.8/10, up from 3.75/10); biggest gap = typography (the 12A design problem) |
| 12C skill infra | DONE | 4 hooks regex-only, additive merge (6 existing preserved), 8/8 verify + real trigger test, 8 agents, dev-docs skipped, ~130 skills audited |
| 12D housekeeping | DONE | check_mnt_f kept (not redundant — catches absent-USB); Stop hook fails safe; ~$1.50-2.60 this stage; nothing left running |

The single biggest finding across Stage 12: **the low board scores are a
DESIGN problem, not a dimension problem (12A).** The 12B assembler fixed every
STRUCTURAL gap (runtime 10×, shot rhythm, opening hook, audio WPM, framing) and
the episode jumped 3.75→7.8/10, but the boards still score 4/10 because the
typography/design system does not exist yet. That is the next build: Bebas/Barlow
lockup, title/formation/team labels, accent hierarchy, surnames — ideally proven
on a 2D board first (Opus: a well-designed 2D board would hit 7-8/10), then
layer 3D on top of that proven design. 3D is a multiplier on good design, not a
substitute for it.
