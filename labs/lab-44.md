# Lab 44: "Cordoba": df is lying (or is it du?)

## Description
 Monitoring reports that the root filesystem is under pressure, but a quick du of /var/log shows almost nothing in the logs of the running application at /var/log/cordoba-app.

Find what is holding the space and reclaim it so df and du agree again for practical purposes; currently there's a ~300 MB discrepancy on the root partition /

The service unit is cordoba-hoarder.service.

Test: df -h / and sudo du -sh / report the same used space after reclaiming the ~300 MB discrepancy.

🔗 **Lab Link:** [SadServers - "Cordoba": df is lying (or is it du?)](https://sadservers.com/scenario/cordoba)

<br>

## 🪜 Steps

````markdown
## Problem

The root filesystem `/` is under disk pressure.

When checking the disk usage:

```bash
df -h /
````

it reports around **5.1 GB used**, while:

```bash
sudo du -sh /
```

reports only around **4.7 GB**.

There is therefore a discrepancy of roughly **300–400 MB**.

The application logs under `/var/log/cordoba-app` appear almost empty, so the missing space is not explained by normal visible files.

---

## Root Cause

The service `cordoba-hoarder.service` was holding a deleted log file open.

A file can be deleted with `rm` while a running process still has it open. In that case:

* The file disappears from the directory tree.
* `du` cannot see or count it.
* The process still holds a file descriptor to it.
* The filesystem blocks remain allocated.
* `df` continues to count the used space.

This creates the discrepancy:

```text
df  → sees the allocated blocks
du  → cannot see the deleted file
```

---

## Investigation

Check for deleted files that are still open:

```bash
# Find deleted files that are still held open by running processes
sudo lsof +L1
```

The output shows the `cordoba-hoarder` process holding a deleted `access.log`.

Find the service's main PID:

```bash
# Get the main PID of the hoarder service
pid=$(systemctl show -p MainPID --value cordoba-hoarder)
```

Inspect its file descriptors:

```bash
# List the file descriptors opened by the process
sudo ls -l /proc/$pid/fd
```

File descriptor `9` points to:

```text
/var/log/cordoba-app/access.log (deleted)
```

---

## Solution

The simplest solution is to stop the service so the process closes the file descriptor:

```bash
# Stop the service holding the deleted file open
sudo systemctl stop cordoba-hoarder
```

Once the last file descriptor is closed, the kernel releases the disk blocks occupied by the deleted file.

Verify the result:

```bash
# Check filesystem usage
df -h /

# Check directory usage
sudo du -sh /

# Check for remaining deleted-but-open files
sudo lsof +L1
```

The `df` and `du` values should now be approximately the same.

---

## Alternative Solution

If the service must remain running, the deleted file can be truncated through its open file descriptor:

```bash
# Get the service's main PID
pid=$(systemctl show -p MainPID --value cordoba-hoarder)

# Truncate file descriptor 9 to zero bytes
sudo truncate -s 0 /proc/$pid/fd/9
```

This releases the disk space without stopping the process.

---

## Key Lessons

* `df` reports filesystem-level disk usage.
* `du` reports disk usage of files accessible through the directory tree.
* Deleted files can continue consuming disk space if a process still has them open.
* `lsof +L1` is useful for finding deleted-but-open files.
* `/proc/<PID>/fd/` exposes the file descriptors of a running process.
* Stopping the process closes the file descriptor and allows the kernel to reclaim the space.
* A large `df` vs `du` discrepancy is a strong indication to check for deleted files still held open by processes.

## Final Fix

```bash
# Stop the process holding the deleted file
sudo systemctl stop cordoba-hoarder

# Verify that disk usage is consistent
df -h /
sudo du -sh /

# Verify that no deleted files are still held open
sudo lsof +L1
```
