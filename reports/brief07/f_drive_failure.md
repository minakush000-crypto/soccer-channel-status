# Brief 07 job 1 BLOCKED: F: drive drops under sustained write

Date: 2026-09-27 21:30-21:40 CDT. Produced by: Flash (glm-5.3-flash:cloud).

## What happened

Job 1 (BACKUP to /mnt/f/backups/) failed twice on hardware. The Memorex USB
flash drive (serial 0A7710C44, model "USB Flash Drive", revision PMAP)
physically disconnected from Windows mid-write, twice, under sustained write
load. Windows re-enumerated it when idle. This is NOT the NTFS corruption
from earlier today: `Get-Volume F` reported `Healthy / OK` both before and
after (that corruption was repaired by chkdsk earlier on 2026-09-27, gate
B29 evidence).

## Evidence trail (all commands run in WSL2, outputs verbatim)

1. `bash tools/disk_guard.sh --report` → `DISK-GUARD-PASS ... avail=19G >= 15G`,
   `DISK-GUARD-PASS f_paths mounted+writable at /mnt/f (4 configured paths)`.
2. `bash tools/f_mount.sh --check` → `F-MOUNT-OK` (small write probe passes).
3. `tar czf /mnt/f/backups/claude-backup-2026-09-27.tar.gz -C ~ .claude .claude.json`
   → exit=2 after 47 s: `tar: /mnt/f/backups/...: Wrote only 4096 of 10240
   bytes` / `Error is not recoverable`. Partial tarball reached
   146,735,032 bytes (~147 MB of ~1.7 GB, ~9%).
4. Immediately after: `ls /mnt/f/backups/` → `Os { code: 19, ... "No such
   device" }`. `bash tools/f_mount.sh --check` → `F-MOUNT-FAIL mounted=no
   writable=no`.
5. `sudo -n /usr/local/bin/f-remount` (fresh drvfs mount) → `F-MOUNT-OK`.
6. `dd if=/dev/zero of=/mnt/f/backups/.wprobe bs=1M count=500` → `IO error:
   Input/output error`. Channel died again (Errno 19 everywhere).
7. Remount again, write ladder: 100 MB dd SUCCEEDED but took 65.2 s
   (1.6 MB/s; a healthy USB stick does 20-40 MB/s). md5sum readback of the
   just-written file returned NOTHING. Next access (300 MB) → `Invalid
   input` / Errno 19. Channel dead again.
8. Windows-side test (`powershell.exe`, `[IO.File]::Create('F:\backups\.winprobe')`)
   → `A device which does not exist was specified.` Then
   `Get-Volume -DriveLetter F` → `No MSFT_Volume objects found with property
   'DriveLetter' equal to 'F'`. Windows itself lost the drive.
9. Windows Event Log, System, 9:35:28 PM (matching the 300 MB attempt):
   NTFS Id 140, `The system failed to flush data to the transaction log.
   Corruption may occur in VolumeId: F:` `Failure status: A device which
   does not exist was specified.` Device manufacturer: **Memorex**, model
   **USB Flash Drive**, Bus type: **USB**, serial 0A7710C44.
10. 8 s later, no load: `Get-Volume F` → `Healthy` again. The drive flaps:
    drops under sustained write, recovers when idle.
11. Fresh remount → `LIST-OK`, `F-MOUNT-OK`. Partial files cleaned.

## Conclusion

The stick disconnects under sustained write and has degraded within the last
hour (brief 10 pushed larger writes to F: successfully earlier today; the
same drive now does 1.6 MB/s with readback failures). Two disconnects under
load in one hour. The rollback tarball is the safety artifact for the ECC
install; writing it to a drive that drops mid-write risks a silently corrupt
backup, which is worse than no backup.

## Decision (per doctrine 8 and brief 07 job 1: "F: is mandatory, never C:")

- Job 1 is BLOCKED at hardware level. No fallback to C: or anywhere else.
- Gates B32 (tarball on F:) and B33 (B2 copy) are ABANDONed with reason;
  they rerun as-is once the drive is healthy.
- Jobs 2-4 continue (clone, dry run, analysis, summary): they do not touch
  F: and do not modify ~/.claude, so the missing backup does not increase
  their risk.
- **No install (jobs 5-8) until job 1 completes on healthy hardware and
  B32/B33 are MET.** Mayo's APPROVED ECC alone is not enough; the backup
  must land first.

## What Mayo needs to do

1. Re-plug the Memorex stick (or a different port/cable). If Windows
   re-detects it, tell me and I rerun job 1 (guard + mount check, tar, B2).
2. If it drops again under load, the stick is dying. Replace it. The
   rollback backup and `soccer-channel-keys` (which has no other local copy)
   both live on F:. Say the word and I archive `soccer-channel-keys` to B2
   as soon as the drive is readable, so the keys outlive the stick.

## Cost

~12 minutes, 0 local files kept (partials deleted), 2 drive drop cycles
observed.