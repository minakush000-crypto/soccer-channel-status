# Brief 07 job 7: rollback path (documented, NOT run)

Date: 2026-09-27. Produced by: Flash (glm-5.3-flash:cloud).

## Option 1: ECC's own uninstall (removes exactly what ECC installed)

    cd ~/ECC && node scripts/uninstall.js --dry-run

Output of the dry run (2026-09-27, captured verbatim):

    Uninstall summary (dry run; nothing was removed):

    - claude-home
      Status: WOULD UNINSTALL (dry run)
      Install-state: /home/muads/.claude/ecc/install-state.json
      Would remove: 816

    Summary (dry run): checked=1, planned=1, partial=0, errors=0

It removes the 816 paths ECC recorded in install-state.json (the 814 copies +
settings hook entries). To actually run it, drop --dry-run. Note: ECC's
uninstall restores settings.json hooks it manages; the unlazy/claude-mem/
disk-guard hooks were never ECC-owned and are not touched by it.

## Option 2: full restore from the pre-ECC backup on B2 (nuclear, exact)

The job-1 backup (continuation option A) is the byte-exact ~/.claude +
~/.claude.json snapshot taken BEFORE the install, verified at 16,545 entries:

    rclone cat b2:mendymax-archive/backups/2026-09-27_claude_home_pre_ecc.tar.gz | tar xzf - -C ~

Run this from WSL with NO Claude Code session open (this very session holds
files in ~/.claude). It overwrites ~/.claude and ~/.claude.json with the
pre-ECC state; anything created after the backup (the whole ECC install, the
context-transfer skill, the rules/precedence.md file, the CLAUDE.md line) is
gone with it. Size-verified 592,458,956 bytes; entry count verified by the
same stream on 2026-09-27.

NOT RUN (job 7 rule): neither option was executed. Both were documented only.