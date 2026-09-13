# GATES.md — current task gates

> **Purpose:** the live gate file for the current task (unlazy discipline). One
> observable outcome per gate; a gate passes when CHECK prints EXPECT.
> **Reader:** every session; the unlazy Stop hook blocks session end while gates
> remain unmet.
> **Last verified against code:** 2026-09-13.
> **Mirrored:** yes (in push_status.sh ALLOW_PROJECT and the mirror .gitignore
> un-ignore list — verified 2026-09-13).

Scope: document reconciliation (Stage 14 interlude) — make the "Last verified
against code" stamp real. PRECEDENCE + STAMP RULE in CLAUDE.md; re-verify or
re-date stamps across canonical docs; self-maintaining SessionStart stamp hook;
GATES.md header; mirror allowlist reconciliation; archive historical files.
The 7 docs with heavy Stage-12B line-drift (CONTEXT/STATUS/PROGRESS/TOOLS/
ARCHITECTURE/LANE_PLAN/GAPS) are reported-with-unverified-claims (STAMP RULE
option b), NOT re-stamped this pass; their stamps stay 2026-09-09 and the hook
flags them. That is the honest end state, not a gap to close silently.

- [x] G1: PRECEDENCE + STAMP RULE blocks present in CLAUDE.md
  CHECK: grep -qF "PRECEDENCE. When two sources disagree" CLAUDE.md && grep -qF "STAMP RULE. Any task that edits" CLAUDE.md && echo precedence_stamp_present
  EXPECT: precedence_stamp_present
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=f281a7609afea7fbbcf8a50760ced69be91e88dda30e0b23113b3a66ea3125a6; output-bytes=25

- [x] G2: CLAUDE.md stamp 2026-09-13 and runpod_annotate row removed
  CHECK: grep -qF "Last verified against code:** 2026-09-13" CLAUDE.md && ! grep -q runpod_annotate CLAUDE.md && echo claude_clean
  EXPECT: claude_clean
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=7dedd6861da6b32b59b7b80d54c50b23e00904104949acc34966d2dc5fde8391; output-bytes=13

- [x] G3: doc-stamp SessionStart hook wired (3 hooks, additive)
  CHECK: python3 -c "import json;d=json.load(open('.claude/settings.json'));ss=[h for blk in d['hooks']['SessionStart'] for h in blk['hooks']];print(len(ss),'check_doc_stamps' in ''.join(h['command'] for h in ss))"
  EXPECT: 3 True
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=1df8186257801f3a0247af7e7af157a615ccc69553182939919be5295e9b75ca; output-bytes=7

- [x] G4: doc_stamp_check.py runs and reports
  CHECK: ~/yt-digest/.venv/bin/python tools/doc_stamp_check.py | grep -q "^doc-stamp-check:" && echo hook_reports
  EXPECT: hook_reports
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=10de85f091ad5bb97e6832b286d8484af213b3a3c7919a32139c601b01cf64f1; output-bytes=13

- [x] G5: DECISIONS.md stamp 2026-09-13 + STAMP RULE entry
  CHECK: grep -qF "Last verified against code:** 2026-09-13" DECISIONS.md && grep -qF "STAMP RULE + known weakness" DECISIONS.md && echo decisions_stamped
  EXPECT: decisions_stamped
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=d16b749d6d882374fa29f65d13bc6fae0f94fbd89cb0720fab5d9a830e455aaf; output-bytes=18

- [x] G6: GATES.md has Purpose/Reader/stamp header
  CHECK: head -12 GATES.md | grep -q "Purpose:" && head -12 GATES.md | grep -q "Last verified against code" && echo gates_header
  EXPECT: gates_header
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=99812583488df9224d4627b4c9ce9a1d9b4be8942f95b922c622497b2d42e053; output-bytes=13

- [x] G7: 3 historical files archived, SKILLS.md kept
  CHECK: test -f archive/RECONCILIATION.md && test -f archive/REPORT_AUDIT.md && test -f archive/VISUAL_QUALITY_ASSESSMENT.md && test -f SKILLS.md && ! test -f RECONCILIATION.md && echo archived
  EXPECT: archived
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=3eb992486b31ee03214bd2688612fb599daaafad29d99081849788a696a9df1d; output-bytes=9

- [x] G8: no .bak files in project (excluding .git and archive)
  CHECK: test -z "$(find . -name '*.bak' -not -path './.git/*' -not -path './archive/*')" && echo no_bak
  EXPECT: no_bak
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=ba02a31e1f95efda21cd29d51197b480eaf239eb1c920e2cda099aaa521c74ea; output-bytes=7

- [x] G9: EPISODE_SPEC + SCRIPT_TEMPLATE stamps 2026-09-13
  CHECK: grep -qF "Last verified against code:** 2026-09-13" EPISODE_SPEC.md && grep -qF "Last verified against code:** 2026-09-13" SCRIPT_TEMPLATE.md && echo specs_stamped
  EXPECT: specs_stamped
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=322860f24f18aafebae3ef37e76b7df6e1acd902d9fa53b6b4b4f091becdfaa8; output-bytes=14

- [x] G10: the 5 re-stamped docs no longer flagged stale by the hook
  CHECK: ~/yt-digest/.venv/bin/python tools/doc_stamp_check.py | grep -E "STALE (CLAUDE|DECISIONS|GATES|EPISODE_SPEC|SCRIPT_TEMPLATE)\.md" >/tmp/g10.txt 2>&1; if [ -s /tmp/g10.txt ]; then echo STALE_REMAINS; else echo none_stale; fi
  EXPECT: none_stale
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=671408d498b4/21 entries; EXPECT=matched; output-sha256=5d7eafc220ff22441a6e2b148dd216d499714b5c7e9ff87a228c159356ed0085; output-bytes=11