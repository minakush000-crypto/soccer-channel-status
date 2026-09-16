# GAPS.md — what nobody has verified

> **Purpose:** registry of unverified claims and open uncertainties.
> **Reader:** every session.
> **Last verified against code:** 2026-09-13.

This file existing and being short is a warning sign, not a success. Each
entry is something that could be true or false and nobody has run the command
to find out. If you verify one, move it to STATUS.md or DECISIONS.md and date
it.

## Disk / B2 archive migration (2026-09-13)

Standing rule added this date: everything produced by any task (renders,
frames, transcripts, artifacts, backups) is archived to B2 (rclone v1.75.1,
remote "b2", bucket "mendymax-archive", verified `rclone lsd b2:` →
`mendymax-archive`). Local copies are working copies, not the record. Doctrine
rule 1 (raw footage -> /mnt/f -> pod -> delete locally) is unchanged; this
extends it to generated outputs. Open items from this migration:

- **Doctrine rule 1 not followed for renders**: produce_v2.py writes renders
  inside /home/muads (`grep -n 'RENDERS' tools/produce_v2.py` → line 38
  `RENDERS = SCRIPT_DIR / "renders"`, with callers at lines 101 and 743).
  SCRIPT_DIR is the project folder under /home/muads, so every render lands
  on the ext4.vhdx and it grows. Being fixed via the B2 archive +
  delete-old-keep-working-copy pattern: archive the render to B2, keep only
  the latest working copy locally, delete the rest after the upload verifies.
  Not yet proven end-to-end; the rule is standing but the cleanup has not been
  run on the full renders/ tree. CLOSING PASS: free-space-after on / after
  the renders/ cleanup runs.

- **ext4.vhdx compaction pending**: `ls -la` on the vhdx reports
  39530266624 bytes (36.82 GB) at
  `/mnt/c/Users/muads/AppData/Local/wsl/{f2ea779f-e5f1-4c82-a1b2-0608e6ab4883}/ext4.vhdx`.
  The vhdx grows and never shrinks on its own; deleting inside Ubuntu frees
  blocks inside the image but the file on C: stays the same size. Compaction
  requires WSL fully stopped (a separate operation Mayo runs outside WSL).
  `df -h /` shows 29G used of 1007G on /, so inode pressure is not the issue;
  the issue is the C: host file (see Context). CLOSING PASS: vhdx size after
  compaction (whenever Mayo runs it).

- **Other-project folders awaiting Mayo's decision**: five non-soccer folders
  under /home/muads are candidates for archive-then-delete. Verified sizes
  (`du -sh`): cv-test 2.7G, mcp-servers 1.3G, quant-solana-gate2 517M,
  skills-lab 474M, .claude-mem 396M. These are archived to B2, NOT deleted;
  Mayo decides whether to delete the local copies. CLOSING PASS: which folders
  were actually deleted and free-space-after.

- **/mnt/f EINVAL write issue**: fresh file creation in /mnt/f fails with
  EINVAL on every tested path — `touch /mnt/f/__w` → "Invalid argument";
  `echo > /mnt/f/__w` → "Invalid argument"; `cp <src> /mnt/f/__new` →
  "Invalid argument"; `mv /tmp/__x /mnt/f/__y` (cross-filesystem) →
  "Invalid argument". Rename of an existing file *within* /mnt/f DOES work
  (`mv /mnt/f/.dropbox.device /mnt/f/__r` → exit 0, rename back → exit 0), so
  a durable-clip-via-rename pattern is the workaround: create via a path
  that yields an existing file, then rename in place. Not yet wired into any
  tool. Falsifier: a tool that relies on `open(..., 'w')` to /mnt/f will fail.

## Cloud / GPU

- **3D Renderer**: The new local `three_scene.js`/`three_render3d.py` path is unverified beyond a single frame. It replaced the unverified Modal/Blender path.
- **EINVAL fallback**: The staging `/mnt/f` EINVAL fallback to `/dev/shm` is untested and undocumented.
- **Player Mapper**: `player_mapper.py` is present but completely untested, undocumented, and orphaned (violating Rule 3).

