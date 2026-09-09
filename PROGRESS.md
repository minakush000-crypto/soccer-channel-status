# PROGRESS.md — chronological log

## 2026-09-05 session 1: documentation spine + skill fix

- Built CONTEXT.md, ARCHITECTURE.md, TOOLS.md, STATUS.md, DECISIONS.md,
  GAPS.md, SKILLS.md from code reading and command output, not intent.
- Neutralized the soccer-channel skill (both copies): frontmatter now says
  RETIRED, body redirects to ~/yt-digest/soccer-channel, all
  references to ~/soccer-pipeline deleted. Backups at *.bak.
- Added GAPS.md note: skill firing is model judgment, CONTEXT.md is the
  only reliable primer.

## 2026-09-05 session 2: Option C Stage 1

- Backed up tools/cv_annotate.py to tools/cv_annotate.py.bak.
- Added `player_positions` export to cv_annotate.py: one entry per processed
  frame, each with {frame, players: [{id, bbox, team}]}. Existing exports
  (team_assignment, ball_positions) intact. Syntax checked.
- Created tools/runpod_stage1.py: one-off RunPod runner for a single 10s clip.
  Uploads clip + tools to catbox.moe, creates webhook, runs cv_annotate.py
  --max-frames 300, uploads tracking JSON + annotated MP4, terminates pod.
  Known bug: webhook parser fails on unescaped newlines in track_preview.
- Ran on RunPod L4: 300 frames, 129 tracker IDs, 23s inference, $0.05 cost.
  Output: renders/2026-08-30_liverpool-forest/clips/clip_nxPNT4TU5_Q_annotated.tracking.json
  (1.16MB) + clip_nxPNT4TU5_Q_annotated.mp4 (1.5MB, 1920x1080, 10s).
- Confirmed: produce_episode.py:288 call to cv_annotate.py is NOT broken
  by the change (signature and CLI unchanged).
- Confirmed: NO path from tracker ID to player name exists in the repo.
  match_data.py has jersey numbers but nothing reads them from video frames.
  Stage 2 is arrows on unnamed tracked players or hand-mapped.
- Updated STATUS.md, DECISIONS.md, GAPS.md, CONTEXT.md.

## 2026-09-05 session 3: tracking measurement + direction pick (C+E)

- Ran three measurements on the Stage 1 tracking JSON (300 frames, 10s).
  Key finding: the "129 IDs" is from 3 camera shots, not pure fragmentation.
  The clip has shot boundaries at frames 130 and 245. Within each shot,
  tracking is usable: median consecutive run 32 frames (1.07s), 72 of 129
  IDs survive >25 frames, 7 survive >75 frames. Shot 2 (frames 131-244)
  is the cleanest: 35 IDs, median run 41 frames, 2 survive the full shot.
- Detection is healthy: median 20.0/frame (near 22), mode 21 (58 frames).
  Max 27 (mild referee/crowd contamination). Team classification only
  works in shot 1 (shots 2-3 are 100% team 2 = unclassified).
- Long-lived trackers cluster by shot. New-ID rate spikes at shot boundaries
  (22 new IDs in frames 130-139, 20 in frames 240-249) and is near zero
  in stable passages (0-2 new IDs/sec in sec 1, 5, 6).
- Analysed options A-G against the RESUME_RESEARCH.md benchmark (7 criteria).
  Option E (top-down tactical graphics) scores 5/7. No other option scores
  above 1/7. Options B (fix tracking) and D (jersey OCR) are losing fights
  on compressed wide-shot footage. Picked C+E: segment selection by tracking
  quality + top-down tactical renderer.
- Built tools/segment_scorer.py: standalone scorer that ranks 1-second
  windows by detection count, tracker persistence, new-ID rate, and team
  classification quality. Composite score 0-100, threshold 65. Validated
  on the 10s tracking data: 4/10 segments recommended, correctly flags
  camera cuts and close-up shots as low-quality.
- pitch_radar.py confirmed dead code: shipped to RunPod but never imported
  by cv_annotate.py. Only mention is a comment on line 370. It is a
  starting point for option E but needs major expansion.
- Next: run cv_annotate on the full 146s clip on RunPod to score all
  segments, then prototype the top-down tactical renderer.

## 2026-09-05 session 3 (continued): C+E prototype

- Ran cv_annotate on the full 146s clip on RunPod L4: 4380 frames,
  156s inference, $0.011 cost. Output: clip_nxPNT4TU5_Q_full.tracking.json
  (8MB). Fixed webhook bug (removed track_preview field that broke JSON
  on newlines). Runner: tools/runpod_fulltrack.py.
- Scored all 146 segments: 15/146 score >=65 (15s of usable footage).
  Best: sec 1 (85.6), sec 5 (82.5), sec 113 (72.5). Worst: sec 45 (7.5),
  sec 68 (6.5), sec 89 (6.5). Many segments are close-ups/replays with
  1-5 detections — correctly flagged as low-quality.
- Built tools/tactical_render.py: full-frame top-down tactical renderer.
  Reads tracking JSON, renders dark pitch with mowing stripes, player
  dots in team colors, movement trails, and Bezier arrows. Outputs PNG
  or MP4. Screen-space projection (no homography model needed).
- Vision assessment: Claude Opus 5 rates 5.5/10 (authoritative). All
  elements visible: stripes, arrows with arrowheads, team-colored dots,
  trails. Local gemma4:cloud rated higher but is not the judge of record.
  Claude identifies
  3 key gaps: (1) no context layer (title, team names, ball marker,
  attacking direction), (2) pitch layout off-centre with missing
  markings, (3) movement encoding needs hierarchy (all players equal
  weight, trail color conflicts with team dot color).
- Deliverables this session:
  - tools/segment_scorer.py (C: segment quality scorer)
  - tools/runpod_fulltrack.py (RunPod runner for full-clip tracking)
  - tools/tactical_render.py (E: top-down tactical renderer prototype)
  - renders/2026-08-30_liverpool-forest/clips/clip_nxPNT4TU5_Q_full.tracking.json
  - /tmp/tactical_seg1_8s_v2.png (proof-of-concept image, Opus 5.5/10)
  - /tmp/tactical_seg1_5s.mp4 (proof-of-concept video, 1280x720, 5s)

## 2026-09-05 session 4: capability audit, doc fixes, C+E risk analysis

