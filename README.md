<!-- push: 2026-09-16T03:32:56Z changed -->
<!-- push: 2026-09-16T02:41:51Z changed -->
<!-- push: 2026-09-16T02:36:03Z changed -->
<!-- push: 2026-09-16T01:49:05Z changed -->
<!-- push: 2026-09-16T01:22:24Z unchanged -->
<!-- push: 2026-09-16T01:22:01Z changed -->
<!-- push: 2026-09-16T00:50:49Z changed -->
<!-- push: 2026-09-16T00:40:53Z unchanged -->
<!-- push: 2026-09-16T00:38:49Z unchanged -->
<!-- push: 2026-09-15T23:01:47Z unchanged -->
<!-- push: 2026-09-14T06:42:55Z unchanged -->
<!-- push: 2026-09-14T06:42:09Z changed -->
<!-- push: 2026-09-14T05:47:09Z unchanged -->
<!-- push: 2026-09-14T05:46:47Z changed -->
<!-- push: 2026-09-14T05:25:00Z unchanged -->
<!-- push: 2026-09-14T05:24:38Z changed -->
<!-- push: 2026-09-14T05:22:04Z unchanged -->
<!-- push: 2026-09-14T05:12:58Z unchanged -->
<!-- push: 2026-09-14T05:12:35Z changed -->
<!-- push: 2026-09-14T05:06:53Z changed -->
<!-- push: 2026-09-14T04:48:53Z changed -->
<!-- push: 2026-09-14T04:47:05Z unchanged -->
<!-- push: 2026-09-14T04:45:39Z changed -->
<!-- push: 2026-09-14T04:43:09Z changed -->
<!-- push: 2026-09-14T04:20:39Z changed -->
<!-- push: 2026-09-14T03:18:39Z changed -->
<!-- push: 2026-09-14T02:39:44Z unchanged -->
<!-- push: 2026-09-14T02:29:20Z unchanged -->
<!-- push: 2026-09-14T02:15:35Z unchanged -->
<!-- push: 2026-09-14T02:10:21Z changed -->
<!-- push: 2026-09-14T02:02:23Z changed -->
<!-- push: 2026-09-14T01:54:04Z changed -->
<!-- push: 2026-09-14T01:44:04Z changed -->
<!-- push: 2026-09-14T01:34:07Z changed -->
<!-- push: 2026-09-14T01:20:15Z changed -->
<!-- push: 2026-09-13T07:38:51Z unchanged -->
<!-- push: 2026-09-13T07:31:06Z unchanged -->
<!-- push: 2026-09-13T07:17:05Z unchanged -->
<!-- push: 2026-09-13T07:16:52Z unchanged -->
<!-- push: 2026-09-13T07:14:47Z unchanged -->
<!-- push: 2026-09-13T07:13:40Z unchanged -->
<!-- push: 2026-09-13T07:13:22Z unchanged -->
<!-- push: 2026-09-13T07:10:05Z unchanged -->
<!-- push: 2026-09-13T07:04:54Z unchanged -->
<!-- push: 2026-09-13T07:04:19Z unchanged -->
<!-- push: 2026-09-13T06:58:45Z unchanged -->
<!-- push: 2026-09-13T06:55:46Z unchanged -->
<!-- push: 2026-09-13T06:54:31Z unchanged -->
<!-- push: 2026-09-13T06:54:16Z changed -->
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
