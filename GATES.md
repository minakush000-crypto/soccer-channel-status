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
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=6d5a70ee0f93080d5c9a2278202b66e9c6932fb75a086949dbade57268a9b472; output-bytes=23
- [x] G10: pipeline smoke: produce_v2 and pod_build still parse
  CHECK: ~/yt-digest/.venv/bin/python -c "import ast; ast.parse(open('tools/produce_v2.py').read()); ast.parse(open('tools/pod_build.py').read()); print('PIPE-OK')"
  EXPECT: PIPE-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=afdbfb412303fe91a9d1e181aec00cdb4cc744f9c34ed876804c71475f5d59d9; output-bytes=8
- [x] B3: one 2D renderer; zero matplotlib importers in tools/ (positive control included)
  CHECK: test -f tools/board_page.html && test -f tools/board_html.js && printf 'import matplotlib\n' > /tmp/b08_pc_b3.py && grep -qE "^\s*(import matplotlib|from matplotlib)" /tmp/b08_pc_b3.py && rm -f /tmp/b08_pc_b3.py && n=$(grep -rlE "^\s*import matplotlib|^\s*from matplotlib" tools/*.py | wc -l) && echo "ONE-RENDERER matplotlib-importers=$n"
  EXPECT: ONE-RENDERER matplotlib-importers=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=075f6c4fda1e62134063d83ea711df518f3738048209a044746cc4fd72f1edb1; output-bytes=36
- [x] B6: main repo confirmed private via gh
  CHECK: gh repo view minakush000-crypto/yt-digest --json isPrivate --jq .isPrivate | grep -q true && echo REPO-PRIVATE
  EXPECT: REPO-PRIVATE
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=9712f53ee30824421e58a3d15d13728d6681cdf6ab6f492faa9d76a02a593bf8; output-bytes=13
- [x] B8: the episode slug is a CLI argument, never a module constant (positive control included)
  CHECK: printf 'SLUG = "x"\n' > /tmp/b08_pc_b8.py && grep -q "^SLUG" /tmp/b08_pc_b8.py && rm -f /tmp/b08_pc_b8.py && n=$(grep -c "^SLUG" tools/pod_build.py || true) && echo "SLUG-ARG consts=$n"
  EXPECT: SLUG-ARG consts=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=aebafe882af44eefd50370f71a8351607d9c7818d8bfb7efa8c9e4d5167613b0; output-bytes=18
- [x] B9: gate evidence logs stay git-ignored; the ignore matcher itself is proven (positive control: a tracked file is NOT ignored)
  CHECK: git check-ignore -q reports/gates/brief04.log && ! git check-ignore -q CLAUDE.md && echo GATELOG-IGNORED
  EXPECT: GATELOG-IGNORED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=b804a7d20b957368e047fdf4eecaf59ea9da56967937149b4fa57e7fe64ee7d0; output-bytes=16
- [x] B16: Madrid board numbers match the RAW fetched data (brief 05 job 3, extended brief 06 job 3)
  CHECK: ~/yt-digest/.venv/bin/python tools/board_data_check.py --slug 2026-09-08_real-madrid-inter
  EXPECT: DATA-CHECK OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=998f8436a0863edcfaa2f1f9e13d6cda3fb70885510e49ebab5221e5d31cd3bd; output-bytes=407
- [x] B17: facts come from Sofascore only; match_data.py retired, no caller left
  CHECK: grep -q "sofascore_client.py" tools/produce_v2.py && test ! -e tools/match_data.py && test -e retired/match_data.py && n=$(grep -nE "^\s*(from|import) match_data\b|match_data\.py\"" tools/*.py 2>/dev/null | wc -l); echo "FACTS-SOFA callers-left=$n"
  EXPECT: FACTS-SOFA callers-left=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=511e49a316d32ff4fa75a256daefcef6c58f8f9e1348f9014e3d83a410ce1d6d; output-bytes=26
- [x] B18: extended data check passes on BOTH episodes against the raw fetch
  CHECK: ~/yt-digest/.venv/bin/python tools/board_data_check.py --slug 2026-09-08_real-madrid-inter >/dev/null && ~/yt-digest/.venv/bin/python tools/board_data_check.py --slug 2026-09-12_bournemouth-brentford >/dev/null && echo DATACHECK-BOTH-OK
  EXPECT: DATACHECK-BOTH-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=6cef1fb7a6b8b10146dca828916831750f1706b1796b9ff5aea14b4e1c6e96a1; output-bytes=18
- [x] B19: the rendered-page audit RUNS and enforces: a deliberately mirrored input must FAIL, the real input must PASS
  CHECK: bash tools/pagecheck_proof.sh
  EXPECT: PAGECHECK-PROOF OK mirrored-fail=1 real-pass=1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=737acfa4b7ded0d2b0cf86f1919063a1067a4f51537cc507205d477cadb17739; output-bytes=47
- [x] B20: census is script-driven and TOOLS.md matches tools/ exactly
  CHECK: ~/yt-digest/.venv/bin/python tools/census.py
  EXPECT: CENSUS OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=7217c2daad4808c992de515116d55471499543f04c6d4a456ff36c0bfbe2dbc4; output-bytes=2065
- [x] B22: all work committed AND pushed; the check stages nothing (replaces G12+B15, brief 08 job 4)
  CHECK: test -z "$(git status --porcelain)" && test "$(git rev-parse HEAD)" = "$(git rev-parse @{u})" && echo CLEAN-PUSHED
  EXPECT: CLEAN-PUSHED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=7e11f8b60ada90f702114686f4c992332a3c591b6458f432fa3aa551c69067a0; output-bytes=13
- [x] B23: brief 09 research artifacts exist (3 CSVs, 3 transcripts, measurements, catalog, gaps, durations)
  CHECK: for f in reports/brief09/frames_A.csv reports/brief09/frames_B.csv reports/brief09/frames_C.csv reports/brief09/transcript_A.txt reports/brief09/transcript_B.txt reports/brief09/transcript_C.txt reports/brief09/measurements.md reports/brief09/overlay_catalog.md reports/brief09/gaps.md reports/brief09/durations.json; do test -f "$f" || exit 1; done && echo BRIEF09-ARTIFACTS-OK
  EXPECT: BRIEF09-ARTIFACTS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=8725c126fb5712822dacf56972d1c276bd74b3a1000a7e1a4a4690f0b13ffc69; output-bytes=21
- [x] B24: CSV row counts match the studied region duration/2 within 1 (reads durations.json recorded before the videos were deleted)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief09_check.py rows
  EXPECT: ROWS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=e09ffd77cfa3560d2be1c996aaf4d1fd9ac247f5e5f4fd0371a000c8fffa9755; output-bytes=143
- [x] B25: brief 09 contact sheets exist and total at most 12
  CHECK: n=$(ls reports/brief09/sheet_*.jpg 2>/dev/null | wc -l) && [ "$n" -ge 1 ] && [ "$n" -le 12 ] && echo SHEETS-OK
  EXPECT: SHEETS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=3565f3bd79550109917471627206ec673bbbf94f398bfc914da1708726fc8cae; output-bytes=10
- [x] B26: local benchmark videos deleted after the pod pull (positive control: the planted /tmp .mp4 the matcher must catch)
  CHECK: mkdir -p /tmp/b09_pc && printf x > /tmp/b09_pc/pc.mp4 && n=$(find /tmp/b09_pc -name "*.mp4" 2>/dev/null | wc -l) && rm -rf /tmp/b09_pc && [ "$n" -eq 1 ] && real=$(find /mnt/f/benchmark /mnt/c/Users/muads/Downloads/benchmark -maxdepth 1 -type f \( -name "*.mp4" -o -name "*.mkv" -o -name "*.webm" \) 2>/dev/null | wc -l) && [ "$real" -eq 0 ] && echo BENCH-LOCAL-DELETED
  EXPECT: BENCH-LOCAL-DELETED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=86f9fd29599306b77941b2361713b79fd810bd35cc6b9062547a414fdae7348c; output-bytes=20
- [x] B27: the disk guard exists and is wired to every entry point (5+ callers grepped, not counted by memory)
  CHECK: test -x tools/disk_guard.sh && n=$(grep -rlE "disk_guard\.sh" tools/ .claude/hooks/ 2>/dev/null | grep -v "disk_guard.sh$" | wc -l) && [ "$n" -ge 5 ] && echo GUARD-WIRED callers=$n
  EXPECT: GUARD-WIRED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=63a1455860d7da6aed83be200f505957a828ed7bed36d8032fc116b81046f595; output-bytes=23
- [x] B28: the guard's positive controls FAIL on faked states (100000G floor; unmounted-F target) and the cleanup exemption PASSES
  CHECK: bash tools/disk_guard.sh --selftest
  EXPECT: GUARD-SELFTEST-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=1096bead3681d9db3c9157d52a4fcb207f9b44c270911a3f5a70dcaedeebdfe3; output-bytes=18
- [x] B30: the five cache+staging env vars resolve to /mnt/d in the sourced env file (path re-pointed brief 13 job 0c to the machine/ rename; XDG_CACHE_HOME still deliberately absent)
  CHECK: s=$HOME/.config/machine/env.sh && test -f "$s" && [ "$(grep -cE '^export (PIP_CACHE_DIR|UV_CACHE_DIR|npm_config_cache|HF_HOME|SOCCER_STAGING_ROOT)=/mnt/d/scratch/' "$s")" -eq 5 ] && echo CACHES-ON-D
  EXPECT: CACHES-ON-D
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=204874917ab8270f4d727a7a9f4347a604bd58d8d0dd626852bb90a603a59b4e; output-bytes=12
- [x] B31: .wslconfig carries the memory cap (half of RAM minus 1 GB, min 4 GB)
  CHECK: grep -qE '^memory=[0-9]+MB' /mnt/c/Users/muads/.wslconfig && echo WSLCONFIG-CAPPED
  EXPECT: WSLCONFIG-CAPPED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=cf10f7c3d0f3cdb388fcf7974ea6e39ba0cd17d76a912efd4182c89338c4b5d5; output-bytes=17
- [x] B32: the keys rescue tarball is on B2 and still lists exactly the file count recorded in its manifest (brief 07 continuation job 1, Mayo option A; reads B2 only, never prints key contents; rewritten 2026-09-27 from the F: version the flapping drive killed — ABANDON resolved by Mayo choosing B2 streaming)
  CHECK: u=b2:mendymax-archive/keys/2026-09-27_soccer-channel-keys.tar.gz; n=$(rclone cat "$u" 2>/dev/null | tar tzf - 2>/dev/null | grep -vc '/$') && m=$(rclone cat b2:mendymax-archive/keys/2026-09-27_soccer-channel-keys.manifest.json 2>/dev/null | ~/yt-digest/.venv/bin/python -c "import json,sys; print(json.load(sys.stdin)['file_count'])") && [ -n "$n" ] && [ "$n" -gt 0 ] && [ "$n" = "$m" ] && echo KEYS-RESCUE-VERIFIED files=$n
  EXPECT: KEYS-RESCUE-VERIFIED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=bc53bc9661ec6008c08f26959e455a003bcc1e22939a5420f6051491d1343f3c; output-bytes=29
- [x] B33: the pre-ECC ~/.claude backup tarball is on B2 with byte-exact size matching its manifest (brief 07 continuation job 2, Mayo option A; reads B2 only; the 16545-entry full listing was verified once at job time by rclone cat | tar tzf - | wc -l and is recorded in the manifest; B2 stores no SHA1 for this rcat large-file, so size-vs-manifest is the durable check)
  CHECK: u=b2:mendymax-archive/backups/2026-09-27_claude_home_pre_ecc.tar.gz; s=$(rclone lsjson "$u" 2>/dev/null | ~/yt-digest/.venv/bin/python -c "import json,sys; print(json.load(sys.stdin)[0]['Size'])") && m=$(rclone cat b2:mendymax-archive/backups/2026-09-27_claude_home_pre_ecc.manifest.json 2>/dev/null | ~/yt-digest/.venv/bin/python -c "import json,sys; print(json.load(sys.stdin)['size_bytes'])") && [ -n "$s" ] && [ "$s" -gt 500000000 ] && [ "$s" = "$m" ] && echo BACKUP-ON-B2 size=$s
  EXPECT: BACKUP-ON-B2
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=b68c9b0f1aaccbfeacef67743ffecb475fda874cd9d24b87fb0f6ad02659ce79; output-bytes=28
- [x] B34: the ECC clone is checked out at tag v2.2.1 exactly (brief 07 job 2)
  CHECK: git -C ~/ECC describe --tags --exact-match 2>/dev/null | grep -qx v2.2.1 && echo ECC-PINNED-V221
  EXPECT: ECC-PINNED-V221
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=d065a72298616d1a0095816c5667e30324999ca25fcd39fd200cdd70b5b14371; output-bytes=16
- [x] B35: the dry-run plan files exist and the JSON one parses (brief 07 job 3)
  CHECK: ~/yt-digest/.venv/bin/python -c "import json; json.load(open('reports/brief07/ecc_plan.json')); print('PLAN-OK')" && test -s reports/brief07/ecc_plan.txt
  EXPECT: PLAN-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=edab1af7911171143f404e059437e3f2580c9477fbf5546f5723e780ab523c8d; output-bytes=8
- [x] B36: the six analysis artifacts are non-empty (counts, clashes, hooks, settings diff, size estimate, context before) (brief 07 job 3)
  CHECK: for f in counts.txt clashes.txt hooks.txt settings_diff.txt size_estimate.txt context_before.txt; do test -s "reports/brief07/$f" || exit 1; done && echo BRIEF07-ANALYSIS-OK
  EXPECT: BRIEF07-ANALYSIS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=f2c719f19f820f314bc8d6935b6c68c7d0c4a0fb514fae40876bf9750c676ad4; output-bytes=20
- [x] B37: C: free space stays above the 15 GB floor at the end of brief 07 jobs 1-4 (brief 07 job 2 stop rule)
  CHECK: a=$(df -BG --output=avail /mnt/c | tail -1 | tr -dc '0-9') && [ "$a" -ge 15 ] && echo C-FLOOR-OK avail=${a}G
  EXPECT: C-FLOOR-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=8ce94429a937dfc41f74177671dd40fa0f4de7403255c4c7558ffa902807b6d0; output-bytes=21

- [x] B39: ECC doctor is clean at source version 2.2.1 after the install (brief 07 job 8)
  CHECK: d=$(node ~/ECC/scripts/ecc.js doctor 2>&1) && echo "$d" | grep -q "errors=0" && l=$(node ~/ECC/scripts/ecc.js list-installed 2>&1) && echo "$l" | grep -q "Source version: 2.2.1" && echo ECC-DOCTOR-CLEAN-V221
  EXPECT: ECC-DOCTOR-CLEAN-V221
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=255fae43e759ec80d73791dc86d43e2cc6cff650ec789ae380f5d7aef0ceab2b; output-bytes=22
- [x] B40: the unlazy Stop hook AND ECC's dispatcher coexist in ~/.claude/settings.json post-install (brief 07 job 8)
  CHECK: grep -q "stop-hook.mjs" ~/.claude/settings.json && grep -q "pre:bash:dispatcher" ~/.claude/settings.json && echo HOOKS-COEXIST
  EXPECT: HOOKS-COEXIST
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=40ad8a1dc2118c44aae5d4af2f45c59389b6e887049073ad1856d01ff1cba4e6; output-bytes=14
- [x] B41: the disk-guard session-start hook is still wired in project settings and the guard selftest passes post-install (brief 07 job 8; hook renamed check_mnt_f.sh → check_scratch.sh by brief 11 job 4, so the grep follows the rename)
  CHECK: grep -q "check_scratch.sh" .claude/settings.json && bash tools/disk_guard.sh --selftest 2>&1 | grep -q GUARD-SELFTEST-OK && echo GUARD-INTACT
  EXPECT: GUARD-INTACT
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=b102eaf1d7bb41f16a7394e9ef211c3d9b6c4253f7f69bfa40bfe6e194df198c; output-bytes=13
- [x] B42: the precedence file exists and the global CLAUDE.md points to it (brief 07 decision 5)
  CHECK: test -s ~/.claude/rules/precedence.md && grep -q "rules/precedence.md" ~/.claude/CLAUDE.md && echo PRECEDENCE-WIRED
  EXPECT: PRECEDENCE-WIRED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=897744703894031f0e70a9909076a2a876df7d761f4813314a2a65046662eefc; output-bytes=17
- [x] B43: the context-transfer skill is installed with valid frontmatter (brief 07 decision 7)
  CHECK: test -f ~/.claude/skills/context-transfer/SKILL.md && grep -q '^name:' ~/.claude/skills/context-transfer/SKILL.md && echo CT-SKILL-PRESENT
  EXPECT: CT-SKILL-PRESENT
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=0278fde26d143fe0d3f1d2c7c808cbfca3ac62e1d11adddd20caa7707f8dc6c2; output-bytes=17
- [x] B44: the ECC data cap is wired: guard selftest (incl. oversized-ECC control 4) passes and the live report shows the ecc_data line under cap (brief 07 job 5, Mayo instruction)
  CHECK: bash tools/disk_guard.sh --selftest 2>&1 | grep -q GUARD-SELFTEST-OK && bash tools/disk_guard.sh --report 2>/dev/null | grep -qE "ecc_data [0-9]+B <= [0-9]+B cap" && echo ECC-CAP-WIRED
  EXPECT: ECC-CAP-WIRED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=4083c7d2c6597821ff2607367b9783836c8103b0643132360e44d11fe266d52e; output-bytes=14
- [x] B45: before/after session-start context numbers are recorded in both artifacts (brief 07 job 3f + job 6)
  CHECK: test -s reports/brief07/context_before.txt && grep -q "39837" reports/brief07/context_before.txt && test -s reports/brief07/context_after.txt && grep -q "50652" reports/brief07/context_after.txt && echo CTX-BEFORE-AFTER-RECORDED
  EXPECT: CTX-BEFORE-AFTER-RECORDED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=ac2250ca18829020a7553ed272f26bc99e0d1ef55727b796535ef960d9afab6f; output-bytes=26
- [x] B46: the brief 11 inventory artifact exists with NTFS evidence for D: and the counted /mnt/f hit list (brief 11 job 1)
  CHECK: test -s reports/brief11/inventory.txt && grep -q "NTFS" reports/brief11/inventory.txt && grep -q "MNTF-HITS=" reports/brief11/inventory.txt && echo INVENTORY-OK
  EXPECT: INVENTORY-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=eb00d203d03f0009faa610a5f3fb941b4f422f59994574396334c361b6607296; output-bytes=13
- [x] B47: scratch_mount.sh exists and its write-probe check passes on D: (brief 11 job 2)
  CHECK: test -x tools/scratch_mount.sh && bash tools/scratch_mount.sh --check 2>&1 | grep -q D-MOUNT-OK && echo SCRATCH-MOUNT-OK
  EXPECT: SCRATCH-MOUNT-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=88649a6ebbe8cb5851f84c998cee105cc420fdc2c6df470a21e663162dcf81c0; output-bytes=17
- [x] B48: the WSL-side 1 GB write and read speeds are recorded with MB/s figures (brief 11 job 3)
  CHECK: test -s reports/brief11/speed.txt && grep -q "WRITE-MBps=" reports/brief11/speed.txt && grep -q "READ-MBps=" reports/brief11/speed.txt && echo SPEED-RECORDED
  EXPECT: SPEED-RECORDED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=56ba360414c87bc3b2c6fd74dd0ccc8d3de30107a2d3aec7e590b21c1e9ae391; output-bytes=15
- [x] B50: no live config or tool VALUE names /mnt/f as a destination (comment lines documenting the retirement are excluded), while the history file still names it (positive control that the grep works); disk_guard.sh's retired-F detector is exempt by design (brief 11 job 6d)
  CHECK: n=$(grep -rn "/mnt/f" ~/.config/machine/env.sh ~/.config/yt-dlp/config tools/staging.py tools/scratch_mount.sh tools/b2_mount.sh tools/backup_env.py tools/produce_v2.py tools/f_mount.sh .claude/hooks/check_scratch.sh .claude/settings.json 2>/dev/null | grep -vE ':[0-9]+:#' | wc -l) && [ "$n" -eq 0 ] && h=$(grep -c "/mnt/f" tools/brief10_elevated_setup.sh) && [ "$h" -gt 0 ] && echo NO-LIVE-MNTF history-hits=$h
  EXPECT: NO-LIVE-MNTF
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=5ba68989a7062eade8a8c90c5f3c0910bd3b01b8d10e52185af0f3d3dba06de0; output-bytes=28
- [x] B51: the guard selftest output with the brief-11 controls is recorded (D: unmounted FAILS, /mnt/f path FAILS, real state PASSES) (brief 11 job 6a)
  CHECK: test -s reports/brief11/controls.txt && grep -q GUARD-SELFTEST-OK reports/brief11/controls.txt && echo CONTROLS-RECORDED
  EXPECT: CONTROLS-RECORDED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=99e82e34c013c3da0895b7dc590924eeac93314055243eaa2384b0918faf84ab; output-bytes=18
- [x] B52: the elevated-setup output proves visudo-validated d-* sudoers and the removed f-* rule (brief 11 jobs 2 and 5, Mayo-run script)
  CHECK: test -s reports/brief11/elevated_setup_output.txt && grep -q "VISUDO-OK" reports/brief11/elevated_setup_output.txt && grep -q "F-SUDOERS-REMOVED" reports/brief11/elevated_setup_output.txt && echo SUDOERS-OK
  EXPECT: SUDOERS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=d861d2b5586336a35c431b4708389e11511a87942e1464f5d91f84f65f80f81a; output-bytes=11
- [x] B53: a real yt-dlp download landed in /mnt/d/scratch (path printed, file deleted after) (brief 11 job 6b)
  CHECK: test -s reports/brief11/ytdlp_download.txt && grep -q "/mnt/d/scratch" reports/brief11/ytdlp_download.txt && echo YTDLP-ON-D
  EXPECT: YTDLP-ON-D
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=fe98c7b4ab8df62d7afa11382cc7129926e93569e5d5d40b6aa0f2c70684122a; output-bytes=11
- [x] B54: a Modal round trip staged its file through /mnt/d/scratch (smallest existing smoke: modal volume put + get) (brief 11 job 6c)
  CHECK: test -s reports/brief11/modal_smoke.txt && grep -q "MODAL-SMOKE-OK" reports/brief11/modal_smoke.txt && grep -q "/mnt/d/scratch" reports/brief11/modal_smoke.txt && echo MODAL-THRU-D
  EXPECT: MODAL-THRU-D
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=e4f430eccebbfcdfc8ad546e10d36b652c2f251cde12c7b525c04ae0fd3245bb; output-bytes=13
- [x] B55: the reverify proof ending ALL MET after the last code commit is published in reports/brief11/ (brief 11 job 8)
  CHECK: test -s reports/brief11/reverify.txt && grep -q "ALL MET" reports/brief11/reverify.txt && echo REVERIFY-PUBLISHED
  EXPECT: REVERIFY-PUBLISHED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=8b4564b0ae760c286f32c8384039bff2a83d419f9ffdfbc945a8dc0c2766b4cf; output-bytes=19

Brief 12 gates (animated boards; added 2026-09-28 before implementation,
per unlazy). The oracle for B56-B61 is tools/brief12_check.py, which reads
reports/brief12/render_manifest.json (the board list job 5 produces) and
touches nothing in the repo (writes only under /tmp).

- [x] B56: every brief-12 timeline spec validates: schema, targets, source stamps, and every player number/name/position traced to the cached raw responses (board_data_check kind timeline)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief12_check.py specs
  EXPECT: TIMELINE-SPECS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=f96a911a48231b70a16714539fb4abf144370f30303ef18a9bf0c9f292237d59; output-bytes=22
- [x] B57: each proof board MP4 lasts its spec duration within one frame (ffprobe vs duration_s at the spec fps)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief12_check.py durations
  EXPECT: DURATIONS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=119ced4aa21b20eaaf9f3629decd22ee9f683739f8d94c111bfa89bd82999501; output-bytes=32
- [x] B58: the per-step keyframe audit passes on every proof timeline (rendered-page audit re-run per step end, in audit-only mode)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief12_check.py keyframes
  EXPECT: KEYFRAME-AUDIT-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=b5a44e622bc27cc6e8cfd17ff300af979e6bfefa3a6a3ca5af5184fcd32cf15a; output-bytes=22
- [x] B59: the swapped-name positive control FAILS the timeline audit and the real spec passes (brief 12 job 4)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief12_check.py swap
  EXPECT: SWAP-CONTROL-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=93afd519868995258b884d600eddf87979171ea0424782e1c74a12ef0a057e94; output-bytes=81
- [x] B60: reports/brief12/ holds a 4x4 640px contact sheet per proof board and per board a 640px GIF or 2-3 keyframe PNGs
  CHECK: ~/yt-digest/.venv/bin/python tools/brief12_check.py sheets
  EXPECT: SHEETS-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=3e160115d73d9ecddeacef06825d215c34ae330c77c2f45a8d849dda98fd6bad; output-bytes=28
- [x] B61: no invented numbers: the numeric-leaf walk accepts the timeline specs (every non-choreography number traced to a cached raw response) and a planted fake position FAILS the check (positive control)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief12_check.py numbers
  EXPECT: NUMBERS-TRACED-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=3d52719de57575995baae287c0ee0c1b6ecdeeaaf861acb35af7be211f2228df; output-bytes=474

Brief 13 gates (board sizing + freeze-frame overlays; added 2026-09-28 before
implementation, per unlazy). The oracle for B62-B68 is tools/brief13_check.py,
which touches nothing in the repo (writes only under /tmp).

- [x] B62: sizing minima hold on every rendered board at every step end: disc radius >= 42 px, number-chip radius >= 26 px, name-label height >= 36 px at 1080p (the 480p readability floor), and a planted sub-minimum entity FAILS the audit (positive control)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief13_check.py sizing
  EXPECT: SIZING-MINIMA-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=3ba82c33bc74dac50825f8f4f0c30a6588808a7a20a35cf709b184042aa14c7a; output-bytes=228
- [x] B63: the chip-label overlap audit is live on every rendered keyframe and a planted chip-on-label case FAILS it (positive control)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief13_check.py overlap
  EXPECT: OVERLAP-CONTROL-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=beea3352d679167e63708ee3ad741b6389c1cddb2f9baa026cb6a3a33b77a8e7; output-bytes=65
- [x] B64: every NAMED player in a brief-13 overlay spec carries identity evidence (source "lineup:<jersey>" matching a jersey in the facts XI); a spec naming a player without evidence is REFUSED (positive control)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief13_check.py identity
  EXPECT: IDENTITY-EVIDENCE-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=32df4151fe4ee13d3883db646f3c0f9fb8b52094b419e93ca4d2108dbba56171; output-bytes=131
- [x] B65: the drift check compares rendered mark positions against the marked foot points and a spec with one ellipse shifted 150 px FAILS it (positive control, brief job 6)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief13_check.py drift
  EXPECT: DRIFT-CONTROL-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=d3d495d7a55162694751ed977edc1339f004349c69c0fae2404504e66f20aa42; output-bytes=278
- [x] B66: neither npm nor yt-dlp cache resolves on C: in a login shell (npm env var and yt-dlp --cache-dir both point at /mnt/d or the WSL home, never /mnt/c)
  CHECK: bash -lic 'c=$(npm config get cache 2>/dev/null); y=$(grep -oE "cache-dir [^ ]+" ~/.config/yt-dlp/config | head -1); case "$c$y" in */mnt/c/*) exit 1;; esac; echo "$c | $y" | grep -q "mnt/d" && echo CACHES-OFF-C' 2>/dev/null
  EXPECT: CACHES-OFF-C
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=bf0bc11d94142cc1f26807c3be72fe2451aaecd0296fdfc8a33fe4b120582356; output-bytes=13
- [x] B67: the machine env rename is live: bash -lic resolves the five scratch vars through the new path, the old path is a one-line shim, and .bashrc sources the new path
  CHECK: bash -lic 'test -f ~/.config/machine/env.sh && test "$(grep -c . ~/.config/soccer/env.sh)" -eq 1 && grep -q "config/machine/env.sh" ~/.bashrc && [ "$SOCCER_STAGING_ROOT" = "/mnt/d/scratch/soccer-staging" ] && echo ENV-SHIM-OK' 2>/dev/null
  EXPECT: ENV-SHIM-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=29a13918b1d57083acedcb291a49cfdb28c7c61d2e91071f546b8fab9b2e8f57; output-bytes=12
- [x] B68: the stale auto-memory files are gone: tactical-dominant-balance.md deleted and stop-adding-start-replacing.md carries no tactical_render/step4 reference
  CHECK: m=/home/muads/.claude/projects/-home-muads-yt-digest-soccer-channel/memory && test ! -f "$m/tactical-dominant-balance.md" && ! grep -qE "tactical_render|step4" "$m/stop-adding-start-replacing.md" && echo MEMORY-CLEAN
  EXPECT: MEMORY-CLEAN
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=1b950285411cb4e78de19aadc8aa162e4aea1c26df2f061056836f8d1e8b16bc; output-bytes=13

Brief 14 gates (lean harness on shared ~/.claude, both pipelines; added
2026-09-28 before implementation, per unlazy). Removal sets live only in
reports/brief14/plan.json, authored in job 3; the oracles read that file, so
no gate hard-codes skill names. Oracle files under reports/brief14/ are
committed alongside the proof.

- [x] B69: the pre-change harness backup exists at
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=61ddeaa0dd8e171d5b692fcbcd3c485ecab261575e2aeb6f57f7ece74306f02d; output-bytes=16
      b2:mendymax-archive/backups/2026-09-28_claude_home_pre_lean.tar.gz and
      verifies: the downloaded tarball's md5 equals the md5 captured on the
      streamed upload (reports/brief14/backup_upload.md5) and its tar listing
      count equals the locally streamed listing count
      (reports/brief14/backup_listing.txt)
  CHECK: a=$(cut -d' ' -f1 reports/brief14/backup_upload.md5) && rclone copyto b2:mendymax-archive/backups/2026-09-28_claude_home_pre_lean.tar.gz /tmp/b14_backup_verify.tar.gz && b=$(md5sum /tmp/b14_backup_verify.tar.gz | cut -d' ' -f1) && n=$(tar tzf /tmp/b14_backup_verify.tar.gz | wc -l) && e=$(tail -1 reports/brief14/backup_listing.txt) && [ "$a" = "$b" ] && [ "$n" -eq "$e" ] && echo BACKUP-VERIFIED
  EXPECT: BACKUP-VERIFIED
- [x] B70: reports/brief14/before.md records the before measurements: the
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=0e66a8d245d7fd486465e9e017e0c7a84de02e7d073ec374efdf69f2e2f1d4e8; output-bytes=16
      session-start token run with the brief-07 headless command (method, raw
      excerpt, first-turn and aggregate input tokens), benchmark run 1 and
      run 2 each with wall time, input tokens, turns and a correctness
      verdict against locally counted truth, and a hooks table listing every
      hook per event with owner and measured runtime
  CHECK: grep -q "SESSION-START-TOKENS" reports/brief14/before.md && grep -q "BENCH-RUN-1" reports/brief14/before.md && grep -q "BENCH-RUN-2" reports/brief14/before.md && grep -q "HOOKS-TABLE" reports/brief14/before.md && echo BEFORE-RECORDED
  EXPECT: BEFORE-RECORDED
- [x] B71: plan.md and plan.json are complete and mutually consistent: every
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=c42a7bce6ac2c131a5f55766770298ffaf806fc06f287854da58ff69efc5ca04; output-bytes=1089
      planned removal names an owner and a reason, all removal/keep keys are
      present, and the md carries the counts before/after block
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief14_check.py plan
  EXPECT: PLAN-CONSISTENT
- [x] B72: the applied filesystem end state matches the plan: every
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=ea77c8689e24252602c96aee2fefbebfe813b7d4874aa03c332f7b8624c188ce; output-bytes=630
      planned-removed skill/agent directory is gone, every planned-kept one
      is present, claude-mem is untouched, and live skills/agents counts
      equal the plan's after counts
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief14_check.py applied
  EXPECT: APPLIED-MATCHES-PLAN
- [x] B73: the hook end state matches the plan across every settings file
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=d8bbf1233ca04d9a1f14a0478845de31a7f2e8d11749e5aca3aa5d9af6126e4d; output-bytes=529
      (global, soccer project, jiheeye project): every planned-removed hook
      snippet absent, every planned-kept snippet present (unlazy Stop, soccer
      push_status, disk guard, jiheeye hooks, claude-mem)
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief14_check.py hooks
  EXPECT: HOOKS-STATE-OK
- [x] B74: the MCP end state matches the plan: filesystem, memory and
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=f3d812d052c5f94293ae025d7742d5f7419eeddd5d47a8a553d75e281e6af14c; output-bytes=206
      sequential-thinking absent from ~/.claude.json, github, brave-search
      and puppeteer still present, and the no-caller grep evidence is saved
      (reports/brief14/mcp_callers.txt)
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief14_check.py mcp
  EXPECT: MCP-STATE-OK
- [x] B75: the read-discipline rule is standing in ~/.claude/CLAUDE.md
  CHECK: grep -q "Read discipline" /home/muads/.claude/CLAUDE.md && grep -q "Grep first" /home/muads/.claude/CLAUDE.md && echo READ-RULE-PRESENT
  EXPECT: READ-RULE-PRESENT
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=e235ecef97c424be071e22e3a4e12a8c181aa14bbc8e9f03a31a3457d1bed834; output-bytes=18
- [x] B76: planned auto-memory files were merged into SYSTEM.md pointers
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=d781c4148e18ceeb92bb8f41be5a0259010997fae68aaa153eae7033ca558d89; output-bytes=258
      (short, name SYSTEM.md) and planned-kept memory files remain in place
      (both memory dirs: yt-digest and soccer-channel)
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief14_check.py memory
  EXPECT: MEMORY-MERGED-OK
- [x] B77: ECC doctor reports zero warnings and zero errors for what remains
  CHECK: node /home/muads/ECC/scripts/ecc.js doctor > /tmp/b14_doctor.txt 2>&1 && grep -q "warnings=0" /tmp/b14_doctor.txt && grep -q "errors=0" /tmp/b14_doctor.txt && echo DOCTOR-CLEAN
  EXPECT: DOCTOR-CLEAN
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=dd49e82c3de041ea16e143bea972a11696d09ed16c56c2d33c20437f6f6b08dc; output-bytes=13
- [x] B78: reports/brief14/after.md carries the before/after table with all
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=42fefc84b182780d3d17af9b3600c730b615eafbecbe59cdf851380b1f617f64; output-bytes=485
      seven done-means rows (session_start_tokens, bench_wall_ms,
      bench_input_tokens, skills, agents, stop_hooks, mcp_tools), each side
      numeric
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief14_check.py after_table
  EXPECT: AFTER-TABLE-OK
- [x] B79: no regression bundle: soccer disk_guard selftest, scratch_mount
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=8929fb6a7804dcee3c7e8375e60d03cc90cbae587211f8e8305eece783612a3f; output-bytes=301
      --check, jiheeye disk_guard selftest (read-only), brief13 board-render
      identity gate re-run locally without Modal, and both after benchmark
      runs' answers equal the same-time locally counted truth
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief14_check.py quality
  EXPECT: QUALITY-NO-REGRESSION
- [x] B80: ~/.claude/SYSTEM.md section 5 reflects the lean end state (new
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=073278c2b63223f056584b89d465acd56553a1c5fa4930fcb23e5a9273564a22; output-bytes=16
      counts, hooks list, MCP list, read rule, dated 2026-09-28) and a copy
      is streamed to b2:mendymax-archive/system/2026-09-28_SYSTEM.md
  CHECK: grep -q "2026-09-28" /home/muads/.claude/SYSTEM.md && grep -qi "brief 14" /home/muads/.claude/SYSTEM.md && rclone lsf b2:mendymax-archive/system/ 2>/dev/null | grep -q "2026-09-28_SYSTEM" && echo SYSTEM-STREAMED
  EXPECT: SYSTEM-STREAMED
- [x] B81: both repos' DECISIONS.md carry a dated brief-14 entry recording
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=f2e9d355c9b873bb40d63304022fb83fe9bfe11e9ab4409c2328a5ba6a79ceb5; output-bytes=21
      what was removed and kept in the shared harness
  CHECK: grep -qi "brief 14" /home/muads/yt-digest/soccer-channel/DECISIONS.md && grep -qi "brief 14" /home/muads/jiheeye-ultra/DECISIONS.md && echo DECISIONS-BOTH-REPOS
  EXPECT: DECISIONS-BOTH-REPOS
- [x] B82: the reverify proof ending ALL MET, run after the last code commit,
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=3be1501a5a88c1302475ed12070303ab4f36ed58c783765a8dc419e12e7c018e; output-bytes=13
      is published in reports/brief14/reverify.txt
  CHECK: test -s reports/brief14/reverify.txt && grep -q "ALL MET" reports/brief14/reverify.txt && echo REVERIFY-B14
  EXPECT: REVERIFY-B14

Brief 15 gates (overlay styling pass + text, title and stat cards; added
2026-09-29 before implementation, per unlazy). Oracle: tools/brief15_check.py
(WIRED, TOOLS.md). The card-spec audits reuse the brief-12/13 page-audit
mechanism (node tools/board_html.js --audit-only with the local cached
chrome; writes only under /tmp).

- [x] B83: the 60-skill marketing kit is removed and verifiable: roster dirs
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=47991cd0aaf81e018e1a5acc81a1778625436ffbe15075bba504fff47bc6cd64; output-bytes=288
      absent under ~/.claude/skills, skills dir count 193, the B2 backup's
      upload/download md5s match (two lines in kit_removal.md), tar listing
      count 413, ecc.js doctor clean, brief-07 session-start tokens recorded
      before AND after, and both repos' DECISIONS.md carry the removal entry
      (report: reports/brief15/kit_removal.md)
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief15_check.py kit
  EXPECT: KIT-REMOVAL-OK
- [x] B84: the overlay restyle is enforced by the page audit: ground
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=fef8e9d7a8d3f12afdc5d2bf7f22030dd21a0387ad22a14df2695c9d25231667; output-bytes=318
      ellipses at least 2x previous size (rx floor 64, ry floor 24 at
      1080p), link/arrow widths at least 3x previous (floor 12), chips
      larger with a thin border (size floor 40, h floor 56), pressing discs
      at least 2x (radius floor 24), polygon more opaque (alpha >= 0.28),
      and the vignette darkening outside the marked area recorded in
      0.25-0.35; a planted sub-minimum ellipse FAILS the audit (positive
      control)
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief15_check.py overlay
  EXPECT: OVERLAY-STYLE-OK
- [x] B85: freeze points re-checked against broadcaster graphics: every
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=caa795606a61ee286bb1f3e1b6ab9639ca4e26400878cdeeb84c6c5a84fcaa8d; output-bytes=2851
      moment records either banner bboxes (hand-measured from the frame via
      the vision report) or a clear verdict, the marked-area extents do not
      intersect the banner bboxes, and a planted intersecting banner FAILS
      the check (positive control); freeze shifts move by whole frame
      counts at the clip fps with the new frozen frame stamped in marks.json
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief15_check.py freeze
  EXPECT: FREEZE-CLEAR-OK
- [x] B86: the five card types exist in the timeline renderer (same
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=d9ef46d615afc6d7aaca6fee249091c5d17d6a575a06b8db8911c1eea7642060; output-bytes=891
      renderer, no second one) and every card number traces to a cached raw
      response (numeric-leaf walk with choreography exemptions); a planted
      wrong number FAILS the walk (positive control)
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief15_check.py cards
  EXPECT: CARDS-FACTS-OK
- [x] B87: every brief-15 render is real: 4 restyled overlay clips and 5
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=5abbfc376a2b191baf30ef80bd55cfe4bee52f7efeb683f9aa49bfb8cedff428; output-bytes=2052
      card renders exist at 1080p H.264 (codec, fps, duration within one
      frame of spec), with a render_manifest.json recording sha256, bytes,
      duration and codec per render; sizes cross-checked against B2 sha1
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief15_check.py renders
  EXPECT: RENDERS-OK
- [x] B88: the visual check is recorded: reports/brief15/visual_check.md
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=02831830911ffdaa827d54448e588f846c839c573a8e0e17618a9a514116292f; output-bytes=16
      covers every render with a keyframe verdict (480p readability, no
      element hiding another, numbers match facts, names carry evidence)
      and ends EVERY-RENDER-CHECKED
  CHECK: test -s reports/brief15/visual_check.md && grep -q "EVERY-RENDER-CHECKED" reports/brief15/visual_check.md && echo VISUAL-CHECK-OK
  EXPECT: VISUAL-CHECK-OK
- [x] B89: the publish artifacts exist: overlay before/after sheets and
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=21632d5713a06131c81e86528205da09cb31d18d37e439926b501d7f28bfc409; output-bytes=518
      per-card sheets at 640px width, photo_sources.md with a full row per
      photo (url, owner/licence, claim-risk note), full-res MP4s archived to
      B2 with a JSON manifest next to them
  CHECK: /home/muads/yt-digest/.venv/bin/python tools/brief15_check.py publish
  EXPECT: PUBLISH-OK
- [x] B90: ~/.claude/SYSTEM.md carries the decision-1 change-log mapping
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=bc577f18223f774f4ff77684e8a86bb2b8ef31c44b9aafeb784156ef6ea39354; output-bytes=14
      line, the kit outcome in section 5, and a dated copy is streamed to
      b2:mendymax-archive/system/2026-09-29_SYSTEM.md
  CHECK: grep -q "audit ledger" /home/muads/.claude/SYSTEM.md && grep -qi "kit" /home/muads/.claude/SYSTEM.md && rclone lsf b2:mendymax-archive/system/ 2>/dev/null | grep -q "2026-09-29_SYSTEM" && echo SYSTEM-B15-OK
  EXPECT: SYSTEM-B15-OK
- [x] B91: the reverify proof ending ALL MET, run after the last code
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=64c5b1d9c4e56779524496bf7977c58ade2d032c7125dcfe94088b6d82e11a08; output-bytes=13
      commit, is published in reports/brief15/reverify.txt
  CHECK: test -s reports/brief15/reverify.txt && grep -q "ALL MET" reports/brief15/reverify.txt && echo REVERIFY-B15
  EXPECT: REVERIFY-B15

## Brief 16: thesis clip list + multi-match footage (gates B92-B101)

Brief source: briefs/brief16.md (jobs 0-8). Oracle: tools/brief16_check.py.
The md artifacts (matches.md, press_facts.md, claims.md, clip_list.md) are
rendered from JSON sidecars next to them; the oracle checks the JSONs
(single source of truth) and a grep ties each md to its sidecar ids.

- [x] B92: the matches table lists 3-4 chosen matches, each with its
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=c7d8d72c1ee080e5377cb0d9806e6999a9a616d091c5ab978969c50c95e6e004; output-bytes=1204
      Sofascore event id, competition, score and highlights URL row, and
      every chosen match has a cached raw on disk
  CHECK: ~/yt-digest/.venv/bin/python tools/brief16_check.py matches
  EXPECT: MATCHES-OK
- [x] B93: every cached brief-16 raw carries a fetched_at stamp reported in
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=f4200146bdbed9aca31cb4b7e8ead4a08a76293aaabd802eff9ed6900154e00d; output-bytes=928
      the report; every chosen event id appears in the cached last-events
      list; the list cache carries a fetched_at stamp of 2026-10-03 or
      later OR a fetch-attempts log proves the re-fetch was challenged
      (every attempt logged, none silently skipped)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief16_check.py freshness
  EXPECT: FRESH-OK
- [x] B94: press_facts.md has one table per chosen match; every number-Row
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=d5cb5eefa4ee04942ea86273a9acb11e7ff9d366e7e6fadfa0237d42035594e2; output-bytes=4317
      traces to an existing raw path and the value is found at that path
      (positive control: a planted wrong value fails)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief16_check.py facts
  EXPECT: FACTS-OK
- [x] B95: claims.md holds 6-10 accepted claims building the thesis, each
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=cdd5aa92eed66ec5300bee8e702a5e93f0789caab7c4600862b4d83eb32a7ecc; output-bytes=514
      with traced numbers and match sources, and every dropped claim states
      its reason
  CHECK: ~/yt-digest/.venv/bin/python tools/brief16_check.py claims
  EXPECT: CLAIMS-OK
- [x] B96: every chosen match's highlights video is archived to B2 with a
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=fefb6997ec7537bcc7bcfc8e6724400f3728f8186c316b367f4781de9d9feb5f; output-bytes=2191
      manifest, present on the pod volume, and DELETED locally (positive
      control: the check flags a footage path that still exists locally)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief16_check.py footage
  EXPECT: FOOTAGE-OK
- [x] B97: clip_list.md: every clip row carries a claim id, match, start/end
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=936d3cda66d965a8914f8641880e7f6006063889ed78765c6ac1a962355f0986; output-bytes=491
      seconds and what the viewer sees; every clip exists in B2 with a
      manifest whose duration matches end-start within 0.2s; every accepted
      claim has at least one clip or says no-clip with a reason
  CHECK: ~/yt-digest/.venv/bin/python tools/brief16_check.py clips
  EXPECT: CLIPS-OK
- [x] B98: the contact sheet exists at 640px wide with exactly one frame per
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=5439838f3aa3018d292d9f971c9cf43a270474380e0e8b0941b90ba6202dcc4c; output-bytes=127
      published clip
  CHECK: ~/yt-digest/.venv/bin/python tools/brief16_check.py sheet
  EXPECT: SHEET-OK
- [x] B99: the sizing minima are enforced in code and proven: the title
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=b87ba39f8ebff658b831883628f21152176e91c3d9a0c6c986e491039c3a294c; output-bytes=680
      audit requires a photo box >= 45 percent of frame width with the drawn
      cutout >= 45 percent of frame height, the overlay audit requires the
      bold ellipse stroke floor, and before/after sheets exist for one card
      and one overlay clip
  CHECK: ~/yt-digest/.venv/bin/python tools/brief16_check.py sizing
  EXPECT: SIZING-OK
- [x] B100: the strongest press moment's freeze-frame overlay is rendered
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=ab8df71590deee7d49130dcac01460f165a65b9f72898617e3da7a8d119c553f; output-bytes=252
      with FOOTAGE-CHECK OK, stored in renders with a manifest entry, and
      archived to B2 with a JSON manifest
  CHECK: ~/yt-digest/.venv/bin/python tools/brief16_check.py overlay
  EXPECT: OVERLAY-OK
- [x] B101: the reverify proof is published after the last brief-16 code
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=4e77f5712b95036a28108818579a00f7bc9570b4d6e10ee9a045134281a24c10; output-bytes=398
      commit, in reports/brief16/reverify.txt AND at the mirror; the file
      documents the full pass-A rerun (reran=88, only the proof gate itself
      unmet by construction) and ENDS with a pass-B rerun verdict printed
      by the runner (positive control: a file claiming ALL MET without the
      pass-A rerun + by-construction evidence fails)
  CHECK: ~/yt-digest/.venv/bin/python tools/brief16_check.py reverify
  EXPECT: REVERIFY-OK
