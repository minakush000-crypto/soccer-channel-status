# Gates: Stage 5 (pod-side path + rule-3 debt)

> **Purpose:** the unlazy gate file for Stage 5 (completed). A working artifact of that stage, kept for history.
> **Reader:** historical reference; the unlazy Stop hook that consumed it has finished. Not read by any pipeline tool.
> **Last verified against code:** 2026-09-09.

OWNS: tools/runpod_download.py, tools/produce_v2.py, tools/runpod_fulltrack.py, tools/runpod_stage1.py, tools/runpod_annotate.py, LANE_PLAN.md

Scope: build the pod-side download path (5A), retire the 4 dead tools and run the standalone smoke test (5C), report on storage (5B), append Stage 5 to LANE_PLAN.md and push.

- [x] G1: runpod_download.py launches with a usage line
  CHECK: ~/yt-digest/.venv/bin/python tools/runpod_download.py --help 2>&1 | grep -m1 -c "usage: runpod_download.py"
  EXPECT: 1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=11abdf0c0c5d/34 entries; EXPECT=matched; output-sha256=4355a46b19d348dc2f57c046f8ef63d4538ebb936000f3c9ee954a27460dd865; output-bytes=2

- [x] G2: produce_v2 routes step3 through the pod path
  CHECK: n=$(grep -c "runpod_download" tools/produce_v2.py); [ "$n" -ge 1 ] && echo OK_POD_WIRED
  EXPECT: OK_POD_WIRED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=11abdf0c0c5d/34 entries; EXPECT=matched; output-sha256=a2efeb0d315cb197d4e31a552e33ea8ef7947e2840fa15bb46e6b56c23e6e340; output-bytes=13

- [x] G3: the pod path uses 1080p, not the old 720p cap
  CHECK: n=$(grep -cE "height<=1080|POD_MAX_HEIGHT" tools/runpod_download.py); [ "$n" -ge 1 ] && echo OK_1080
  EXPECT: OK_1080
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=11abdf0c0c5d/34 entries; EXPECT=matched; output-sha256=296f2dfc3d5a03237823a32f69e5c2f922ffd3f6062d07ccf7144b655f3ef652; output-bytes=8

- [x] G4: the real pod run outcome is recorded in LANE_PLAN.md
  CHECK: n=$(grep -c "5A.3" LANE_PLAN.md); [ "$n" -ge 1 ] && echo OK_PROOF_RECORDED
  EXPECT: OK_PROOF_RECORDED
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=11abdf0c0c5d/34 entries; EXPECT=matched; output-sha256=04070b27f98dabdbdc1533101439f675fda458e766e3267bfe98245130095a93; output-bytes=18

- [x] G5: no local file over the 200MB guard was written during the run
  CHECK: n=$(find renders -newermt "2026-09-09 00:00" -size +200M -type f 2>/dev/null | wc -l); [ "$n" -eq 0 ] && echo OK_NO_OVERSIZE
  EXPECT: OK_NO_OVERSIZE
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=11abdf0c0c5d/34 entries; EXPECT=matched; output-sha256=11cf125a5f9509007bc2632479d326380739f305e4d4082a9bab265d1d287488; output-bytes=15

- [x] G6: the 4 DEAD tools are deleted from the tree
  CHECK: for t in tactical_overlay pitch_radar render_video check_and_download; do [ ! -f tools/$t.py ] && echo gone; done | wc -l | tr -d ' '
  EXPECT: 4
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=11abdf0c0c5d/34 entries; EXPECT=matched; output-sha256=7de1555df0c2700329e815b93b32c571c3ea54dc967b89e81ab73b9972b72d1d; output-bytes=2

- [x] G7: no remaining pitch_radar references in tools
  CHECK: n=$(grep -rl "pitch_radar" tools/ --include="*.py" 2>/dev/null | wc -l); [ "$n" -eq 0 ] && echo OK_NO_PITCH_RADAR
  EXPECT: OK_NO_PITCH_RADAR
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=11abdf0c0c5d/34 entries; EXPECT=matched; output-sha256=795d69c5cb04aeff441ef4153733f601f1ad340f869072f25e87abc62021b725; output-bytes=18

- [x] G8: the 27-tool standalone smoke test completed
  CHECK: grep -c "SMOKE DONE" /tmp/smoke_results.txt
  EXPECT: 1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=11abdf0c0c5d/34 entries; EXPECT=matched; output-sha256=4355a46b19d348dc2f57c046f8ef63d4538ebb936000f3c9ee954a27460dd865; output-bytes=2

- [x] G9: Stage 5 appended to LANE_PLAN.md
  CHECK: grep -c "STAGE 5" LANE_PLAN.md
  EXPECT: 1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=11abdf0c0c5d/34 entries; EXPECT=matched; output-sha256=4355a46b19d348dc2f57c046f8ef63d4538ebb936000f3c9ee954a27460dd865; output-bytes=2

- [x] G10: Stage 5 committed and pushed
  CHECK: git log --oneline -1 | grep -c "Stage 5"
  EXPECT: 1
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=11abdf0c0c5d/34 entries; EXPECT=matched; output-sha256=4355a46b19d348dc2f57c046f8ef63d4538ebb936000f3c9ee954a27460dd865; output-bytes=2