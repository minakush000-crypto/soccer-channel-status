# brief06 gates — FROZEN (historical, not run)

> Moved out of the live ledger by brief 08 job 2 (Mayo decision 2):
> these gates snapshot a PAST state (durations, dates, copied counts,
> one-time deletions/proofs). They are kept verbatim with their last
> evidence for the record. They are NOT run by gate-check, and their
> stored evidence must never be read as current truth — brief 06's
> '33/33 gates pass' false-pass is exactly that failure.

- [x] B21: episode rebuilt from the facts step, every board segment frame checked
  CHECK: d=$(ffprobe -v error -show_entries format=duration -of csv=p=0 renders/2026-09-08_real-madrid-inter/final_video.mp4 | cut -d. -f1); n=$(ls reports/brief06/final_boards/*.png 2>/dev/null | wc -l); echo "FINAL-REBUILT dur=${d}s frames=$n"
  EXPECT: FINAL-REBUILT dur=140s frames=7
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=98aaefd8d840/21 entries; EXPECT=matched; output-sha256=37cea1a994d472a1394f1d007ceec2f36cd628a43fbd18c175ee1654021aaa83; output-bytes=32
