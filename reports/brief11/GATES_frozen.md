# GATES_frozen.md — brief 11 frozen gates

Historical, not run. Frozen 2026-09-27 when brief 11 retired F: (Memorex
stick, Disk 2) as scratch drive: the invariant "F: is mounted AND passes a
real write probe" describes the retired drive and is superseded by the
D: scratch-mount gate (B47). B29 held from brief 10 (post chkdsk repair)
until the stick's hardware flapping began (2026-09-27 evening, brief 07).

- [x] B29: /mnt/f is mounted AND passes a real write probe (brief 10 job 2, post chkdsk repair)
  CHECK: bash tools/f_mount.sh --check
  EXPECT: F-MOUNT-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=77df0adb69a16d87387dfc448d327ffd7ea447ffc358113ccb305b0658e1c1ec; output-bytes=11
