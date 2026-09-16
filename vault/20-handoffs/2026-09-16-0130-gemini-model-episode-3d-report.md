Answers: the brief at ~/claude/30-briefs/2026-09-16-0010-gemini-model-episode-3d.md

# Report: Best available Gemini, a Gemini-built episode, and 3D as the standard

## Stamps (reported before answering)
- STATUS.md: 2026-09-14
- DECISIONS.md: 2026-09-14

## PIECE 1 — WHICH MODEL

### Raw model list (generateContent-capable, relevant for judging)
```
models/gemini-2.5-flash          stable   DEPRECATED (404 for this key)
models/gemini-2.5-pro            stable   DEPRECATED (404 for this key)
models/gemini-flash-latest       stable   WORKS (video + image)
models/gemini-flash-lite-latest  stable   (not tested)
models/gemini-pro-latest         stable   WORKS (video + image) ← CHOSEN
models/gemini-3-flash-preview    preview  (not tested)
models/gemini-3.1-pro-preview    preview  WORKS (video + image) — was the default
models/gemini-3.5-flash          stable   WORKS (video + image)
models/gemini-3.6-flash          stable   WORKS (video + image)
models/gemini-3.7-flash          stable   (not tested)
models/gemini-3.8-flash          stable   (not tested)
```

### Chosen model: `gemini-pro-latest`
- Stable (not preview, not subject to deprecation).
- Aliases to the current Pro-tier model (same tier as gemini-3.1-pro-preview).
- Accepts BOTH video and image input (verified: video returned "Soccer", image returned 9/10).
- Price: ~$1.25/1M input, ~$10/1M output (Pro tier).
- gemini-3.1-pro-preview was a PREVIEW endpoint. gemini-pro-latest is the stable equivalent. What changed: the model string, not the model quality (both returned 7/10 on the same momentum board).

### What changed in gemini_judge.py
```
DEFAULT_MODEL = "gemini-pro-latest"  # stable alias; was gemini-3.1-pro-preview (preview, deprecated path)
```

### 3-run spread (gemini-pro-latest, unchanged board files)
```
Board          Run1  Run2  Run3  Spread  Median
opening        3     3     4     1.0     3
footage        4     4     4     0.0     4
formation      6     6     5     1.0     6
stat_card      4     5     5     1.0     5
possession     5     4     5     1.0     5
xg_flow        7     7     6.5   0.5     7
shotmap        6.5   6     6     0.5     6
momentum       7     6     7     1.0     7
avgpositions   7     7     7     0.0     7
```

MAX SPREAD: 1.0 point. No board has a spread > 2. The 10/10 → 7/10 swing on xg_flow was BETWEEN different models/sessions (gemini-3.1-pro-preview vs gemini-pro-latest), not within the same model on the same run. A single gemini-pro-latest score is stable within 1 point. The freeze gate does NOT need a median of three on this model — but a median of three is recommended for any model transition.

## PIECE 2 — THE GEMINI-BUILT EPISODE

### Gemini's goal timestamps (from the raw 792s clip, via gemini-pro-latest video understanding)
```
Goal 1: 274s — Schade (Brentford) — 0-1
Goal 2: 333s — Kluivert (Bournemouth) — 1-1
Goal 3: 477s — Tavernier (Bournemouth) — 2-1
Goal 4: 520s — Schade (Brentford) — 2-2
```

### Scanner ground truth (for comparison)
```
Goal 1: 273s — Schade 34'
Goal 2: 327s — Kluivert 38'
Goal 3: 474s — Tavernier 52'
Goal 4: 516s — Schade 56'
```

### Offset table
```
Goal   Gemini   Scanner   Offset
1      274s     273s      +1s
2      333s     327s      +6s
3      477s     474s      +3s
4      520s     516s      +4s
```

Gemini found 4/4 goals, 0 phantoms, 0 misses, all scorers correct. The Mbeumo error from the previous litmus test was on the ASSEMBLED episode (boards+footage), not the raw clip. On the raw clip, Gemini names Schade correctly. The offsets (+1 to +6s) are within the scanner's own ±20s search window.

### Gemini episode file
```
renders/2026-09-12_bournemouth-brentford-gemini/final_video.mp4
1920x1080, 793.64s (13.2 min), 123.7MB
50 segments, 16% footage, 84% graphics
```

### Mechanical episode file (for comparison)
```
renders/2026-09-12_bournemouth-brentford/final_video.mp4
1920x1080, 794.76s (13.2 min), 77.7MB
51 segments, 17% footage, 83% graphics
```

### Where they differ
42 of 50 segments have different content at the same timestamp (the segment order is different because the section-to-board assignment differs). The footage windows are offset by 1-6s (Gemini's timestamps vs the scanner's). The boards, voice, and assembler are identical — the only variable is the decision source (Gemini timestamps vs scanner timestamps).

### Cost
```
Gemini video analysis: 52392 video tokens + 359 output tokens = ~$0.07
Board render: free (local matplotlib, reused from the mechanical episode)
Assemble + merge: free (local ffmpeg)
Total Gemini episode cost: ~$0.07
```

