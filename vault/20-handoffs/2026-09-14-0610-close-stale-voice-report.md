# Close the stale-voice hole, settle two open questions — report

**Report date:** 2026-09-14
**Brief:** ~/claude/30-briefs/2026-09-14-0610-close-stale-voice-and-settle-two-questions.md
**Stamps reported before answering:** STATUS.md 2026-09-13, DECISIONS.md 2026-09-14.
STATUS.md has since been updated to 2026-09-14 (STAMP RULE, see below).

pwd: /home/muads/yt-digest/soccer-channel

## PIECE 1 — THE VOICE

### The failure (verified)
The 2026-09-12_bournemouth-brentford episode scored **5.5/10** (below the 7/10
gate). The single disqualifier was EPISODE_SPEC §10 (narration source-of-truth,
MUST FAIL): the voice track named a fabricated scorer. Root cause:
`voice_elevenlabs.mp3` (mtime 2026-09-13 02:16) was TTS'd from the PRE-FIX
script (1911 spoken words, "Igor Thiago needed just one big chance to score").
The script was fixed at 20:33 (Igor Thiago → Kevin Schade — Schade scored both
Brentford goals, match_data.json: Schade 34', 56'). `.script_verified` was set
at 20:33, AFTER the voice was generated. Every re-run reused the pre-fix voice
because `step6_voice` skipped on `voice_path.exists()` alone. The gate checked
the marker existed, not that the voice matched the current script.

### Re-render + proof (all ●, commands run 2026-09-14)
Old voice moved aside: `voice_elevenlabs.mp3` → `voice_elevenlabs_pre_fix.mp3`.

```
$ # spoken word count of the script used (clean_script_for_tts)
1819
$ # old voice duration
719.078083
$ # new voice duration
794.676458
$ # timestamps in order (stat -c '%y %n')
2026-09-13 20:33:27 scripts/2026-09-12_bournemouth-brentford.md
2026-09-13 20:33:38 renders/2026-09-12_bournemouth-brentford/.script_verified
2026-09-14 01:12:21 renders/2026-09-12_bournemouth-brentford/voice_elevenlabs.mp3
2026-09-14 01:12:21 renders/2026-09-12_bournemouth-brentford/.voice_script_hash
$ # hash match (sha256)
recorded: a4a70df3409a25e38e522c708f0f83293f7ec2d68fa2c56c8b5abb699f16eef7
computed: a4a70df3409a25e38e522c708f0f83293f7ec2d68fa2c56c8b5abb699f16eef7  scripts/2026-09-12_bournemouth-brentford.md
```
The gate (20:33:38) precedes the voice (01:12:21). The recorded hash equals the
current script hash — the voice is provably from the fixed script. ◑ the audio
itself says Schade not Thiago (inferred from hash + script content; no
speech-to-text tool exists to prove the audio content directly).

Note on word count: the brief said "1751 words"; the measured spoken count
(clean_script_for_tts) is **1819**. The "1751" was an approximate prediction
from the prior session. 1819 is the actual. The new voice is LONGER (794.7s)
than the old (719.1s) despite fewer spoken words (1819 < 1911) because ElevenLabs
paced the new TTS slower (137 WPM vs the old 159 WPM). This is a regression
(see re-score §7).

### The skip-logic fix (diff)
`step6_voice` now compares a recorded script hash, not just file existence.
`import hashlib` added. The changed function:

```python
    script_path = SCRIPTS / f"{slug}.md"
    voice_path = render_dir / "voice_elevenlabs.mp3"
    hash_path = render_dir / ".voice_script_hash"

    def _script_hash():
        return hashlib.sha256(script_path.read_bytes()).hexdigest()

    if voice_path.exists():
        if hash_path.exists():
            try:
                recorded = hash_path.read_text(encoding="utf-8").strip()
            except Exception:
                recorded = ""
            current = _script_hash()
            if recorded and recorded == current:
                print(f"  Voice already exists and script unchanged: {voice_path.name}")
                return True
            print("  Voice exists but script hash mismatch; regenerating from current script.")
        else:
            print("  Voice exists but no script hash recorded; regenerating to be safe.")
    cmd = [PYTHON, str(TOOLS / "generate_voice.py"), slug, "--tts-only"]
    ok, _ = run(cmd, "STEP 6: Generate voice (ElevenLabs TTS)", timeout=600)
    if ok:
        hash_path.write_text(_script_hash(), encoding="utf-8")
        print(f"  Recorded script hash -> {hash_path.name}")
    return ok
```
"Voice file exists" is no longer a valid skip condition. Mismatch or missing
hash → regenerate. Parse OK (`ast.parse`).

