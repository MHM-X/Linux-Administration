# Lab 45: "Montevideo": restore test snapshot would clobber production

## Description
This host runs a small app whose live data lives under /production/. Nightly snapshot backups land under /snapshots/ in an rsnapshot-style layout.

A recent bug in the snapshot job may have pointed some files in the latest rotation (/snapshots/daily-2026-06-01/) at the live tree instead of making independent copies.

Ops scheduled a dry-run restore from that snapshot into /production/. If any snapshot path still aliases live data, the restore would overwrite production in place.

Read /home/admin/backup-notes.txt. Find every incorrectly shared file between /production/ and /snapshots/daily-2026-06-01/, then repair the snapshot so it is safe to restore from. Do not damage live production data, and do not break intentional space-saving deduplication inside older snapshot rotations.

Note: the tools rsync and dd are available in this server.

Test: Every file in /snapshots/daily-2026-06-01/ mirroring a name in /production/ must be an independent copy: restoring the snapshot must not alias or overwrite live files.
Older rotations (snapshot-1–snapshot-3) must keep shared config.ini hard links.

🔗 **Lab Link:** [SadServers - "Montevideo": restore test snapshot would clobber production](https://sadservers.com/scenario/montevideo)

<br>

## 🪜 Steps

## Understanding the Problem

A hard link means that two different filenames point to the same inode:

```text
/production/users.db
        │
        ▼
     inode 123
        ▲
        │
/snapshots/daily-2026-06-01/users.db
```

In this situation, the snapshot is not an independent backup.

If the snapshot is used for a restore operation, it could potentially overwrite or modify the live production data because both paths refer to the same underlying inode.

The affected files were:

```text
users.db
inventory.db
```

---

## Solution

The solution is to create a completely new file from each affected snapshot file and then replace the old snapshot entry with the new file.

### Fix `users.db`

```bash
# Create an independent copy with a new inode
cp -p /snapshots/daily-2026-06-01/users.db /tmp/users.db

# Replace the old hard-linked snapshot entry with the new independent file
mv /tmp/users.db /snapshots/daily-2026-06-01/users.db
```

### Fix `inventory.db`

```bash
# Create an independent copy with a new inode
cp -p /snapshots/daily-2026-06-01/inventory.db /tmp/inventory.db

# Replace the old hard-linked snapshot entry with the new independent file
mv /tmp/inventory.db /snapshots/daily-2026-06-01/inventory.db
```

---

## Why `cp` + `mv` Works

Initially:

```text
production/users.db ──────┐
                          ├── inode 123
snapshot/users.db ────────┘
```

After `cp`:

```text
production/users.db ──────→ inode 123
snapshot/users.db ────────→ inode 123

/tmp/users.db ────────────→ inode 456
```

The original snapshot file still points to the production inode at this point.

After `mv`:

```text
production/users.db ──────→ inode 123

snapshot/users.db ────────→ inode 456
```

Now the production file and snapshot file are completely independent.

The production inode is not modified or replaced.

---

## Why Not Simply Copy the File Directly?

The important objective is not only to copy the data.

We need the snapshot file to have a **different inode** from the production file.

Using:

```bash
cp -p source /tmp/file
```

creates a new filesystem object with a new inode.

Then:

```bash
mv /tmp/file destination
```

makes the snapshot path point to that new independent file.

---

## Important Note

Do **not** remove all hard links from the snapshot directories.

Some hard links between older snapshot rotations are intentional and are used for deduplication to save disk space.

Only the incorrect links between:

```text
/production/
/snapshots/daily-2026-06-01/
```

needed to be fixed.

---

## Key Concepts Learned

* **Inode**: The filesystem structure that represents a file and stores its metadata.
* **Hard link**: Multiple filenames pointing to the same inode.
* **Independent copy**: A separate file with its own inode.
* `cp -p`: Creates a new independent copy while preserving file metadata where possible.
* `mv`: Moves/renames the new file and replaces the old snapshot directory entry.
* Hard links can be useful for deduplication, but they are dangerous when used accidentally between production data and backups.
* A backup must have independent storage from the live data if it is expected to safely restore production.