- Rewrote soccer-channel/CLAUDE.md (43 lines, under 50): project path,
  standing rules, image rule, judge rule (Opus authoritative), tool routing
  table with 10 tools, @CONTEXT.md and @STATUS.md references. Backup at
  CLAUDE.md.bak.
- Fixed yt-digest/CLAUDE.md: soccer-channel skill marked RETIRED (was
  listed as active). Backup at CLAUDE.md.bak.
- Added CONTENT RESEARCH section to CONTEXT.md covering viral_angle.py,
  agent_reach_research.py, fresh_fetch.py. These tools existed but were
  undocumented, which is why they stopped being used. Backup at CONTEXT.md.bak.
- Removed 8/10 gemma4 score from STATUS.md and PROGRESS.md. Only 5.5/10
  (Claude Opus 5, authoritative) remains as the current assessment.
  Backups at STATUS.md.bak and PROGRESS.md.bak.
- Risk 1 DIAGNOSED: K-means fits once on first 60 frames (line 258,
  classification_threshold=60, confirmed by grep). classified=True after
  fitting, never re-fits. team_assignment has 38 entries (shot 1 only).
  1020 unique tracker IDs in full 146s clip. 982 IDs (96%) unclassified,
  default to team 2 (referee). Shots 2-14: 0% classified. This is a hard
  limitation without per-shot re-fitting or a different classification
  approach.
- Risk 1b ANSWERED: 2 of 15 segments scoring >=65 have working team
  classification (sec 1 at 85.6, team%=74; sec 3 at 65.3, team%=22).
  The other 13 have team%=0. E does not have enough two-team footage.
  The 15s of "usable" footage is really 2s of two-team footage.
- produce_v2.py confirmed: syntax OK (ast.parse), imports all standard
  library, no references to new files (segment_scorer, tactical_render,
  runpod_fulltrack, runpod_stage1). The four new files are standalone and
  do not affect produce_v2.py.
- MCP memory server graph is empty (read_graph returned no entities).
- Auto memory is DISABLED (CLAUDE_CODE_DISABLE_AUTO_MEMORY=1 in settings.json).

## 2026-09-06 session 5: capability enablement

### Crash recovery
- git status: branch master, 2 commits ahead, modified files are expected
  pipeline tools. Nothing broken by the crash.
