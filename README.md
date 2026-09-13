<!-- push: 2026-09-13T06:40:29Z changed -->
<!-- push: 2026-09-13T06:24:51Z unchanged -->
<!-- push: 2026-09-13T06:21:37Z changed -->
<!-- push: 2026-09-13T06:11:49Z changed -->
<!-- push: 2026-09-13T06:06:23Z unchanged -->
<!-- push: 2026-09-13T05:36:48Z changed -->
<!-- push: 2026-09-13T04:53:29Z changed -->
<!-- push: 2026-09-13T03:48:58Z changed -->
<!-- push: 2026-09-13T03:42:03Z changed -->
<!-- push: 2026-09-13T03:39:56Z unchanged -->
<!-- push: 2026-09-13T03:39:05Z changed -->
<!-- push: 2026-09-13T03:37:12Z changed -->
<!-- push: 2026-09-13T03:33:56Z changed -->
<!-- push: 2026-09-13T03:31:46Z unchanged -->
<!-- push: 2026-09-13T03:25:48Z changed -->
# soccer-channel-status

Public mirror of the soccer-channel project state: docs + small
machine-readable artifacts + downscaled verification frames.
Readable without authentication via the raw host. Do NOT use the
github.com/.../blob/... HTML view, it is not reliably fetchable by
automated readers.

## One-fetch full state

ALL_STATUS.md concatenates every status doc below into one file:

```
https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/ALL_STATUS.md
```

## Individual docs (raw URLs)

- [CONTEXT.md](https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/CONTEXT.md)
- [STATUS.md](https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/STATUS.md)
- [PROGRESS.md](https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/PROGRESS.md)
- [GAPS.md](https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/GAPS.md)
- [DECISIONS.md](https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/DECISIONS.md)
- [TOOLS.md](https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/TOOLS.md)
- [ARCHITECTURE.md](https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/ARCHITECTURE.md)

## Machine-readable artifacts (artifacts/)

Small JSON outputs Claude can recompute and check arithmetic from:
- scoreboard/ — raw scoreline per sampled frame + detected changes
- gemini_inventory/ — Gemini footage inventory JSON
- publish-log/ — YouTube upload log entries
- tracking_summary/ — trimmed per-tracker tracking summaries
Files live at https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/artifacts/<type>/<file>

## Verification frames (frames/)

Downscaled 640px-wide images behind visual claims, named by
timestamp (e.g. frame_0253.png). Files at https://raw.githubusercontent.com/minakush000-crypto/soccer-channel-status/main/frames/<...>
