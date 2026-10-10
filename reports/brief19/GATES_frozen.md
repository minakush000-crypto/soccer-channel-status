# GATES_frozen.md — historical, not run

Moved 2026-10-10 (brief 19) under the brief-08 ledger rule: LIVE = an
invariant that must hold on every future commit. FROZEN = snapshots a past
state, moved verbatim with its evidence here, headed historical, not run.

- [x] B3: one 2D renderer; zero matplotlib importers in tools/ (positive control included)
  CHECK: test -f tools/board_page.html && test -f tools/board_html.js && printf 'import matplotlib\n' > /tmp/b08_pc_b3.py && grep -qE "^\s*(import matplotlib|from matplotlib)" /tmp/b08_pc_b3.py && rm -f /tmp/b08_pc_b3.py && n=$(grep -rlE "^\s*import matplotlib|^\s*from matplotlib" tools/*.py | wc -l) && echo "ONE-RENDERER matplotlib-importers=$n"
  EXPECT: ONE-RENDERER matplotlib-importers=0
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=f6ab58f858b3/19 entries; EXPECT=matched; output-sha256=075f6c4fda1e62134063d83ea711df518f3738048209a044746cc4fd72f1edb1; output-bytes=36

Replaced in the live ledger by B125 (two-renderer invariant) on 2026-10-10,
brief 19 (DECISIONS.md decision 25). Reason: brief 19 job 1 mandates a
mplsoccer/matplotlib diagram renderer in tools/diagram_gen.py; the
coordinator's brief outranks the brief-04 decision, and B3's one-renderer-
per-family intent is preserved live by B125 (exactly one matplotlib
importer in tools/, exactly one HTML board renderer, planted control).
Nothing in tools/ referenced B3 (grep for "GATES:B3"/"gate B3" empty,
checked 2026-10-10).