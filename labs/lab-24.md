# Lab 01: "Cape Town": Borked Nginx

## Description
There's an Nginx web server installed and managed by systemd. Running curl -I 127.0.0.1:80 returns curl: (7) Failed to connect to localhost port 80: Connection refused , fix it so when you curl you get the default Nginx page.

Test: curl -Is 127.0.0.1:80|head -1 returns HTTP/1.1 200 OK

🔗 **Lab Link:** [SadServers - "Cape Town": Borked Nginx](https://sadservers.com/scenario/capetown)

<br>

## 🪜 Steps

### 1. What does the lab require?

We have a web server, Nginx, managed by systemd.

We need:

```bash
curl -Is 127.0.0.1:80 | head -1
```

to return:

```
HTTP/1.1 200 OK
```

But at the start, it returned:

```
Connection refused
```

Meaning: no Nginx process is available to accept connections on port 80.

### 2. Check Nginx's status

First command:

```bash
sudo systemctl status nginx
```

Since Nginx is managed by systemd, we want to know: is it running or has it failed?

We found:

```
Active: failed
```

And something important:

```
ExecStartPre=/usr/sbin/nginx -t
(code=exited, status=1/FAILURE)
```

This means that before starting Nginx, systemd runs `nginx -t` to validate the configuration — and that check failed.

### 3. Check the Nginx configuration ourselves

```bash
sudo nginx -t
```

Output:

```
nginx: [emerg] unexpected ";" in /etc/nginx/sites-enabled/default:1
```

Now we know exactly:

- **Problem:** unexpected `;`
- **File:** `/etc/nginx/sites-enabled/default`
- **Line:** 1

### 4. Look at the offending line

```bash
sudo head -5 /etc/nginx/sites-enabled/default
```

Output:

```
;
##
# You should look at...
```

Notice the first line is just:

```
;
```

That's the error — Nginx doesn't expect a lone `;` at the start of the configuration.

### 5. Fix the first issue

Delete the first line:

```bash
sudo sed -i '1d' /etc/nginx/sites-enabled/default
```

Breaking it down:

- `sed` — text editing tool
- `-i` — modify the file in place
- `1d` — delete line 1

The configuration is now valid. Restart Nginx:

```bash
sudo systemctl restart nginx
```

### 6. A new error appears: HTTP 500

After fixing the config, Nginx can now run — but `curl` returns:

```
HTTP 500
```

This matters. Compare:

**Before the fix:**
```
curl
 ↓
Nginx ❌
 ↓
Connection refused
```

**After fixing the config:**
```
curl
 ↓
Nginx ✅
 ↓
HTTP 500
```

We've gotten past the first problem, but there's a second issue happening while Nginx handles the request.

### 7. Check the logs

The clue tells us to look in `/var/log/`.

```bash
sudo ls -lah /var/log/
```

We find `nginx/`:

```bash
sudo ls -lah /var/log/nginx/
```

Inside:

```
access.log
error.log
```

Since we're looking for the cause of an error, we check `error.log`.

### 8. Read the most recent errors

Instead of reading the whole file:

```bash
sudo tail -50 /var/log/nginx/error.log
```

The key lines:

```
open() "/var/www/html/index.nginx-debian.html" failed
(24: Too many open files)
```

and:

```
accept4() failed (24: Too many open files)
```

So the second problem is: **Nginx has hit its limit on open file descriptors.**

### 9. How do we find the limit?

Since the problem is "too many open files," the question becomes: what controls how many files/descriptors Nginx can open?

Nginx is managed by systemd, so we inspect its unit:

```bash
systemctl cat nginx
```

(Note: this isn't the regular `cat` — `systemctl` has a `cat` subcommand that prints the unit file.)

We found:

```
[Service]
...
LimitNOFILE=10
```

🔥 That's the real cause of the second problem.

### 10. What does `LimitNOFILE=10` mean?

It means systemd only allows Nginx to use a very small number of file descriptors — just 10.

Nginx needs descriptors for:

- Network connections
- Files
- Sockets
- ...

10 is far too few, which results in:

```
Too many open files
```

### 11. Fix the limit

Open the unit for editing:

```bash
sudo systemctl edit --full nginx
```

Change:

```
LimitNOFILE=10
```

to a much larger value, e.g.:

```
LimitNOFILE=65535
```

### 12. Tell systemd the unit changed

After editing the service file:

```bash
sudo systemctl daemon-reload
```

This tells systemd: "re-read the unit files, since one of them changed."

### 13. Restart Nginx

```bash
sudo systemctl restart nginx
```

Nginx will now run with the new limit.

### 14. Final test

```bash
curl -Is 127.0.0.1:80 | head -1
```

Breaking it down:

- `curl` — send an HTTP request
- `-I` — headers only (HEAD request)
- `-s` — silent mode
- `127.0.0.1:80` — connect to localhost on port 80
- `|` — pipe the output
- `head -1` — show only the first line

Expected:

```
HTTP/1.1 200 OK
```

Lab complete. ✅

### Full Troubleshooting Map

```
curl 127.0.0.1:80
        ↓
Connection refused
        ↓
systemctl status nginx
        ↓
nginx = failed
        ↓
nginx -t
        ↓
unexpected ";" line 1
        ↓
inspect default
        ↓
found ";"
        ↓
delete it
        ↓
restart nginx
        ↓
curl again
        ↓
HTTP 500
        ↓
check /var/log/nginx/error.log
        ↓
Too many open files
        ↓
systemctl cat nginx
        ↓
LimitNOFILE=10
        ↓
increase LimitNOFILE
        ↓
daemon-reload
        ↓
restart nginx
        ↓
curl
        ↓
HTTP/1.1 200 OK ✅
```

### Commands Used, in Order

```bash
# 1. Check Nginx status
sudo systemctl status nginx

# 2. Test Nginx configuration
sudo nginx -t

# 3. Inspect the beginning of the config file
sudo head -5 /etc/nginx/sites-enabled/default

# 4. Remove the broken first line
sudo sed -i '1d' /etc/nginx/sites-enabled/default

# 5. Restart Nginx
sudo systemctl restart nginx

# 6. Check Nginx logs
sudo ls -lah /var/log/
sudo ls -lah /var/log/nginx/

# 7. Read recent errors
sudo tail -50 /var/log/nginx/error.log

# 8. Inspect the systemd unit
systemctl cat nginx

# 9. Edit the unit and increase the file-descriptor limit
sudo systemctl edit --full nginx

# 10. Reload systemd configuration
sudo systemctl daemon-reload

# 11. Restart Nginx
sudo systemctl restart nginx

# 12. Verify
curl -Is 127.0.0.1:80 | head -1
```

**Core takeaway:** this lab wasn't a single-issue problem. A configuration error blocked Nginx from starting at all; after fixing it, a resource-limit error (`LimitNOFILE=10`) surfaced. It's a good example of incremental troubleshooting: fix the visible problem → retest → read the new error → keep diagnosing.