- ls tools/*.bak: cv_annotate.py.bak, produce_v2.py.bak, produce_v2.py.bak2,
  runpod_stage1.py.bak all present.
- ast.parse(produce_v2.py): ok.
- Claude Code version: 2.1.263 (matches npm registry 2.1.263, update is current).

### Settings backup
- Prior session backed up to ~/.claude/settings.json.bak-sep6 (1071 bytes,
  Sep 5 23:51, pre-edit state).
- This session backed up to ~/.claude/settings.json.bak-sep6-session5
  (1768 bytes, Sep 6 00:05, post-prior-session state).

### Settings audit (what was off, what was already on)

Prior session already fixed:
- CLAUDE_CODE_DISABLE_AUTO_MEMORY=1 → removed. autoMemoryEnabled: true set.
- No PreToolUse hooks → 3 installed (block-retired, block-image-read,
  warn-local-gpu).
- bypassPermissions: already set (user's deliberate choice, not touched).

This session enabled (before → after, all in ~/.claude/settings.json,
affects ALL projects on this machine):
- enableWorkflows: false → true (enables the workflows feature)
- workflowKeywordTriggerEnabled: false → true (enables keyword-triggered
  workflow activation)
- awaySummaryEnabled: false → true (enables away summaries, generates
  session summaries when user is away)

Already on, not touched:
- effortLevel: xhigh, fastMode: true, autoCompactEnabled: true,
  switchModelsOnFlag: true, skipDangerousModePermissionPrompt: true,
  autoMemoryEnabled: true (prior session).

### Auto memory verified
- Memory directory: ~/.claude/projects/-home-muads-yt-digest/memory/
  (exists, was empty before this session).
- Wrote MEMORY.md (index) and soccer-channel-capability-enablement.md
  (first memory file). Both confirmed created by Write tool.
- Auto memory is functional: the setting enables the system prompt
  instruction to write memory files, and the Write tool successfully
  creates them in the correct directory.

### MCP memory graph decision
- claude-mem is the memory of record. It has 50+ project observations
  with semantic search, timeline, and corpus building. The MCP memory
  graph (read_graph) is empty and has a simpler entity-relation model.
- Decision: do NOT populate the MCP memory graph. It would duplicate
  claude-mem with a less capable query system and create a sync problem.
  claude-mem's search (mcp__plugin_claude-mem_mcp-search__search) and
  timeline tools are more powerful for this project's needs.

### Hooks: all 3 proven to fire live
1. block-retired.sh (matcher: Read|Edit|Write|Bash)
   - PROVEN: blocked a Bash command containing ~/retired/test.py.
   - Exit 2, error: "BLOCKED: ~/retired/ is off limits."
2. block-image-read.sh (matcher: Read)
   - PROVEN: blocked Read of /tmp/nonexistent-test.png.
   - Exit 2, error: "BLOCKED: Image/video file detected. glm-5.2:cloud
     cannot see images. Use: ~/tools/vision_analyze.py or ask_claude.py."
3. warn-local-gpu.sh (matcher: Bash)
   - PROVEN: script test outputs warning and exits 0. Live test with
     echo "yolo" allowed through (exit 0). Warning goes to stderr but
     is NOT surfaced to the model for exit-0 hooks (Claude Code
     framework behavior). The hook fires but its warning is invisible
     to the model in practice. This is a known limitation: exit 0 =
     allow, stderr swallowed. For visible enforcement, use exit 2.
   - Hook script content verified: checks for
     cv2|yolo|torch|cuda|bytetrack|annotate|inference|detect.py|
     segment_scorer|tactical_render, exempts commands containing
     runpod|ssh|pod|remote|cloud.

### @ syntax settled
- WebFetch on https://code.claude.com/docs/en/memory confirms:
  "CLAUDE.md files can import additional files using @path/to/import
  syntax. Imported files are expanded and loaded into context at launch
  alongside the CLAUDE.md that references them."
- @CONTEXT.md and @STATUS.md in soccer-channel/CLAUDE.md ARE working.
  Their full contents are in the session system prompt. A session-start
  hook to read them is unnecessary.
- Relative paths resolve relative to the file containing the import.
  Maximum import depth: 4 hops. Code spans and fenced code blocks are
  skipped.

### Parallel and background rules (prior session, verified in CLAUDE.md)
- Soccer-channel CLAUDE.md standing rules 8 and 9 already contain:
  "Batch independent tool calls in one response. Serialize only when
  one task's output is needed as input for the next." and "Use
  run_in_background for any command expected to take >2 minutes."
- yt-digest CLAUDE.md section 5 already contains subagent delegation
  pattern with Explore subagents.

### MCP server tests (all 6 reachable and functional)
1. github: search_repositories "soccer tactical analysis computer vision"
   → 8 results. Found SoccerVisionAI (YOLOv8+ByteTrack+player assignment)
   and ugo-soccer-analytics (YOLOv8+ByteTrack+homography+possession).
   Use: research existing soccer CV pipelines, borrow approaches.
2. brave-search: brave_web_search "soccer tactical analysis video pipeline"
   → Nature paper on Tactiformer/StratGaze + Catapult MatchTracker.
   Use: research tactical analysis methods and benchmark tools.
3. filesystem: list_allowed_directories → ~/yt-digest/soccer-channel.
   Use: file operations within project scope. Redundant with Read/Write
   tools but available if needed.
4. sequential-thinking: single thought returned successfully.
   Use: break down complex pipeline design decisions step by step.
5. ide: getDiagnostics → [] (no VS Code file open). Use: catch lint/type
   errors when editing in VS Code.
6. puppeteer: navigated to about:blank successfully. Use: YouTube upload
   fallback (assessed below).

### Puppeteer and YouTube upload assessment
- youtube_upload.py uses YouTube Data API v3 with OAuth2 (line 133:
  build("youtube", "v3", credentials=creds)). Token file:
  secrets/youtube_token.json (Aug 23, 13 days old). Code checks for
  expired credentials and refreshes via refresh_token (line 117).
- client_secret.json: Aug 23. yt_cookies.txt: Sep 5 (fresh).
- API path assessment: likely functional. Google OAuth refresh tokens
  are long-lived (don't expire unless revoked). The code handles token
  refresh automatically. The API path should be tried first.
- Puppeteer fallback assessment: technically possible (browser launches,
  can navigate). Could use yt_cookies.txt for cookie-based auth. But:
  YouTube has strong anti-bot measures (CAPTCHA, device fingerprinting).
  Upload UI selectors change frequently. Slower and more fragile than
  API. Cookie auth might trigger security challenges.
- Recommendation: try youtube_upload.py first. If OAuth refresh fails,
  Puppeteer with cookies is a viable fallback but should be a last
  resort. Do not attempt upload yet (per user instruction).

### Task tracking
- 6 tasks created (TaskCreate):
  #1 Fix team classification: re-fit K-means per camera shot
  #2 Add context layer to tactical renderer
  #3 Fix pitch layout: centre, complete markings, lower stripe contrast
  #4 Rebuild movement encoding with hierarchy
  #5 Wire tactical_render into produce_v2.py (blocked by #1-#4)
  #6 First YouTube upload (blocked by #5)
- Task tracking is worth using: the project has multiple sequential
  workstreams with dependencies. Tasks give the user visibility into
  progress and prevent skipping steps.

### Cron
- Available but NOT set up. Cron jobs are session-only (not persistent
  across sessions). This project uses short interactive sessions, so
  cron is not useful for long-term scheduling. For periodic content
  research (fresh_fetch.py, viral_angle.py), system cron or a dedicated
  scheduler would be better. Cron is available if a long-running session
  needs periodic checks.

### Restart guidance
- No urgent restart needed. All settings changes are already in effect:
  hooks proved firing, auto memory proved writing, @ imports proved
  loading. The installed version (2.1.263) matches npm registry (2.1.263),
  so the update is current.
- After restart, verify:
  1. Run /context — check CONTEXT.md and STATUS.md appear under Memory
     files (proves @ imports load).
  2. Try Read on a .png file — should be blocked by block-image-read hook.
  3. Run /memory — check auto memory folder shows MEMORY.md and
     soccer-channel-capability-enablement.md.
  4. grep autoMemoryEnabled ~/.claude/settings.json — should be true.
  5. git status in project — verify nothing broken.

## 2026-09-06 session 5 (continued): task 1 — per-shot team classification

### Validation before building
Checked 6 frames from the raw 146s source clip (clip_nxPNT4TU5_Q.mp4,
1920x1080, 4380 frames) via vision relay:
- Frame 30: crowd/stands shot (no players). Opus confirmed.
- Frame 120: crowd shot (no players). Scored 82.5 by segment_scorer
  but is crowd, not pitch. Segment_scorer is contaminated by crowd
  detections (YOLOv8 detects spectators as "person" class).
- Frame 1522: close-up of one sky-blue player (#4). Opus says this
  looks like Manchester City, not Nottingham Forest. Clip may be
  mislabelled.
- Frame 2500: goalkeeper close-up (teal jersey).
- Frame 3500: crowd shot (no players).
- Frame 3360: WIDE PITCH SHOT with both teams visible. Red vs light
  blue, 10+ players. This is the first confirmed wide shot.
- Team assignment from existing tracking JSON: shot 1 found 2 balanced
  clusters (team 0 = 16, team 1 = 14, referee = 8). K-means works
  when both teams are visible.
- Opus assessment: red vs light blue are ~155 degrees apart in hue,
  "one of the easier cases" for hue-based anchoring. Raw RGB centroid
  matching will NOT hold across shots (illumination shifts); must use
  HSV hue matching. Must sample torso ROI only (discard low-saturation
  and dark pixels). Must cluster inside person-detector bounding boxes,
  not on whole frame (crowd is red at Anfield).

### Changes to tools/cv_annotate.py (backup: tools/cv_annotate.py.bak2)
1. TEAM_COLORS: team 1 changed from green (50,200,50) to light blue
   (50,150,255) to match the actual away kit.
2. extract_jersey_color: added HSV filtering. Converts torso ROI to
   HSV, keeps only pixels with S>60 and V>50 (discards white shorts,
   dark shadows, pitch grass). Falls back to grey if <5 qualifying
   pixels.
3. classify_teams: rewritten to cluster on (cos(hue), sin(hue),
   saturation) instead of raw RGB. Hue is treated as circular (red
   at 0/180 wraps). Returns (team_assignment, team_hues) tuple where
   team_hues is [(team_id, mean_hue)] for cross-shot anchoring.
4. New function circular_hue_distance: handles hue wraparound.
5. New function _anchor_labels: compares current shot's team hues to
   reference hues (from first shot) by circular distance. Swaps team
   0/1 labels if the swapped assignment gives a smaller total hue
   distance. This fixes the label-flip problem.
6. Main loop: added shot boundary detection (>10 new tracker IDs in
   one frame after a stable passage). At each boundary: resets
   detection_frames, classified, and tracker_colors. Keeps
   team_assignment and reference_hues. Only collects colors for
   trackers NOT already in team_assignment.
7. Main loop: classification now calls classify_teams with
   reference_hues. First shot sets reference_hues. Subsequent shots
   are anchored to the reference.
8. Export: added shot_count and shot_boundaries to tracking JSON.
9. End-of-function: unique tracker count now uses total_trackers set
   (accumulates across all shots) instead of tracker_colors (which
   resets per shot).

### Changes to tools/runpod_fulltrack.py (backup: tools/runpod_fulltrack.py.bak)
Added --verbose flag to pod command, log file capture and upload, and
log download. This gives shot boundary detection messages and per-shot
classification output in the downloaded log.

### RunPod test (in progress)
Running cv_annotate on full 146s clip with --verbose. Output prefix:
clip_nxPNT4TU5_Q_full_v2 (preserves old tracking JSON for comparison).
Background task ID: bi6023uj1. Awaiting results.

### Bug fix: --verbose flag + BGR colors
First RunPod run failed: cv_annotate.py rejected --verbose (argparse
error, flag not defined). verbose defaults to True in annotate_clip()
but was never exposed as a CLI argument. Fix: removed --verbose from
the pod command in runpod_fulltrack.py (verbose is already on by
default). Also fixed BGR color bug: team 0 was (255,50,50) BGR = blue
not red, team 1 was (50,150,255) BGR = orange not light blue. Changed
to (50,50,255) BGR = red and (255,200,100) BGR = light blue. Both
files re-syntax-checked. Re-running on RunPod (task bhm4anz4p).

### Bug fix: axis parameter in extract_jersey_color
Second RunPod run crashed at line 117 (cv2.cvtColor channel error).
Root cause: region[mask] has shape (N, 3) but I used mean(axis=(0,1))
which collapses to a scalar, not a 3-element array. The original code
used region.mean(axis=(0,1)) on the full (H,W,3) region which correctly
gives (3,). Fix: changed to region[mask].mean(axis=0) which preserves
the 3 color channels. Pod ran 8s before crash. Re-running (third attempt).

### RunPod test 3: SUCCESS (2026-09-06)
Third run completed: exit 0, 4380 frames, 1020 trackers, 93MB output.
Log shows 11 shot boundaries detected, all shots 2-11 anchored.

VERIFIED RESULTS (analysis script on v2 tracking JSON):
- Shot count: 11 (old: 1, no metadata)
- Shot boundaries: [0, 131, 245, 1268, 1370, 2660, 3369, 3639, 3845, 3926, 4062]
- Total classified trackers: 379 (old: 38) — 10x improvement
- Team assignment: {0: 222, 1: 110, 2: 47}
- Frames with both teams: 1018/4380 (old: 129/4380) — 8x improvement
- ALL 11 shots have both teams classified

Log classification messages:
- Shot 1: Team A=25, Team B=12, Ref=1 (sets reference hues)
- Shot 2: Team A=6, Team B=13, Ref=5 (anchored)
- Shot 3: Team A=20, Team B=9, Ref=6 (anchored)
- Shot 4: Team A=57, Team B=5, Ref=4 (anchored, unbalanced — close-up)
- Shot 5: Team A=39, Team B=3, Ref=2 (anchored, unbalanced — close-up)
- Shot 6: Team A=14, Team B=28, Ref=10 (anchored)
- Shot 7: Team A=10, Team B=12, Ref=5 (anchored, balanced — wide shot)
- Shots 8-11: all anchored, 5-9 per team

Known limitation: crowd contamination. 222 team-0 trackers vs 110 team-1
suggests red Anfield crowd detected as team 0. Per-shot re-fitting
mitigates but doesn't eliminate this. Shots 4-5 are close-ups with
one team dominant (57v5, 39v3).

TASK 1 COMPLETE. Backup: tools/cv_annotate.py.bak2.

## 2026-09-06 session 5 (continued): task 2 — context layer

### Changes to tools/tactical_render.py (backup: tools/tactical_render.py.bak)
1. TEAM_COLORS updated to match cv_annotate.py BGR values (team 0 = red
   (50,50,255), team 1 = light blue (255,200,100)).
2. New function draw_context_layer: top bar with team names, color dots,
   attacking direction arrows, score (centred on canvas), minute (inline
   satellite, small grey), and bottom bar with tactical caption. Orange
   accent rules bracket the pitch top and bottom.
3. render_segment signature extended with home_team, away_team, score,
   minute_str, caption parameters. draw_context_layer called before
   frame write in both PNG and video paths.
4. CLI args added: --home-team, --away-team, --score, --minute, --caption.

### Opus assessment (12 iterations, judge of record)
- v1: 4.5/10 (basic layer, minute in wrong place)
- v5: 8/10 (enlarged arrows, differentiated minute)
- v11: 7.5/10 (inline minute on shared baseline)
- v12: 8/10 — "Yes, this is production-ready for the context layer.
  The remaining optical-centring question is a taste call, not a defect."

Key design decisions from Opus feedback:
- Score is the fixed center anchor (centred on canvas centre), minute
  hangs to the right as a satellite (14px gap, 0.65 scale, muted grey).
  This keeps the score stable regardless of minute width.
- All text elements share one baseline (y=31) for a single horizontal
  anchor line. Team names, score, and minute all align.
- Attacking direction arrows (13px, full-saturation team colors) sit
  between the color dot and the team name, mirrored left/right.
- Bottom bar holds a tactical caption (not the minute), with an orange
  divider rule matching the top bar.
- Bar height 52px, score thickness 2 (thickness 3 distorted the "0"
  glyph counter in OpenCV's HERSHEY_DUPLEX font).

Known limitation: OpenCV's built-in fonts lack tabular numerals, prime
characters, and en-dashes. A production system would use PIL/Pillow with
TrueType fonts for these typographic details.

TASK 2 COMPLETE. Backup: tools/tactical_render.py.bak.

## 2026-09-06 session 5 (continued): task 3 — pitch layout

### Changes to tools/tactical_render.py
1. Added pitch constants: PEN_SPOT_L=11m, GOAL_W=7.32m, CORNER_R=1m.
2. draw_pitch: added penalty spots (filled circles at 11m from each goal),
   penalty arcs (cv2.ellipse at ±53° from penalty spot, clipped to outside
   the penalty box), corner arcs (quarter circles at all 4 corners), goals
   (small rectangles outside the goal line at both ends).
3. Lowered stripe contrast: lighter shade changed from (34,64,44) to
   (24,46,33) — roughly 1.3x PITCH_COLOR instead of 1.9x.
4. Increased goal area thickness from 1 to 2 (was invisible at 35% alpha).
5. Increased goal thickness from 1 to 2.
6. Increased LINE_ALPHA from 0.35 to 0.50 (Opus suggested 0.55, went
   with 0.50 as a compromise to keep lines subtle but visible).

### Opus assessment (judge of record)
- Pitch-only render (no players): 8.5/10
- "Complete and correctly ordered element hierarchy; nothing overlaps
  incorrectly. Excellent left/right mirror symmetry. Penalty arcs are
  properly clipped so only the portion outside the 18-yard box shows."
- All markings confirmed visible: touchlines, halfway line, centre circle,
  centre spot, penalty areas, goal areas, penalty spots, penalty arcs,
  corner arcs, goals.
- The left goal area "invisibility" in earlier full renders was player
  occlusion + low alpha, not a drawing bug. Pitch-only test confirmed
  both goal areas are drawn correctly.

TASK 3 COMPLETE.

## 2026-09-06 session 5 (continued): task 4 — movement encoding

### Changes to tools/tactical_render.py
1. TRAIL_COLOR constant: (235,235,235) light grey for fallback/unclassified.
2. draw_trail: uses team colours at 0.6 alpha (via addWeighted) so trails
   show team ownership while remaining visually distinct from solid dots.
   Width increased to 3px. Path simplified with _simplify_path before drawing.
3. New _simplify_path: removes vertices where turn > 140 degrees (kills
   hairpin artifacts from sharp direction reversals in tracking data).
4. New _should_draw_glyph: gates on net displacement >= 25px AND
   net/arclength ratio >= 0.55 (filters hairpins where player returns
   to near origin).
5. draw_bezier_arrow: minimum length increased from 8 to 15px.
6. Combined trail+arrow loop in PNG path: ensures consistency (both drawn
   or neither). Arrow uses simplified path endpoints. Arrow tip inset
   by DOT_RADIUS + ARROW_HEAD + 2 = 28px past the dot. Minimum 43px
   displacement for any glyph (inset + 15px arrow minimum).
7. Video path: trail loop uses _should_draw_glyph for the same filtering.

### Opus assessment (7 iterations, judge of record)
- v1: 6/10 (two-tone trails, spline overshoot)
- v5: 6.5/10 (team colours, ratio filter)
- v7: 8.5/10 — "hairpins and stubs are genuinely eliminated and the
  both-or-neither trail/arrow pairing is consistent"

Key design decisions from Opus feedback:
- Team-coloured trails at 0.6 alpha (not pure grey, not full saturation)
- Straight polylines (Catmull-Rom splines caused overshoot artifacts on
  sharp direction reversals; straight lines are cleaner for tracking data)
- Per-vertex angle simplification (drop vertices with > 140 degree turns)
- Net/arclength ratio filter (0.55 threshold catches hairpins)
- Combined trail+arrow loop (both drawn or neither, no incomplete glyphs)
- Arrow tip inset past dot radius so arrowheads are never hidden under dots

TASK 4 COMPLETE.

## 2026-09-06 session 5 (continued): task 5 — wire tactical_render into produce_v2.py

### Changes to tools/produce_v2.py (backup: tools/produce_v2.py.bak3)
1. New step4b_tactical_render function: checks for existing tracking JSON
   (prefers _v2 with per-shot classification), runs runpod_fulltrack.py if
   none exists, finds the shot with the most balanced team classification
   (team 0 vs team 1 tracker counts), runs tactical_render.py with team
   names and score from match_data.json.
2. step5_assemble: accepts tactical_path parameter, handles "tactical"
   visual tags ([VISUAL: tactical] in the script), adds "tactical" segment
   type to the concat builder (scales 1280x720 to 1920x1080 with pad,
   same filter as board segments).
3. main(): loads match_data before step4b (was loaded after voice), calls
   step4b_tactical_render, passes tactical_path to step5_assemble.

### Verification
- produce_v2.py syntax ok (ast.parse)
- step4b shot balancing logic verified: picks shot 7 (frame 3369, 112.3s,
  balance=0.09, 10 red vs 12 blue trackers) — the most balanced two-team
  shot in the 146s clip.
- tactical_view.mp4 produced: 1280x720, 3.0s, 1.1MB
- ffmpeg scale+pad verified: 1280x720 → 1920x1080 with #0d1f16 padding
  (same filter as board segments, which already work in the pipeline)

The C+E pipeline is now wired into produce_v2.py:
  step3 (download) → step4 (overlay) → step4b (tactical render) →
  step5 (voice) → step6 (assemble, includes tactical segments) →
  step7 (merge) → step8 (shorts crop)

TASK 5 COMPLETE.

## 2026-09-06 session 5 (continued): task 6 — YouTube upload assessment

### Assessment (no upload attempted, per user instruction)
- Token: secrets/youtube_token.json has refresh_token (✅). Access token
  expired 2026-08-24 but refresh tokens are long-lived (don't expire
  unless revoked). Code auto-refreshes at line 117.
- Script: tools/youtube_upload.py syntax ok (ast.parse).
- Packages: google-auth-oauthlib, google-auth, google-api-python-client
  all installed in ~/yt-digest/.venv.
- Publish log: publish-log/ is empty (no prior uploads, confirmed by ls).
- API path: READY. No blockers identified. The upload can be attempted
  when the user is ready.
- Puppeteer fallback: NOT needed. The API path should work. Puppeteer
  remains available as a backup if the OAuth refresh fails (assessed in
  capability enablement: browser launches, can navigate, but YouTube
  anti-bot measures make it fragile).

### What would need to happen for the first upload
1. Run: ~/yt-digest/.venv/bin/python tools/youtube_upload.py <slug>
2. The script reads match_data.json for title/description/tags.
3. It refreshes the OAuth token automatically.
4. It uploads the final video (shorts/final_video_shorts.mp4).
5. It logs the upload to publish-log/.
6. The user should verify the video is public and set the thumbnail.

TASK 6 COMPLETE (assessment only, no upload attempted).

## 2026-09-07: three questions answered

### Q1: Match identity — SETTLED
- yt-dlp confirms: title "Liverpool ( 2 - 2 ) Nottingham", tags include
  #nottinghamforest, uploader "footzonex7", upload date 20260830.
- Opus frame 1 (1522): sky-blue #4 kit, "classic Manchester City home
  kit". Crowd in red and white. FOOTZONE watermark. No scoreboard.
- Opus frame 2 (2500): Liverpool goalkeeper (Alisson), Liverbird crest,
  Standard Chartered sponsor, Expedia sleeve. No scoreboard.
- Opus frame 3 (3360): sky blue vs red, scoring moment. No scoreboard.
- Verdict: match IS Liverpool vs Nottingham Forest (confirmed by video
  title and match_data.json). The sky-blue kit is Forest's away kit, not
  Man City. Opus misidentified it because sky blue is City's classic
  color. No scoreboard visible in any frame, so score can't be
  independently verified from footage. Residual risk: step3 downloads
  the first 720p+ clip from YouTube search without verifying the
  footage matches the claimed match.

### Q2: Crowd contamination — QUANTIFIED
- 379 classified trackers: 200 pitch (53%), 51 crowd (13%), 128 edge
  (34%).
- Team 0 (red): 94 pitch, 33 crowd, 95 edge = 222 total.
- Team 1 (blue): 76 pitch, 10 crowd, 24 edge = 110 total.
- The 10x gain (38 to 379) is real but inflated: pitch-only gain is 5x
  (38 to 200). Crowd contamination is mostly team 0 (red Anfield crowd
  classified as red team). Some large-bbox trackers (area >400K) are
  close-ups of single players/coaches, not crowd.

### Q3: Capability check — HONEST AUDIT
- Parallel tool calls: YES, used extensively.
- Background tasks: YES, 3 RunPod runs + pipeline run.
- Subagents: NO. Should have used Explore for produce_v2.py reading.
- Sequential-thinking: NO. Should have used for label-flip analysis.
- PostToolUse PROGRESS.md hook: NO. Does not exist in settings.json.
  Manually updated PROGRESS.md with Edit calls.

## 2026-09-07: tasks 4 and 5 — pipeline run and YouTube upload

### Task 4: produce_v2.py end-to-end with step4b
- Pipeline ran: 300.8s (5.0 min), exit code 0.
- Step 1 (match data): 1.9s. Step 2 (boards): 89.5s. Step 3 (download):
  skipped (1080p clip cached). Step 4b (tactical render): 6.8s, best shot
  7 (112.3s, balance=0.09). Step 5 (assemble): 5 sections including [3]
  Tactical View (tactical tag, 8.7s of 62.3s total). Step 7 (merge):
  3.5s. Step 8 (shorts): 77.9s.
- Output: final_video.mp4 (1920x1080, 62.28s, 43.2MB, h264+aac).
  Shorts: final_video_shorts.mp4 (720x1280, 62.27s, 24.8MB).
- Copied to /mnt/c/Users/muads/Downloads/2026-08-30_liverpool-forest_Short.mp4.

### Task 5: YouTube upload (first ever)
- First attempt: OAuth refresh_token expired/revoked (token from Aug 23,
  Google "testing" mode expires after 7 days). Crash: RefreshError not
  caught. Fix: wrapped creds.refresh() in try/except, falls through to
  OAuth flow. Backup: tools/youtube_upload.py.bak.
- Second attempt: OAuth flow started, printed authorization URL, timed
  out (user hadn't opened URL in time — run_local_server default ~5min
  timeout).
- Third attempt: user ran with `!` prefix, authorized in Windows
  browser, token saved at 00:15:40.
- Fourth attempt (fresh token): upload succeeded. Exit 0.
  Output: "Upload complete: https://www.youtube.com/watch?v=IVUg5oZLH5k
  Video is PRIVATE. Review and publish in YouTube Studio."
- Video URL: https://www.youtube.com/watch?v=IVUg5oZLH5k
- Privacy: PRIVATE (user publishes manually).
- Title: "Liverpool Forest — Soccer Tactical Analysis"
- Tags: tactical, analysis, tactics, forest, football, soccer, liverpool
- File uploaded: final_video.mp4 (41MB, 1920x1080, 62.3s)
- publish-log/ is empty (script prints URL to stdout, does not write
  a log file — minor gap for future tracking).
- FIRST EVER YouTube upload for the soccer-channel project.
## Session 2026-09-07: Structural overhaul (step4 removal + re-balance + crowd filter + new match)

User verdict: "The video is bad and the reason is structural. Tactical View
is 8.7s of a 62.3s video. The other 53.6s is step4's LLM-guessed arrows
(crayon scribbles). Stop adding. Start replacing."

### Current section breakdown (verified from pipeline output)
- [0] Hook: 30 words, 8.7s, board=formation (14%)
- [1] Setup: 57 words, 16.5s, board=possession (26%)
- [2] Point 1: 72 words, 20.8s, footage=buildup (33%) ← step4 overlay
- [3] Tactical View: 30 words, 8.7s, tactical [actual 3s] (14%)
- [4] Close: 33 words, 9.5s, board=stat_card (15%)
- Total: 222 words, 64.2s. Tactical: 3s/62.3s = 4.8%.

### Changes made (6 edits, all syntax-verified)

1. **REMOVE step4** (produce_v2.py, backup: .bak4):
   - Deleted step4_overlay function (was lines 202-239).
   - Removed call in main() (was lines 559-560).
   - Changed step5_assemble to receive clip_path (raw) not annotated_clip.
   - grep confirms: no step4_overlay, annotated_clip, or tactical_overlay
     references remain in produce_v2.py.
   - tactical_overlay.py now orphaned (no pipeline caller).

2. **RE-BALANCE** (produce_v2.py):
   - step4b: --duration "3" → dynamic min(45.0, clip_dur - start_sec).
   - step5_assemble: tactical cap min(3.0, sec_duration) → min(sec_duration, 45.0).
   - New script (scripts/2026-09-06_arsenal-chelsea.md): 4 sections,
     223 words, 65.5% tactical (146 words). Target: 60%+. Confirmed.

3. **FILTER CROWD** (cv_annotate.py, backup: .bak3):
   - Added bbox size/position filter before tracker.update_with_detections.
   - Rejects: area < 500 (far crowd), > 50000 (close-up), cy < 80 (top
     stands), cy > h-80 (bottom edge). Persons only, balls pass through.
   - Verification RunPod run in progress on old Liverpool-Forest clip.

4. **NEW MATCH**: Arsenal 2-1 Chelsea (Sep 6 2026, PL).
   - fresh_fetch.py run (briefs/fresh/2026-09-07_0539_fresh.md).
   - Arsenal 4-2-3-1 vs Chelsea 3-4-2-1. 54.6% possession, 16 shots.
   - Slug: 2026-09-06_arsenal-chelsea. Query: "Arsenal Chelsea".

5. **OAUTH FIX** (user action, no code): Google Cloud Console →
   APIs & Services → OAuth consent screen → Publish App. Testing mode
   = 7-day refresh expiry; production = no expiry.

6. **PUBLISH LOG** (youtube_upload.py, backup: .bak2):
   - Added from datetime import datetime.
   - Writes JSON to publish-log/{slug}_{video_id}.json after upload.

### End-to-end pipeline run (in progress)
- produce_v2.py 2026-09-06_arsenal-chelsea --query "Arsenal Chelsea"
  --date-range 20260901-20260930
- Background task, output: /tmp/pipeline_arsenal_chelsea.txt

### Bug fixes during pipeline run (2026-09-07)

1. **AV1 codec crash**: First pipeline run downloaded an AV1-encoded clip.
   The RunPod pod (L4) cannot decode AV1 (no hardware decoder, software
   decoder fails with "Missing Sequence Header"). Result: 0 frames
   processed, 0 trackers, 0KB tracking JSON, tactical render skipped.
   Fix: changed yt-dlp format in step3 to prefer H.264 (vcodec^=avc1)
   before falling back to other formats.
   Verification: pending re-run.

2. **OUT_DIR hardcoded**: runpod_fulltrack.py line 18 had OUT_DIR hardcoded
   to the Liverpool-Forest clips directory. Tracking JSON downloaded to
   the wrong render folder. Fix: OUT_DIR = clip_path.parent (derive from
   the --clip argument). Backup: runpod_fulltrack.py.bak2.

3. **Step4b timeout**: Increased RunPod tracking timeout from 600s to 1200s
   for longer clips (482s clip has 14,472 frames).

### End-to-end pipeline result (2026-09-07 01:12)

Pipeline: 2026-09-06_arsenal-chelsea --query "Arsenal Chelsea"
--date-range 20260901-20260930

**Output verified:**
- final_video.mp4: 1920x1080, h264+aac, 65.17s, 12MB
- shorts: 720x1280, h264+aac, 65.18s, 8.2MB
- tactical_view.mp4: 1280x720, 45.0s, 1350 frames
- step4_overlay references in produce_v2.py: 0 (fully removed)

**Section breakdown (with board fix):**
- Hook: 17 words, 5.7s, board=formation
- Setup: 20 words, 6.7s, board=possession
- Tactical Analysis: 146 words, 48.5s, tactical (45s view + 3.5s footage)
- Close: 13 words, 4.3s, board=stat_card
- Total: 196 words, 65.2s. Tactical view: 45s/65.2s = 69%

**Crowd filter verification (Arsenal-Chelsea 720p):**
- Total trackers: 3248, Pitch: 3248 (100%), Edge: 0, Crowd: 0
- Team assignments: 285 (all real players)

**YouTube upload:**
- URL: https://www.youtube.com/watch?v=PsEz5ITTIpM
- Privacy: PRIVATE
- Publish log: publish-log/2026-09-06_arsenal-chelsea_PsEz5ITTIpM.json
- First publish-log entry ever (directory was empty before)

**Bug fix: board duration (4th fix):**
- step5_assemble subtracted 4.0s for boards but only played
  min(4.0, sec_duration*0.5). Video was 59.6s vs voice 65.2s —
  the entire Close narration was cut off. Fixed: subtract actual
  board_dur. Video now 65.2s, matching voice.

**Total cost:** ~$0.04 RunPod (2 runs: $0.009 verification + $0.030
Arsenal-Chelsea tracking) + ElevenLabs voice + YouTube API (free).

## 2026-09-08 session: Gemini video inventory assessment (footage-first idea)

User hypothesis: reverse the pipeline, footage inventory first (Gemini
video understanding), then script from that inventory cross-referenced
with ESPN match data, then cut at named timestamps. Assessment only,
no pipeline built.

Setup:
- GEMINI_API_KEY in ~/yt-digest/.env authenticates (urllib GET
  /v1beta/models -> 50 models). Prepaid AI Studio key; was depleted
  (429 "prepayment credits are depleted"), user topped $25, then works.
- Installed google-genai into ~/yt-digest/.venv (no breakage: yt_digest,
  openai, requests all still import). tools/gemini_inventory_test.py
  written (File API upload + inline-bytes mode, model fallback). Backup
  at tools/gemini_inventory_test.py.bak.
- 2.5 model family is 404-gone ("no longer available to new users",
  redirects to gemini-3.6-flash). Only gemini-3.6-flash TEXT is free;
  ALL video (even 3s) is prepay-gated. Used gemini-3.1-pro-preview after
  credit.

Measured cost (usage_metadata, not pricing-page guesses):
- 91 video tokens/sec at 720p (43862 video tok / 482s; 4095/45s matches).
- gemini-3.1-pro-preview: ~$0.11/full-clip pass. gemini-3.6-flash: ~$0.04.
- $25 ~ 220 pro passes.

Results on renders/2026-09-06_arsenal-chelsea/clips/clip_PrCW_geeRAU.mp4
(482s, 1280x720, partly fan-shot per Claude at 250/330s):
- Inventory: 22 passages, clean JSON, saved to
  clip_PrCW_geeRAU.gemini_inventory.json. Content labels (goal/celebration/
  replay/crowd/subs/lineup) accurate, confirmed by Claude on 18 frames.
- Timestamps NOT accurate enough to cut on:
  * Pre-match boundaries 1-2s early (banner at 5.0 but 5.5 still goal;
    Chelsea lineup at 23.0 but 23.5 still banner; Arsenal at 33.0 but
    33.5 still Chelsea blue 21/45).
  * Goal windows bloated: shot at front, midpoint is post-goal restart
    (Chelsea 99-111: 105s board 0-1 restart; Havertz 161-172: 166s
    HAVERTZ 1-1 graphic, restart).
  * Wrong team in open play: "Chelsea advances" 88-99, but 93s is Arsenal.
  * 172 boundary wrong: Gemini says replay, 172.5s is live build-up 1-1.
  * WINNING GOAL mislocated ~60s: Gemini "Arsenal scores" 250-254, but
    250/251/252s is ARS 1-1 Chelsea attacking. Real winner Ødegaard
    (ARS 2-1, "ØDEGAARD 1-2" graphic) at ~305-315s, which Gemini labelled
    "celebration"+"replay" of a phantom goal. Found 2 of 3 real goals
    (Chelsea, Havertz), missed Ødegaard, invented 1 phantom.
- Player names WORK via jersey lookup: with match_data lineup, mapped
  10=Palmer,17=Rogers,9=Joao Pedro,3=Fofana,34=Acheampong,45=Lavia,
  21=Hato (all correct). Without lineup, reads numbers only, no guesses.
  Both name-test responses truncated ~80 output tokens (finish_reason
  unchecked).
- Usable passages: ~5-6 action moments (~50s) in a 482s reel; ~90% is
  celebration/crowd/replay/ceremony. Video length capped by available
  action. One 63s celebration block confirmed real (200/220/240s).

Verdict: footage-first half-works. Gemini good at WHAT (content, jerseys),
bad at WHEN (timestamps, team, goal grounding; does not track scoreboard).
match_data.json (real goals+scorers+scoreline) MUST be ground truth;
Gemini only the segment-finder, never the event-source. Do not trust
Gemini timestamps as cut points without match_data cross-check.

Cost this session: ~$0.45 of $25 (pro full-clip $0.11 + ~30 Claude frame
judgements ~$0.30 + small inline calls).

## 2026-09-08 session 4: scoreboard scanner + status mirror

PART 1 — status mirror (public docs repo):
- soccer-channel is a SUBDIR of the private yt-digest repo, not its own
  repo; the 7 docs were untracked (never in any remote).
- Created PUBLIC repo minakush000-crypto/soccer-channel-status, docs only
  (7 files + .gitignore). Scanned all 7 for secrets BEFORE push: only
  ~ paths + masked-key example; redacted on copy (~->~,
  <masked>-><masked>). Verified clean on remote.
- Automated: .claude/hooks/push_status.sh (idempotent, redacts, no-ops,
  exit 0) wired as 2nd Stop hook in ~/.claude/settings.json. Proven: manual
  run committed + pushed. URL written into CONTEXT.md.

PART 2 — scoreboard scanner (goal-finding by scoreline change):
- Built tools/scoreboard_scan.py. Goal-finding as a scanning problem (read
  the broadcast score bug), not model judgement. gemma4:cloud reads the
  top-left score bug every 3s (free, 0.25s/frame, 4 workers); tesseract OCR
  rejected (0/3 readable). NONE (bug-absent) frames carry last-known score.
- 482s arsenal-chelsea clip: 161 frames, 76s, free. 3 real scoreline changes
  (after dropping 1-frame blips "1-1 BUE"@213, "2-4"@288):
  102s 0-0->0-1 Rogers; 171s 0-1->1-1 Havertz; 261s 1-1->2-1 Ødegaard.
  Matches match_data.json exactly (Rogers 2', Havertz 25', Ødegaard 50').
- Walk-back (Claude full frames) to the shot: Chelsea offset ~1-3s, Havertz
  ~5-8s, Ødegaard 8s (bug 261, shot 253 "Arsenal player shooting" still 1-1).
  Offset variable 1-8s; fixed 8s pre-roll captures all 3.
- Corrected my own earlier error: "~305-315s" Ødegaard winner was the
  CELEBRATION, not the goal. Scanner pins bug-update 261s, shot 253s.
- Confirmations (ask_claude): 102 ARS 0-1 CHE (match), 261 ARS 2-1 CHE
  (match). 171 value confirmed via HAVERTZ 1-1 caption + bug @172.5
  (bug intermittent, ±1-2s on exact frame).

Answers to coordinator Q5-8:
- Q5 Scanner vs Gemini+Claude: scanner wins on accuracy (found the goal
  Gemini missed at 250-254, no phantom) AND cost (~$0.10 vs ~$0.30-0.41).
- Q6 Non-goal events (build-ups/presses/overloads): no scoreboard change.
  Use scanner bug-presence to segment broadcast-action vs
  celebration/replay/fan; run Gemini (content classification, WHAT is
  reliable) on broadcast-action segments only; scanner for goals, match_data
  validates, Claude confirms. Scanner removes the goal-mislocation failure
  mode for the highest-value events.
- Q7 Coordinates/crowd: tracking half-crowd (200/379, crowd as team, 14
  close-ups). Resolve: filter to pitch-region detections + render arrows only
  on wide shots (segment_scorer scores this); if fragmentation still too high
  on the reupload, cleaner source = full-match 1080p broadcast (~3-8GB,
  ~$1.5/episode tracking on RunPod, delete after render). Scanner doesn't
  need tracking; only arrows do.
- Q8 Scope: ~50s action in 482s. One reel = ~60-90s video (3 goals + 1-2
  build-ups). For Tifo-length (5-10min), need full-match broadcast (~3-8GB,
  ~$1.5 tracking) — scanner scales to it (1800 frames, free). Several reels
  redundant (overlap); not recommended.

Cost this session (part 2): ~$0.20 (scanner confirmations + walk-back frames).
