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
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=6d5a70ee0f93080d5c9a2278202b66e9c6932fb75a086949dbade57268a9b472; output-bytes=23
- [x] G10: pipeline smoke: produce_v2 and pod_build still parse
  CHECK: ~/yt-digest/.venv/bin/python -c "import ast; ast.parse(open('tools/produce_v2.py').read()); ast.parse(open('tools/pod_build.py').read()); print('PIPE-OK')"
  EXPECT: PIPE-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=afdbfb412303fe91a9d1e181aec00cdb4cc744f9c34ed876804c71475f5d59d9; output-bytes=8
- [x] B3: one 2D renderer; zero matplotlib importers in tools/ (positive control included)
  CHECK: test -f tools/board_page.html && test -f tools/board_html.js && printf 'import matplotlib\n' > /tmp/b08_pc_b3.py && grep -qE "^\s*(import matplotlib|from matplotlib)" /tmp/b08_pc_b3.py && rm -f /tmp/b08_pc_b3.py && n=$(grep -rlE "^\s*import matplotlib|^\s*from matplotlib" tools/*.py | wc -l) && echo "ONE-RENDERER matplotlib-importers=$n"
  EXPECT: ONE-RENDERER matplotlib-importers=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=075f6c4fda1e62134063d83ea711df518f3738048209a044746cc4fd72f1edb1; output-bytes=36
- [x] B6: main repo confirmed private via gh
  CHECK: gh repo view minakush000-crypto/yt-digest --json isPrivate --jq .isPrivate | grep -q true && echo REPO-PRIVATE
  EXPECT: REPO-PRIVATE
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=9712f53ee30824421e58a3d15d13728d6681cdf6ab6f492faa9d76a02a593bf8; output-bytes=13
- [x] B8: the episode slug is a CLI argument, never a module constant (positive control included)
  CHECK: printf 'SLUG = "x"\n' > /tmp/b08_pc_b8.py && grep -q "^SLUG" /tmp/b08_pc_b8.py && rm -f /tmp/b08_pc_b8.py && n=$(grep -c "^SLUG" tools/pod_build.py || true) && echo "SLUG-ARG consts=$n"
  EXPECT: SLUG-ARG consts=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=aebafe882af44eefd50370f71a8351607d9c7818d8bfb7efa8c9e4d5167613b0; output-bytes=18
- [x] B9: gate evidence logs stay git-ignored; the ignore matcher itself is proven (positive control: a tracked file is NOT ignored)
  CHECK: git check-ignore -q reports/gates/brief04.log && ! git check-ignore -q CLAUDE.md && echo GATELOG-IGNORED
  EXPECT: GATELOG-IGNORED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=b804a7d20b957368e047fdf4eecaf59ea9da56967937149b4fa57e7fe64ee7d0; output-bytes=16
- [x] B16: Madrid board numbers match the RAW fetched data (brief 05 job 3, extended brief 06 job 3)
  CHECK: ~/yt-digest/.venv/bin/python tools/board_data_check.py --slug 2026-09-08_real-madrid-inter
  EXPECT: DATA-CHECK OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=998f8436a0863edcfaa2f1f9e13d6cda3fb70885510e49ebab5221e5d31cd3bd; output-bytes=407
- [x] B17: facts come from Sofascore only; match_data.py retired, no caller left
  CHECK: grep -q "sofascore_client.py" tools/produce_v2.py && test ! -e tools/match_data.py && test -e retired/match_data.py && n=$(grep -nE "^\s*(from|import) match_data\b|match_data\.py\"" tools/*.py 2>/dev/null | wc -l); echo "FACTS-SOFA callers-left=$n"
  EXPECT: FACTS-SOFA callers-left=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=511e49a316d32ff4fa75a256daefcef6c58f8f9e1348f9014e3d83a410ce1d6d; output-bytes=26
- [x] B18: extended data check passes on BOTH episodes against the raw fetch
  CHECK: ~/yt-digest/.venv/bin/python tools/board_data_check.py --slug 2026-09-08_real-madrid-inter >/dev/null && ~/yt-digest/.venv/bin/python tools/board_data_check.py --slug 2026-09-12_bournemouth-brentford >/dev/null && echo DATACHECK-BOTH-OK
  EXPECT: DATACHECK-BOTH-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=6cef1fb7a6b8b10146dca828916831750f1706b1796b9ff5aea14b4e1c6e96a1; output-bytes=18
- [x] B19: the rendered-page audit RUNS and enforces: a deliberately mirrored input must FAIL, the real input must PASS
  CHECK: bash tools/pagecheck_proof.sh
  EXPECT: PAGECHECK-PROOF OK mirrored-fail=1 real-pass=1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=737acfa4b7ded0d2b0cf86f1919063a1067a4f51537cc507205d477cadb17739; output-bytes=47
- [x] B20: census is script-driven and TOOLS.md matches tools/ exactly
  CHECK: ~/yt-digest/.venv/bin/python tools/census.py
  EXPECT: CENSUS OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=ca53858eec5910f6dc2e05983ce1c953f48d57886cc87a269e4973d722b3e577; output-bytes=1422
- [x] B22: all work committed AND pushed; the check stages nothing (replaces G12+B15, brief 08 job 4)
  CHECK: test -z "$(git status --porcelain)" && test "$(git rev-parse HEAD)" = "$(git rev-parse @{u})" && echo CLEAN-PUSHED
  EXPECT: CLEAN-PUSHED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=7e11f8b60ada90f702114686f4c992332a3c591b6458f432fa3aa551c69067a0; output-bytes=13
- [x] B23: brief 09 research artifacts exist (3 CSVs, 3 transcripts, measurements, catalog, gaps, durations)
  CHECK: for f in reports/brief09/frames_A.csv reports/brief09/frames_B.csv reports/brief09/frames_C.csv reports/brief09/transcript_A.txt reports/brief09/transcript_B.txt reports/brief09/transcript_C.txt reports/brief09/measurements.md reports/brief09/overlay_catalog.md reports/brief09/gaps.md reports/brief09/durations.json; do test -f "$f" || exit 1; done && echo BRIEF09-ARTIFACTS-OK
  EXPECT: BRIEF09-ARTIFACTS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=8725c126fb5712822dacf56972d1c276bd74b3a1000a7e1a4a4690f0b13ffc69; output-bytes=21
- [x] B24: CSV row counts match the studied region duration/2 within 1 (reads durations.json recorded before the videos were deleted)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief09_check.py rows
  EXPECT: ROWS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=e09ffd77cfa3560d2be1c996aaf4d1fd9ac247f5e5f4fd0371a000c8fffa9755; output-bytes=143
- [x] B25: brief 09 contact sheets exist and total at most 12
  CHECK: n=$(ls reports/brief09/sheet_*.jpg 2>/dev/null | wc -l) && [ "$n" -ge 1 ] && [ "$n" -le 12 ] && echo SHEETS-OK
  EXPECT: SHEETS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=3565f3bd79550109917471627206ec673bbbf94f398bfc914da1708726fc8cae; output-bytes=10
- [x] B26: local benchmark videos deleted after the pod pull (positive control: the planted /tmp .mp4 the matcher must catch)
  CHECK: mkdir -p /tmp/b09_pc && printf x > /tmp/b09_pc/pc.mp4 && n=$(find /tmp/b09_pc -name "*.mp4" 2>/dev/null | wc -l) && rm -rf /tmp/b09_pc && [ "$n" -eq 1 ] && real=$(find /mnt/f/benchmark /mnt/c/Users/muads/Downloads/benchmark -maxdepth 1 -type f \( -name "*.mp4" -o -name "*.mkv" -o -name "*.webm" \) 2>/dev/null | wc -l) && [ "$real" -eq 0 ] && echo BENCH-LOCAL-DELETED
  EXPECT: BENCH-LOCAL-DELETED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=86f9fd29599306b77941b2361713b79fd810bd35cc6b9062547a414fdae7348c; output-bytes=20
- [x] B27: the disk guard exists and is wired to every entry point (5+ callers grepped, not counted by memory)
  CHECK: test -x tools/disk_guard.sh && n=$(grep -rlE "disk_guard\.sh" tools/ .claude/hooks/ 2>/dev/null | grep -v "disk_guard.sh$" | wc -l) && [ "$n" -ge 5 ] && echo GUARD-WIRED callers=$n
  EXPECT: GUARD-WIRED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=b784b78d958ced207308d63c2ef966414513164ac01f938602eb103b7b5cb867; output-bytes=23
- [x] B28: the guard's positive controls FAIL on faked states (100000G floor; unmounted-F target) and the cleanup exemption PASSES
  CHECK: bash tools/disk_guard.sh --selftest
  EXPECT: GUARD-SELFTEST-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=1096bead3681d9db3c9157d52a4fcb207f9b44c270911a3f5a70dcaedeebdfe3; output-bytes=18
- [ ] B30: the five cache+staging env vars resolve to /mnt/d in the sourced env file (rewritten brief 11 job 4 from the brief-10 F: version; XDG_CACHE_HOME still deliberately absent)
  CHECK: s=$HOME/.config/soccer/env.sh && test -f "$s" && [ "$(grep -cE '^export (PIP_CACHE_DIR|UV_CACHE_DIR|npm_config_cache|HF_HOME|SOCCER_STAGING_ROOT)=/mnt/d/scratch/' "$s")" -eq 5 ] && echo CACHES-ON-D
  EXPECT: CACHES-ON-D
- [x] B31: .wslconfig carries the memory cap (half of RAM minus 1 GB, min 4 GB)
  CHECK: grep -qE '^memory=[0-9]+MB' /mnt/c/Users/muads/.wslconfig && echo WSLCONFIG-CAPPED
  EXPECT: WSLCONFIG-CAPPED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=cf10f7c3d0f3cdb388fcf7974ea6e39ba0cd17d76a912efd4182c89338c4b5d5; output-bytes=17
- [x] B32: the keys rescue tarball is on B2 and still lists exactly the file count recorded in its manifest (brief 07 continuation job 1, Mayo option A; reads B2 only, never prints key contents; rewritten 2026-09-27 from the F: version the flapping drive killed — ABANDON resolved by Mayo choosing B2 streaming)
  CHECK: u=b2:mendymax-archive/keys/2026-09-27_soccer-channel-keys.tar.gz; n=$(rclone cat "$u" 2>/dev/null | tar tzf - 2>/dev/null | grep -vc '/$') && m=$(rclone cat b2:mendymax-archive/keys/2026-09-27_soccer-channel-keys.manifest.json 2>/dev/null | ~/yt-digest/.venv/bin/python -c "import json,sys; print(json.load(sys.stdin)['file_count'])") && [ -n "$n" ] && [ "$n" -gt 0 ] && [ "$n" = "$m" ] && echo KEYS-RESCUE-VERIFIED files=$n
  EXPECT: KEYS-RESCUE-VERIFIED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=bc53bc9661ec6008c08f26959e455a003bcc1e22939a5420f6051491d1343f3c; output-bytes=29
- [x] B33: the pre-ECC ~/.claude backup tarball is on B2 with byte-exact size matching its manifest (brief 07 continuation job 2, Mayo option A; reads B2 only; the 16545-entry full listing was verified once at job time by rclone cat | tar tzf - | wc -l and is recorded in the manifest; B2 stores no SHA1 for this rcat large-file, so size-vs-manifest is the durable check)
  CHECK: u=b2:mendymax-archive/backups/2026-09-27_claude_home_pre_ecc.tar.gz; s=$(rclone lsjson "$u" 2>/dev/null | ~/yt-digest/.venv/bin/python -c "import json,sys; print(json.load(sys.stdin)[0]['Size'])") && m=$(rclone cat b2:mendymax-archive/backups/2026-09-27_claude_home_pre_ecc.manifest.json 2>/dev/null | ~/yt-digest/.venv/bin/python -c "import json,sys; print(json.load(sys.stdin)['size_bytes'])") && [ -n "$s" ] && [ "$s" -gt 500000000 ] && [ "$s" = "$m" ] && echo BACKUP-ON-B2 size=$s
  EXPECT: BACKUP-ON-B2
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=b68c9b0f1aaccbfeacef67743ffecb475fda874cd9d24b87fb0f6ad02659ce79; output-bytes=28
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

- [x] B39: ECC doctor is clean at source version 2.2.1 after the install (brief 07 job 8)
  CHECK: d=$(node ~/ECC/scripts/ecc.js doctor 2>&1) && echo "$d" | grep -q "errors=0" && l=$(node ~/ECC/scripts/ecc.js list-installed 2>&1) && echo "$l" | grep -q "Source version: 2.2.1" && echo ECC-DOCTOR-CLEAN-V221
  EXPECT: ECC-DOCTOR-CLEAN-V221
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=255fae43e759ec80d73791dc86d43e2cc6cff650ec789ae380f5d7aef0ceab2b; output-bytes=22
- [x] B40: the unlazy Stop hook AND ECC's dispatcher coexist in ~/.claude/settings.json post-install (brief 07 job 8)
  CHECK: grep -q "stop-hook.mjs" ~/.claude/settings.json && grep -q "pre:bash:dispatcher" ~/.claude/settings.json && echo HOOKS-COEXIST
  EXPECT: HOOKS-COEXIST
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=40ad8a1dc2118c44aae5d4af2f45c59389b6e887049073ad1856d01ff1cba4e6; output-bytes=14
- [x] B41: the disk-guard session-start hook is still wired in project settings and the guard selftest passes post-install (brief 07 job 8)
  CHECK: grep -q "check_mnt_f.sh" .claude/settings.json && bash tools/disk_guard.sh --selftest 2>&1 | grep -q GUARD-SELFTEST-OK && echo GUARD-INTACT
  EXPECT: GUARD-INTACT
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=b102eaf1d7bb41f16a7394e9ef211c3d9b6c4253f7f69bfa40bfe6e194df198c; output-bytes=13
- [x] B42: the precedence file exists and the global CLAUDE.md points to it (brief 07 decision 5)
  CHECK: test -s ~/.claude/rules/precedence.md && grep -q "rules/precedence.md" ~/.claude/CLAUDE.md && echo PRECEDENCE-WIRED
  EXPECT: PRECEDENCE-WIRED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=897744703894031f0e70a9909076a2a876df7d761f4813314a2a65046662eefc; output-bytes=17
- [x] B43: the context-transfer skill is installed with valid frontmatter (brief 07 decision 7)
  CHECK: test -f ~/.claude/skills/context-transfer/SKILL.md && grep -q '^name:' ~/.claude/skills/context-transfer/SKILL.md && echo CT-SKILL-PRESENT
  EXPECT: CT-SKILL-PRESENT
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=0278fde26d143fe0d3f1d2c7c808cbfca3ac62e1d11adddd20caa7707f8dc6c2; output-bytes=17
- [x] B44: the ECC data cap is wired: guard selftest (incl. oversized-ECC control 4) passes and the live report shows the ecc_data line under cap (brief 07 job 5, Mayo instruction)
  CHECK: bash tools/disk_guard.sh --selftest 2>&1 | grep -q GUARD-SELFTEST-OK && bash tools/disk_guard.sh --report 2>/dev/null | grep -qE "ecc_data [0-9]+B <= [0-9]+B cap" && echo ECC-CAP-WIRED
  EXPECT: ECC-CAP-WIRED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=4083c7d2c6597821ff2607367b9783836c8103b0643132360e44d11fe266d52e; output-bytes=14
- [x] B45: before/after session-start context numbers are recorded in both artifacts (brief 07 job 3f + job 6)
  CHECK: test -s reports/brief07/context_before.txt && grep -q "39837" reports/brief07/context_before.txt && test -s reports/brief07/context_after.txt && grep -q "50652" reports/brief07/context_after.txt && echo CTX-BEFORE-AFTER-RECORDED
  EXPECT: CTX-BEFORE-AFTER-RECORDED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=ac2250ca18829020a7553ed272f26bc99e0d1ef55727b796535ef960d9afab6f; output-bytes=26
- [ ] B46: the brief 11 inventory artifact exists with NTFS evidence for D: and the counted /mnt/f hit list (brief 11 job 1)
  CHECK: test -s reports/brief11/inventory.txt && grep -q "NTFS" reports/brief11/inventory.txt && grep -q "MNTF-HITS=" reports/brief11/inventory.txt && echo INVENTORY-OK
  EXPECT: INVENTORY-OK
- [ ] B47: scratch_mount.sh exists and its write-probe check passes on D: (brief 11 job 2)
  CHECK: test -x tools/scratch_mount.sh && bash tools/scratch_mount.sh --check 2>&1 | grep -q D-MOUNT-OK && echo SCRATCH-MOUNT-OK
  EXPECT: SCRATCH-MOUNT-OK
- [ ] B48: the WSL-side 1 GB write and read speeds are recorded with MB/s figures (brief 11 job 3)
  CHECK: test -s reports/brief11/speed.txt && grep -q "WRITE-MBps=" reports/brief11/speed.txt && grep -q "READ-MBps=" reports/brief11/speed.txt && echo SPEED-RECORDED
  EXPECT: SPEED-RECORDED
- [ ] B50: no live config or tool VALUE names /mnt/f as a destination (comment lines documenting the retirement are excluded), while the history file still names it (positive control that the grep works); disk_guard.sh's retired-F detector is exempt by design (brief 11 job 6d)
  CHECK: n=$(grep -rn "/mnt/f" ~/.config/soccer/env.sh ~/.config/yt-dlp/config tools/staging.py tools/scratch_mount.sh tools/b2_mount.sh tools/backup_env.py tools/produce_v2.py tools/f_mount.sh .claude/hooks/check_scratch.sh .claude/settings.json 2>/dev/null | grep -vE ':[0-9]+:#' | wc -l) && [ "$n" -eq 0 ] && h=$(grep -c "/mnt/f" tools/brief10_elevated_setup.sh) && [ "$h" -gt 0 ] && echo NO-LIVE-MNTF history-hits=$h
  EXPECT: NO-LIVE-MNTF
- [ ] B51: the guard selftest output with the brief-11 controls is recorded (D: unmounted FAILS, /mnt/f path FAILS, real state PASSES) (brief 11 job 6a)
  CHECK: test -s reports/brief11/controls.txt && grep -q GUARD-SELFTEST-OK reports/brief11/controls.txt && echo CONTROLS-RECORDED
  EXPECT: CONTROLS-RECORDED
- [ ] B52: the elevated-setup output proves visudo-validated d-* sudoers and the removed f-* rule (brief 11 jobs 2 and 5, Mayo-run script)
  CHECK: test -s reports/brief11/elevated_setup_output.txt && grep -q "VISUDO-OK" reports/brief11/elevated_setup_output.txt && grep -q "F-SUDOERS-REMOVED" reports/brief11/elevated_setup_output.txt && echo SUDOERS-OK
  EXPECT: SUDOERS-OK
- [ ] B53: a real yt-dlp download landed in /mnt/d/scratch (path printed, file deleted after) (brief 11 job 6b)
  CHECK: test -s reports/brief11/ytdlp_download.txt && grep -q "/mnt/d/scratch" reports/brief11/ytdlp_download.txt && echo YTDLP-ON-D
  EXPECT: YTDLP-ON-D
- [ ] B54: a Modal round trip staged its file through /mnt/d/scratch (smallest existing smoke: modal volume put + get) (brief 11 job 6c)
  CHECK: test -s reports/brief11/modal_smoke.txt && grep -q "MODAL-SMOKE-OK" reports/brief11/modal_smoke.txt && grep -q "/mnt/d/scratch" reports/brief11/modal_smoke.txt && echo MODAL-THRU-D
  EXPECT: MODAL-THRU-D
- [ ] B55: the reverify proof ending ALL MET after the last code commit is published in reports/brief11/ (brief 11 job 8)
  CHECK: test -s reports/brief11/reverify.txt && grep -q "ALL MET" reports/brief11/reverify.txt && echo REVERIFY-PUBLISHED
  EXPECT: REVERIFY-PUBLISHED