### Re-assemble
Re-ran `produce_v2.py 2026-09-12_bournemouth-brentford --lane A --clip
clips/video_footage.mp4`. Output:
```
final_video.mp4: 1920x1080, 794.758005s, 77.7MB, 2026-09-14 01:33:35
PIPELINE COMPLETE (lane A), total time 1707.3s. Shorts crop FAILED (non-fatal).
```

### Re-score (same method as the 5.5 run, Gemini authoritative)
9 frames judged by gemini-3.1-pro-preview. Raw responses pasted below. Frames
extracted from the assembled video / board MP4s (boards were reused by step2,
not re-rendered, so visual judgments are directly comparable to the 5.5 run).

**Raw Gemini responses (every frame):**
```
===== opening (2s) =====
(a) Establishing aerial footage with a superimposed team crest.
(b) 3/10
(c) The image successfully fills the entire 1920x1080 frame without margins.
(d) It lacks typography, data, and tactical analysis elements entirely.
===== footage (7.5s) =====
(a) Establishing aerial footage of a stadium.
(b) 3/10
(c) The image successfully utilizes the entire 16:9 frame without leaving empty margins.
(d) It lacks the required dark background, argumentative title, and any actual data visualization elements.
===== formation (3D) =====
(a) 3D tactical pitch board showing player positions.
(b) 4/10
(c) Effectively uses full-frame space with direct labels on player tokens.
(d) Completely lacks a title or explanatory text to state the argument.
===== stat_card =====
(a) Match statistics graphic board.
(b) 4/10
(c) The color scheme effectively uses a dark background with distinct team accent colors.
(d) The bar lengths are completely arbitrary and fail to reflect the actual data values.
===== possession =====
(a) Full-screen data graphic (possession bar chart).
(b) 5/10
(c) Effectively uses a dark background with bright accent colors and direct data labels.
(d) The title is generic rather than argumentative, and the layout wastes significant screen space.
===== xg_flow =====
(a) Cumulative expected goals (xG) line chart.
(b) 7/10
(c) The title clearly states the main argument alongside highly readable direct data labels.
(d) The axis labels are slightly small compared to the dominant title text.
===== shotmap =====
(a) 2D tactical pitch graphic (shot map).
(b) 6/10
(c) The dark background with distinct red and blue accents provides excellent visual contrast.
(d) The title acts more as a descriptive label than a strong, argumentative message.
===== momentum =====
(a) Full-screen momentum area chart showing territorial dominance.
(b) 7/10
(c) Strong color contrast and direct labeling make the data instantly readable.
(d) The subtitle text is quite small and lacks visual hierarchy.
===== avgpositions =====
(a) 2D pitch map displaying average player positions.
(b) 6/10
(c) Direct labeling and contrasting team colors work effectively against the dark background.
(d) The title lacks a specific tactical argument and the side margins waste screen space.
```

**EPISODE_SPEC section-by-section:**
| § | Section | Score | Verdict |
|---|---|---|---|
| 1 | Runtime 8-14min | 2/2 | 13.25 min. PASS |
| 2 | Shot rhythm | 1.5/2 | 51 segments, mean 15.6s (PASS), longest 27s (below 30s min) |
| 3 | Content mix | 2/2 | 87% graphics, 13% footage. PASS |
| 4 | Graphic types | 1.5/2 | All 3 MUSTs + 4 new; SHOULD arrows/lower-third absent |
| 5 | Typography/colour | 1/2 | 4 new 2D boards have Bebas/Barlow; 3D formation + stat_card + possession don't |
| 6 | Opening | 1.5/2 | Footage-led hook (PASS); basic aerial, not dynamic action |
| 7 | Audio | 0.5/2 | **137.3 WPM (BELOW 155-195 MUST)**; ambience present; voice from fixed script |
| 8 | Framing | 2/2 | 1920x1080, no pillarbox. PASS |
| 9 | Source-footage provenance | 1.5/2 | Footage frames clean aerial (no watermark; 5.5 "Google watermark" was a false positive) |
| 10 | Narration source-of-truth | **2/2** | **RESOLVED.** Voice from fixed script (hash proven), no fabricated scorer |
| 11 | Board design properties | 1/2 | 4 new 2D boards have the 4 properties; 3D formation + stat_card + possession don't |
| **Total** | | **16.5/22 (7.5)** | §10 disqualifier RESOLVED |

