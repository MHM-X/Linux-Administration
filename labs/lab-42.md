# Lab 42: "Sumé": Tied in a Knot

## Description
A DNS server running Knot DNS is serving the zone sadservers.internal (see ls /var/lib/knot/zones/), but users are reporting that they cannot access blog.sadservers.internal neither api.sadservers.internal. Your task is to diagnose and fix the DNS issues so the services become accessible.
You can manage Knot DNS with sudo knotc commands.

Note: the 203.0.113.0/24 range is part of TEST-NET-3, a block reserved by RFC 5737 for documentation and examples, making it a Bogon IP range.

IMPORTANT. Do not change the Nginx configurations under /opt/services/ for the solution to work.

Test: You are able to access the blog and the API services: curl blog.sadservers.internal returns Welcome to blog.sadservers.internal
curl api.sadservers.internal returns {"status": "ok", "service": "api.sadservers.internal"}

🔗 **Lab Link:** [SadServers - "Sumé": Tied in a Knot](https://sadservers.com/scenario/sume)

<br>

## 🪜 Steps


## Scenario

A DNS server running **Knot DNS** serves the `sadservers.internal` zone.

Users cannot access:

- `blog.sadservers.internal`
- `api.sadservers.internal`

The goal is to diagnose and fix the DNS configuration so both services become accessible.

The Nginx configurations under `/opt/services/` must not be modified.

---

## Architecture

```text
User
 │
 │ DNS query
 ▼
Knot DNS
 │
 ├── blog.sadservers.internal
 │       │
 │       └── CNAME
 │             ↓
 │       www.sadservers.internal
 │             │
 │             └── A → 203.0.113.10
 │
 └── api.sadservers.internal
         │
         └── A → 203.0.113.20

## Step 1 – Check the DNS Zone

First, validate the zone syntax:

```bash
# Check the syntax and validity of the DNS zone
sudo knotc zone-check sadservers.internal
```

The result showed:

```text
warning: [sadservers.internal.] check, node sadservers.internal., missing glue record
```

This was only a **warning**, not a syntax error.

---

## Step 2 – Inspect the Current DNS Records

Read the records currently loaded by Knot:

```bash
# Display the records currently loaded for the zone
sudo knotc zone-read sadservers.internal
```

The important records were:

```text
api.sadservers.internal. 3600 A 203.0.113.20
blog.sadservers.internal. 3600 CNAME www.sadservers.internal.
```

The `blog` record was a valid CNAME:

```text
blog.sadservers.internal
        ↓
www.sadservers.internal.
```

However, there was no `A` record for `www.sadservers.internal`.

Therefore DNS could resolve:

```text
blog → www
```

but could not resolve:

```text
www → IP address
```

---

## Step 3 – Understand the Problem

A **CNAME** does not contain an IP address.

It points one hostname to another hostname.

For example:

```text
blog.sadservers.internal
        ↓ CNAME
www.sadservers.internal.
        ↓ A
203.0.113.10
```

So the `www` hostname must have an `A` record.

Without it, the DNS resolution chain is incomplete.

---

## Step 4 – Add the Missing `www` Record

Use a Knot DNS transaction to add the correct `A` record:

```bash
# Start a DNS zone transaction
sudo knotc zone-begin sadservers.internal

# Add the correct IPv4 address for the www record
sudo knotc zone-set sadservers.internal www A 203.0.113.10

# Commit the transaction and apply the change
sudo knotc zone-commit sadservers.internal
```

The resulting records are:

```text
www.sadservers.internal.   A      203.0.113.10
blog.sadservers.internal.  CNAME  www.sadservers.internal.
api.sadservers.internal.   A      203.0.113.20
```

---

## Step 5 – Test the Services

Test the blog:

```bash
# Test the blog service through its DNS hostname
curl blog.sadservers.internal
```

Expected:

```text
Welcome to blog.sadservers.internal
```

Test the API:

```bash
# Test the API service through its DNS hostname
curl api.sadservers.internal
```

Expected:

```json
{"status": "ok", "service": "api.sadservers.internal"}
```

---

## Root Cause

There were multiple DNS configuration problems:

1. The `blog` CNAME record initially had an invalid record type (`CNAM`) and needed to be changed to `CNAME`.
2. The CNAME target needed a trailing dot to make it an absolute FQDN:

   ```text
   www.sadservers.internal.
   ```
3. The `www` A record was missing/wrong and needed to point to:

   ```text
   203.0.113.10
   ```
4. The `api` A record was missing and needed to point to:

   ```text
   203.0.113.20
   ```

---

## Key Commands

```bash
# Validate zone syntax
sudo knotc zone-check sadservers.internal

# Read the currently loaded zone
sudo knotc zone-read sadservers.internal

# Start a DNS transaction
sudo knotc zone-begin sadservers.internal

# Remove an incorrect record
sudo knotc zone-unset sadservers.internal www

# Add an A record
sudo knotc zone-set sadservers.internal www A 203.0.113.10

# Add the API A record
sudo knotc zone-set sadservers.internal api A 203.0.113.20

# Commit DNS changes
sudo knotc zone-commit sadservers.internal
```

---

## Key Lessons

* **A record** → maps a hostname to an IPv4 address.
* **CNAME record** → maps one hostname to another hostname.
* A CNAME target ending with `.` is an **absolute FQDN**.
* `zone-check` validates DNS zone syntax.
* `zone-read` shows the records currently loaded by Knot.
* `zone-begin` / `zone-set` / `zone-commit` are used to safely modify a Knot DNS zone.
* When a hostname cannot be resolved, check the **entire DNS resolution chain**, not just the original hostname.

### Final DNS Resolution

```text
blog.sadservers.internal
        │
        ▼
CNAME
        │
        ▼
www.sadservers.internal.
        │
        ▼
A
        │
        ▼
203.0.113.10
```

```text
api.sadservers.internal
        │
        ▼
A
        │
        ▼
203.0.113.20
```

```
```
