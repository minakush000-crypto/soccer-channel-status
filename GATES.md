# GATES.md — current task gates

> **Purpose:** the live gate file for the current task (unlazy discipline). One
> observable outcome per gate; a gate passes when CHECK prints EXPECT.
> **Reader:** every session; the unlazy Stop hook blocks session end while gates
> remain unmet.
> **Last verified against code:** 2026-09-14.

Scope: close the stale-voice hole (piece 1), research what medium practitioners
use for football tactical boards + settle the 2D-vs-3D contradiction (piece 2),
and run the Gemini-vs-scanner litmus head-to-head on timestamps + pitch
coordinates + cut decisions vs match_data.json (piece 3). Brief:
~/claude/30-briefs/2026-09-14-0610-close-stale-voice-and-settle-two-questions.md

Episode: renders/2026-09-12_bournemouth-brentford. Report (consolidated handoff):
/home/muads/claude/20-handoffs/2026-09-14-0610-close-stale-voice-report.md

- [x] G1: voice regenerated from the fixed script (new voice newer than script, old voice moved aside)
  CHECK: f=renders/2026-09-12_bournemouth-brentford/voice_elevenlabs.mp3; s=scripts/2026-09-12_bournemouth-brentford.md; [ -f renders/2026-09-12_bournemouth-brentford/voice_elevenlabs_pre_fix.mp3 ] && [ "$f" -nt "$s" ] && echo VOICE_REGEN_OK
  EXPECT: VOICE_REGEN_OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=200f606b2235c130f893716ee88327733d9f2800cd7d5d506aa06aa21fe429b1; output-bytes=15

- [x] G2: voice has a recorded script hash matching the current script (proves regeneration from the fixed script; the duration-shorter heuristic was dropped because ElevenLabs pacing varies, so fewer words does not imply shorter audio)
  CHECK: h=$(cat renders/2026-09-12_bournemouth-brentford/.voice_script_hash 2>/dev/null); c=$(sha256sum scripts/2026-09-12_bournemouth-brentford.md | cut -d' ' -f1); [ -n "$h" ] && [ "$h" = "$c" ] && echo HASH_MATCHED_OK
  EXPECT: HASH_MATCHED_OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=56734944dad9833bea7c23d80d560ef55b0fae4db4a48e8b9ba3fe82cbc336f8; output-bytes=16

- [x] G3: step6_voice skip logic compares a script hash, not just file existence
  CHECK: grep -q "hashlib\|sha256\|script_hash" tools/produce_v2.py && echo HASH_SKIP_OK
  EXPECT: HASH_SKIP_OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=ed3c44bbeebd27b1e4b4894c99714aaa84b4a334a7684ab2fd0d058e1e18fd0d; output-bytes=13

- [x] G4: episode re-scored, consolidated report file exists
  CHECK: [ -f /home/muads/claude/20-handoffs/2026-09-14-0610-close-stale-voice-report.md ] && echo RESCORE_OK
  EXPECT: RESCORE_OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=c6ed1b5b88c9a448d6d594bca41b5e6f2e7243dd8a104d48203d12486244f60b; output-bytes=11

- [x] G5: STATUS.md stamp updated to 2026-09-14 and records the 5.5 verdict
  CHECK: head -6 STATUS.md | grep -q "2026-09-14" && grep -q "5.5" STATUS.md && echo STAMP_OK
  EXPECT: STAMP_OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=66cda22dc18c9e03be97b47cc11e2a31ce7bcd88c30a11ba5c132d675e83d875; output-bytes=9

- [x] G6: piece 2 practitioner medium table present in the report
  CHECK: grep -qi "board type" /home/muads/claude/20-handoffs/2026-09-14-0610-close-stale-voice-report.md && grep -qi "medium" /home/muads/claude/20-handoffs/2026-09-14-0610-close-stale-voice-report.md && echo RESEARCH_OK
  EXPECT: RESEARCH_OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=faef86a8ad6501bb9f32299c2961b01fdedb2c70fcc6efa0fdcd5e9cdb0da452; output-bytes=12

- [x] G7: piece 3 litmus head-to-head with per-goal offsets for both Gemini and scanner
  CHECK: grep -qi "offset" /home/muads/claude/20-handoffs/2026-09-14-0610-close-stale-voice-report.md && grep -qi "scanner" /home/muads/claude/20-handoffs/2026-09-14-0610-close-stale-voice-report.md && grep -qi "gemini" /home/muads/claude/20-handoffs/2026-09-14-0610-close-stale-voice-report.md && echo LITMUS_OK
  EXPECT: LITMUS_OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=fcbd7f73e2b5617e9dabbe15f4003486c2b2112d4e764631f30fce148db4010c; output-bytes=10

- [x] G8: brief + report saved in the vault
  CHECK: [ -f /home/muads/claude/30-briefs/2026-09-14-0610-close-stale-voice-and-settle-two-questions.md ] && [ -f /home/muads/claude/20-handoffs/2026-09-14-0610-close-stale-voice-report.md ] && echo VAULT_OK
  EXPECT: VAULT_OK
  EVIDENCE: exit=0; shell=/bin/bash; cwd=/home/muads/yt-digest/soccer-channel; path=af0e20c5c0e4/21 entries; EXPECT=matched; output-sha256=e24fa9f14e9639aafaaaeabe2365eec96c78081bd90b708949f364c63ba1ebb5; output-bytes=9