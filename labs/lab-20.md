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

### Step 1:  Read the handoff notes: cat /home/admin/incident-notes.txt. Confirm the API is healthy: systemctl status sad-api and curl -s http://127.0.0.1:9090/. List the launcher scripts: ls -l /opt/sad/.

```bash
cat /home/admin/incident-notes.txt
```

<img width="1147" height="600" alt="image" src="https://github.com/user-attachments/assets/4a89aebc-2f89-44e5-b15c-82d18f46f79b" />
