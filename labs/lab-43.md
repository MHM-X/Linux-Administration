# Lab 43: "Stockholm": DNS health check issue

## Description
 The internal status portal on this host should answer on http://127.0.0.1:9167/ with a body containing OK.

It worked until operations ran a package cleanup.

The portal service (stockholm-portal) only runs after a DNS health check at /usr/local/bin/stockholm-dns-check.sh succeeds.

Make the necessary changes so the portal works again.

Do not modify /usr/local/bin/stockholm-dns-check.sh.

Test: The health script /usr/local/bin/stockholm-dns-check.sh runs successfully, stockholm-portal is active, and curl http://127.0.0.1:9167/ returns a response whose body contains OK.

🔗 **Lab Link:** [SadServers - "Stockholm": DNS health check issue](https://sadservers.com/scenario/stockholm)

<br>

## 🪜 Steps

## Architecture

```text
stockholm-dns-health
        │
        │ runs
        ▼
stockholm-dns-check.sh
        │
        │ uses
        ▼
       dig
        │
        ▼
       DNS
        │
        ▼
Health check succeeds
        │
        ▼
stockholm-portal
        │
        ▼
127.0.0.1:9167
```

---

## Problem

First, check the portal and related services:

```bash
# Check the portal, DNS health check, and DNS service
systemctl status stockholm-portal stockholm-dns-health dnsmasq
```

Then inspect the logs:

```bash
# Inspect recent logs from the health check and DNS service
journalctl -u stockholm-dns-health -u dnsmasq --no-pager -n 40
```

The health-check script can also be executed manually:

```bash
# Run the DNS health check manually
/usr/local/bin/stockholm-dns-check.sh
```

The script fails because:

```text
dig: command not found
```

The problem is therefore not the DNS configuration itself. The `dig` command is missing.

---

## Root Cause

The operations package cleanup removed the package that provides the `dig` command.

The package history can be checked with:

```bash
# Check APT history for removed or purged packages
grep -E 'Remove|Purge' /var/log/apt/history.log
```

`dig` is provided by the DNS utilities package:

```text
bind9-dnsutils
```

Therefore:

```text
Package cleanup
      ↓
bind9-dnsutils removed
      ↓
dig removed
      ↓
DNS health check fails
      ↓
stockholm-dns-health fails
      ↓
stockholm-portal does not start
```

---

## Solution

Restore the missing package:

```bash
# Install the package that provides the dig command
sudo apt-get install bind9-dnsutils
```

Verify that `dig` is available:

```bash
# Verify that dig is installed and available in PATH
which dig
```

Expected:

```text
/usr/bin/dig
```

The health check can now be tested again:

```bash
# Run the DNS health check again
/usr/local/bin/stockholm-dns-check.sh
```

---

## Restart the Services

Restart the DNS health check and portal:

```bash
# Restart the DNS health check service
sudo systemctl restart stockholm-dns-health

# Restart the portal service
sudo systemctl restart stockholm-portal
```

Verify their status:

```bash
# Verify that the health check and portal are active
sudo systemctl status stockholm-dns-health stockholm-portal
```

---

## Final Test

Test the status portal:

```bash
# Verify that the portal responds successfully
curl http://127.0.0.1:9167/
```

Expected response:

```text
OK
```

---

## Key Lessons

* A service can fail because of a **missing dependency**, even when its own configuration is correct.
* `dig` is a command-line tool used to perform DNS queries and troubleshoot DNS.
* Package cleanup can accidentally remove tools required by system services.
* `journalctl` is useful for identifying why a systemd service failed.
* Running a service's health-check script manually can reveal the exact failure.
* Always fix the underlying dependency instead of modifying or bypassing the health-check script.

## Final Fix

```bash
# Restore the missing DNS utilities package
sudo apt-get install bind9-dnsutils

# Verify dig
which dig

# Restart the health check
sudo systemctl restart stockholm-dns-health

# Restart the portal
sudo systemctl restart stockholm-portal

# Test the portal
curl http://127.0.0.1:9167/
```

```
```
