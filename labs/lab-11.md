# Lab 11: "Fukuoka": Forbidden Association

## Description
There's a web server running on this host but curl localhost returns the default 404 Not Found page.

Fix the issue so that a file is served correctly and the message Welcome to the Real Site! is returned.

<img width="508" height="184" alt="image" src="https://github.com/user-attachments/assets/53d793fa-9877-4e4b-b3a1-8db6e08a47ed" />

🔗 **Lab Link:** [SadServers - "Fukuoka": Forbidden Association](https://sadservers.com/scenario/fukuoka)

<br>

## 🪜 Steps

## Step 1 — Find the file Nginx is trying to serve

Review the Nginx configuration to see which file it's looking for:

```bash
sudo nginx -T 2>&1 | grep -E 'root|index'
# -T: Test the Nginx configuration and dump the full configuration.
```

Example output:

```
root /var/www/html;
index index.html;
```

This means a request to `curl localhost` will make Nginx look for:

```
/var/www/html/index.html
```

---

## Step 2 — Check `index.html`

```bash
sudo ls -l /var/www/html/index.html
```

Example output:

```
lrwxrwxrwx 1 root root ... /var/www/html/index.html -> /opt/site-content/real_index.html
```

The leading `l` means `index.html` is a **symbolic link**, and the arrow shows where it actually points:

```
/opt/site-content/real_index.html
```

---

## Step 3 — Check the real file

```bash
sudo ls -l /opt/site-content/real_index.html
```

Example output:

```
-rw-r----- 1 root root ... real_index.html
```

Permissions breakdown (`640`):

| Owner | Group | Others |
|-------|-------|--------|
| `rw-` | `r--` | `---`  |

The problem: Nginx runs as `www-data`. If the file's group is `root`, `www-data` has no read access at all.

---

## Step 4 — Confirm the Nginx worker user

```bash
ps aux | grep nginx
```

Example output:

```
root      ... nginx: master process
www-data  ... nginx: worker process
```

The **worker process** — the one that actually handles requests — runs as `www-data`.

---

## Step 5 — Grant `www-data` read access

Change the file's group to `www-data`:

```bash
sudo chown root:www-data /opt/site-content/real_index.html
```

Set safe, minimal permissions:

```bash
sudo chmod 640 /opt/site-content/real_index.html
```

Resulting permissions:

| Owner (`root`) | Group (`www-data`) | Others |
|-----------------|---------------------|--------|
| `rw-`           | `r--`               | `---`  |

Nginx can now read the file.

---

## Step 6 — Test

```bash
curl localhost
```

If you still don't see the expected content (e.g. `Welcome to the Real Site!`), there's another issue to fix.

---

## Step 7 — Check the parent directory

```bash
sudo ls -ld /var/www
```

`/var/www` may be missing the execute (`x`) permission for **others**. On a directory, `x` allows a process to traverse into it and reach the files inside — even if the target file itself is readable, Nginx must be able to walk the full path to get there.

---

## Step 8 — Allow directory traversal

```bash
sudo chmod o+x /var/www
```

Breakdown:

| Symbol | Meaning |
|--------|---------|
| `o`    | others  |
| `+`    | add     |
| `x`    | execute |

---

## Step 9 — Final test

```bash
curl localhost
```

Expected output:

```
Welcome to the Real Site!
```

---

## Complete Solution

```bash
# 1. Find Nginx root/index configuration
sudo nginx -T 2>&1 | grep -E 'root|index'

# 2. Check the web file
sudo ls -l /var/www/html/index.html

# 3. Check the symbolic link target
sudo ls -l /opt/site-content/real_index.html

# 4. Check the Nginx worker user
ps aux | grep nginx

# 5. Change the group to www-data
sudo chown root:www-data /opt/site-content/real_index.html

# 6. Set minimal file permissions
sudo chmod 640 /opt/site-content/real_index.html

# 7. Check the parent directory
sudo ls -ld /var/www

# 8. Allow directory traversal
sudo chmod o+x /var/www

# 9. Test
curl localhost
```

## Key Takeaways

- A symlinked `index.html` means the **target file's** ownership and permissions matter, not just the link itself.
- Nginx's worker process runs as `www-data` (not `root`) — it needs explicit read access.
- Directory traversal (`x`) permission is required on **every directory in the path**, not just the final file.



