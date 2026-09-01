# Lab 23: "Manhattan": can't write data into database.

## Description
 Your objective is to be able to insert a row in an existing Postgres database. The issue is not specific to Postgres and you don't need to know details about it (although it may help).

Helpful Postgres information: it's a service that listens to a port (:5432) and writes to disk in a data directory, the location of which is defined in the data_directory parameter of the configuration file /etc/postgresql/14/main/postgresql.conf. In our case Postgres is managed by systemd as a unit with name postgresql.

Test: (from default admin user) sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt

🔗 **Lab Link:** [SadServers - "Manhattan": can't write data into database.](https://sadservers.com/scenario/manhattan)

<br>

## 🪜 Steps

### 1. What does the lab require?

The lab asks you to be able to insert a row into PostgreSQL.

The test is:

```bash
sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt
```

If everything is correct, you'll see:

```
INSERT 0 1
```

Meaning: one person was successfully added to the database.

### 2. What's the problem?

The core problem is that the **PostgreSQL server itself can't run**. So when we try to use `psql` to connect to the database, we can't — because the server itself is down.

The initial picture:

```
PostgreSQL Server
       ❌
       ↓
Database "dt"
       ❌
       ↓
INSERT
       ❌
```

Important: the problem is **not** in the SQL or in the `persons` table. The problem is that PostgreSQL itself isn't running.

### 3. First step: check PostgreSQL

The clue gives us:

```bash
sudo systemctl status postgresql
```

This asks systemd: what's PostgreSQL's status?

In our case, we found that the actual cluster:

```
postgresql@14-main.service
```

was:

```
failed
```

Meaning PostgreSQL never started.

### 4. Try starting it

The second clue says:

```bash
sudo systemctl start postgresql
```

Why try starting it? Because PostgreSQL might simply be stopped normally. But if it fails to start, that tells us something is preventing it from starting — and that's where troubleshooting begins.

### 5. Look for the cause of the failure

We use:

```bash
journalctl -p err
```

`journalctl` shows system and systemd logs, and `-p err` means "show only error-level messages."

Here we saw:

```
Duplicate entry in /etc/fstab
```

This led us to suspect `/etc/fstab` was the problem — but that turned out to be a **misleading diagnosis** in the context of this lab. The full set of clues points to the correct path: after checking the service's errors, move on to checking the system and disk space.

### 6. Check disk space

The clue says:

```bash
df -h
```

This tells you how much storage space exists on each filesystem, and how much is left. For example, you might see:

```
Filesystem      Size  Used Avail Use%
/dev/xvdf1       1G   1G     0  100%
```

If `Use%` = 100%, the disk is full.

### 7. Why does a full disk stop PostgreSQL from running?

This is the most important point in the lab.

PostgreSQL constantly needs to write to disk:

```
PostgreSQL
    ↓
writes data
    ↓
/opt/pgdata/main
    ↓
Disk
```

If the disk is full:

```
PostgreSQL
    ↓
tries to write a file
    ↓
Disk FULL ❌
    ↓
"No space left on device"
    ↓
PostgreSQL fails to start
```

### 8. How do we confirm it's actually "No Space"?

The clue gives you:

```bash
grep -i 'no space left' /var/log/syslog
```

This searches the system log for the message "No space left." Finding it is strong evidence the problem is: not enough space on the disk.

### 9. So what filled up the disk?

The next clue says:

```bash
du -sh /opt/pgdata/main
```

`du` = Disk Usage. `-s` means "give me the total only." `-h` means "make the size human-readable" (e.g. `800M`, `1.2G`).

We use it to find out how much space the PostgreSQL data directory is using.

### 10. Compare with the size of `/opt/pgdata`

The idea:

```
Volume
/opt/pgdata
       ↓
limited space

PostgreSQL Data
/opt/pgdata/main
       ↓
consumes space
```

If `/opt/pgdata` = 1G, but `/opt/pgdata/main` = 950M, and the disk is nearly full, we start looking for the files that took up the remaining space.

### 11. What did we find?

The solution tells you the problem files are:

```
/opt/pgdata/file*.bk
```

Meaning there are backup-like files named:

```
file1.bk
file2.bk
file3.bk
...
```

inside `/opt/pgdata/`. These files aren't needed for PostgreSQL to run, but they consumed a large chunk of the volume's space:

```
/opt/pgdata
│
├── main/          ← PostgreSQL data
│
├── file1.bk       ← unnecessary backup
├── file2.bk       ← unnecessary backup
├── file3.bk       ← unnecessary backup
└── ...
```

These files are what's filling up the disk.

### 12. The fix

We delete the files:

```
/opt/pgdata/file*.bk
```

For example:

```bash
sudo rm /opt/pgdata/file*.bk
```

**Careful:** this command permanently deletes the matching files — only run it because the lab specifically identifies these as the files to remove.

After that, we have enough free space again.

### 13. Start PostgreSQL again

```bash
sudo systemctl start postgresql
```

PostgreSQL should now be able to start, since the issue blocking it — the full disk — has been resolved.

Then confirm:

```bash
sudo systemctl status postgresql
```

The actual cluster should now be running.

### 14. Finally, run the test

```bash
sudo -u postgres psql -c "insert into persons(name) values ('jane smith');" -d dt
```

Expected:

```
INSERT 0 1
```

And that's the end of the lab.

### Full Picture of the Lab

```
                PostgreSQL
                     │
                     ↓
          needs to read/write to disk
                     │
                     ↓
              /opt/pgdata
                     │
              ┌──────┴──────┐
              ↓             ↓
             main/        file*.bk
              │             │
              │             │
       PostgreSQL data   takes up space
                            │
                            ↓
                       Disk becomes full
                            │
                            ↓
                 PostgreSQL can't start
                            │
                            ↓
                  postgresql@14-main
                         FAILED
                            │
                            ↓
                     psql can't connect
```

<img width="1175" height="382" alt="image" src="https://github.com/user-attachments/assets/d459fe02-c5f9-4194-a67a-25558d736db6" />
