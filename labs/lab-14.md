# Lab 14: "Cairo": Time for a Timer

## Description
A critical health check script at /opt/scripts/health.sh is supposed to run every 10 seconds. This check is triggered by a systemd timer.
The script's job is to check the local Nginx server and write its status (e.g., "STATUS: OK") to the log file at /var/log/health.log.
The log file is not being updated, and it appears the health check is failing.

Find out why the health check system is broken and fix it. The check will pass once the /var/log/health.log file is being correctly updated by the timer with a STATUS: OK message.

🔗 **Lab Link:** [SadServers - "Cairo": Time for a Timer](https://sadservers.com/scenario/cairo)

<br>

## 🪜 Steps

### Step 1: 

```bash
```

