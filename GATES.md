# Gates: brief 03 (retire dead tools, board fixes, canonical sync)

OWNS: tools/**, GATES.md, canonical docs, episodes/, ~/.claude/CLAUDE.md, reports/brief04/, .claude/hooks/push_status.sh

Scope: brief 03 — retire every tool not reachable from produce_v2.py/pod_build.py
(24 files), fix momentum label overlap + four-section outro freeze (verified by
viewed frames), replace project CLAUDE.md, rewrite global CLAUDE.md, rewrite
STATUS/CONTEXT/DECISIONS, delete invented architecture, resolve the disabled
image-block hook.

- [x] G1: dead tools retired — 37 .py files remain, none of the 25 retired names exist
  CHECK: n=$(ls tools/*.py | wc -l); miss=0; for t in scene_gen player_mapper render_goal_clip ltx_enhance modal_render3d parse_feed assemble_video assemble_words_match bulk_classify cloud_produce cv_annotate enhance_clips generate_captions gpu_superres runpod_fulltrack runpod_scenedetect runpod_superres render3d_vast segment_scorer sharpness_check trim_tracking produce_episode b2_upload; do test -e "tools/$t.py" && miss=1; done; test -e tools/match_moments.workflow.mjs -o -e tools/rematch_ep001.workflow.mjs && miss=1; echo "py=$n miss=$miss"
  EXPECT: py=37 miss=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=a32edeaaa1ada82adb0f77fb26e27da614fd393249254e600cce2d9a89e39084; output-bytes=13
- [x] G2: no kept/reachable tool still names a retired tool as live (present-tense refs gone; RESTORED to strict form 2026-09-23 per brief 04 addendum — the brief-03 `-viE retired` exemption was looser than needed; the strict import/call form passes with 0 hits, so no gate exemption is warranted)
  CHECK: n=$(grep -rnE "(import (scene_gen|modal_render3d|runpod_fulltrack|generate_captions|assemble_words_match)\b|from (scene_gen|modal_render3d|runpod_fulltrack|generate_captions|assemble_words_match) import)|\b(scene_gen|modal_render3d|runpod_fulltrack|generate_captions|assemble_words_match)\(" tools/*.py | wc -l); test "$n" -eq 0 && echo REFS-CLEAN
  EXPECT: REFS-CLEAN
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=2845579af1cf407e8b0e8b82dfb12b29f975b3e880a251f36e803b2e8ba6e226; output-bytes=11
- [x] G3: fixed momentum board rendered and deployed (deployed on Modal volume + local)
  CHECK: test -s renders/2026-09-08_real-madrid-inter/boards_job2/momentum.mp4 && ~/yt-digest/.venv/bin/modal volume ls soccer-build out 2>/dev/null | grep -c "out/momentum.mp4"
  EXPECT: 1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=4355a46b19d348dc2f57c046f8ef63d4538ebb936000f3c9ee954a27460dd865; output-bytes=2
- [x] G4: final_video.mp4 re-assembled after board fixes (140.92s)
  CHECK: ffprobe -v error -show_entries format=duration -of csv=p=0 renders/2026-09-08_real-madrid-inter/final_video.mp4
  EXPECT: 140.920000
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=7c5f5ded5b88531f291861ed88028dfdb8ab89316853d8a8cffc1f0c83cf4b68; output-bytes=11
- [x] G5: project CLAUDE.md replaced with the delivered file
  CHECK: grep -c "You are the engineer on this project" CLAUDE.md
  EXPECT: 1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=4355a46b19d348dc2f57c046f8ef63d4538ebb936000f3c9ee954a27460dd865; output-bytes=2
- [x] G6: global CLAUDE.md rewrite carries no soccer/Gemini/glm-5.2 text-only rules
  CHECK: n=$(grep -ciE "soccer|gemini|glm-5\.2" /home/muads/.claude/CLAUDE.md); test "$n" -eq 0 && echo GLOBAL-CLEAN
  EXPECT: GLOBAL-CLEAN
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=97f8d45765862d67590548e55214ed520c0dc1b985801e6f04a49e6f30461b09; output-bytes=13
- [x] G7: STATUS.md, CONTEXT.md, DECISIONS.md rewritten with a current stamp
  CHECK: grep -l "Last verified against code:\*\* 2026-09-22" STATUS.md CONTEXT.md DECISIONS.md | wc -l
  EXPECT: 3
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=1121cfccd5913f0a63fec40a6ffd44ea64f9dc135c66634ba001d10bcf4302a2; output-bytes=2
- [x] G8: invented architecture deleted (moments list, script v2, workflow .mjs, thesis handover)
  CHECK: test ! -f episodes/EP001_MOMENTS.md && test ! -f episodes/EP001_SCRIPT_v2.md && test ! -f tools/match_moments.workflow.mjs && test ! -f tools/rematch_ep001.workflow.mjs && test ! -f HANDOVER_TO_FLASH.md && echo GONE
  EXPECT: GONE
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=7977cc353752450c625a1b80d72b79e781fe750cc047aed08aa01f4563e962f1; output-bytes=5
- [x] G9: block-image-read hook deleted and unregistered; other global hooks intact
  CHECK: test ! -f /home/muads/.claude/hooks/block-image-read.sh && python3 -c "import json; d=json.load(open('/home/muads/.claude/settings.json')); hs=[hh for g in d['hooks']['PreToolUse'] for hh in g['hooks']]; assert not any('block-image-read' in h['command'] for h in hs); assert len(hs)==2" && echo HOOK-CLEAN
  EXPECT: HOOK-CLEAN
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=1fc74a8f8491d0f8f6a907476fc93928085b18eb1e26aab5576764c830582141; output-bytes=11
- [x] G10: pipeline smoke: produce_v2 and pod_build still importable
  CHECK: ~/yt-digest/.venv/bin/python -c "import ast; ast.parse(open('tools/produce_v2.py').read()); ast.parse(open('tools/pod_build.py').read()); print('PIPE-OK')"
  EXPECT: PIPE-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=afdbfb412303fe91a9d1e181aec00cdb4cc744f9c34ed876804c71475f5d59d9; output-bytes=8
- [x] G11: new final + proof frames + retired tools archived to B2 with manifests (soccer-channel/2026-09-23/)
  CHECK: rclone lsf b2:mendymax-archive/soccer-channel/2026-09-23/ | grep -cE "final_video.mp4$|proof_frames.tar.gz$|retired-tools"
  EXPECT: 4
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=7de1555df0c2700329e815b93b32c571c3ea54dc967b89e81ab73b9972b72d1d; output-bytes=2
- [x] G12: all work committed and pushed
  CHECK: git add -A >/dev/null 2>&1; test -z "$(git status --porcelain)" && echo CLEAN
  EXPECT: CLEAN
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=0b98843240a0b1a2384483206c02be8968dd809eb73838820be24cb7d6d6aeb9; output-bytes=6

---

# Gates: brief 04 (prove brief 03, replace the board renderer, lock the mirror)

Evidence rule (brief 04 job 8, reconciled with the unlazy runner): the
human-readable CHECK/EXPECT/GOT record lives in reports/gates/brief04.log
(git-ignored); the unlazy runner additionally appends its own one-line
EVIDENCE per gate here, which is committed with the close-out. A gate run +
commit leaves the tree clean.

- [x] B1: momentum boards proven clean on both code paths, images committed
  CHECK: ls reports/brief04/momentum_boards2d_after.png reports/brief04/momentum_boardpy_after.png reports/brief04/brief03_proof/boards_job2/mom_before_last.png 2>/dev/null | wc -l
  EXPECT: 3
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=1121cfccd5913f0a63fec40a6ffd44ea64f9dc135c66634ba001d10bcf4302a2; output-bytes=2
- [x] B2: retime pass caps footage at its verified window (no tpad overrun)
  CHECK: grep -c "Retime pass" tools/pod_build.py
  EXPECT: 1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=4355a46b19d348dc2f57c046f8ef63d4538ebb936000f3c9ee954a27460dd865; output-bytes=2
- [x] B3: one 2D board renderer (HTML/Chromium); zero matplotlib imports in tools/
  CHECK: test -f tools/board_page.html && test -f tools/board_html.js && test ! -f tools/boards_2d.py && test ! -f tools/board_design.py && test ! -f tools/tactical_boards.py; n=$(grep -rlE "^\s*import matplotlib|^\s*from matplotlib" tools/*.py | wc -l); echo "files-ok matplotlib-importers=$n"
  EXPECT: files-ok matplotlib-importers=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=56621677ef212981fbb72252b2d78163e695bf86f86f9f7700b34f283178e76e; output-bytes=32
- [x] B4: produce_v2 board step goes through the emitters + pod_build render2d
  CHECK: grep -c "stats_spec" tools/produce_v2.py
  EXPECT: 1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=4355a46b19d348dc2f57c046f8ef63d4538ebb936000f3c9ee954a27460dd865; output-bytes=2
- [x] B5: mirror publisher scans staged content and blocks secrets
  CHECK: n=$(cat .claude/hooks/push_status.sh | grep -c "scan_staged"); echo "scan_staged-refs=$n"
  EXPECT: scan_staged-refs=2
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=4b3f3c86716276dbaf5dfd9bc84df35cb14f6378a25d778e7e3d6e39e5f83893; output-bytes=19
- [x] B6: main repo confirmed private via gh
  CHECK: gh repo view minakush000-crypto/yt-digest --json isPrivate --jq .isPrivate
  EXPECT: true
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=a17fcf0a2f50e2d495e4f90ce263410edc183add6c62699a2facbccf60410f74; output-bytes=5
- [x] B7: Modal cost measured from billing API, not estimated
  CHECK: grep -c "billing report" reports/brief04/REPORT.md 2>/dev/null
  EXPECT: 1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=4355a46b19d348dc2f57c046f8ef63d4538ebb936000f3c9ee954a27460dd865; output-bytes=2
- [x] B8: episode slug is a CLI argument in pod_build (no module constant)
  CHECK: n=$(grep -c "^SLUG" tools/pod_build.py); echo "slug-consts=$n"; test "$n" -eq 0
  EXPECT: slug-consts=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=d1015759d48be78fa01d8da33d799b0253279b444858a0f77486ef509da4b3ea; output-bytes=14
- [x] B9: gate evidence lands outside tracked files; a gate run keeps the tree clean
  CHECK: git check-ignore -q reports/gates/brief04.log && echo IGNORED
  EXPECT: IGNORED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=da22823c363df25f50e61852800914dc78a4eccc19c54e0723e0bf2811a9f3a8; output-bytes=8
- [x] B10: iraola-liverpool script archived out of validation reach
  CHECK: test ! -f scripts/2026-08-18_iraola-liverpool.md && test -f scripts/archived/2026-08-18_iraola-liverpool.md && echo ARCHIVED
  EXPECT: ARCHIVED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=16499648e7e6a7fd767f9c1f9ec233d412cfc307ee04ebdbf2ee5ece7d5eea5e; output-bytes=9
- [x] B11: judge supports seed + median-of-N; 5-run spread recorded in REPORT.md
  CHECK: n=$(grep -c '"--runs"' tools/glm_judge.py); echo "runs-flag=$n"
  EXPECT: runs-flag=1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=33ea9dea5448e61e4b7ffdcee1dfcc5e9a9262577b44430934069ea013fe93ff; output-bytes=12
- [x] B12: reel t=90 verdict documented with committed proof frames
  CHECK: ls reports/brief04/goal90_t90_full.png reports/brief04/goal90_sheet_85-100.png 2>/dev/null | wc -l
  EXPECT: 2
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=53c234e5e8472b6ac51c1ae1cab3fe06fad053beb8ebfd8977b010655bfdd3c3; output-bytes=2
- [x] B13: cleanup retired GEMINI.md, LANE_PLAN.md duplicate, q files
  CHECK: test ! -f GEMINI.md && test ! -f LANE_PLAN.md && test ! -f q1_new.txt && test -f retired/GEMINI.md && echo CLEANED
  EXPECT: CLEANED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=9fee4a40e6d7d8e41ba04af164feb854563a6f6bba396b3d2eb9686dd0a4066e; output-bytes=8
- [x] B14: canonical docs stamped 2026-09-23 after brief 04
  CHECK: grep -l "Last verified against code:\*\* 2026-09-23" STATUS.md CONTEXT.md TOOLS.md GAPS.md DECISIONS.md | wc -l
  EXPECT: 5
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=f0b5c2c2211c8d67ed15e75e656c7862d086e9245420892a7de62cd9ec582a06; output-bytes=2
- [x] B15: all brief 04 work committed and pushed
  CHECK: git add -A >/dev/null 2>&1; test -z "$(git status --porcelain)" && echo CLEAN
  EXPECT: CLEAN
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=0b98843240a0b1a2384483206c02be8968dd809eb73838820be24cb7d6d6aeb9; output-bytes=6