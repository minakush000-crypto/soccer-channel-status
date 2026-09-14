# GATES.md — current task gates

> **Purpose:** the live gate file for the current task (unlazy discipline). One
> observable outcome per gate; a gate passes when CHECK prints EXPECT.
> **Reader:** every session; the unlazy Stop hook blocks session end while gates
> remain unmet.
> **Last verified against code:** 2026-09-13.
> **Mirrored:** yes (in push_status.sh ALLOW_PROJECT line 13 and the mirror
> .gitignore line 18 `!GATES.md` — both verified 2026-09-13).

Scope: B2 migration + disk reclamation + board #1 (xG flow chart) build.
Everything produced by any task is archived to B2 (rclone remote "b2", bucket
"mendymax-archive", verified `rclone lsd b2:` → mendymax-archive). Local copies
are working copies, not the record. Doctrine rule 1 (raw footage -> /mnt/f ->
pod -> delete locally) is unchanged; this extends it to generated outputs.

B2 archive state verified 2026-09-13:
- backups/: soccer-pipeline-backup-sep5.tar.gz, yt-digest-backup-sep5.tar.gz
  (already uploaded; `rclone lsf b2:mendymax-archive/backups/` confirmed).
- retired-archive/: soccer-channel/, soccer-pipeline/ (already uploaded;
  `rclone lsf b2:mendymax-archive/retired-archive/` confirmed).
- yt-digest/: full repo snapshot incl .venv/ fragments (5 entries: .gitignore,
  bin/, lib/, pyvenv.cfg, share/) — these are the G4 cleanup target.
- Bucket size: 8.154 GiB / 29037 objects (`rclone size b2:mendymax-archive/`,
  verified 2026-09-13; growing as uploads continue).
- Old renders NOT yet uploaded (local: 2026-08-18_iraola-liverpool 1.3G,
  2026-09-06_arsenal-chelsea 753M, 2026-08-30_liverpool-forest_sep1 538M,
  2026-08-30_liverpool-forest 457M, _20min_test 245M, _real_soccer_test 169M,
  _sharp_test 39M, 2026-09-12_preview-manc-derby 1.7M,
  _fullmatch_arsenal-chelsea-carabao 20K; KEEP 2026-09-12_bournemouth-brentford).

Control scores for G5 (verified from STATUS.md line 396, Gemini authoritative):
- 2D possession control: 4/10 (Gemini, line 389/396).
- 3D formation non-buggy: 6/10 (Gemini, line 394/396).
- The amendment's original numbers (5/10 and 4/10) do not match STATUS.md and
  were NOT used; the verified numbers are used instead.

CLOSING PASS placeholders (after-numbers, filled after file moves):
- Free space after deletions (df -h /).
- Exact list of local dirs/files deleted.
- Bucket size after render upload + .venv cleanup (rclone size).
- Exact B2 path for the old-renders archive.

- [x] G1: regenerable caches cleared
  CHECK: test "$(du -s ~/.cache 2>/dev/null | cut -f1)" -lt 51200 && echo caches_cleared
  EXPECT: caches_cleared
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=51c807a7a98edf7cf251739116fc9ad638df07d49bec55cda478800379d4111d; output-bytes=15
  NOTE: ~/.cache was 3.3G, now 44K (verified `du -sh ~/.cache`). Other regenerable
  caches (npm 330M, bun 5.6M) accepted; they rebuild on demand.

- [x] G2: sep5 backups + retired + old renders archived to B2 and verified
  CHECK: rclone lsf b2:mendymax-archive/backups/ 2>/dev/null | grep -q "yt-digest-backup-sep5" && rclone lsf b2:mendymax-archive/retired-archive/ 2>/dev/null | grep -q "soccer-channel" && rclone lsf b2:mendymax-archive/yt-digest/renders/ 2>/dev/null | grep -q "/" && echo b2_archived
  EXPECT: b2_archived
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=20de6d98c54a7bbdbf6a8dad5867a53d87f774e2c42d27268bfc73e9c8f36bf0; output-bytes=12
  CLOSING PASS: record exact render archive path + object count + bucket size after upload.

- [x] G3: local deletions done (after docs amended + B2 uploads verified)
  CHECK: test ! -f /home/muads/yt-digest-backup-sep5.tar.gz && test ! -f /home/muads/soccer-pipeline-backup-sep5.tar.gz && test ! -d /home/muads/retired && test ! -d renders/_20min_test && test ! -d renders/_real_soccer_test && test ! -d renders/_sharp_test && test ! -d renders/_fullmatch_arsenal-chelsea-carabao && test -d renders/2026-09-12_bournemouth-brentford && echo local_deleted
  EXPECT: local_deleted
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=599d81355de68369aa4be87caccd4ba81e437a82ec2e56fd67a746fee4629979; output-bytes=14
  CLOSING PASS: record df -h / free space after deletions + confirm ~/retired removed.

- [x] G4: bucket .venv fragments cleaned (b2:mendymax-archive/yt-digest/.venv/)
  CHECK: test "$(rclone lsf b2:mendymax-archive/yt-digest/.venv/ 2>/dev/null | wc -l | tr -d ' ')" -eq 0 && echo venv_cleaned
  EXPECT: venv_cleaned
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=8d6f8f1194486ffcb02234ba4a2f3011d19316f096c36792e97dc813fa98f342; output-bytes=13
  CLOSING PASS: record how many objects were purged from the .venv path.

- [x] G5: board #1 (xG flow chart) built and Gemini-judged >= 2D control (4/10) and >= 3D formation (6/10)
  CHECK: test -f artifacts/frames/xg_flow_judgment.txt && grep -qE '([6-9]|10)/10' artifacts/frames/xg_flow_judgment.txt && echo board1_passed
  EXPECT: board1_passed
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=3a553610ed1a41a5c7938ac7e0bdae86725a9c79c29edbfda62922916bfde43b; output-bytes=14
  NOTE: the judgment file is written by gemini_judge.py --out. Board #1 is an
  xG flow chart (~50 lines, 2D timeline, data=shotmap from sofascore_client).
  Control thresholds verified from STATUS.md: 2D possession control = 4/10,
  3D formation non-buggy = 6/10 (Gemini authoritative). Board must score >= 6/10
  (the higher control) to beat both. CLOSING PASS: record exact board path,
  Gemini score, and judgment text.

- [x] G6: doc_stamp_check clean on every touched doc
  CHECK: ~/yt-digest/.venv/bin/python tools/doc_stamp_check.py 2>&1 | grep -E "STALE (GATES|CONTEXT|STATUS|DECISIONS|ARCHITECTURE|TOOLS|LANE_PLAN|GAPS|PROGRESS|EPISODE_SPEC|SCRIPT_TEMPLATE)\.md" >/tmp/g6_stale.txt 2>&1; if [ -s /tmp/g6_stale.txt ]; then cat /tmp/g6_stale.txt; else echo no_stale_touched; fi
  EXPECT: no_stale_touched
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=695f5ef985f662ddd54003b13655a3636b4238f325fd0449f4046ddf0569adce; output-bytes=17
  NOTE: SKILLS.md is currently stale (stamp 2026-09-09) but is NOT touched in
  this task, so it does not block G6. Only docs touched in this task must be
  clean. Verified: `doc_stamp_check.py` reports 1 stale (SKILLS.md) at write time.