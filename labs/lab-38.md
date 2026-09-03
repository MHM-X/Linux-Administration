# Lab 38: "Bizerte": The Slow Application

## Description
A Python web application running on port 5000 from the /opt directory is experiencing severe performance issues; every request takes more than 5 seconds to complete.
The application is supposed to use the redis-server cache service for speed.

Your mission is to diagnose the performance bottleneck and restore the application to its normal, fast response time.

Do not change the Python application file slow_app.py.

Test: curl localhost:5000 returns Data from FAST cache!

🔗 **Lab Link:** [SadServers - "Bizerte": The Slow Application](https://sadservers.com/scenario/bizerte)

<br>

## 🪜 Steps

## Architecture

The application communicates with Redis:

```text
User
  │
  ▼
Python Application :5000
  │
  ▼
Redis :6379
```

Redis is an **in-memory data store** commonly used as a cache.

Because Redis stores frequently accessed data in memory, it can respond much faster than repeatedly querying a database.

---

## Step 1 — Check the Application

Check whether the application is running:

```bash
# Check the status of the slow application
sudo systemctl status slow-app
```

The application should be active and listening on port `5000`.

---

## Step 2 — Check Redis

Check whether Redis is running:

```bash
# Check the Redis service
sudo systemctl status redis-server
```

Then connect to Redis:

```bash
# Connect to Redis on localhost using the default Redis port
redis-cli -h 127.0.0.1 -p 6379
```

Inside Redis:

```text
PING
```

Expected response:

```text
PONG
```

This confirms that Redis is running and accepting connections.

---

## Step 3 — Inspect the Application and Service

First, inspect the Python application:

```bash
# Inspect the application code to understand how it connects to Redis
cat /opt/slow_app.py
```

Then inspect the systemd service:

```bash
# Display the systemd configuration for the application
sudo systemctl cat slow-app
```

We discover:

```ini
[Service]
Environment="REDIS_HOST=127.0.0.2"
```

This is suspicious because Redis is running on:

```text
127.0.0.1:6379
```

while the application is trying to connect to:

```text
127.0.0.2:6379
```

---

## Root Cause

The application has the **wrong Redis host** configured.

```text
Application
    │
    │ tries to connect
    ▼
127.0.0.2:6379  ❌
```

But Redis is actually running at:

```text
127.0.0.1:6379  ✅
```

Therefore, the application cannot reach Redis normally and falls back to the slow behavior.

---

## Step 4 — Fix the systemd Configuration

Instead of modifying the Python application, modify its systemd service:

```bash
# Edit the complete systemd service file
sudo systemctl edit --full slow-app
```

Change:

```ini
Environment="REDIS_HOST=127.0.0.2"
```

to:

```ini
Environment="REDIS_HOST=127.0.0.1"
```

Save and exit.

---

## Step 5 — Reload systemd

After modifying a systemd unit file, systemd needs to reread the configuration:

```bash
# Make systemd reload the modified unit file
sudo systemctl daemon-reload
```

---

## Step 6 — Restart the Application

Restart the application so it starts with the corrected environment variable:

```bash
# Restart the application with the new Redis configuration
sudo systemctl restart slow-app
```

---

## Step 7 — Test

Finally, test the web application:

```bash
# Test the application through its HTTP endpoint
curl localhost:5000
```

Expected result:

```text
Data from FAST cache!
```

---

## Final Configuration

The correct architecture is:

```text
             HTTP
User ──────────────────► Python App :5000
                           │
                           │ Redis connection
                           ▼
                       Redis :6379
                       127.0.0.1
```
