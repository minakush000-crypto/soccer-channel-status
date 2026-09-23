# Gates: brief 03 (retire dead tools, board fixes, canonical sync)

OWNS: tools/**, GATES.md, canonical docs, episodes/, ~/.claude/CLAUDE.md

Scope: brief 03 — retire every tool not reachable from produce_v2.py/pod_build.py
(24 files), fix momentum label overlap + four-section outro freeze (verified by
viewed frames), replace project CLAUDE.md, rewrite global CLAUDE.md, rewrite
STATUS/CONTEXT/DECISIONS, delete invented architecture, resolve the disabled
image-block hook.

- [x] G1: dead tools retired — 37 .py files remain, none of the 25 retired names exist
  CHECK: n=$(ls tools/*.py | wc -l); miss=0; for t in scene_gen player_mapper render_goal_clip ltx_enhance modal_render3d parse_feed assemble_video assemble_words_match bulk_classify cloud_produce cv_annotate enhance_clips generate_captions gpu_superres runpod_fulltrack runpod_scenedetect runpod_superres render3d_vast segment_scorer sharpness_check trim_tracking produce_episode b2_upload; do test -e "tools/$t.py" && miss=1; done; test -e tools/match_moments.workflow.mjs -o -e tools/rematch_ep001.workflow.mjs && miss=1; echo "py=$n miss=$miss"
  EXPECT: py=37 miss=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=a32edeaaa1ada82adb0f77fb26e27da614fd393249254e600cce2d9a89e39084; output-bytes=13
- [x] G2: no kept/reachable tool still names a retired tool as live (present-tense refs gone; lines that say "retired" are historical notes)
  CHECK: n=$(grep -rnE "scene_gen|modal_render3d|runpod_fulltrack|generate_captions|assemble_words_match" tools/*.py | grep -vE "produce_v2|runpod_download|pod_check" | grep -viE "retired" | wc -l); test "$n" -eq 0 && echo REFS-CLEAN
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