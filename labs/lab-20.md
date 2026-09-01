# Lab 20: "Genova": cgroups problem

## Description
This small VM runs sad-api (a lightweight health endpoint on port 9090) and sad-batch (a nightly ETL-style job that allocates a lot of RAM).

After a recent deploy, starting sad-batch caused memory use to spike and sad-api was killed by the OOM killer. On-call stopped the batch service before handing you the host.

A legacy cgroup v2 launcher under /opt/sad/ is supposed to enforce a 128M hard limit on cgroup sad-batch, but the cap never applies.

sad-batch is intentionally stopped and disabled when you log in. Read /home/admin/incident-notes.txt for context. Fix the cgroup configuration so /sys/fs/cgroup/sad-batch/memory.max is 134217728 before you start the batch job again.

Do not change sad-api; it should keep running on 127.0.0.1:9090.

Test: sad-api is active and curl http://127.0.0.1:9090/ returns SadServers - API OK.

The cgroup v2 hard limit is in place: cat /sys/fs/cgroup/sad-batch/memory.max prints 134217728 (128 MiB).

🔗 **Lab Link:** [SadServers - "Genova": cgroups problem](https://sadservers.com/scenario/genova)

<br>

## 🪜 Steps

### Step 1:  Lab Idea:

We have two services:

- `sad-api`
- `sad-batch`

### `sad-api`

A small service running on:

```
127.0.0.1:9090
```

It must stay up and running at all times, and **must not be modified**.

Test it with:

```bash
curl -s http://127.0.0.1:9090/
```

Expected output:

```
SadServers - API OK
```

### `sad-batch`

A service that runs a Batch/ETL job and consumes a very large amount of RAM.

**The problem that happened before:**

```
sad-batch
    ↓
High RAM usage
    ↓
OOM Killer kicks in
    ↓
Kills sad-api ❌
```

### Goal

The Operations team wants to set a **memory limit** on `sad-batch` only, so that if it exceeds its memory usage, it gets throttled/killed instead of taking down `sad-api` again — i.e., isolate memory consumption between the two services.


## What's Required?

We want `sad-batch` to run inside a cgroup named:

```
sad-batch
```

With a maximum memory limit of:

```
128 MiB
```

The test will check:

```bash
cat /sys/fs/cgroup/sad-batch/memory.max
```

And it should return:

```
134217728
```

Because:

```
128 × 1024 × 1024 = 134217728 bytes
```

### 🧠 What is a cgroup?

Simply put, a **cgroup** (control group) is a set of processes whose resource usage we can control.

For example:

```
cgroup: sad-batch
┌──────────────────────────┐
│                          │
│      sad-batch           │
│                          │
│   Memory limit: 128 MiB  │
│                          │
└──────────────────────────┘
```

Instead of letting `sad-batch` consume all the RAM available on the server, we tell it:
you're allowed a maximum of 128 MiB.


### Step 2:  The lab told as to view the incident notes:

```bash
cat /home/admin/incident-notes.txt
```

<img width="1170" height="567" alt="image" src="https://github.com/user-attachments/assets/e1c62bcd-b979-4fc9-a006-2086d74049d9" />

## Where is this cgroup located?

Linux has a dedicated filesystem for cgroups:

```
/sys/fs/cgroup/
```

The lab tells us there's a cgroup named:

```
sad-batch
```

So:

```bash
ls /sys/fs/cgroup/
```

You should find:

```
sad-batch
```

And inside it, files that control its resources. For example:

```bash
ls /sys/fs/cgroup/sad-batch/
```

You'll find things like:

```
memory.max
memory.high
...
```

### 3. The most important file: `memory.max`

This file tells Linux the maximum amount of RAM this cgroup is allowed to use.

The lab shows you:

```bash
cat /sys/fs/cgroup/sad-batch/memory.max
```

If it returns:

```
max
```

that means there's no hard limit set — which explains why `sad-batch` was able to consume so much RAM that it got `sad-api` killed.

What we want instead is:

```
134217728
```

which is:

```
128 MiB
```

### 4. What about `/opt/sad/`?

The lab notes tell us:

> Memory control is handled by a legacy cgroup v2 launcher under `/opt/sad/`

This means there are scripts under:

```
/opt/sad/
```

responsible for creating/configuring this cgroup. The notes give their names:

```
setup-cgroup.sh
run-batch.sh
```

Roughly:

```
setup-cgroup.sh
       ↓
sets up the cgroup and the memory limit

run-batch.sh
       ↓
runs sad-batch
```

### Step 3: Checking the Status of the Services

```bash
systemctl status sad-api sad-cgroup-setup sad-batch
```

This will help us find out:

- Is `sad-api` running?
- Is `sad-cgroup-setup` running, or did it fail?
- Is `sad-batch` stopped?

After that, we move on to inspecting the scripts themselves.

<img width="1176" height="670" alt="image" src="https://github.com/user-attachments/assets/ee3b151c-e968-41fa-9ee7-f4fa4c0ec433" />
> They are all good.

### Step 4: Inspecting `setup-cgroup.sh`

The incident notes gave us the file paths:

```
/opt/sad/setup-cgroup.sh
/opt/sad/run-batch.sh
```

Now we want to see what `setup-cgroup.sh` actually does. Run:

```bash
sudo cat /opt/sad/setup-cgroup.sh
```

<img width="846" height="228" alt="Screenshot 2026-09-01 095831" src="https://github.com/user-attachments/assets/62b33308-9f96-421c-ba96-2e4aac23d7f7" />

## Found It — The Bug

The important line:

```bash
echo 134217728 > "$CGROUP/memory.high"
```

**This is the problem.** 👀

It writes:

```
134217728
```

to:

```
memory.high
```

instead of:

```
memory.max
```
## But, before editing any thing let as check also `/opt/sad/run-batch.sh`

```bash
sudo cat /opt/sad/run-batch.sh
```

<img width="822" height="206" alt="image" src="https://github.com/user-attachments/assets/23e400b2-e9fa-4ca5-b0d0-fd30eb1f1287" />
>There's no problems here.

## What `run-batch.sh` Does

The `run-batch.sh` script is responsible for running `sad-batch` inside its dedicated cgroup.

1. First, it runs `setup-cgroup.sh` to set up the cgroup and configure the memory limits.
2. Then, the line:

```bash
   echo $$ > /sys/fs/cgroup/sad-batch/cgroup.procs
```

   takes the PID of the current process (`$$`) and writes it into `cgroup.procs`. This tells Linux that this process now belongs to the `sad-batch` cgroup, and will therefore be subject to the resource limits defined in it.

3. Finally:

```bash
   exec /opt/sad/sad-batch.sh
```

   replaces the current shell with the actual batch process, while keeping it inside the same cgroup.


### Step 5: Finally, solving the error

```bash
sudo vim /opt/sad/setup-cgroup.sh
```

Change `echo 134217728 > "$CGROUP/memory.high"` to `echo 134217728 > "$CGROUP/memory.max"`

restart the service:

```bash
sudo systemctl restart sad-cgroup-setup
```

<img width="1171" height="137" alt="image" src="https://github.com/user-attachments/assets/fd0d346d-163f-4416-b027-8ee0dcec1679" />
