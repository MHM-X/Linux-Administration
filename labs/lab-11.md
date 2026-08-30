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

<img width="817" height="193" alt="image" src="https://github.com/user-attachments/assets/e086391b-e166-45da-aedd-776cfadc8d41" />

This means a request to `curl localhost` will make Nginx look for:

```
/var/www/html/index.html
```

---

## Step 2 — Check `index.html`

```bash
sudo ls -l /var/www/html/index.html
```

<img width="1123" height="47" alt="image" src="https://github.com/user-attachments/assets/bf661434-7d15-4b15-985a-def3031696e5" />

The leading `l` means `index.html` is a **symbolic link**, and the arrow shows where it actually points:

```
/opt/site-content/real_index.html
```

---

## Step 3 — Check the real file

```bash
sudo ls -l /opt/site-content/real_index.html
```

<img width="861" height="45" alt="image" src="https://github.com/user-attachments/assets/e6e62246-f645-4244-8f19-c713e9a49c43" />

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

<img width="1178" height="137" alt="image" src="https://github.com/user-attachments/assets/955d1d65-e49e-4b01-9927-8fad1fdf394b" />


The **worker process** — the one that actually handles requests — runs as `www-data`.

 
> **Note — why the worker's identity matters:** the worker is the process that actually opens and reads `index.html`, so Linux checks *its* identity against the file's permissions, not root's. Linux checks in order: is the process the file's **owner**? Is it a member of the file's **group**? If neither, it falls under **others**. Since `www-data` was neither the owner (`root`) nor in the group (`root`), it landed in `others` — which had no permissions (`---`) — and the read was denied. That's exactly why the fix is to make `www-data` the file's group, not just leave ownership at `root`.
 
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

<img width="602" height="51" alt="image" src="https://github.com/user-attachments/assets/03eee12d-ff4c-4b35-8ea5-38ffe4a889da" />

`/var/www` may be missing the execute (`x`) permission for **others**. On a directory, `x` allows a process to traverse into it and reach the files inside — even if the target file itself is readable, Nginx must be able to walk the full path to get there.

---

## Step 8 — Allow directory traversal

```bash
sudo chmod o+x /var/www
```

<img width="597" height="47" alt="image" src="https://github.com/user-attachments/assets/47c28a50-5883-424b-a8a1-c63fcac51332" />

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

<img width="746" height="52" alt="image" src="https://github.com/user-attachments/assets/24dee3ab-4042-43f7-807e-03956e7efd2e" />