- **runpod_annotate.py end-to-end run**: RETIRED Stage 6 (deleted). The tool
  no longer exists on disk (`ls tools/runpod_annotate.py` → No such file).
  Its function (ship cv_annotate.py to RunPod) is now handled by
  `runpod_fulltrack.py`, which IS proven (146s clip, 156s wall, $0.011, per
  STATUS.md). No further verification needed — the tool is gone.

- **runpod_shorts.py / vastai_shorts.py produce a 1080x1920 output**: RETIRED
  Stage 6 (both deleted). `ls tools/runpod_shorts.py tools/vastai_shorts.py`
  → No such file for either. The 1080x1920 iraola output (Aug 29) predates
  the v2 pipeline and these tools. shorts_crop.py (still present) handles
  vertical output locally now.

- **$0.15 / 5 min per episode on an L4**: PARTIALLY VERIFIED 2026-09-05.
  One 10s clip cost $0.05 (720s pod uptime at $0.25/hr, 23s inference) via
  runpod_annotate.py (now RETIRED Stage 6, deleted). The replacement tool
  runpod_fulltrack.py is proven cheaper: 146s clip, 156s wall, $0.011 (per
  STATUS.md). The original $0.15 estimate was for 7 clips via the deleted
  tool; the current tool's per-clip cost is ~$0.011, making 7 clips ~$0.08.
  The 5-min estimate is plausible for inference alone but not for full pod
  uptime.

## Tracker-to-player mapping

- **Tracker IDs can map to player names**: VERIFIED 2026-09-05 — no path
  exists. grep of cv_annotate.py and match_data.py found no jersey OCR, no
  position heuristics, no manual mapping. match_data.py has jersey numbers
  (line 141) but nothing reads them from video. cv_annotate.py now exports
  per-frame player_positions (Stage 1 done) but every tracker is unnamed.
  Stage 2 is arrows on unnamed players, or hand-mapped. An automated path
  (jersey OCR on the bbox → match to ESPN lineup) would be a separate stage.

## Upload

- **Hooks/Sandboxing**: Gemini CLI currently runs with NO sandbox and NO automatic hooks. Tools like `push_status.sh` and `check_pods.sh` must be executed manually. This gap exposes the system to drift if the human operator forgets to run them.

- **youtube_upload.py OAuth tokens are valid**: PARTIALLY VERIFIED 2026-09-06.
  `secrets/youtube_token.json` exists (mtime Sep 8, not Aug 23 as previously
  claimed). publish-log/ has 2 entries (2026-09-06_arsenal-chelsea
  PsEz5ITTIpM + WFi2LBwXINU), confirming 2 successful private uploads on
  2026-09-06. Tokens were valid as of that date; expiry since then is still
  possible. Falsifier: run `youtube_upload.py <slug>` today and see if it
  401s.

## Visual quality

- **cv_annotate.py's output looks acceptable today**: UNVERIFIED. The
  "professional broadcast tracking" verdict in CONTEXT.md is from a prior
  session. Re-run `~/tools/vision_analyze.py` on a current annotated frame
  before relying on it.

- **The produce_v2.py output is visually acceptable**: UNVERIFIED.
  The liverpool-forest shorts file (referenced below) has mtime Sep 7 (not
  "Sep 5" as previously claimed). It is no longer the latest output: the
  latest produce_v2 output is
  `renders/2026-09-12_bournemouth-brentford/final_video.mp4` (mtime Sep 13
  21:27, 1920x1080, 718.0s, 143MB per `stat -c '%s'` = 149425972). ffprobe
  confirms dimensions and duration, not quality. Run `~/tools/vision_analyze.py` on a frame from either file before
  claiming the pipeline produces good video.

## Skill reliability

- **Skill firing is model judgment, not enforced.** The harness lists skills
  and the model decides whether to invoke one; nothing forces a match
  (SKILLS.md). So CONTEXT.md is the only reliable primer — it is read every
  session, skills are not.

## Orphans

- **ORPHANED tools in TOOLS.md**: RESOLVED. `grep -c ORPHANED TOOLS.md` → 1,
  which is the definition line itself (`"ORPHANED" = no caller anywhere`).
  Zero tool rows are labeled ORPHANED. The term is a definition, not a
  category in active use. The prior "~20 ORPHANED tools" claim was wrong.