**Overall: 7.5/10.** §10 (the credibility disqualifier) is resolved. BUT §7
regressed to 137 WPM (below the 155 MUST) — ElevenLabs paced the new TTS
slower. The episode numerically clears the 7.0 gate, but §7 is a MUST
violation. **Recommendation: regenerate the voice with an ElevenLabs speed
setting that hits 155+ WPM before declaring the freeze lifted.**

**Three biggest failures (re-score):**
1. **§7 audio: 137 WPM, below the 155 MUST** (regression from the fix). The
   voice is correct (Schade, hash-proven) but too slow. Fix: ElevenLabs speed
   parameter, or trim silence/pauses.
2. **stat_card 4/10: bar lengths arbitrary, don't reflect data values** (same
   as the 5.5 run — the 2D stat_card from tactical_boards.py lacks the shared
   design layer).
3. **3D formation 4/10: no title/explanatory text, no shared design layer**
   (standard sans-serif, not bold condensed). Plus it is not reliably
   reproducible (Stage 12) and costs ~$0.09/render on Modal.

## PIECE 2 — WHAT PRACTITIONERS ACTUALLY USE

`~/claude/40-lessons/viz-design-findings.md` was checked FIRST (per the standing
rule). It covers the 2D-vs-3D principle per board type (line 364: "3D
perspective distorts 2D positional data... all four recommended boards are
2D... 3D is not the bottleneck; design is"). The research workflow (13 agents:
7 named channels + 5 tooling sources, 1.08M tokens, 0 errors) supplemented the
per-practitioner medium table with sources. Sources beyond the named channels:
mplsoccer toolkit, StatsBomb/Opta visual style, After Effects motion workflows,
broadcast pre-match 3D (Sky/BT/BBC), Tableau football viz — each chosen because
it is a tooling/style source where practitioners describe their own setup or
where the tool's architecture imposes a medium.

### Medium table (board type | medium | tool | reason | source)
| Board type | Medium | Tool | Stated reason | Source |
|---|---|---|---|---|
| formation | 2D animated (top-down) | fmstadio.com | not stated; fmstadio is 2D-only, channel built on it | Football Meta (~400K subs) |
| formation | 2D animated (top-down, birds-eye) | Once Video Analyzer PRO + After Effects | not stated; all sources describe 2D top-down dots | Football Made Simple |
| formation | 2D/3D mixed | After Effects (2D) + Football Manager 3D engine (3D, sponsored) | not stated; 3D is a sponsored partnership, not a stylistic choice. FM 3D used for POSITIONAL analysis throughout, not just pre-match | Coaches' Voice |
| formation | 3D (AR on LED floor) | Unreal Engine, Vizrt, Pixotope, Piero | "Flat screens limit engagement. AR expands it" (Vizrt) — spectacle + pundit interaction | Sky/BT/BBC broadcasters |
| formation | 2D static (top-down) | mplsoccer (Pitch.formation()) | not stated; matplotlib is 2D-only by architecture | mplsoccer + StatsBomb/Opta |
| formation | 2D static/animated | After Effects (Shape Layers, Trim Paths) | 2D top-down eliminates perspective distortion, shows true distances | AE motion workflows |
| formation | 2D static | Tableau (Background Images, X/Y) | not stated; Tableau plots X/Y on axes, not a 3D engine | Tableau practitioners |
| pass-network | 2D static (nodes + weighted edges) | mplsoccer (pitch.lines + scatter) | nodes are mean x,y on a flat 2D pitch; 3D adds occlusion + distance ambiguity | mplsoccer + StatsBomb/Opta |
| pass-network | 2D animated (telestration arrows) | Once Video Analyzer PRO | not stated; no 3D node-and-edge graph found | Football Made Simple |
| shotmap | 2D for aggregate; 3D only for single-shot reconstruction | mplsoccer (2D) / StatsBomb IQ Live (3D freeze-frame) | 2D for aggregate location analysis; 3D only where GK sightline + ball trajectory matter | StatsBomb/Opta |
| xG-flow / momentum | 2D (timeline charts) | matplotlib / AE | time-series is inherently 2D; a Z-axis adds no information | viz-design-findings.md §7 |
| broadcast pre-match | 3D (fly-through) | Cinema 4D / Unreal Engine | spectacle, branding, cinematic flow — the "hype phase," distinct from the "insight phase" (2D) | AE/broadcast workflows |

