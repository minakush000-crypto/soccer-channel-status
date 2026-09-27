# Brief 09 — gate plan (written before implementation, added to the GATES.md ledger in job 8)

Four live gates cover the new artifacts, per brief 09 job 8: files exist,
CSV row counts match duration/2 within 1, sheets <= 12, local video deleted.
Durations are recorded at job 1 by ffprobe into reports/brief09/durations.json
BEFORE the videos are deleted (the gate reads that file; the videos are gone).

## B23: brief 09 research artifacts exist (transcripts, CSVs, docs, catalog examples)

CHECK: cd "$SRC" && for f in reports/brief09/frames_A.csv reports/brief09/frames_B.csv reports/brief09/frames_C.csv reports/brief09/transcript_A.txt reports/brief09/transcript_B.txt reports/brief09/transcript_C.txt reports/brief09/measurements.md reports/brief09/overlay_catalog.md reports/brief09/gaps.md reports/brief09/durations.json; do test -f "$f" || exit 1; done && echo BRIEF09-ARTIFACTS-OK

EXPECT: BRIEF09-ARTIFACTS-OK

## B24: CSV row counts match duration/2 within 1 (positive control: a tampered count fails)

CHECK: .venv/bin/python tools/brief09_check.py rows
EXPECT: ROWS-OK

## B25: contact sheets exist and total <= 12

CHECK: n=$(ls reports/brief09/sheet_*.jpg 2>/dev/null | wc -l) && [ "$n" -ge 1 ] && [ "$n" -le 12 ] && echo SHEETS-OK
EXPECT: SHEETS-OK

## B26: local benchmark videos deleted (positive control: the planted .mp4 the matcher must catch)

CHECK: mkdir -p /tmp/b09_pc && printf x > /tmp/b09_pc/pc.mp4 && n=$(find /tmp/b09_pc -name "*.mp4" 2>/dev/null | wc -l) && rm -rf /tmp/b09_pc && [ "$n" -eq 1 ] && real=$(find /mnt/f/benchmark -maxdepth 1 -type f \( -name "*.mp4" -o -name "*.mkv" -o -name "*.webm" \) 2>/dev/null | wc -l) && [ "$real" -eq 0 ] && echo BENCH-LOCAL-DELETED
EXPECT: BENCH-LOCAL-DELETED

Notes:
- B24 runs from a script (tools/brief09_check.py) because the within-1
  comparison needs arithmetic; B19 set this pattern (bash script as gate
  CHECK). The script must not modify the repo (reads only).
- All four gates are read-only: no git add, no writes outside /tmp (B26
  plants its control under /tmp/b09_pc and removes it).
- B24's tolerance "within 1" follows the brief: rows may differ from
  floor(duration/2) by at most 1 (frame boundary rounding).