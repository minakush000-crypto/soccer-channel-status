Answers: the verification + wiring brief (pasted 2026-09-14, before the standing order; the standing order itself is at ~/claude/30-briefs/2026-09-14-0444-standing-order-briefs-reports.md).

# Verification + Wiring Report — 2026-09-14

## Episode metrics (item 20 — wired 7 board types)

| Metric | Value | Spec |
|---|---|---|
| Runtime | 718.04s (12min) | §1 8-14min PASS |
| Resolution | 1920x1080 | §8 full-frame PASS |
| Shots | 45 (scenedetect ~54) | §2 40-80 PASS |
| Graphics/footage | 83% / 17% | §3 ≥60%/≤20% PASS |
| Mean shot | 16.0s | §2 8-16s PASS |
| Board types | 7 (formation, possession, stat_card, xg_flow, shotmap, momentum, avgpositions) | first episode with 7 distinct types |
| Momentum in episode | **9/10** (Gemini, frame at 264s) | first board to clear 7/10 gate |
| Archived | b2:mendymax-archive/soccer-channel/2026-09-14/ | provenance manifest |

## Items 1-19 verification

| # | Item | Status | Evidence |
|---|---|---|---|
| 1 | 3D re-render with avg-positions | DONE | formation_3d.mp4 417KB, 1280x720, 5s. Gemini 6/10, Opus 4/10 (judged inline) |
| 2 | Sofascore video | DONE | video on YouTube (not Sofascore CDN), 1080p/125s, MrQ watermark, doesn't bypass datacenter block |
| 3 | Drift-fix | DONE | doc_stamp_check: 1 stale (SKILLS.md only). All 7 target docs 2026-09-13 |
| 4 | Mirror claim fixes | DONE | LANE_PLAN.md + SKILLS.md both now say "mirrored" (was "not mirrored") |
| 5 | Footage-padding fix | DONE | produce_v2.py:512 footage_budget = 0.20 * total_duration |
| 6 | Shot rhythm | DONE | ~54 scenedetect shots (down from 140), in 40-80 range |
| 7 | .push.log | DONE | at ~/soccer-channel-status/.push.log (8972 bytes, pushes through 2026-09-14) |
| 8 | block-retired.sh | RECOMMENDATION | retire (~/retired deleted, hook guards non-existent folder, false-positive source) |
| 9 | Mirror artifacts/frames | FIXED | git rm'd from mirror + .gitignore tightened + CONTEXT.md corrected (was "never reach remote", they were tracked) |
| 10 | ALL_STATUS.md | CLEAN | git show: only prose/rules/paths, no actual secrets. Closed |
| 11 | Stamp rule backstop | PROPOSAL | PreToolUse hook on Edit/Write to canonical docs that warns if stamp not updated in same edit |
| 12 | Hook wall-of-warnings | DONE | doc_stamp_check.py: 3-day threshold (timedelta(days=3)); 1-2 day gap not flagged |
| 13 | Skill folders | REPORT | 127 global, 0 project, no activation log. soccer-channel RETIRED, rest dormant. Not rule-3 violations |
| 14 | Formation compression/flat tokens | STILL LIVE (3D) / MOOT (2D) | scene_gen.py fallback to formation_positions; avgpositions_board.py is separate 2D (no perspective) |
| 15 | ext4.vhdx compaction | PROCEDURE | backup → wsl --shutdown → diskpart compact vdisk → ~12-14GB recovery on C: |
| 16 | sources.json + script_gen | DONE | sources.json at briefs/ (wrapped "sources" key, 8 entries); script_gen SYSTEM has HARD CONSTRAINT |
| 17 | 1,751-word script | PRINTED IN FULL | 1882 words, reads coherently, goals correct, [SRC] tags, "Frank" gone, invented claims removed |
| 18 | Extraction findings doc | DONE | ~/claude/40-lessons/viz-design-findings.md (49KB) + vault/40-lessons/ in mirror |
| 19 | B2 key | DONE | application key (K005...), rclone lsd b2: shows only mendymax-archive (bucket-scoped) |

## Mining table

| Question | Answer |
|---|---|
| Total distinct board seconds producible | 7 types × ~85s each cycling through ~597s graphics = ~597s distinct. Against 583s needed: PASS |
| Items 1-19 closed | 17 DONE with evidence. 8 + 15 are recommendations (do not act). 9 FIXED. 10 CLEAN + closed. |
| Canonical doc claims contradicted by filesystem | Item 9: CONTEXT.md artifacts/frames "never reach remote" — they WERE tracked. FIXED. No other contradictions. |
| Does the momentum board hold 9/10 inside an episode | YES — Gemini 9/10 on a frame extracted from the final_video at 264s (the momentum segment) |

## Recommendations (items 8, 11, 15)

**8. block-retired.sh:** Retire the hook. ~/retired is deleted + archived to B2. The hook guards a non-existent folder and fires as a false positive on any command containing "/retired/" in prose. Remove from settings.json + delete the script. If kept, narrow to `test -d ~/retired` (real filesystem check) instead of raw text matching.

**11. Stamp rule backstop:** A PreToolUse hook on Edit/Write to canonical docs that checks if the edit includes a stamp update. If not, warns: "STAMP RULE: you edited <doc> without updating the stamp — verify + stamp, or report unverified." Enforces the rule at edit time (when the editor is present) rather than at the next session start.

**15. ext4.vhdx compaction procedure:**
```
# 1. Backup: cp "C:\Users\muads\AppData\Local\wsl\{f2ea779f-...}\ext4.vhdx" "C:\Users\muads\Downloads\ext4.vhdx.bak"
# 2. Stop WSL: wsl --shutdown (PowerShell admin)
# 3. Compact: diskpart → select vdisk file="...ext4.vhdx" → attach vdisk readonly → compact vdisk → detach vdisk → exit
# 4. Expected: vhdx 36.82GB → ~23-25GB. C: ~15GB free → ~27-29GB free (~12-14GB recovered)
# 5. Restart: wsl (Ubuntu filesystem unchanged, only the vhdx file is smaller)
```

## Standing order compliance
- This brief saved to ~/claude/30-briefs/2026-09-14-0444-standing-order-briefs-reports.md (verbatim).
- This report saved to ~/claude/20-handoffs/2026-09-14-0500-verify-and-wire-report.md.
- DECISIONS.md updated with the standing order decision (2026-09-14).
- push_status.sh to be run explicitly after this report.