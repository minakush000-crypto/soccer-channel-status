# Gates: the LIVE ledger (rewritten brief 08, jobs 2-4)

What happened (why this file was rewritten): brief 06 reported "33/33 gates
pass" from a default `gate-check` run, which only EXECUTES gates whose boxes
are unchecked; the 28 pre-checked gates were trusted from stored evidence.
The coordinator re-ran four by hand on 2026-09-27 and all four FAIL today
(G1 py=37 vs 35, G4 140.920000 vs 140.960000, G7 stamp 2026-09-22 vs docs
restamped 2026-09-27, B14 5 stamps vs 1). The brief-06 work itself checked
out where tested; the ledger was the failure.

Rules for this ledger (Mayo, brief 08 closed decisions 1-4):
- LIVE = an invariant that must hold on every future commit. Everything that
  snapshots a past state (durations, dates, copied counts, one-time
  deletions/proofs) is FROZEN: moved verbatim with its evidence to
  reports/<its brief>/GATES_frozen.md, headed "historical, not run" (22
  gates: brief03 10, brief04 11, brief06 1 — table with one-line reasons in
  reports/brief08/gate_triage.md).
- No CHECK may modify the repo: no `git add`, no writes outside /tmp.
- Every absence check carries a positive control (a planted file the matcher
  must catch) — an absence check without one proves nothing.
- `--status` reads stored evidence and is NEVER proof. Only `--reverify`
  output that post-dates the last code commit, published to the mirror,
  counts (doctrine rule 7).