### Is any positional board rendered in 3D by a credible practitioner?
**YES, one:** Coaches' Voice, in its Football-Manager-powered Masterclass series
(sponsored by Sports Interactive). The FM 3D match engine shows formations,
pressing shapes, and positional shifts in true 3D throughout the video, not
confined to pre-match fly-throughs. Broadcasters (Sky/BT/BBC) render formations
in 3D AR on LED studio floors, but that is studio presentation with pundits
physically interacting, not a produced analysis board. **Every other
practitioner keeps positional boards 2D** (Football Meta/fmstadio, Football
Made Simple, mplsoccer, StatsBomb/Opta, Tableau, After Effects). DK FALCON is
ambiguous (search summaries say "isometric/3D perspective" but cannot be
distinguished from a 2D pitch viewed at an angle without frame observation).
The dominant industry standard is **2D top-down**. 3D for positional analysis
is rare and tied to a sponsorship.

### Inferred principle
**"2D for information, 3D for spectacle."** Positional/tactical boards = 2D
top-down (XY data maps directly to a 2D plane; 2D preserves spatial accuracy,
distances, and structure without perspective distortion or occlusion).
Statistical timelines = 2D (time vs value). Shot maps = 2D for aggregate, 3D
only for single-shot reconstruction. Broadcast pre-match/fly-through = 3D
(spectacle). Studio reveals = 3D AR (engagement). The one exception (Coaches'
Voice + FM) uses a game engine, not a purpose-built 3D analytics tool.

### Settling the contradiction
**NEITHER on-record claim survives.** Both compare UNLIKE board types.
- STATUS.md 2026-09-13: "3D 6/10 vs 2D control 4/10, 3D ahead by 2" = a 3D
  FORMATION board vs a 2D POSSESSION board (same match, different board types).
  The +2 gap reflects formation-vs-possession design quality, not 2D-vs-3D.
- The handoff (2026-09-14-0545): "2D beats 3D because momentum 9/10 beats
  formation 3D 6/10" = a 2D MOMENTUM board vs a 3D FORMATION board (different
  types, different data). Momentum is a timeline (inherently 2D, no 3D version
  makes sense); formation is a pitch scatter (the one board where 3D was tried).

**PLAIN ANSWER: the SAME board with the SAME data has NEVER been rendered both
2D and 3D in this project.** The 3D was always a formation board (scene_gen.py).
The 2D controls were possession, stat_card, momentum, xG flow, shotmap,
average-positions — all different board types. The closest to like-for-like is
2D average-positions (5/10) vs 3D formation (6/10), but even those are
different board types with different pipelines.

**What survives: "DESIGN not DIMENSION" (Stage 12).** Score variance is driven
by design quality (typography, labels, colour discipline, title-as-message,
selective visibility), not dimension. The 2D boards that score highest (xG flow
7-10, momentum 7-9) use the shared design layer (board_design.py, Bebas
Neue/Barlow Condensed). The 3D formation (4-6/10) does NOT use it. The 2D
boards that score low (possession 5, stat_card 4) ALSO lack it. A like-for-like
test (same formation board, same data, one 2D top-down + one 3D, BOTH using the
shared design layer) has never been run and is the only test that would settle
the dimension question.

