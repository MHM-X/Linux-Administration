# Lab 17: "Valladolid": Cleaner not cleaning

## Description
The systemd service log-cleaner.service is supposed to be run manually (not a timer or cron job) and delete log files older than 7 days in the /var/log/app directory.

The service runs successfully (exit code 0), but no logs are ever deleted.

Fix the service and/or the script so that old_data.log (older than 7 days) is deleted, but recent_data.log is preserved.

If you accidentally delete the wrong files while debugging, run ~/reset_logs.sh to restore them.

Test: Running sudo systemctl restart log-cleaner deletes the file /var/log/app/old_data.log but not /var/log/app/recent_data.log

🔗 **Lab Link:** [SadServers - "Valladolid": Cleaner not cleaning](https://sadservers.com/scenario/valladolid)

<br>

## 🪜 Steps

### Step 1: What's wrong with the service?

```bash
sudo systemctl cat log-cleaner.service
```
<img width="1171" height="261" alt="Screenshot 2026-08-31 195746" src="https://github.com/user-attachments/assets/b7a8d6ee-b20a-43e4-a487-cf88064389ca" />

### Step 2: revuing the scribt

```bash
cat opt/scribts/log-cleaner.sh
```
<img width="857" height="310" alt="Screenshot 2026-08-31 195813" src="https://github.com/user-attachments/assets/c9ae7d8b-5d0d-4d88-a9c9-b80a6bd90e7d" />

### Problems Found

The script had two issues. First, `find .` uses `.` to mean the **current directory**, instead of the `/var/log/app` directory stored in `$LOG_DIR`. Second, `-mtime -7` means files **less than 7 days old**, while we need the files more than 7 days old `-mtime +7` means files **more than 7 days old**. The correct command is `find "$LOG_DIR" -maxdepth 1 -name "*.log" -type f -mtime +7 -print -delete`, where `$LOG_DIR` is the log directory, `-maxdepth 1` limits the search to that directory, `-type f` selects files, `-mtime +7` selects files older than 7 days, and `-delete` removes them.

So, we need to change it to:

```bash
find "$LOG_DIR" -maxdepth 1 -name "*.log" -type f -mtime +7 -print -delete
```

Checking our solution:

<img width="837" height="140" alt="Screenshot 2026-08-31 200626" src="https://github.com/user-attachments/assets/30068efc-6f7e-4849-a7e7-759d4679b879" />