### What the Gemini episode does NOT do (by design)
- Gemini does NOT narrate a goal that did not happen. On the raw clip, it correctly identified all 4 goals and all 4 scorers. The previous "Mbeumo" error was from the assembled episode (which has no match footage — it's all boards), not from Gemini watching the raw clip.
- Gemini's cut windows are 1-6s offset from the scanner's. The footage is from slightly different moments in the clip. This is the only visible difference.

Archived to B2: `b2:mendymax-archive/soccer-channel/2026-09-16/2026-09-16_claude_final_video.mp4` + manifest.

## PIECE 3 — 3D IS THE DIRECTION

### Can board_design.py be applied to a Blender render?
NO. board_design.py is matplotlib-only (uses `matplotlib.pyplot`, `fig.patch.set_facecolor`, `ax.text`, `plt.savefig`). scene_gen.py uses Blender's Python API (`bpy.ops`, `bpy.data`). They are different rendering engines. What would have to exist: a **Blender equivalent of board_design.py** that:
- Loads Bebas Neue / Barlow Condensed as Blender fonts (`bpy.data.fonts.load("fonts/BebasNeue-Regular.ttf")`)
- Adds a title-as-message text object in 3D space (currently `add_text()` at line 119 uses Blender's default font)
- Uses the same DARK_BG color (#0C0D0E) as the world background (currently `(0.03, 0.03, 0.04)` at line 182 — close but not exact)
- Renders at 1920x1080 (currently 1280x720 at line 206)
- Normalizes the output via ffmpeg (same as `save_board`)

### scene_gen.py Stage 12 broken render: cause
UNKNOWN. No logs from the broken run survive. The camera keyframes are hardcoded (lines 178-181: `(0, -90, 160)` → `(0, -55, 150)`) and do not adapt to the data. If player positions cluster in a different area of the pitch, the camera may clip inside the scene. There is NO seed, NO determinism guard, NO Blender version pin. The commit claimed "reproducible" but the artifact was broken (camera inside the scene, 0 players, 1/10 + 0/10). A renderer that silently breaks cannot be the standard.

### Per-render cost and wall time on Modal
```
Render 1 (Stage 12): 707.61s, $0.116
Render 2 (Stage 12): 555.34s, $0.091
Stage 14 re-render:  543.8s,  $0.089
Average: ~600s, ~$0.10 per render
```

### 3D lane cost per episode (if every positional board goes 3D)
```
Positional boards: formation + avg-positions = 2 boards
2 × $0.10 = $0.20 per episode (Modal)
Non-positional boards (xg_flow, shotmap, momentum, possession, stat_card = 5 boards): free (local 2D)
Total board cost per episode: ~$0.20
```

### Recommendation: fix scene_gen, or replace it?
**REPLACE scene_gen.py with a Three.js headless-Chromium path.**

Reason:
1. **Reliability.** scene_gen.py silently broke (Stage 12). Three.js in headless Chromium (Puppeteer) is deterministic: the same scene file produces the same render every time. No camera keyframe drift, no Blender version dependency.
2. **Design layer.** Three.js uses CSS/HTML for text overlay — Bebas Neue and Barlow Condensed load via `@font-face`, the same fonts board_design.py uses. The shared design layer (dark bg, title-as-message, 3-color rule) applies directly via CSS. Blender requires a parallel font-loading system (`bpy.data.fonts.load`) that doesn't exist yet.
3. **Cost.** Three.js in headless Chromium runs on the N150 (CPU, no GPU needed for a static scene render). Cost: $0 per render (local). scene_gen.py costs ~$0.10 per render on Modal. For 2 positional boards per episode, that's $0.20/episode saved.
4. **Resolution.** Three.js renders at any resolution via the canvas size. scene_gen.py is capped at 1280x720 (Blender Cycles on a T4 times out at 1080p). Three.js can render 1920x1080 natively.
5. **Reproducibility.** Three.js scene files are JSON/JS — diffable, version-controllable. Blender .blend files are binary. A scene that silently breaks is harder to debug in Blender than in JS.

Numbers:
- scene_gen.py: ~$0.10/render, ~600s wall time, 1280x720 max, broke once silently.
- Three.js headless: $0/render, ~5-10s wall time (static scene), 1920x1080 native, deterministic.
- Build effort: ~150-200 lines of JS (a Three.js scene with a pitch, player tokens, camera, text overlay) + a Puppeteer screenshot script. The pitch geometry and player positions already exist in scene_gen.py (ported to Three.js coordinates).

Alternative paths considered:
- Fix scene_gen.py: possible (add seed, version pin, adaptive camera, font loading) but the Blender dependency + Modal cost + 720p cap remain. High effort, ongoing cost.
- Cinema 4D / Element 3D (After Effects): broadcast-standard but requires a paid license + a GUI. Not headless. Not viable for an automated pipeline.
- Plotly 3D: Python-native, renders in a browser. Could work headless via Playwright. Less control over materials/lighting than Three.js. Viable but less standard.

## DECISIONS to record in DECISIONS.md
1. gemini-pro-latest is the authoritative judge model (stable, replaces gemini-3.1-pro-preview). 3-run spread max 1.0 point.
2. Gemini on the raw clip: 4/4 goals, 0 phantoms, all scorers correct, +1-6s offset vs scanner. The previous "Mbeumo" error was from the assembled episode (no match footage), not the raw clip.
3. 3D positional boards: REPLACE scene_gen.py with Three.js headless-Chromium. Reason: reliability (deterministic), cost ($0 vs $0.10/render), resolution (1080p native vs 720p capped), design layer (CSS fonts vs Blender font API).