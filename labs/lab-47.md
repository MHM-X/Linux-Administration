# Lab 47: "Vienna": The Vanishing Process

## Description
Our monitoring agent monagent keeps "crashing" — or at least that's what it looks like. Every few minutes it seems to disappear, but:

- systemctl status monagent always reports it as active (running)
- journalctl -u monagent shows no errors, panics, or OOM kills
- CPU and memory usage are completely normal
- Restarting the service "fixes" it, but only for a few minutes

Find out what is actually happening and correct /usr/local/bin/monagent-watchdog.sh so it tracks the running process correctly, without disabling the watchdog..

Test: The watchdog unit is still active and, after a monagent restart, it does not kill/restart the newly started worker due to stale PID tracking.

🔗 **Lab Link:** [SadServers - "Vienna": The Vanishing Process](https://sadservers.com/scenario/vienna)

<br>

## 🪜 Steps

## Lab Description

The `monagent` monitoring service appeared to be crashing because its process seemed to disappear every few minutes.

However:

* `systemctl status monagent` always reported `active (running)`.
* `journalctl -u monagent` showed no crashes, panics, or OOM kills.
* CPU and memory usage were normal.
* Restarting the service only fixed the problem temporarily.

The goal was to determine what was actually happening and fix the watchdog script without disabling the watchdog.

---

## Investigation

First, check the service:

```bash
systemctl status monagent
```

Then monitor its main PID:

```bash
watch -n1 systemctl show monagent -p MainPID
```

The PID changes periodically, indicating that the process is being restarted even though the service itself remains `active (running)`.

Inspect the service configuration:

```bash
systemctl cat monagent.service
```

The important setting is:

```ini
WatchdogSec=120s
```

The application does not send `sd_notify(WATCHDOG=1)` messages to systemd.

Therefore, systemd assumes that the service has become unresponsive after the watchdog timeout and restarts it.

---

## The Second Problem — Stale PID Tracking

The watchdog script was also tracking the process using stale PID information.

After systemd restarted `monagent`, the service received a new PID:

```text
Old process → PID 1234
New process → PID 1500
```

If the watchdog continued using the old PID, it could incorrectly conclude that `monagent` was dead or target the wrong process.

The correct source for the current PID is:

```text
/run/monagent/monagent.pid
```

---

## Solution

Rewrite `/usr/local/bin/monagent-watchdog.sh` so it reads the live PID from the PID file and verifies that the process actually exists and is a `python3` process.

```bash
#!/usr/bin/env bash
set -euo pipefail

PIDFILE="/run/monagent/monagent.pid"

LIVE_PID=""
if [[ -f "$PIDFILE" ]]; then
    LIVE_PID=$(cat "$PIDFILE")
fi

if [[ -n "$LIVE_PID" ]] && [[ -d "/proc/${LIVE_PID}" ]] && grep -q "^Name:.*python3" "/proc/${LIVE_PID}/status"; then
    logger -t monagent-watchdog "OK: pid ${LIVE_PID} alive"
    exit 0
fi

logger -t monagent-watchdog "ALERT: pid missing or dead — restarting"
pkill -f "monagent.py" || true
systemctl restart monagent.service
```

Make the script executable:

```bash
sudo chmod +x /usr/local/bin/monagent-watchdog.sh
```

Clear the previous failure state:

```bash
sudo systemctl reset-failed monagent.service
```

---

## How the Fixed Watchdog Works

```text
/run/monagent/monagent.pid
            │
            ▼
       Read current PID
            │
            ▼
    Does /proc/PID exist?
            │
            ▼
     Is it a python3 process?
        /           \
      YES            NO
       │              │
       ▼              ▼
    Exit 0          Restart
```

The watchdog now checks the **current live PID** instead of relying on stale PID information.

---

## Key Lessons

* `systemctl status` showing `active (running)` does not mean the same process/PID has been running continuously.
* A systemd service and its underlying process are different concepts.
* `WatchdogSec` allows systemd to restart a service that fails to report that it is still responsive.
* Applications using systemd watchdog functionality normally need to send `WATCHDOG=1` notifications.
* A stale PID can cause monitoring scripts to track the wrong process.
* `/run/monagent/monagent.pid` provides the current PID used by the application.
* `/proc/<PID>` can be used to verify whether a process currently exists.
* The watchdog should be fixed rather than disabled when the monitoring mechanism itself is required.
