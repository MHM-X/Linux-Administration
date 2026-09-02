# Lab 33: "Bekasi": Supervisor is still around

## Description
There is an nginx service running on port 443, it is the main web server for the company and looks like a new employee has deployed some changes to the configuration of supervisor and now it is not working as expected.

If you try to access curl -k https://bekasi it should return Hello SadServers! but for some reason it is not.

You cannot modify files from the /home/admin/bekasi folder in order to pass the check.sh

You must find out what the issue is and fix it.

Test: curl -k https://bekasi returns Hello SadServers!

🔗 **Lab Link:** [SadServers - "Bekasi": Supervisor is still around](https://sadservers.com/scenario/bekasi)

<br>

## 🪜 Steps

## 1. What's required?

A company has a main web server, **Nginx**, running on:

```
HTTPS → port 443
```

We need this command:

```bash
curl -k https://bekasi
```

to return:

```
Hello SadServers!
```

but it currently doesn't return the correct result.

**Important constraint:** it's forbidden to modify anything inside `/home/admin/bekasi`. So if we find a problem inside the application, we cannot fix it by editing the application's files — we need to discover that the issue is in *how the application is run*.

---

## 2. Understand the architecture first

We have several pieces:

```
Client
  │
  │ HTTPS :443
  ▼
Nginx
  │
  ▼
Application
```

But the application isn't run directly. A program called **Supervisor** manages it:

```
Supervisor
    │
    │ starts/manages
    ▼
  uWSGI
    │
    │ runs
    ▼
 Flask Application
```

So the full picture:

```
curl
  │
  ▼
Nginx :443
  │
  ▼
uWSGI
  │
  ▼
Flask Application

Supervisor
    │
    └──── manages uWSGI
```

---

## 3. What is each piece?

**Nginx** — web server and reverse proxy. Receives the `curl -k https://bekasi` request on 443, then forwards it to the application.

**Flask** — a Python framework for building web applications; contains the app's logic, e.g. `return "Hello SadServers!"`.

**WSGI** — an interface/standard letting a Python web app communicate with an application server.

**uWSGI** — an application server that runs a Python/Flask app via WSGI.

**Supervisor** — a process manager. Its job: start uWSGI, restart it if it stops, and manage the process. So:

```
Supervisor
     ↓
   uWSGI
     ↓
 Flask
```

---

## 4. Start troubleshooting

First question: is Nginx running?

```bash
systemctl status nginx
```

Result:

```
Active: active (running)
```

So Nginx ✅ — no need to touch Nginx.

---

## 5. Is Supervisor running the app?

```bash
sudo supervisorctl status
```

Result:

```
bekasi    RUNNING    pid 829
```

So Supervisor ✅ and the `bekasi` process ✅. Important note: **`RUNNING` doesn't necessarily mean the app is working correctly** — only that Supervisor successfully started the process.

---

## 6. See how Supervisor runs the app

We have the file:

```
/etc/supervisor/conf.d/uwsgi.conf
```

Read it:

```bash
cat /etc/supervisor/conf.d/uwsgi.conf
```

Found:

```ini
[program:bekasi]
autorestart=true
command=/home/admin/bekasi/bin/uwsgi --ini /home/admin/bekasi/bekasi.ini
directory=/home/admin/bekasi
redirect_stderr=true
stdout_logfile=/var/log/bekasi.log
```

The key line:

```ini
command=/home/admin/bekasi/bin/uwsgi --ini /home/admin/bekasi/bekasi.ini
```

means: when Supervisor wants to run `bekasi`, it starts uWSGI using the `bekasi.ini` config.

---

## 7. What's inside `bekasi.ini`?

```bash
cat /home/admin/bekasi/bekasi.ini
```

Found:

```ini
[uwsgi]
module = wsgi:app

master = true
processes = 5

socket = bekasi.sock
chmod-socket = 666
uid = admin
gui = admin

vacuum = true

die-on-term = true
```

The line `module = wsgi:app` roughly means:

```
wsgi.py
   │
   └── app
        │
        ▼
     Flask app
```

and uWSGI runs this application.

---

## 8. The clue gives us an important idea

The logs didn't give us a clear error. So the clue says: *start at the beginning and see if the app works.* Instead of continuing to inspect Nginx and Supervisor, try **the app itself, directly**.

```bash
cd /home/admin/bekasi
```

then:

```bash
bin/python wsgi.py
```

This runs the app directly on port 5000. Then, from another terminal, try reaching it. The point here isn't that this is the fix — the point is: **compare the app when I run it directly myself vs. when Supervisor runs it.**

---

## 9. Why does this comparison matter?

When you run `bin/python wsgi.py`, you're running it from **your own shell**, which has its own environment variables. We can see them with:

```bash
env | grep '^BEKASI_'
```

or by checking:

```bash
cat ~/.bashrc
```

We find:

```
BEKASI_SERVER=bekasi.sadservers.com
BEKASI_USER=admin
```

So your shell's environment contains `BEKASI_SERVER` and `BEKASI_USER`.

---

## 10. What about Supervisor?

Supervisor is what actually runs the app. The process we saw was:

```
bekasi RUNNING pid 829
```

We can inspect its environment:

```bash
sudo cat /proc/829/environ | tr '\0' '\n' | grep '^BEKASI_'
```

Why `/proc/829/environ`? Linux stores information about every process under `/proc/<PID>/`, so `/proc/829/environ` holds the environment of process 829.

`tr '\0' '\n'` is needed because environment variables in `/proc/.../environ` are separated by NULL characters, not newlines.

`grep '^BEKASI_'` filters for only the variables starting with `BEKASI_`.

---

## 11. Here's where we find the problem

We have two different environments.

**When run manually:**

```
Your shell
   │
   ├── BEKASI_SERVER=bekasi.sadservers.com
   └── BEKASI_USER=admin
   │
   ▼
Python application
```

**When run by Supervisor:**

```
Supervisor
   │
   ├── BEKASI_SERVER ❌ missing
   └── BEKASI_USER   ❌ missing
   │
   ▼
uWSGI
   │
   ▼
Python application
```

**The same application runs in two different environments.** That explains why manual execution gives one result and Supervisor execution gives a different one. The application itself isn't broken — **Supervisor just isn't passing the environment variables the app needs.**

---

## 12. What does `BEKASI_SERVER` mean?

It's just an **environment variable** chosen by the app's developer:

```
BEKASI_SERVER
       ↓
bekasi.sadservers.com
```

and likewise:

```
BEKASI_USER
     ↓
admin
```

These aren't Linux- or Supervisor-specific names — they're variables specific to the Bekasi application.

---

## 13. Why does the app need them?

Imagine the app has something like:

```python
import os

server = os.getenv("BEKASI_SERVER")
user = os.getenv("BEKASI_USER")
```

`os.getenv()` returns the value of that environment variable if it exists (e.g. `bekasi.sadservers.com`); if it doesn't exist, it returns `None`, and the app may behave differently.

---

## 14. The fix

We don't modify `/home/admin/bekasi` — the lab forbids it. Instead, we edit **the Supervisor configuration**:

```
/etc/supervisor/conf.d/uwsgi.conf
```

```bash
sudo nano /etc/supervisor/conf.d/uwsgi.conf
```

Add, inside `[program:bekasi]`:

```ini
environment=BEKASI_SERVER="bekasi.sadservers.com",BEKASI_USER="admin"
```

So it becomes:

```ini
[program:bekasi]
autorestart=true
command=/home/admin/bekasi/bin/uwsgi --ini /home/admin/bekasi/bekasi.ini
directory=/home/admin/bekasi
redirect_stderr=true
stdout_logfile=/var/log/bekasi.log
environment=BEKASI_SERVER="bekasi.sadservers.com",BEKASI_USER="admin"
```

---

## 15. What did this line do?

We're telling Supervisor: when you run the `bekasi` program, make these environment variables present inside the process:

```
Supervisor
    │
    │ environment:
    │ BEKASI_SERVER=bekasi.sadservers.com
    │ BEKASI_USER=admin
    ▼
   uWSGI
    │
    ▼
 Flask
```

Now Supervisor's environment matches the one the app ran in manually.

---

## 16. But is editing the file enough?

No. Supervisor may already be running with the old configuration loaded in memory, so we need to make it re-read the change:

```bash
sudo supervisorctl
```

then:

```
reread
```

**What does `reread` do?** Tells Supervisor to re-read the configuration files and detect changes.

then:

```
update
```

**What does `update` do?** Tells Supervisor to apply the changes it detected.

Then confirm with:

```
status
```

---

## 17. Final test

After applying the change:

```bash
curl -k https://bekasi
```

should return:

```
Hello SadServers!
```

Lab solved.

---

## Summary

The problem wasn't:

```
Nginx ❌
```

nor:

```
Supervisor ❌
```

nor:

```
uWSGI ❌
```

nor:

```
Flask ❌
```

It was:

```
Supervisor
    │
    │ starts
    ▼
  uWSGI
    │
    │ missing environment variables ❌
    ▼
 Flask
```

And the fix:

```
/etc/supervisor/conf.d/uwsgi.conf
              │
              ▼
environment=BEKASI_SERVER="bekasi.sadservers.com",BEKASI_USER="admin"
              │
              ▼
Supervisor
              │
              ▼
            uWSGI
              │
              ▼
            Flask
              │
              ▼
       Hello SadServers!
```

**The real lesson from this lab:** when an app is `RUNNING` but behaves differently depending on how it's launched, don't just look at the process and the logs — compare the **environment**, `PATH`, user, working directory, and configuration between manual execution and running as a service.
