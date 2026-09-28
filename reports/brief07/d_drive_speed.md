# Brief 07 continuation job 3: D: speed test (one-time measurement)

Date: 2026-09-27 ~22:05-22:10 CDT. Produced by: Flash (glm-5.3-flash:cloud).

## Why Windows-side, not /mnt/d

Mayo asked for /mnt/d/speedtest.bin. /mnt/d does not exist in WSL (mounted
drives: c, e, f), but Windows HAS a D: drive (FAT32, Healthy, 29.8 GB, empty,
Disk 1 "Multiple Card Reader"). Mounting it in WSL needs `sudo mount`, and
passwordless sudo covers only the f-* helpers (checked: `sudo -n -l`), so
adding a rule was not done without Mayo. The test ran Windows-side on the
SAME physical drive (PowerShell, D:\speedtest.bin). A WSL-side rerun needs
Mayo to type: `! sudo mount -t drvfs D: /mnt/d -o rw,uid=1000,gid=1000,umask=22`
(drvfs adds overhead, expect somewhat lower numbers).

## Critical hardware fact

Windows partition map (Get-Partition, run in this job):
- C: = Disk 0 (internal SAMSUNG SSD, 118 GB)
- **F: = Disk 2 and G: = Disk 2** (both on the Memorex stick; G: is a 0-byte
  leftover FAT partition on it)
- **D: = Disk 1** (a different physical device, the "Multiple Card Reader")

So D: is NOT the flapping stick. It is a separate device and a valid
replacement candidate.

## Measurements (500 MB, written then read back, then deleted)

- WRITE: 500 MB in 40.7 s = **12.3 MB/s** (PowerShell FileStream, 50x10MB
  writes, Flush(true) at end)
- READ+MD5: 500 MB in 1.7 s = **288.7 MB/s** (CAVEAT: the file was just
  written, so this read ran largely from Windows file cache; a cold read
  would be slower. md5=3DAA474F (prefix))
- File deleted: True. D: free after: 29.8 GB, HealthStatus Healthy (no
  disconnect during or after the test, unlike F: under load)

## Comparison against the same day's F: numbers

| | F: (Memorex, flapping) | D: (Disk 1, card reader) |
|---|---|---|
| 100 MB write | 65.2 s = 1.6 MB/s (degraded) | n/a |
| 500 MB write | I/O error, drive dropped | 40.7 s = 12.3 MB/s |
| readback of just-written file | FAILED (no hash) | OK, md5 matched write |
| drive survived test | NO (disconnected twice) | YES (Healthy after) |

Verdict: D: sustained-write is 7.6x the degraded F: number and completed a
500 MB write+readback+delete without a single drop. It is usable as the
staging/replacement candidate while the Memorex stick stays suspect. Nothing
was changed on D: beyond creating and deleting speedtest.bin.

Note: measured while the 592 MB B2 upload ran concurrently (network + WSL
disk only; D: unaffected).