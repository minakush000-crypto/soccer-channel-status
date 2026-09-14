## STANDING ORDER — briefs and reports are files, not messages

Effective immediately, for every task from any agent.

### Every brief is saved before work starts

When Mayo pastes a brief, write it verbatim to:

```
~/claude/30-briefs/YYYY-MM-DD-HHMM-<short-slug>.md
```

Unedited. Do not summarise, do not improve it. Then push.

### Every report is saved when work ends

Write the full report, with the pasted command output, to:

```
~/claude/20-handoffs/YYYY-MM-DD-HHMM-<same-slug>-report.md
```

The report names the brief file it answers, in its first line. Then push.

### Push after both

Run `.claude/hooks/push_status.sh` explicitly at the end of every task rather than relying on the Stop hook, which only fires on a clean session end and produced a three-day silent gap once already. Report the push line it printed.

### Why

The coordinator can read the public mirror directly. When brief and report are both files, the brief can be checked against the report by reading, rather than by trusting a pasted summary. Nothing is verified today: every report this session reached the coordinator as text, with no artifact behind it.

### Confirm the path works

`~/claude/30-briefs` and `~/claude/20-handoffs` are both in `push_status.sh`'s vault list. Confirm both appear under `vault/30-briefs/` and `vault/20-handoffs/` in the mirror after the first push, and paste the `git ls-files` output showing them.

### Also save the decision record

Any decision Mayo makes mid-task (an answer to a question, a reversal, a ruling) goes into DECISIONS.md with the date, in his words, before the work continues. A decision that lives only in a chat message is lost when the chat ends.