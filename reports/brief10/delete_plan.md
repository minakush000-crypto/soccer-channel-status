# Brief 10 — delete plan (job 5)

Rule (Mayo decision 3): anything already archived to B2 (verified by rclone
listing/size match, not memory) AND not used by the current episode is
deleted automatically; everything else goes on the MAYO-YES list below, not
deleted. "Current episode" = renders/2026-09-08_real-madrid-inter (EP001,
the flash-lane episode of record). Sizes are `du -sh` / `stat` measurements
from 2026-09-27. Git-tracked files under renders/ (board_specs, match_data,
sofascore_raw JSONs — the inputs of record) are NEVER deleted; only untracked
media goes.

## AUTO (archive to B2 first if missing, verify by size, then delete)

| path | size | B2 archive path | used by current episode? | action |
|---|---|---|---|---|
| renders/2026-09-12_bournemouth-brentford-gemini/ (final_video.mp4 123,744,975 B matches B2 exactly) | 436M | already archived: b2:.../2026-09-16/2026-09-16_claude_final_video.mp4 (same bytes; clips are inputs of a retired Gemini lane, re-derivable) | no | DONE 2026-09-27: full media also tarred to b2:.../2026-09-27/2026-09-27_glm_b10_bournemouth.tar.gz (681,109,391 B, size-verified), untracked files deleted, tracked match_data.json kept (dir now 20K) |
| renders/2026-09-12_bournemouth-brentford/ (final 77.7MB + clips 66M + boards 1.9M) | 222M | NOT archived yet -> archive as b09-era tar first | no | DONE 2026-09-27: same B2 tarball as the gemini row (size-verified), untracked media deleted, board_specs+match_data+sofascore_raw kept (dir now 68K) |
| ~/.cache/uv | 543M | regenerable package cache (no archive: it is a cache) | no | DONE 2026-09-27 23:13Z: deleted (F: repair pending, so no redirect to wait for — regenerates wherever the env file points once job 4 lands) |
| ~/.npm | 381M | regenerable npm cache | no | DONE 2026-09-27 23:13Z: deleted |
| ~/.cache/node-gyp | 65M | regenerable build cache | no | DONE 2026-09-27 23:13Z: deleted |
| /var/log journal + /var/cache/apt | ~700M of the 828M+254M | logs and package indexes, regenerable | no | DONE 2026-09-27 23:12Z: f-clean via the passwordless sudoers rule freed 728.5M of journals; /var/log now 100M |

MEASURED FREED so far: df `/` used went 26G -> 24G (2 GiB inside the vhdx:
~657M render media + ~989M caches + ~729M journals). fstrim -av reported
980.4 GiB trimmed on /dev/sdd, so the Windows compact (step 3 below) can
reclaim it. NOTE: C: free fell 12G -> 7.4G during the same window WITHOUT
the vhdx changing (30,586,961,920 bytes before and after; swap.vhdx 36MB) -
the growth is Windows-side activity outside this disk (the Claude Desktop
Cowork VM is a known 10.8GB resident there and out of scope per the brief).

## MAYO-YES list (listed, NOT deleted)

| path | size | note |
|---|---|---|
| ~/cv-test | 2.7G | .venv 2.3G + data 395M; status unknown to this pipeline — your call (a stale experiment venv is the likeliest 2.3G win) |
| ~/.rustup | 1.4G | user rust toolchain; nothing in this pipeline builds Rust (system uutils rust lives in /usr/lib/cargo, separate) |
| ~/mcp-servers | 1.3G | MCP server sources; check which are still configured before deleting |
| ~/.claude/plugins | 1.3G | Claude Code plugins IN USE by this harness (claude-mem etc.) — keep unless you want a re-download |
| ~/.vscode-server | 1.8G | VS Code remote server — keep if you use VS Code with WSL |
| ~/.claude-mem | 523M | the memory worker's DB (claude-mem.db 114M + chroma 357M) — IN USE |
| ~/.claude/projects | 407M | past session transcripts (harness data) |
| ~/.cache/chroma | 167M | looks like a stale copy of claude-mem's chroma (the live one is ~/.claude-mem/chroma) — confirm, then it becomes AUTO |
| ~/.claude-mem/logs | 48M | worker logs |
| ~/.gemini | 70M | Gemini is retired; its CLI state |
| ~/.linkedin-mcp | 37M | LinkedIn MCP state |
| ~/quant-solana-gate2 | 517M | another project — outside this pipeline's authority |
| ~/skills-lab | 474M | another project — outside this pipeline's authority |
| ~/.venv (home root) | 313M | a second venv at $HOME — check what uses it |
| ~/.nvm ~/.bun ~/.cargo | ~1.0G | node/bun/rust version managers — in use by the harness and tools |

## KEEP (not candidates)

| path | why |
|---|---|
| renders/2026-09-08_real-madrid-inter/ (315M) | the current episode: v7 final (75.6MB, ARCHIVED to B2 2026-09-27_glm_final_video.mp4, size-verified 75581383), boards, frames, all specs tracked |
| ~/yt-digest (3.8G incl .venv 2.4G) | the pipeline itself |
| ~/soccer-channel-status (30M) | the public mirror |
| ~/.claude except plugins/projects | harness config, skills, hooks |