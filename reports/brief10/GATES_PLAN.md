# Brief 10 — gate plan (written before implementation; added to GATES.md in job 9)

Five live gates per brief 10 job 9. Drafted now, wired at job 9.

## B27: the disk guard exists and is wired to every entry point (grep the callers)
CHECK: test -x tools/disk_guard.sh && n=$(grep -rlE "disk_guard\.sh" tools/*.py .claude/hooks/*.sh .claude/settings.json 2>/dev/null | wc -l) && [ "$n" -ge 3 ] && echo GUARD-WIRED callers=$n
EXPECT: GUARD-WIRED

## B28: the guard's positive controls FAIL on faked states, real state PASSES
CHECK: bash tools/disk_guard.sh --selftest
EXPECT: GUARD-SELFTEST-OK

## B29: F: is mounted AND writable (write test through the guard's own probe)
CHECK: bash tools/f_mount.sh --check
EXPECT: F-MOUNT-OK

## B30: caches resolve to /mnt/f via the env file (no venv/node/git/playwright moved)
CHECK: bash -c 'source ~/.config/soccer/env.sh && [ "$(pip config get cache-dir 2>/dev/null || env | grep -o "PIP_CACHE_DIR=[^ ]*")" ] && case "$PIP_CACHE_DIR$npm_config_cache$HF_HOME$XDG_CACHE_HOME" in */mnt/f*) echo CACHES-ON-F;; *) echo CACHES-NOT-ON-F; exit 1;; esac'
EXPECT: CACHES-ON-F

## B31: .wslconfig carries the memory cap
CHECK: grep -qiE "^memory=" /mnt/c/Users/muads/.wslconfig && echo WSLCONFIG-CAPPED
EXPECT: WSLCONFIG-CAPPED

Notes:
- All CHECKs read-only (B28's selftest fakes states via a TEST override var,
  not by touching the real mount or floor).
- B29 will fail until Mayo's one-time remount command lands; that is the
  brief's own STOP design, not a gate bug.
- reverify + mirror at job 9 follow the brief 08/09 pattern (proof to /tmp
  first, then committed as the last artifact).
