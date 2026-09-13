# Gates: 3D formation board re-run + multi-lens judge (design vs dimension)

OWNS: artifacts/frames/**, renders/2026-09-06_arsenal-chelsea/**, PROGRESS.md, STATUS.md, LANE_PLAN.md

Scope: re-run scene_gen.py on Modal against arsenal-chelsea match_data.json (as written, no edits), extract one frame, judge it against the 2D possession.png control through multiple independent vision lenses, and record a design-vs-dimension verdict plus the contradiction with the relay.

- [x] G1: arsenal-chelsea 3D MP4 rendered and valid
  CHECK: ffprobe -v error -show_entries stream=width,height,codec_name -show_entries format=duration -of default=noprint_wrappers=1 /mnt/f/soccer-staging/3dpoc_2026-09-06_arsenal-chelsea.mp4 | tr '\n' ' '
  EXPECT: codec_name=h264 width=1280 height=720 duration=5.000000
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=38bcba114a2d66890d83be5cc37a4408674d474f7503f9ae43c14d55ba67ac2e; output-bytes=56

- [x] G2: arsenal-chelsea 3D frame extracted at t=2.5s
  CHECK: test -s artifacts/frames/3dpoc_arsenal-chelsea_t2500.png && ffprobe -v error -show_entries stream=width,height -of csv=p=0:s=x artifacts/frames/3dpoc_arsenal-chelsea_t2500.png
  EXPECT: 1280x720
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=3cbae94edfc4034591954cc8ed588e2db0b4ee4f34f53eeeba9d9076443c1c28; output-bytes=9

- [x] G3: Opus (authoritative) scored the 3D frame on the 2D-board scale
  CHECK: test -s artifacts/frames/opus_3d_verdict.txt && grep -qE '[0-9]+/10' artifacts/frames/opus_3d_verdict.txt && echo opus_3d_scored
  EXPECT: opus_3d_scored
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=1b5bfba2604966de08b5d6e81c4c5167470e5c8d7007dd0421ad8c8aecd63495; output-bytes=15

- [x] G4: Opus (authoritative) scored the 2D possession.png control on the same scale
  CHECK: test -s artifacts/frames/opus_2d_control_verdict.txt && grep -qE '[0-9]+/10' artifacts/frames/opus_2d_control_verdict.txt && echo opus_2d_scored
  EXPECT: opus_2d_scored
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=53506ca0df7eda75d5adae69bbfbc9a49652286422af7ad5a0e54e43adaff5b9; output-bytes=15

- [x] G5: multi-lens synthesis verdict written (design vs dimension, with the relay contradiction)
  CHECK: test -s artifacts/frames/synthesis_verdict.md && grep -qi 'design' artifacts/frames/synthesis_verdict.md && grep -qi 'dimension\|3d' artifacts/frames/synthesis_verdict.md && grep -qi 'contradict' artifacts/frames/synthesis_verdict.md && echo synthesis_written
  EXPECT: synthesis_written
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=e3e72846126d77be278e7519c2663b6cb083caca385c5b4eb82afb60b35c8dac; output-bytes=18

- [x] G6: PROGRESS.md records this run (provider, cost, wall time, output path, frame path)
  CHECK: grep -q '3dpoc_2026-09-06_arsenal-chelsea' PROGRESS.md && grep -qi 'modal' PROGRESS.md && echo progress_updated
  EXPECT: progress_updated
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=70b3a7e57858ffd7be87d9f243b91662a0e3c63420851b2828c9e83a45218520; output-bytes=17