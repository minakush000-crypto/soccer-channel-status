# GAPS.md — what nobody has verified

> **Purpose:** registry of unverified claims and open uncertainties.
> **Reader:** every session; mirrored to the public status repo.
> **Last verified against code:** 2026-09-09.

This file existing and being short is a warning sign, not a success. Each
entry is something that could be true or false and nobody has run the command
to find out. If you verify one, move it to STATUS.md or DECISIONS.md and date
it.

## Cloud / GPU

- **runpod_annotate.py end-to-end run**: UNVERIFIED. It exists, it
  tarballs tools/ and ships them, but no log of a full pod run is in the repo.
  CONTEXT.md says "RunPod and Vast.ai both authenticate" but does not show a
  completed annotation job. Falsifier: run it with a slug and check the pod
  log + downloaded tracking JSON.

- **runpod_shorts.py / vastai_shorts.py produce a 1080x1920 output**: UNVERIFIED
  today. The 1080x1920 iraola output (Aug 29) predates the v2 pipeline. Could
  be that cloud encoding works but nobody ran it after the v2 changes.

- **$0.15 / 5 min per episode on an L4**: PARTIALLY VERIFIED 2026-09-05.
  One 10s clip cost $0.05 (720s pod uptime at $0.25/hr, 23s inference). The
  old estimate was for 7 clips; extrapolating linearly gives $0.35 for 7
  clips, but setup is one-time so amortized cost would be lower. The 5-min
  estimate is plausible for inference alone but not for full pod uptime.

## Tracker-to-player mapping

- **Tracker IDs can map to player names**: VERIFIED 2026-09-05 — no path
  exists. grep of cv_annotate.py and match_data.py found no jersey OCR, no
  position heuristics, no manual mapping. match_data.py has jersey numbers
  (line 141) but nothing reads them from video. cv_annotate.py now exports
  per-frame player_positions (Stage 1 done) but every tracker is unnamed.
  Stage 2 is arrows on unnamed players, or hand-mapped. An automated path
  (jersey OCR on the bbox → match to ESPN lineup) would be a separate stage.

## Upload

- **youtube_upload.py OAuth tokens are valid**: UNVERIFIED.
  `secrets/youtube_token.json` exists (Aug 23) but tokens expire. publish-log/
  is empty so no upload has confirmed them. Falsifier: run
  `youtube_upload.py <slug>` and see if it 401s.

## Visual quality

- **cv_annotate.py's output looks acceptable today**: UNVERIFIED. The
  "professional broadcast tracking" verdict in CONTEXT.md is from a prior
  session. Re-run `~/tools/vision_analyze.py` on a current annotated frame
  before relying on it.

- **The latest produce_v2.py output (Sep 5) is visually acceptable**: UNVERIFIED.
  ffprobe confirms dimensions and duration, not quality. Run
  `~/tools/vision_analyze.py` on a frame from
  `renders/2026-08-30_liverpool-forest/shorts/final_video_shorts.mp4` before
  claiming the pipeline produces good video.

## Skill reliability

- **Skill firing is model judgment, not enforced.** The harness lists skills
  and the model decides whether to invoke one; nothing forces a match
  (SKILLS.md). So CONTEXT.md is the only reliable primer — it is read every
  session, skills are not.

## Orphans

- **The ~20 ORPHANED tools in TOOLS.md are truly dead**: UNVERIFIED. They may
  be invoked by hand, by cloud_produce.py on the pod (not grepped line-by-line
  for every tool), or by scripts/ not yet examined. Before deleting any, grep
  the whole tree including scripts/ and briefs/.