- [x] G2: no kept tool references a retired tool as live (positive control included)
  CHECK: printf 'import scene_gen\nscene_gen("x")\n' > /tmp/b08_pc_g2.py && grep -qE "(import scene_gen\b)|(\bscene_gen\()" /tmp/b08_pc_g2.py && rm -f /tmp/b08_pc_g2.py && n=$(grep -rnE "(import (scene_gen|modal_render3d|runpod_fulltrack|generate_captions|assemble_words_match)\b|from (scene_gen|modal_render3d|runpod_fulltrack|generate_captions|assemble_words_match) import)|\b(scene_gen|modal_render3d|runpod_fulltrack|generate_captions|assemble_words_match)\(" tools/*.py | wc -l) && echo "REFS-CLEAN live-refs=$n"
  EXPECT: REFS-CLEAN live-refs=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=6d5a70ee0f93080d5c9a2278202b66e9c6932fb75a086949dbade57268a9b472; output-bytes=23
- [x] G10: pipeline smoke: produce_v2 and pod_build still parse
  CHECK: ~/yt-digest/.venv/bin/python -c "import ast; ast.parse(open('tools/produce_v2.py').read()); ast.parse(open('tools/pod_build.py').read()); print('PIPE-OK')"
  EXPECT: PIPE-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=afdbfb412303fe91a9d1e181aec00cdb4cc744f9c34ed876804c71475f5d59d9; output-bytes=8
- [x] B3: one 2D renderer; zero matplotlib importers in tools/ (positive control included)
  CHECK: test -f tools/board_page.html && test -f tools/board_html.js && printf 'import matplotlib\n' > /tmp/b08_pc_b3.py && grep -qE "^\s*(import matplotlib|from matplotlib)" /tmp/b08_pc_b3.py && rm -f /tmp/b08_pc_b3.py && n=$(grep -rlE "^\s*import matplotlib|^\s*from matplotlib" tools/*.py | wc -l) && echo "ONE-RENDERER matplotlib-importers=$n"
  EXPECT: ONE-RENDERER matplotlib-importers=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=075f6c4fda1e62134063d83ea711df518f3738048209a044746cc4fd72f1edb1; output-bytes=36
- [x] B6: main repo confirmed private via gh
  CHECK: gh repo view minakush000-crypto/yt-digest --json isPrivate --jq .isPrivate | grep -q true && echo REPO-PRIVATE
  EXPECT: REPO-PRIVATE
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=9712f53ee30824421e58a3d15d13728d6681cdf6ab6f492faa9d76a02a593bf8; output-bytes=13
- [x] B8: the episode slug is a CLI argument, never a module constant (positive control included)
  CHECK: printf 'SLUG = "x"\n' > /tmp/b08_pc_b8.py && grep -q "^SLUG" /tmp/b08_pc_b8.py && rm -f /tmp/b08_pc_b8.py && n=$(grep -c "^SLUG" tools/pod_build.py || true) && echo "SLUG-ARG consts=$n"
  EXPECT: SLUG-ARG consts=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=aebafe882af44eefd50370f71a8351607d9c7818d8bfb7efa8c9e4d5167613b0; output-bytes=18
- [x] B9: gate evidence logs stay git-ignored; the ignore matcher itself is proven (positive control: a tracked file is NOT ignored)
  CHECK: git check-ignore -q reports/gates/brief04.log && ! git check-ignore -q CLAUDE.md && echo GATELOG-IGNORED
  EXPECT: GATELOG-IGNORED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=b804a7d20b957368e047fdf4eecaf59ea9da56967937149b4fa57e7fe64ee7d0; output-bytes=16
- [x] B16: Madrid board numbers match the RAW fetched data (brief 05 job 3, extended brief 06 job 3)
  CHECK: ~/yt-digest/.venv/bin/python tools/board_data_check.py --slug 2026-09-08_real-madrid-inter
  EXPECT: DATA-CHECK OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=998f8436a0863edcfaa2f1f9e13d6cda3fb70885510e49ebab5221e5d31cd3bd; output-bytes=407
- [x] B17: facts come from Sofascore only; match_data.py retired, no caller left
  CHECK: grep -q "sofascore_client.py" tools/produce_v2.py && test ! -e tools/match_data.py && test -e retired/match_data.py && n=$(grep -nE "^\s*(from|import) match_data\b|match_data\.py\"" tools/*.py 2>/dev/null | wc -l); echo "FACTS-SOFA callers-left=$n"
  EXPECT: FACTS-SOFA callers-left=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=511e49a316d32ff4fa75a256daefcef6c58f8f9e1348f9014e3d83a410ce1d6d; output-bytes=26
- [x] B18: extended data check passes on BOTH episodes against the raw fetch
  CHECK: ~/yt-digest/.venv/bin/python tools/board_data_check.py --slug 2026-09-08_real-madrid-inter >/dev/null && ~/yt-digest/.venv/bin/python tools/board_data_check.py --slug 2026-09-12_bournemouth-brentford >/dev/null && echo DATACHECK-BOTH-OK
  EXPECT: DATACHECK-BOTH-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=6cef1fb7a6b8b10146dca828916831750f1706b1796b9ff5aea14b4e1c6e96a1; output-bytes=18
- [x] B19: the rendered-page audit RUNS and enforces: a deliberately mirrored input must FAIL, the real input must PASS
  CHECK: bash tools/pagecheck_proof.sh
  EXPECT: PAGECHECK-PROOF OK mirrored-fail=1 real-pass=1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=737acfa4b7ded0d2b0cf86f1919063a1067a4f51537cc507205d477cadb17739; output-bytes=47
- [x] B20: census is script-driven and TOOLS.md matches tools/ exactly
  CHECK: ~/yt-digest/.venv/bin/python tools/census.py
  EXPECT: CENSUS OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=ca53858eec5910f6dc2e05983ce1c953f48d57886cc87a269e4973d722b3e577; output-bytes=1422
- [x] B22: all work committed AND pushed; the check stages nothing (replaces G12+B15, brief 08 job 4)
  CHECK: test -z "$(git status --porcelain)" && test "$(git rev-parse HEAD)" = "$(git rev-parse @{u})" && echo CLEAN-PUSHED
  EXPECT: CLEAN-PUSHED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=7e11f8b60ada90f702114686f4c992332a3c591b6458f432fa3aa551c69067a0; output-bytes=13
- [x] B23: brief 09 research artifacts exist (3 CSVs, 3 transcripts, measurements, catalog, gaps, durations)
  CHECK: for f in reports/brief09/frames_A.csv reports/brief09/frames_B.csv reports/brief09/frames_C.csv reports/brief09/transcript_A.txt reports/brief09/transcript_B.txt reports/brief09/transcript_C.txt reports/brief09/measurements.md reports/brief09/overlay_catalog.md reports/brief09/gaps.md reports/brief09/durations.json; do test -f "$f" || exit 1; done && echo BRIEF09-ARTIFACTS-OK
  EXPECT: BRIEF09-ARTIFACTS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=8725c126fb5712822dacf56972d1c276bd74b3a1000a7e1a4a4690f0b13ffc69; output-bytes=21
- [x] B24: CSV row counts match the studied region duration/2 within 1 (reads durations.json recorded before the videos were deleted)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief09_check.py rows
  EXPECT: ROWS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=e09ffd77cfa3560d2be1c996aaf4d1fd9ac247f5e5f4fd0371a000c8fffa9755; output-bytes=143
- [x] B25: brief 09 contact sheets exist and total at most 12
  CHECK: n=$(ls reports/brief09/sheet_*.jpg 2>/dev/null | wc -l) && [ "$n" -ge 1 ] && [ "$n" -le 12 ] && echo SHEETS-OK
  EXPECT: SHEETS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=3565f3bd79550109917471627206ec673bbbf94f398bfc914da1708726fc8cae; output-bytes=10
- [x] B26: local benchmark videos deleted after the pod pull (positive control: the planted /tmp .mp4 the matcher must catch)
  CHECK: mkdir -p /tmp/b09_pc && printf x > /tmp/b09_pc/pc.mp4 && n=$(find /tmp/b09_pc -name "*.mp4" 2>/dev/null | wc -l) && rm -rf /tmp/b09_pc && [ "$n" -eq 1 ] && real=$(find /mnt/f/benchmark /mnt/c/Users/muads/Downloads/benchmark -maxdepth 1 -type f \( -name "*.mp4" -o -name "*.mkv" -o -name "*.webm" \) 2>/dev/null | wc -l) && [ "$real" -eq 0 ] && echo BENCH-LOCAL-DELETED
  EXPECT: BENCH-LOCAL-DELETED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=86f9fd29599306b77941b2361713b79fd810bd35cc6b9062547a414fdae7348c; output-bytes=20
- [x] B27: the disk guard exists and is wired to every entry point (5+ callers grepped, not counted by memory)
  CHECK: test -x tools/disk_guard.sh && n=$(grep -rlE "disk_guard\.sh" tools/ .claude/hooks/ 2>/dev/null | grep -v "disk_guard.sh$" | wc -l) && [ "$n" -ge 5 ] && echo GUARD-WIRED callers=$n
  EXPECT: GUARD-WIRED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=b784b78d958ced207308d63c2ef966414513164ac01f938602eb103b7b5cb867; output-bytes=23
- [x] B28: the guard's positive controls FAIL on faked states (100000G floor; unmounted-F target) and the cleanup exemption PASSES
  CHECK: bash tools/disk_guard.sh --selftest
  EXPECT: GUARD-SELFTEST-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=1096bead3681d9db3c9157d52a4fcb207f9b44c270911a3f5a70dcaedeebdfe3; output-bytes=18
- [x] B29: /mnt/f is mounted AND passes a real write probe (brief 10 job 2, post chkdsk repair)
  CHECK: bash tools/f_mount.sh --check
  EXPECT: F-MOUNT-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=77df0adb69a16d87387dfc448d327ffd7ea447ffc358113ccb305b0658e1c1ec; output-bytes=11
- [x] B30: the four cache env vars resolve to /mnt/f in the sourced env file (XDG_CACHE_HOME deliberately absent: Chrome stays off the flash drive)
  CHECK: s=$HOME/.config/soccer/env.sh && test -f "$s" && [ "$(grep -cE '^(export )?(PIP_CACHE_DIR|UV_CACHE_DIR|npm_config_cache|HF_HOME)=/mnt/f' "$s")" -eq 4 ] && echo CACHES-ON-F
  EXPECT: CACHES-ON-F
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=9815489de0d2971471393e6d10eaecb363cfab04b3be1a056114c06d3226abb8; output-bytes=12
- [x] B31: .wslconfig carries the memory cap (half of RAM minus 1 GB, min 4 GB)
  CHECK: grep -qE '^memory=[0-9]+MB' /mnt/c/Users/muads/.wslconfig && echo WSLCONFIG-CAPPED
  EXPECT: WSLCONFIG-CAPPED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=cf10f7c3d0f3cdb388fcf7974ea6e39ba0cd17d76a912efd4182c89338c4b5d5; output-bytes=17
- [ ] B32: the ~/.claude + ~/.claude.json backup tarball exists on F: and its entry list is counted (brief 07 job 1)
  CHECK: t=/mnt/f/backups/claude-backup-2026-09-27.tar.gz && test -s "$t" && n=$(tar tzf "$t" | wc -l) && [ "$n" -gt 100 ] && echo BACKUP-ON-F entries=$n
  EXPECT: BACKUP-ON-F
- [ ] B33: the backup tarball is on B2 under soccer-channel/2026-09-27/ with a byte-identical size (brief 07 job 1)
  CHECK: l=$(stat -c%s /mnt/f/backups/claude-backup-2026-09-27.tar.gz) && r=$(rclone lsl b2:mendymax-archive/soccer-channel/2026-09-27/claude-backup-2026-09-27.tar.gz | awk '{print $1}') && [ -n "$r" ] && [ "$l" = "$r" ] && echo B2-SIZE-MATCH bytes=$l
  EXPECT: B2-SIZE-MATCH
- [x] B34: the ECC clone is checked out at tag v2.2.1 exactly (brief 07 job 2)
  CHECK: git -C ~/ECC describe --tags --exact-match 2>/dev/null | grep -qx v2.2.1 && echo ECC-PINNED-V221
  EXPECT: ECC-PINNED-V221
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=d065a72298616d1a0095816c5667e30324999ca25fcd39fd200cdd70b5b14371; output-bytes=16
- [x] B35: the dry-run plan files exist and the JSON one parses (brief 07 job 3)
  CHECK: ~/yt-digest/.venv/bin/python -c "import json; json.load(open('reports/brief07/ecc_plan.json')); print('PLAN-OK')" && test -s reports/brief07/ecc_plan.txt
  EXPECT: PLAN-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=edab1af7911171143f404e059437e3f2580c9477fbf5546f5723e780ab523c8d; output-bytes=8
- [x] B36: the six analysis artifacts are non-empty (counts, clashes, hooks, settings diff, size estimate, context before) (brief 07 job 3)
  CHECK: for f in counts.txt clashes.txt hooks.txt settings_diff.txt size_estimate.txt context_before.txt; do test -s "reports/brief07/$f" || exit 1; done && echo BRIEF07-ANALYSIS-OK
  EXPECT: BRIEF07-ANALYSIS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=f2c719f19f820f314bc8d6935b6c68c7d0c4a0fb514fae40876bf9750c676ad4; output-bytes=20
- [x] B37: C: free space stays above the 15 GB floor at the end of brief 07 jobs 1-4 (brief 07 job 2 stop rule)
  CHECK: a=$(df -BG --output=avail /mnt/c | tail -1 | tr -dc '0-9') && [ "$a" -ge 15 ] && echo C-FLOOR-OK avail=${a}G
  EXPECT: C-FLOOR-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=f319052f28fd45e453c4d6dc2a05894750324b34089c2bbef6cd67569570178e; output-bytes=21
- [x] B38: HARD STOP honored: summary exists and ECC's rules dir is NOT installed, with a planted positive control proving the matcher (brief 07 job 4; FREEZE this gate once Mayo types APPROVED ECC — jobs 5-8 install into rules/)
  CHECK: m=$(mktemp -d /tmp/b07pc.XXXXXX) && mkdir -p "$m/rules/ecc" && test -e "$m/rules/ecc" && rm -rf "$m" && test -s reports/brief07/summary.md && test ! -e ~/.claude/rules/ecc && echo HARD-STOP-OK
  EXPECT: HARD-STOP-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=342e3ccdbf3cb67c6dedda020e29f7f325c8dc33cbc29cd327dc15c724d8c6b9; output-bytes=13

ABANDON: B32 F: drive (Memorex USB, serial 0A7710C44) disconnected from Windows twice under sustained write on 2026-09-27 (NTFS event 140, 21:35 CDT; 1.6 MB/s with readback failures); tarball reached only 147 MB of ~1.7 GB; rollback backup on a flapping drive risks silent corruption; handoff to Mayo in reports/brief07/f_drive_failure.md, gate reruns as-is once the drive is healthy
ABANDON: B33 depends on B32's tarball, which does not exist because of the same F: hardware failure; handoff to Mayo in reports/brief07/f_drive_failure.md
