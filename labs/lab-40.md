# Lab 40: "Bondo": Split my pile!

## Description
A developer wants to run a program that splits their pile of their data into compressed parts for efficient transport across their network. Unfortunately when the tool runs it never completes.

The application binary in question is called bondo located in /home/admin/bondo.

Run it, then debug and help the developer find the issue.

Test: Executing /home/admin/bondo as admin returns part files generation completed!.

The file /etc/fstab has not been modified and the solution would work on reboot.
🔗 **Lab Link:** [SadServers - "Bondo": Split my pile!](https://sadservers.com/scenario/bondo)

<br>

## 🪜 Steps

## Step 1 — Run the Application

First, execute the application and observe the error:

```bash
# Run the bondo application
./bondo
```

The application reports:

```text
No space left on device
```

At first, this suggests that the disk is full.

---

## Step 2 — Check Disk Space

Check the available disk space:

```bash
# Check available disk space on all mounted filesystems
df -h
```

The filesystem appears to have enough free space.

This means the problem is probably not the amount of available disk storage.

---

## Step 3 — Find Where the Application Is Writing

We need to determine where `bondo` is trying to create its files.

Use system call tracing:

```bash
# Trace failed system calls made by the application
sudo perf trace --failure ./bondo
```

The trace shows that the application is trying to create files under:

```text
/srv/bondo/
```

This gives us an important clue:

```text
bondo
  |
  v
/srv/bondo/
  |
  v
Create part files
  |
  v
No space left on device
```

Since `df -h` showed that there is enough disk space, we need to investigate another filesystem resource.

---

## Step 4 — Check Inodes

A filesystem does not only have a limited amount of disk space. It also has a limited number of **inodes**.

Every file and directory requires an inode.

Check inode usage with:

```bash
# Check inode usage and availability
df -i
```

The filesystem used by `/srv/bondo` has no free inodes.

This means the filesystem is suffering from:

**Inode Exhaustion**

---

## Understanding the Problem

There are two different resources that can become exhausted:

```text
Disk Space
    |
    +-- Checked with: df -h
    |
    +-- Stores file data


Inodes
    |
    +-- Checked with: df -i
    |
    +-- Stores file metadata
```

A filesystem can therefore have:

```text
Disk Space → Enough free space
Inodes     → 100% used
```

In this situation, Linux cannot create new files even though there is still free disk space.

Since `bondo` needs to create multiple part files inside `/srv/bondo/`, it fails with:

```text
No space left on device
```

---

## Root Cause

The filesystem mounted on `/srv/bondo` has exhausted its available inodes.

The problem is **not disk capacity**.

The problem is:

```text
Inode Exhaustion
```

---

## Step 5 — Unmount the Filesystem

Before recreating the filesystem with more inodes, unmount `/srv/bondo`:

```bash
# Unmount the filesystem from /srv/bondo
sudo umount /srv/bondo
```

Verify that it is no longer mounted:

```bash
# Verify that /srv/bondo is no longer mounted
mount | grep /srv/bondo
```

---

## Step 6 — Recreate the Filesystem with More Inodes

The filesystem is located on:

```text
/dev/vg-backup/lv-bondo
```

Recreate the ext4 filesystem with 1024 inodes:

```bash
# Create a new ext4 filesystem with 1024 inodes
sudo mkfs.ext4 -N 1024 /dev/vg-backup/lv-bondo
```

The `-N 1024` option specifies the number of inodes to create.

**Note:** `mkfs.ext4` recreates the filesystem, so this is normally a destructive operation. It is appropriate here because this is the intended solution for the lab.

---

## Step 7 — Mount the Filesystem Again

Mount the filesystem back on `/srv/bondo`:

```bash
# Mount the filesystem using the configuration in /etc/fstab
sudo mount /srv/bondo
```

The command can use only `/srv/bondo` because `/etc/fstab` already contains the information about which device should be mounted there.

The configuration effectively tells Linux:

```text
/dev/vg-backup/lv-bondo
        |
        v
   /srv/bondo
```

Verify the inode availability:

```bash
# Verify the filesystem and available inodes
df -i /srv/bondo
```

---

## Step 8 — Fix Ownership

Because the filesystem was recreated, make sure the `admin` user owns the directory:

```bash
# Give the admin user ownership of the bondo directory
sudo chown admin:admin /srv/bondo
```

This allows the application running as `admin` to create the required files.

---

## Step 9 — Run the Application

Run `bondo` again:

```bash
# Run the bondo application after fixing the filesystem
./bondo
```

Expected result:

```text
part files generation completed!
```

---

## Command Sequence

```bash
# 1. Run the application and observe the error
./bondo

# 2. Check available disk space
df -h

# 3. Trace failed system calls to find where the application is writing
sudo perf trace --failure ./bondo

# 4. Check inode usage
df -i

# 5. Unmount the bondo filesystem
sudo umount /srv/bondo

# 6. Verify that the filesystem is unmounted
mount | grep /srv/bondo

# 7. Recreate the ext4 filesystem with more inodes
sudo mkfs.ext4 -N 1024 /dev/vg-backup/lv-bondo

# 8. Mount the filesystem again using /etc/fstab
sudo mount /srv/bondo

# 9. Verify inode availability
df -i /srv/bondo

# 10. Restore ownership to the admin user
sudo chown admin:admin /srv/bondo

# 11. Run the application again
./bondo
```

---

## Final Architecture

```text
/home/admin/bondo
        |
        | creates part files
        v
   /srv/bondo
        |
        v
/dev/vg-backup/lv-bondo
        |
        +----------------------+
        |                      |
   Disk Space              Inodes
   df -h                    df -i
        |                      |
   Enough space             Exhausted
        |                      |
        +----------+-----------+
                   |
                   v
             No space left
             on device
```

After recreating the filesystem with enough inodes:

```text
bondo
  |
  v
/srv/bondo
  |
  v
Enough inodes
  |
  v
Files can be created
  |
  v
part files generation completed!
```

## Key Lessons

* `df -h` checks **disk space**.
* `df -i` checks **inode usage**.
* `No space left on device` does not always mean the disk is full.
* A filesystem can have free disk space but no available inodes.
* Every file and directory requires an inode.
* `perf trace --failure` can help identify failed system calls and reveal where an application is trying to write.
* `mkfs.ext4 -N` can be used to create an ext4 filesystem with a specified number of inodes.
* `mount /srv/bondo` can use `/etc/fstab` to determine which device belongs at that mount point.
* After recreating a filesystem, ownership and permissions may need to be restored.

```
```