## PIECE 3 — THE LITMUS HEAD-TO-HEAD

Same episode, same 718.04s footage (`clips/video_footage.mp4`, mtime 2026-09-14
00:17). match_data.json ground truth: 4 goals — Schade 34' (0-1), Kluivert 38'
(1-1), Tavernier 52' (2-1), Schade 56' (2-2).
- Scanner: `scoreboard_scan.py`, gemma4:cloud via vision_analyze.py, **free,
  68.6s**, interval 3s, 240 frames.
- Gemini: `gemini_inventory_test.py`, **gemini-3.1-pro-preview**, 47388 video
  tokens, ~$0.11. (The 2026-09-08 test used the same model family.)

### Timestamps — per-goal offset, misses, phantoms

**Scanner (raw output, ground truth via scoreboard):**
```
duration: 718.04 elapsed: 68.6
CHANGES (4 real + OCR blips that revert in 3-6s):
  60.0s:  BOU 0-0 -> 0-1  (Schade)   REAL — persists to 102s
 102.0s:  BOU 0-1 -> 1-1  (Kluivert) REAL — persists to 117s
 117.0s:  BOU 1-1 -> 2-1  (Tavernier) REAL — persists to 162s
 162.0s:  BOU 2-1 -> 2-2  (Schade)   REAL — final score
 blips: 69/87/93/96/99 (BRIG/BRU OCR noise), 108/111, 156/159 (2-0 OCR error)
```
Scanner: **4/4 found, 0 misses, 0 phantoms.** (The blips are gemma4 OCR errors
that revert within 3-6s; a phantom would be a persistent scoreline change with
no matching goal. None here.)

**Gemini (raw TASK1 output, goals only):**
```
 {"start": 88.0, "end": 101.0, "is_goal": true, "goal_timestamp": 97.0,  "description": "Brentford builds up an attack and scores a goal."}
 {"start": 105.0, "end": 117.0, "is_goal": true, "goal_timestamp": 113.0, "description": "Bournemouth attacks and scores a goal."}
 {"start": 147.0, "end": 158.0, "is_goal": true, "goal_timestamp": 155.0, "description": "Brentford attacks and scores a goal."}
```
Gemini found 3 goals. It called the real 60s goal (passage 45-60) a "shot saved
by the goalkeeper" + "replay of the save" (is_goal: false) — **MISS**. It
invented a goal at 97s where no scoreline changed (0-1 persisted 60→102) —
**PHANTOM**. It missed the 117s goal (2-1 Tavernier) — **MISS**. It placed goal
2 at 113s (+11s vs scanner 102s) and goal 4 at 155s (-7s vs scanner 162s).
Gemini: **2/4 found, 2 misses, 1 phantom.**

**Per-goal comparison (scanner = ground truth for reel-time goal moment):**
| Goal | match_data | scanner | Gemini | offset | verdict |
|---|---|---|---|---|---|
| 1 (0-1 Schade) | 34' | 60s | (called it a saved shot) | — | Gemini MISS |
| phantom | — | — | 97s | — | Gemini PHANTOM (no scoreline change) |
| 2 (1-1 Kluivert) | 38' | 102s | 113s | +11s | found, late |
| 3 (2-1 Tavernier) | 52' | 117s | (not found) | — | Gemini MISS |
| 4 (2-2 Schade) | 56' | 162s | 155s | -7s | found, early |

**Result reproduces the 2026-09-08 finding on a different clip, same model
family: Gemini invents phantoms, misses real goals, and its found-goal offsets
are 7-11s off. Scanner: 4/4, free. Gemini: 2/4, ~$0.11. Gemini does NOT win on
timestamps.**

