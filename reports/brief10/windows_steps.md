# Brief 10 — Windows steps for Mayo (job 8)

Everything below runs in **Windows PowerShell**, nothing is run by the agent.

## 0. REPAIR THE F: DRIVE FIRST (added 2026-09-27 after the remount attempt)

The remount did NOT fix /mnt/f, because the mount was never the problem.
Windows itself reports the drive is damaged:

    Get-Volume F -> OperationalStatus: Full Repair Needed, HealthStatus: Warning
    a Windows-side write test to F:\ failed: "The file or directory is
    corrupted and unreadable."

That is why every write through WSL failed Errno 22/19 while reads worked.
Repair it in PowerShell **as Administrator** (close any Explorer window
showing F: first; nothing in WSL is holding it):

```powershell
Repair-Volume -DriveLetter F -Scan
Repair-Volume -DriveLetter F -Repair
```

Success looks like: `Repair-Volume` prints `No errors found` or a repair log
ending clean, and `Get-Volume -DriveLetter F` shows `Healthy`. If Repair
reports an unfixable error, the drive is failing — replace it before the
pipeline relies on it again (brief 10 decision 8: no C: fallback).
After the repair, tell the session; it reruns `tools/f_mount.sh --fix` from
the WSL side (now passwordless via the sudoers rule) and continues jobs 4
and 7.
Purpose: reclaim the space the Ubuntu disk (ext4.vhdx) already freed, and let
WSL keep reclaiming it automatically afterwards. Do this when no agent session
is active — `wsl --shutdown` kills every WSL session including this one.

Before size (measured from WSL, 2026-09-27):
`C:\Users\muads\AppData\Local\wsl\{f2ea779f-e5f1-4c82-a1b2-0608e6ab4883}\ext4.vhdx`
= **30,586,961,920 bytes (28.5 GiB)**. Ubuntu's own `df` shows 26G used of the
virtual 1007G, so compaction can only reclaim what THIS brief's cleanup freed
first (caches, logs) plus anything already deleted inside Ubuntu. Expect a
modest one-time shrink (roughly what job 5 freed, GiB for GiB) — sparse mode
is what keeps it shrinking afterwards.

## 1. Turn on sparse VHD (automatic reclaim) — PowerShell, no Admin needed

```powershell
wsl --manage Ubuntu --set-sparse true
```

Syntax checked against this machine's `wsl.exe --help` (2.7.8.0), which lists:
`--manage <Distro> <Options...>` with `--set-sparse, -s <true|false>` —
"Set the VHD of distro to be sparse, allowing disk space to be automatically
reclaimed." Success looks like: the command prints nothing or a short
confirmation and `wsl -l -v` still shows Ubuntu.

## 2. Shut WSL down — PowerShell, no Admin needed

```powershell
wsl --shutdown
```

(The `memory=4957MB` cap this brief wrote into `C:\Users\muads\.wslconfig`
also takes effect here. Success looks like: `wsl -l -v` shows Ubuntu
`Stopped`.)

## 3. One-time compact of the vhdx — PowerShell **as Administrator**

```powershell
wsl --shutdown
diskpart
```

then inside diskpart (the path is the one the coordinator measured):

```
select vdisk file="C:\Users\muads\AppData\Local\wsl\{f2ea779f-e5f1-4c82-a1b2-0608e6ab4883}\ext4.vhdx"
compact vdisk
exit
```

Success looks like: `DiskPart successfully compacted the virtual disk file.`
and the file's size in Explorer drops by roughly what job 5 report as freed
(check with `dir` on the path above).

## 4. Restart and verify — PowerShell, no Admin needed

```powershell
wsl -l -v                 # Ubuntu Starting/Stopped, then start a session
Get-Volume C | Select-Object SizeRemaining
```

What success looks like: C: free space back ABOVE 15 GB (the brief 10 floor)
and the vhdx file smaller than 30,586,961,920 bytes. If sparse mode works,
the file should shrink again on its own in the following days.

## Notes

- The F: fix does NOT need PowerShell: `sudo bash
  ~/yt-digest/soccer-channel/tools/brief10_elevated_setup.sh` in a **WSL2
  terminal** does the whole job (remount helper + narrow sudoers + fstrim).
- `pagefile.sys` (11.01 GB) is Windows-managed and out of this brief's scope;
  the coordinator already measured it. Nothing here touches it.
- `swap=16GB` was kept in .wslconfig per Mayo decision 4 ("keep any existing
  lines"). Be aware the WSL swap file lives on C: — if C: stays tight, that
  line is the next thing to revisit (your call, not changed here).
