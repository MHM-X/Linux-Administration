# Lab 36: "Batumi": Troubleshoot "A" cannot connect to "B"

## Description
(To learn the skills to solve this challenge, see Can't Connect to a Service: Linux Troubleshooting Guide)

There is a web server (Caddy) on HTTP port :80 but curl http://127.0.0.1 doesn't work. Find out what's wrong and make the necessary fixes so the web server returns a URL.

Note: as a limitation, the file /home/admin/db_connector.py must not be modified so that the challenge is considered solved properly.
The web server has to respond on the IP address 127.0.0.1; not only on "localhost".

Test: The command curl http://127.0.0.1 returns a URL address.

The "Check My Solution" button runs the script /home/admin/agent/check.sh, which you can see and execute.

🔗 **Lab Link:** [SadServers - "Batumi": Troubleshoot "A" cannot connect to "B"](https://sadservers.com/scenario/batumi)

<br>

## 🪜 Steps

### Step 1: 

## Problem

The web server is running on port `80`, but:

```bash
curl http://127.0.0.1
```

hangs and eventually times out.

The goal is to troubleshoot the complete request path and make the web server return the expected URL.

The application architecture is:

```text
Client
  │
  │ HTTP :80
  ▼
Caddy
  │
  │ Reverse Proxy :5050
  ▼
Python Application
db_connector.py
  │
  │ PostgreSQL connection
  ▼
PostgreSQL :5432
```

There are multiple problems in this chain.

---

# Troubleshooting Approach

Instead of changing configurations randomly, troubleshoot the request from the outside toward the backend:

```text
curl
 ↓
Firewall
 ↓
Caddy :80
 ↓
Python :5050
 ↓
PostgreSQL
 ↓
Database connection configuration
```

At each layer, verify whether it is working before moving to the next one.

---

# 1. Check if a Web Server is Listening on Port 80

First, check the listening network ports:

```bash
sudo netstat -tlpn
```

Caddy should appear as listening on port `80`.

Example:

```text
tcp6  0  0  :::80  :::*  LISTEN  619/caddy
```

We can also use:

```bash
sudo ss -lntp | grep ':80'
```

Example:

```text
LISTEN 0 4096 *:80 *:* users:(("caddy",pid=619,...))
```

This confirms that Caddy is running and listening on port `80`.

---

# 2. Test the Connection

Test the web server:

```bash
curl -v --connect-timeout 5 http://127.0.0.1
```

The connection times out:

```text
Trying 127.0.0.1:80...
Connection timed out
```

Since Caddy is listening on port `80`, but the TCP connection is timing out, investigate the local firewall.

---

# 3. Check the Firewall

The lab instructs us to check the `INPUT` chain:

```bash
iptables -L INPUT
```

The environment may not have `iptables` installed. If it is missing, install it on a Debian/Ubuntu system with:

```bash
sudo apt update
sudo apt install -y iptables
```

Then check:

```bash
sudo iptables -L INPUT -n -v
```

There is one `INPUT` rule blocking the connection.

Delete the first rule:

```bash
sudo iptables -D INPUT 1
```

Test again:

```bash
curl http://127.0.0.1
```

The request can now reach Caddy.

---

# 4. Inspect the Caddy Configuration

Caddy is a web server and reverse proxy.

A reverse proxy receives a request and forwards it to another application.

The flow is:

```text
curl :80
   │
   ▼
Caddy :80
   │
   │ reverse_proxy
   ▼
Python :5050
```

Inspect the Caddy service:

```bash
sudo systemctl cat caddy
```

Then inspect its configuration:

```bash
cat /etc/caddy/Caddyfile
```

The configuration shows that Caddy forwards requests to port `5050`.

For example:

```text
reverse_proxy 127.0.0.1:5050
```

Therefore, Caddy itself is not the final application. It depends on the Python application running on port `5050`.

---

# 5. Check the Python Application

Find the process running on port `5050`:

```bash
ps auxf | grep python3
```

We find:

```text
python3 /home/admin/db_connector.py
```

So the backend application is:

```text
/home/admin/db_connector.py
```

The architecture is now:

```text
curl
  │
  ▼
Caddy :80
  │
  ▼
db_connector.py :5050
```

Test the backend directly:

```bash
curl http://127.0.0.1:5050
```

This helps isolate Caddy from the Python application.

---

# 6. Investigate the Python Application

The Python application connects to a PostgreSQL database.

Inspect its logs:

```bash
sudo journalctl -u db_connector
```

The application is failing because it cannot connect to PostgreSQL.

Check whether PostgreSQL is running:

```bash
sudo systemctl status postgresql
```

PostgreSQL is stopped.

---

# 7. Start PostgreSQL

Start the database:

```bash
sudo systemctl start postgresql
```

Verify:

```bash
sudo systemctl status postgresql
```

Then test the Python backend again:

```bash
curl http://127.0.0.1:5050
```

However, the application still cannot connect to PostgreSQL.

This means there is another problem.

---

# 8. Check the PostgreSQL Port

Check the listening ports:

```bash
sudo netstat -tlpn
```

PostgreSQL is listening on:

```text
5432
```

This is the standard PostgreSQL port.

However, the Python application is configured to connect to:

```text
5433
```

So there is a port mismatch:

```text
Python application
       │
       └── tries to connect to :5433
                    X
                    │
              PostgreSQL
              listening on :5432
```

The database is running, but the application is connecting to the wrong port.

---

# 9. Check the Application Configuration

The database connection configuration is stored in:

```text
/home/admin/.env
```

Check it:

```bash
cat /home/admin/.env
```

The PostgreSQL port is configured as:

```text
5433
```

Change it to:

```text
5432
```

For example, using `sed`:

```bash
sudo sed -i 's/5433/5432/g' /home/admin/.env
```

Verify the change:

```bash
cat /home/admin/.env
```

---

# 10. Restart the Application

The Python application needs to reload the environment configuration.

Restart the service:

```bash
sudo systemctl restart db_connector
```

Then test the backend:

```bash
curl http://127.0.0.1:5050
```

If the backend now responds correctly, test the complete request path:

```bash
curl http://127.0.0.1
```

The web server should now return the expected URL.

---

# 11. Enable PostgreSQL at Boot

In a real environment, we also want PostgreSQL to start automatically after a reboot:

```bash
sudo systemctl enable postgresql
```

This does not start PostgreSQL immediately; it configures it to start automatically during boot.

---

# Final Solution

The lab contains several independent issues:

### Problem 1 — Firewall

The `INPUT` firewall rule was blocking connections to port `80`.

Fix:

```bash
sudo iptables -D INPUT 1
```

### Problem 2 — PostgreSQL was stopped

Fix:

```bash
sudo systemctl start postgresql
```

### Problem 3 — Wrong PostgreSQL port

The application was configured to use:

```text
5433
```

while PostgreSQL was listening on:

```text
5432
```

Fix:

```bash
sudo sed -i 's/5433/5432/g' /home/admin/.env
```

Then restart the application:

```bash
sudo systemctl restart db_connector
```

---

# Complete Troubleshooting Sequence

```bash
# Check listening ports
sudo netstat -tlpn

# Test the web server
curl -v --connect-timeout 5 http://127.0.0.1

# Check the firewall
sudo iptables -L INPUT -n -v

# Delete the blocking INPUT rule
sudo iptables -D INPUT 1

# Check Caddy configuration
sudo systemctl cat caddy
cat /etc/caddy/Caddyfile

# Check the Python application
ps auxf | grep python3

# Test the backend directly
curl http://127.0.0.1:5050

# Check the database service
sudo systemctl status postgresql

# Start PostgreSQL
sudo systemctl start postgresql

# Check PostgreSQL's listening port
sudo netstat -tlpn

# Check the database connector logs
sudo journalctl -u db_connector

# Check the application environment
cat /home/admin/.env

# Change PostgreSQL port from 5433 to 5432
sudo sed -i 's/5433/5432/g' /home/admin/.env

# Restart the application so it loads the new configuration
sudo systemctl restart db_connector

# Test the backend
curl http://127.0.0.1:5050

# Test the complete application through Caddy
curl http://127.0.0.1

# Enable PostgreSQL at boot
sudo systemctl enable postgresql
```

---

# What This Lab Teaches

The most important lesson is **systematic troubleshooting**.

When:

```bash
curl http://127.0.0.1
```

fails, don't immediately edit the application.

Follow the request path:

```text
          curl
            │
            ▼
     Firewall / Network
            │
            ▼
        Caddy :80
            │
            ▼
 Python db_connector :5050
            │
            ▼
     PostgreSQL :5432
```

Find the **first broken component**, fix it, test again, and continue.

This is a very important troubleshooting pattern for Linux and DevOps because real production applications often have multiple services chained together.