### Pitch coordinates — Gemini TASK2 vs mechanical (Sofascore avg positions)
Gemini was asked to estimate Brentford's average positions from the footage. It
reported it "extracted directly from the Average Positions graphic shown at
04:55." Raw Gemini output:
```
{"team": "Brentford", "players": [
 {"jersey": 1, "x": 10.0, "y": 50.0}, {"jersey": 2, "x": 35.0, "y": 85.0},
 {"jersey": 18, "x": 30.0, "y": 65.0}, {"jersey": 5, "x": 30.0, "y": 35.0},
 {"jersey": 20, "x": 35.0, "y": 15.0}, {"jersey": 8, "x": 45.0, "y": 70.0},
 {"jersey": 6, "x": 40.0, "y": 50.0}, {"jersey": 27, "x": 45.0, "y": 30.0},
 {"jersey": 19, "x": 60.0, "y": 85.0}, {"jersey": 9, "x": 60.0, "y": 50.0},
 {"jersey": 11, "x": 60.0, "y": 15.0}], "confidence": "high"}
```
Mechanical (Sofascore, in match_data.json away_avg_positions), raw + mirrored:
```
jersey  1 (Kelleher)  raw x=11.0 y=50.3   | mirrored x=89.0 y=49.7
jersey  2 (Hickey)    raw x=46.3 y=22.9   | mirrored x=53.7 y=77.1
jersey 44 (Schuster)  raw x=38.6 y=36.3   | mirrored x=61.4 y=63.7
jersey 20 (Ajer)      raw x=35.2 y=66.7   | mirrored x=64.8 y=33.3
jersey 23 (Lewis-Potter) raw x=49.6 y=88.0 | mirrored x=50.4 y=12.0
jersey 18 (Sangare)   raw x=48.7 y=41.0   | mirrored x=51.3 y=59.0
jersey 27 (Janelt)    raw x=43.7 y=62.7   | mirrored x=56.3 y=37.3
jersey 19 (Anthony)   raw x=53.7 y=23.1   | mirrored x=46.3 y=76.9
jersey  6 (Yarmoliuk) raw x=59.7 y=57.9   | mirrored x=40.3 y=42.1
jersey  7 (Schade)    raw x=61.5 y=73.0   | mirrored x=38.5 y=27.0
jersey  9 (Igor Thiago) raw x=60.7 y=52.2 | mirrored x=39.3 y=47.8
```
**Side by side (matched jerseys, Gemini vs raw Sofascore):**
| jersey | Gemini (x,y) | Sofascore raw (x,y) | delta |
|---|---|---|---|
| 1 (GK) | 10, 50 | 11.0, 50.3 | ~1 (close) |
| 2 | 35, 85 | 46.3, 22.9 | x -11, y +62 (wrong) |
| 18 | 30, 65 | 48.7, 41.0 | x -19, y +24 (wrong) |
| 20 | 35, 15 | 35.2, 66.7 | x ~0, y -52 (wrong) |
| 6 | 40, 50 | 59.7, 57.9 | x -20, y -8 (wrong) |
| 27 | 45, 30 | 43.7, 62.7 | x +1, y -33 (wrong) |
| 19 | 60, 85 | 53.7, 23.1 | x +6, y +62 (wrong) |
| 9 | 60, 50 | 60.7, 52.2 | ~1 (close) |
**Gemini jerseys that do not exist in the lineup: 5, 8, 11** (real: 44, 23, 7).

**Verdict: Gemini cannot produce usable pitch coordinates.** It got only the GK
and striker roughly right (the two positions anyone can guess) and used 3
nonexistent jersey numbers. The board shows away MIRRORED (100-x), but Gemini
reported RAW coordinates — it did not actually read the dots. It hallucinated a
plausible "football-shaped" 4-2-3-1 from its own prior knowledge and labelled it
"high confidence." This is the part never tested before; the result is
decisively negative.

### Cut decisions — Gemini TASK3 vs mechanical (cut_list_gen)
Gemini (raw TASK3):
```
{"goal": "Mbeumo", "segment_start": 88.0, "segment_end": 101.0,
 "cuts": [{"include": true, "start": 88.0, "end": 97.0, "reason": "build-up"},
          {"include": true, "start": 97.0, "end": 101.0, "reason": "celebration"}]}
```
- **Wrong scorer:** "Mbeumo" — the real scorer is Schade. Mbeumo is not in this
  match's Brentford lineup (match_data.json: the forwards are Schade, Igor
  Thiago, Anthony, Yarmoliuk).
- **Wrong segment:** Gemini picked 88-101s (its phantom goal at 97s), not the
  real first goal at 60s.
