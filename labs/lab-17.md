# Lab 17: "Valladolid": Cleaner not cleaning

## Description
The systemd service log-cleaner.service is supposed to be run manually (not a timer or cron job) and delete log files older than 7 days in the /var/log/app directory.

The service runs successfully (exit code 0), but no logs are ever deleted.

Fix the service and/or the script so that old_data.log (older than 7 days) is deleted, but recent_data.log is preserved.

If you accidentally delete the wrong files while debugging, run ~/reset_logs.sh to restore them.

🔗 **Lab Link:** [SadServers - "Valladolid": Cleaner not cleaning](https://sadservers.com/scenario/valladolid)

<br>

## 🪜 Steps

### Step 1: 

```bash
```
