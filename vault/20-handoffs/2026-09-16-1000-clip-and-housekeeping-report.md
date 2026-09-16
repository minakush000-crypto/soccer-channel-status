Answers brief: 2026-09-16-1000-clip-and-housekeeping.md

# Task A: Housekeeping & Git Push

**1. Git Push Fix**
The broken `git push` in the mirror repo was caused by CLI tools injecting non-interactive environment variables (`GIT_ASKPASS`, `GIT_CONFIG_GLOBAL`, etc.) that bypassed the `store` credential helper. This was fixed without exposing any tokens by explicitly scrubbing these variables using `env -u` inside `.claude/hooks/push_status.sh`. 
- Modified `.claude/hooks/push_status.sh` to use: `env -u GIT_CONFIG_GLOBAL -u GIT_CONFIG_SYSTEM -u GIT_CONFIG_NOSYSTEM -u GIT_CONFIG_COUNT -u GIT_CONFIG_KEY_0 -u GIT_CONFIG_VALUE_0 -u GIT_ASKPASS git push origin HEAD`
- Proof:
```
Enumerating objects: 21, done.
Writing objects: 100% (15/15), 5.61 KiB | 1.40 MiB/s, done.
Total 15 (delta 9), reused 0 (delta 0), pack-reused 0 (from 0)
To https://github.com/minakush000-crypto/soccer-channel-status.git
   d276d9a..0a7ee82  main -> main
```
Commit hashes pushed: `fb1fb2a` and `0a7ee82`.

**2. Judge Model Pinning**
The model `gemini-pro-latest` was incorrectly stated as the stable model; the actual stable visual model is `gemini-2.5-pro` (the older `gemini-3.1-pro-preview` was deprecated and throws 404s). `gemini_judge.py` was updated to pin `gemini-2.5-pro`.
- Stability test spread on unchanged boards:
  - `board_formation.png`: 5/10, 5/10, 4/10 (Spread = 1)
  - `board_stat_card.png`: 3/10, 3/10, 2/10 (Spread = 1)
  - `board_xg_flow.png`: 9/10, 9/10, 9/10 (Spread = 0)
The maximum spread is 1 point. All older unpinned scores are superseded.

# Section 10: Environment Drift
The project's canonical documents (`GEMINI.md`, `CONTEXT.md`, `STATUS.md`, `ARCHITECTURE.md`, `TOOLS.md`, `GAPS.md`, `DECISIONS.md`) were completely out of sync with the reality of running under Gemini CLI. 
- **GEMINI.md completely rewritten** to assert that the agent is Gemini CLI, automatic hooks (like `push_status.sh` and `check_pods.sh`) DO NOT FIRE automatically and must be called manually, and that the execution environment has NO sandbox or permission guards.
- All documents updated and stamped `2026-09-16`.

# Task B: Answer The Format Question
**Does EPISODE_SPEC describe the product, or does the benchmark video?**
The *benchmark video* (the Cesc Fabregas tactical analysis) describes the intended product. `EPISODE_SPEC` describes a data-heavy, chart-centric format that the owner has explicitly rejected ("It just shows playing notes and match statistics that are in 2D"). 

**Measurements:** I cannot watch the video to measure the exact footage-to-graphics ratio mechanically. However, based on the transcript analysis (Section 2) stating that "Nearly every tactical point is made by playing or freezing a real clip and drawing on it" and "Almost no charts", the ratio is structurally inverted from the spec.

**If the spec should change, what MUSTs change?**
1. **The 60% Graphics Floor & 20% Footage Ceiling:** This must be inverted or removed entirely. The new format requires footage to be the primary visual carrier (likely >70% footage).
2. **The Three MUST Graphic Types:** Pitch diagram, formation board, and stat card MUST be removed as mandatory items. They can be demoted to "SHOULD" or "MAY" for previews, but are not the core analysis.
3. **Board Design Criteria (Dark Background / Selective Visibility):** These criteria cannot apply to live broadcast footage. Footage requires its own criteria: tracking clarity, accurate anchor points, and contrasting overlay colors (e.g. bright orange arrows over natural green pitches).

# Task C: Build One Goal Clip
A 13-second raw AV1 clip of Schade's 1-0 goal (`clip_HfX417hSwPM.mp4`) was extracted from the B2/cache storage. It was re-encoded to h264 and run through `cv_annotate.py` to get per-frame YOLOv8/ByteTrack tracker coordinates. Tracker ID 15 was identified manually as Schade making the run.

A python script (`render_goal_clip.py`) was written to draw an orange arrow, an ellipse base, and a "K. Schade" name tag over the player dynamically, using real coordinates from the JSON, producing `goal_clip_tactical.mp4`.

**Did it look like the benchmark, or like the retired clipart overlay?**
*Blunt answer: It looked like the retired clipart overlay.* 
The pinned judge (`gemini-2.5-pro`) failed due to 404 (quota/access), but the fallback `gemini-3.6-flash` accurately summarized it: *"No, it lacks the polished graphics, tracked lighting, and clean broadcast integration... It resembles a basic visual graphic or retired clip-art overlay rather than a modern professional telestration tool. The overlay consists of a solid black text box with yellow font... A basic orange arrow... A simple orange oval."*

**What would twelve of these cost, in time and money, end to end?**
- **Time:** Generating tracking JSON locally took ~15 seconds per clip. The real cost is human time. Finding the correct tracker ID from 83 generated IDs took ~5 minutes of manual inspection. For 12 clips, that's 60 minutes of human mapping per episode.
- **Money:** The inference was run on a local CPU (0 cost). Even on RunPod, the cost for 12 short clips would be < $0.10.

**What breaks if you scale from one clip to a full episode of them?**
1. **Tracker Fragmentation:** Any camera angle change creates a new tracker ID. The human operator would have to map the player *again* for every cut within the same highlight.
2. **Jitter:** Unsmoothed bounding boxes from ByteTrack cause the drawn arrows and name tags to vibrate noticeably frame-to-frame.
3. **Typography/Styling:** Raw OpenCV `cv2.putText` cannot easily render custom OTF fonts (Bebas Neue) or do broadcast-level blending (drop shadows, tracking lighting) without a heavier graphics framework (like After Effects or a complex HTML canvas overlay pipeline).
4. **AV1 Decoding:** The raw YouTube downloads were AV1, which broke `cv_annotate.py`'s native OpenCV reader, requiring an intermediate ffmpeg h264 transcode step.