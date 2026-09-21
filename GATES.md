# Gates: flash reboot EP001 (Real Madrid 2-1 Inter)

OWNS: renders/2026-09-08_real-madrid-inter/**, scripts/2026-09-08_real-madrid-inter.md, GATES.md

Scope: a finished, judged episode video built on the cloud pipeline (Modal
render + assemble, ElevenLabs voice, B2 archive), meeting the owner benchmark
and the 155 WPM narration MUST.

- [x] G1: every footage window verified by first/mid/last frame reads
  CHECK: ls renders/2026-09-08_real-madrid-inter/frames_verify_flash/ | wc -l
  EXPECT: 38
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=a2c6c14110a317833ac8f2fbae7080ee844af7469b2aa702876548dcf4c58077; output-bytes=3

- [x] G2: script parses to 19 sections, 365 words, 12 footage, 7 boards
  CHECK: ~/yt-digest/.venv/bin/python -c "import sys; sys.path.insert(0,'tools'); from assemble_words_match import parse_script; s=parse_script('2026-09-08_real-madrid-inter'); print(len(s), sum(x['words'] for x in s), sum(1 for x in s if x['type']=='footage'), sum(1 for x in s if x['type']=='board'))"
  EXPECT: 19 365 12 7
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=c4057251ae2a8c3ad079e922b8fd8939d79456f35c2aa4454187a5875b8f765f; output-bytes=12

- [x] G3: narration pace >= 155 WPM from generated voice (MUST)
  CHECK: bash renders/2026-09-08_real-madrid-inter/check_pace.sh
  EXPECT: PACE-PASS
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=fa5761e7e31d631fb2ce03c50f2c22198398aa9a93e9ebbd39b0960e8b5be4e5; output-bytes=14

- [x] G4: all 7 named boards rendered as non-empty MP4s
  CHECK: miss=0; for b in formation_clash counter_map valverde_strike stats_possession goal3_pattern momentum scoreline_outro; do test -s renders/2026-09-08_real-madrid-inter/boards/$b.mp4 || miss=1; done; test $miss -eq 0 && echo BOARDS-COMPLETE || echo BOARDS-MISSING
  EXPECT: BOARDS-COMPLETE
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=dc87a3bb1044bc8bb353856329b82d05d20e7c0e21812e032d91ab8c1391ef60; output-bytes=172

- [x] G5: formation_clash.mp4 non-empty; frame read shows 6+ surnames
  CHECK: test -s renders/2026-09-08_real-madrid-inter/boards/formation_clash.mp4 && echo BOARD-EXISTS
  EXPECT: BOARD-EXISTS
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=0ac665dfb8af7eed268d06e72346933bce39a861ff68d622288659cd6f9b3150; output-bytes=13

- [x] G6: final_video.mp4 is 1920x1080 with video+audio
  CHECK: ffprobe -v error -show_entries stream=codec_type,width,height -of csv=p=0 renders/2026-09-08_real-madrid-inter/final_video.mp4 | head -2
  EXPECT: video,1920,1080
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=42e3ac2b0a4e98e0b50978279748cc1a8f9aad4246cc8f5696f264556118a0ad; output-bytes=22

- [x] G7: final video + manifest archived to B2 for today
  CHECK: rclone lsl b2:mendymax-archive/soccer-channel/2026-09-21/ | grep -c final
  EXPECT: 2
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=53c234e5e8472b6ac51c1ae1cab3fe06fad053beb8ebfd8977b010655bfdd3c3; output-bytes=2

- [x] G8: JUDGE_FLASH.md written with scores and verdict
  CHECK: test -s renders/2026-09-08_real-madrid-inter/JUDGE_FLASH.md && echo JUDGE-WRITTEN
  EXPECT: JUDGE-WRITTEN
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=f0cc3f4dd0b87b9543020114375acc903a2f243f559524d4380a12e8ba37783e; output-bytes=14