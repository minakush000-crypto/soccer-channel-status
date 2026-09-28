# GATES_frozen.md — brief 07 frozen gates

Historical, not run. Frozen 2026-09-27 on Mayo's APPROVED ECC instruction:
"Before job 5, freeze gate B38 as historical (it proved no-install before
approval)." Per the ledger rules, a gate that snapshots a past state moves
verbatim with its evidence here. B38's invariant (ECC's rules dir NOT
installed) was true from authoring until approval and is now superseded by
the approved install (jobs 5-8).

- [x] B38: HARD STOP honored: summary exists and ECC's rules dir is NOT installed, with a planted positive control proving the matcher (brief 07 job 4; FREEZE this gate once Mayo types APPROVED ECC — jobs 5-8 install into rules/)
  CHECK: m=$(mktemp -d /tmp/b07pc.XXXXXX) && mkdir -p "$m/rules/ecc" && test -e "$m/rules/ecc" && rm -rf "$m" && test -s reports/brief07/summary.md && test ! -e ~/.claude/rules/ecc && echo HARD-STOP-OK
  EXPECT: HARD-STOP-OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=ec7f84a9a937/20 entries; EXPECT=matched; output-sha256=342e3ccdbf3cb67c6dedda020e29f7f325c8dc33cbc29cd327dc15c724d8c6b9; output-bytes=13