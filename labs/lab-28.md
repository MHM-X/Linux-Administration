# Lab 28: "Kihei": Surely Not Another Disk Space Scenario

## Description
There is a /home/admin/kihei program. Make the changes necessary so it runs succesfully, without deleting the /home/admin/datafile file.

Test: Running /home/admin/kihei returns Done..

🔗 **Lab Link:** [SadServers - "Kihei": Surely Not Another Disk Space Scenario](https://sadservers.com/scenario/kihei)

<br>

## 🪜 Steps

### The Requirement

There's a program at `/home/admin/kihei`. Make it succeed, **without deleting** `/home/admin/datafile`.

On success:

```bash
/home/admin/kihei
```

should print:

```
Done.
```

### 1. First thing: just run it

As a Linux admin, when someone hands you a program and says "it doesn't work," the first move isn't to read a solution — it's to run it:

```bash
/home/admin/kihei
```

Output:

```
panic: exit status 1

goroutine 1 [running]:
main.main()
        ./main.go:64 +0x47d
```

It doesn't matter that you don't know Go. All we need to know is:

```
kihei
  ↓
tries to do something
  ↓
that operation fails
  ↓
returns exit status 1
  ↓
the Go program panics
  ↓
kihei stops
```

The panic isn't the diagnosis — it's just telling us "something I was doing failed."

### 2. Identify what `kihei` is

```bash
file /home/admin/kihei
```

Output:

```
ELF 64-bit LSB executable, x86-64
statically linked
Go BuildID=...
not stripped
```

So: a Linux executable, built for x86-64, written/built with Go — not a shell script. Importantly, **we don't need to know Go to fix this.** As Linux admins, we want to know: what is this program trying to do on the system?

### 3. Use `strace`

This is the key move in the debugging process.

```bash
strace /home/admin/kihei
```

Why `strace`? Because the program's own output — `panic: exit status 1` — is vague. `strace` shows us the program's interactions with the Linux kernel: opening files, writing files, creating processes, reading files, mounting filesystems, etc.

The first clue is that running:

```bash
strace ./kihei
```

or:

```bash
./kihei -v
```

reveals that the program is trying to create:

```
/home/admin/data/newdatafile
```

at a size of:

```
1,500,000,000 bytes
```

— roughly **1.5 GB**. Now we know the real cause:

```
kihei
  ↓
tries to create newdatafile
  ↓
required size = 1.5 GB
  ↓
not enough space
  ↓
operation fails
  ↓
exit status 1
  ↓
panic
```

### 4. But why isn't there enough space?

Back to the filesystem:

```bash
df -h /home/admin/datafile
```

Output:

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p1  7.7G  6.7G  576M  93% /
```

So `/home` lives on `/dev/nvme0n1p1`, which has 7.7G total, 6.7G used, and 576M available. The program wants 1.5 GB, but only 576 MB is free:

```
Required: 1.5 GB
Available: 576 MB

1.5 GB > 576 MB
```

Confirmed: this is a disk space problem.

### 5. An important constraint

The lab says: **without deleting `/home/admin/datafile`**.

This matters, because the "obvious" lazy fix would've been:

```bash
rm /home/admin/datafile
```

freeing up 5 GB (we confirmed `/home/admin/datafile` = 5.0G) — but that's not allowed. The goal isn't to delete data; it's to **add new storage** for the directory the program needs.

### 6. Look for additional disks

```bash
lsblk -l
```

`lsblk` = list block devices — show the disks, partitions, and block devices on the machine.

Found:

```
/dev/nvme1n1   1G
/dev/nvme2n1   1G
```

Two 1 GB disks, totaling 2 GB — enough for the program's 1.5 GB requirement. But they're two separate disks, and we want something close to one 2 GB space. That's why the clue says: *create a logical volume that spans both disks.* Time for LVM.

### 7. What is LVM?

**LVM = Logical Volume Manager.** Think of it as a pooling layer between physical disks and the filesystem. Instead of:

```
Disk 1 → filesystem
Disk 2 → filesystem
```

we can do:

```
/dev/nvme1n1 ─┐
              ├── Volume Group ── Logical Volume
/dev/nvme2n1 ─┘
```

and then put a filesystem on top of the Logical Volume:

```
Physical Disks

/dev/nvme1n1  1GB ──┐
                    ├── VG: vg ── LV: lv ≈ 2GB
/dev/nvme2n1  1GB ──┘
                              │
                              ▼
                           ext4
                              │
                              ▼
                     /home/admin/data
```

This is exactly why the lab gives you two 1 GB disks instead of a single 2 GB one — it wants you to practice LVM.

### 8. Step one: `pvcreate`

As root:

```bash
sudo pvcreate /dev/nvme1n1 /dev/nvme2n1
```

This converts the disks into Physical Volumes (PVs) that LVM understands. Before, they were just block devices; after, LVM can use them as PVs.

### 9. Group them into a Volume Group

```bash
sudo vgcreate vg /dev/nvme1n1 /dev/nvme2n1
```

This creates a Volume Group named `vg`, pooling both PVs into roughly 2 GB of usable space.

### 10. Take all the space and create a Logical Volume

```bash
sudo lvcreate -n lv -l 100%FREE vg
```

Breaking it down:

- `lvcreate` — create a Logical Volume
- `-n lv` — name it `lv`
- `-l 100%FREE` — use 100% of the free space in the Volume Group (basically all 2 GB)
- `vg` — from the Volume Group named `vg`

Result: `vg` now contains `lv`, exposed as the device `/dev/vg/lv`.

### 11. A Logical Volume alone isn't enough

This is an important point. After creating `/dev/vg/lv`, there's still no filesystem on it — it's essentially a brand-new disk. Linux needs a filesystem (ext4, xfs, etc.) before it can store files on it:

```bash
sudo mkfs.ext4 /dev/vg/lv
```

`mkfs` = make filesystem; `ext4` is the filesystem type. Now:

```
/dev/nvme1n1 ─┐
              ├── VG vg
/dev/nvme2n1 ─┘
                 │
                 └── LV lv
                       │
                       └── ext4
```

is ready to be mounted.

### 12. Mount it

The program tries to create `/home/admin/data/newdatafile` — notice the `/home/admin/data/` directory. That's exactly where we need to attach the new space:

```bash
sudo mount /dev/vg/lv /home/admin/data
```

Now, when the program writes to `/home/admin/data/newdatafile`, the data goes to the new filesystem, not the old, nearly-full root filesystem.

### 13. But there's a permissions problem

The program runs as `admin`, but a freshly mounted filesystem is typically owned by `root`. If `admin` tries to create a file there, it may get `Permission denied`. Fix the ownership:

```bash
sudo chown -R admin: /home/admin/data
```

Now `/home/admin/data` is owned by `admin`, and the program can write into it.

### 14. Run the program

At this point:

```
Old root filesystem
/dev/nvme0n1p1
        │
        ├── /home/admin/datafile
        └── rest of the system

New storage
/dev/nvme1n1 ─┐
              ├── LVM → /dev/vg/lv → ext4
/dev/nvme2n1 ─┘
                         │
                         ▼
                  /home/admin/data
                         │
                         ▼
                    newdatafile
```

```
/home/admin/kihei
       │
       ▼
/home/admin/data/newdatafile
       │
       ▼
NEW 2GB filesystem
       │
       └── has ≥ 1.5GB available
```

Running `~/kihei` now succeeds and prints:

```
Done.
```

### 15. Why can we run it more than once?

The last clue says the program deletes `newdatafile` if it already exists, before recreating it — so each run is roughly:

```
kihei
 ↓
if newdatafile exists
 ↓
delete it
 ↓
recreate it at 1.5GB
 ↓
Done
```

So the fix can be tested repeatedly.

### Summary — the Linux Troubleshooting Chain

This is the logic worth remembering, not the commands:

```
1. Run the program
        ↓
2. Program fails
        ↓
3. strace → discover what it is trying to do
        ↓
4. It wants to create a 1.5GB file
        ↓
5. df → only 576MB free on /
        ↓
6. Disk space is insufficient
        ↓
7. Cannot delete datafile (lab constraint)
        ↓
8. lsblk → discover two unused 1GB disks
        ↓
9. Combine disks with LVM
        ↓
10. Create filesystem
        ↓
11. Mount it on /home/admin/data
        ↓
12. Give admin ownership
        ↓
13. Run kihei
        ↓
14. Done.
```

### Final Commands, in Order

```bash
sudo pvcreate /dev/nvme1n1 /dev/nvme2n1
sudo vgcreate vg /dev/nvme1n1 /dev/nvme2n1
sudo lvcreate -n lv -l 100%FREE vg
sudo mkfs.ext4 /dev/vg/lv
sudo mount /dev/vg/lv /home/admin/data
sudo chown -R admin: /home/admin/data
~/kihei
```

**The real lesson here isn't `pvcreate` or `lvcreate`.** It's the reasoning chain: from "a Go program gave me a panic," to "the program is trying to write 1.5GB," to "the filesystem only has 576MB," to "I have two spare disks," to "use LVM to combine them and put the new filesystem exactly where the program writes." That chain of reasoning is what Linux/system-administrator troubleshooting actually looks like.
