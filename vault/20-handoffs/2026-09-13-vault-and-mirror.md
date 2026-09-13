# Handoff 2026-09-13 — vault and mirror

## Built tonight
- Obsidian vault at ~/claude, six folders: 00-inbox 10-projects 20-handoffs
  30-briefs 40-lessons 90-archive. Obsidian app on Windows, opened via
  \\wsl.localhost\Ubuntu\home\muads\claude
- push_status.sh at ~/yt-digest/soccer-channel/.claude/hooks/
  Allowlist only. Copies named project docs plus vault folders 10/20/30/40.
  Deletes any .env it finds in the mirror. Stamps README with time + verdict.
  Recovers from a rejected push by pulling and retrying. Logs to .push.log.
- Registered as a Stop hook. Project hooks 6 -> 7. Backup at settings.json.bak
- Mirror: github.com/minakush000-crypto/soccer-channel-status (Public)

## CORRECTIONS to the previous handoff
- There was never an auto-push hook. No push_status.sh existed, no cron,
  no Stop hook. GLM pushed by hand. The "3-day stop-hook gap" was simply
  GLM not remembering for three days.
- Hook count was 6. It is 10 across both settings files, now 11.
- Stage 12C is partly installed already: skill-activation-prompt.ts,
  skill-verification-guard.ts, skill-activation-tracker.ts, node_modules,
  all dated 2026-09-12 21:51.
- ALL_STATUS.md (137KB) removed from the mirror. Too big to read and frozen.

## Standing rules added
- Two allowlists exist and must agree: the one inside push_status.sh and the
  one in the mirror's .gitignore. A file must pass both or it vanishes silently.
- Claude cannot fetch a URL until it appears in the chat. Paste the mirror link
  at the start of any session where Claude should read it.
- Never edit the mirror on the GitHub website. It puts the remote ahead of the
  laptop. The script now recovers, but do not rely on it.

## Still open
- 12B assembler: pipeline was running 2026-09-12 ~21:12, outcome never confirmed
- The 1439-word script draft: nobody has read it
- Whether the Stop hook actually fires
- What is inside the mirror's artifacts/ and frames/
- Total provider spend across RunPod, Vast, Modal