- **No editorial discrimination:** the cut list just includes the whole segment
  (no replay exclusion, no filler filtering, no broadcast/filler classification).

Mechanical (cut_list_gen, the windows actually used in the episode):
```
[0-15], [258-285], [312-339], [459-486], [501-528], [780-792]  (source_window)
```
These are ±20s relay-verified windows around scoreboard changes, with
broadcast_filler classification (broadcast = {shot, build-up} non-replay;
filler = {non-action, replay}). The mechanical path excludes replays and
non-action; Gemini's does not.

**Verdict: Gemini's cut decision is inferior — wrong scorer, wrong segment, no
filtering. The mechanical tools win.**

### Delegation verdict
Delegate to Gemini ONLY where it wins on the pasted numbers. It wins NOWHERE
here: timestamps (scanner 4/4 free vs Gemini 2/4 + 1 phantom, paid),
coordinates (Gemini hallucinates), cut decisions (Gemini wrong scorer, no
filtering). The mechanical tools (scanner + Sofascore data + cut_list_gen) win
on every mechanical output. Gemini's demonstrated value is CONTENT
CLASSIFICATION (what kind of passage: build-up, replay, crowd), not
WHEN/WHERE/WHAT-NUMBERS. The 2026-09-08 finding holds on a second clip.

## COST
- Voice re-render: ElevenLabs TTS (1819 words) — included in the existing
  ElevenLabs subscription (not separately metered here).
- Gemini litmus: gemini-3.1-pro-preview, 59844 tokens (~$0.11).
- Gemini re-score: 9 frame judgments, gemini-3.1-pro-preview (~$0.05).
- Research workflow: 1.08M subagent tokens (glm-5.2:cloud via Ollama, local, no
  marginal API cost).
- produce_v2 re-assembly: 28 min local CPU (N150, no GPU, no cloud cost).
- No B2 archive needed this run (no new durable artifacts; frames are
  regenerable, gitignored).

## What this changes
- The fabricated-scorer hole is closed: the voice is from the fixed script
  (hash-proven), and the skip logic that let it happen is fixed (hash gate).
- The episode moves from 5.5 (§10 MUST FAIL, disqualified) to 7.5 (§10
  resolved), but §7 WPM regressed to 137 (below 155) — fix before lifting the
  freeze.
- The 2D-vs-3D contradiction is settled: neither on-record comparison was
  like-for-like; "DESIGN not DIMENSION" survives; a like-for-like test was
  never run. The industry standard for positional boards is 2D top-down (one
  sponsored exception: Coaches' Voice + Football Manager).
- The Gemini litmus is decisive: Gemini loses to the mechanical tools on every
  mechanical output (timestamps, coordinates, cut decisions). Delegate to
  Gemini only for content classification.

## Verification marks
● verified — voice regenerated from fixed script (sha256 hash match: a4a70df3...)
● verified — new voice 794.676s, old 719.078s (ffprobe)
● verified — gate precedes voice (stat timestamps: 20:33:38 < 01:12:21)
● verified — scanner 4/4 goals, 0 phantoms (scoreboard.json changes, 68.6s free)
● verified — Gemini 2/4 goals, 1 phantom, 2 misses (raw TASK1 output pasted)
● verified — Gemini coordinates hallucinated (3 wrong jerseys, board shows mirrored but Gemini reported raw)
● verified — final_video.mp4 1920x1080, 794.76s (ffprobe)
◑ believed — the audio says Schade not Thiago (inferred from hash + script; no speech-to-text)
◑ believed — Gemini "Mbeumo" scorer is a hallucination from training data (Mbeumo is a real Brentford player but not in this match)
○ unchecked — whether an ElevenLabs speed setting fixes §7 WPM without re-introducing risk
○ unchecked — total spend across RunPod/Vast/Modal/ElevenLabs this session
★ fragile — step6_voice hash gate depends on the hash file surviving; if .voice_script_hash is deleted, the voice regenerates (safe) but if the script is edited without re-verifying, the gate forces a regen (correct but may surprise)
★ fragile — scene_gen.py 3D board is not reliably reproducible (Stage 12); still wired into step2_boards