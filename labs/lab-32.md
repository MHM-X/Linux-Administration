# Lab 32: "Moyogalpa": Security Snag. The Trials of Mary and John

## Description

Mary and John are working on a Golang web application, and the security team has asked them to implement security measures. Unfortunately, they have broken the application, and it no longer functions. They need your help to fix it.

The fixed application should be able to allow clients to communicate with the application over HTTPS without ignoring any checks. (eg: curl https://webapp:7000/users.html) and serve its static files.

Test: curl https://webapp:7000/users.html should return the content of file.

🔗 **Lab Link:** [SadServers - "Moyogalpa": Security Snag. The Trials of Mary and John](https://sadservers.com/scenario/moyogalpa)

<br>

## 🪜 Steps

### Solution Summary

```bash
# ============================================================
# STAGE 1 — Find the initial problem
# ============================================================
# Problem:
# The webapp service is running, but it cannot access the TLS
# certificate and private key in /home/webapp/pki/.
#
# As a result, the application cannot properly start HTTPS.
#
# This command shows the webapp service logs so we can identify
# the exact error.

sudo journalctl -u webapp -ef


# ============================================================
# STAGE 2 — Fix TLS file permissions
# ============================================================
# Problem:
# The PKI files are owned by root, while the webapp runs as
# the "webapp" user.
#
# Therefore, webapp gets "Permission denied" when trying to
# read server.crt and server.pem.
#
# Solution:
# Make webapp the owner and group of the PKI directory.
# Then give the owner (webapp) read/write permissions only.
#
# 0600:
# owner  = rw-
# group  = ---
# others = ---
#
# This is especially important for server.pem because it
# contains the private key.

chown -vR webapp.webapp /home/webapp/pki
chmod -v 0600 /home/webapp/pki/server.*


# ============================================================
# STAGE 3 — Fix static file permissions
# ============================================================
# Problem:
# The webapp also needs to read files from:
#
#     /home/webapp/static-files/
#
# But these files are also owned by root.
#
# Solution:
# Make webapp the owner and give it read/write access.
# Give the group read access and deny access to others.
#
# 0640:
# owner  = rw-
# group  = r--
# others = ---

chown -Rv webapp.webapp /home/webapp/static-files
chmod -Rv 0640 /home/webapp/static-files/*


# ============================================================
# STAGE 4 — Trust the private Certificate Authority (CA)
# ============================================================
# Problem:
# The webapp uses a certificate signed by a private CA.
# Ubuntu does not trust this CA by default.
#
# Therefore, curl will reject the server certificate even
# though the certificate itself may be valid.
#
# Solution:
# Copy the CA certificate into the system's trusted CA directory,
# then update the system CA trust store.

sudo cp -v /home/webapp/pki/CA.crt /usr/local/share/ca-certificates/CA.crt
sudo chmod 0644 /usr/local/share/ca-certificates/CA.crt
sudo update-ca-certificates


# ============================================================
# STAGE 5 — Check the hostname in the server certificate
# ============================================================
# Problem:
# If we run:
#
#     curl https://localhost:7000
#
# the hostname being requested is "localhost".
#
# However, the server certificate may be issued for "webapp".
#
# TLS verifies that the hostname we connect to matches the
# hostname listed in the certificate.
#
# This command displays the certificate's Subject and
# Subject Alternative Name (SAN) so we can see which hostname
# the certificate is valid for.

echo QUIT | openssl s_client -connect localhost:7000 -showcerts 2>/dev/null | openssl x509 -noout -subject -ext subjectAltName


# ============================================================
# STAGE 6 — Make "webapp" resolve to the local server
# ============================================================
# Problem:
# The certificate is valid for "webapp", so we want to use:
#
#     https://webapp:7000
#
# instead of:
#
#     https://localhost:7000
#
# But the system does not know what IP address "webapp"
# refers to.
#
# Solution:
# Add a hostname-to-IP mapping to /etc/hosts:
#
#     webapp → 127.0.10.1
#
# Now the hostname used by curl matches the hostname in
# the certificate.

echo '127.0.10.1 webapp' >> /etc/hosts


# ============================================================
# STAGE 7 — Fix the AppArmor restriction
# ============================================================
# Problem:
# At this point, normal Linux file permissions allow webapp
# to read the static files.
#
# However, AppArmor is another security layer.
#
# The AppArmor profile for webapp still prevents the application
# from reading:
#
#     /home/webapp/static-files/
#
# This causes the application to return:
#
#     403 Forbidden
#
# Solution:
# Edit the AppArmor profile.

sudo nano /etc/apparmor.d/usr.local.bin.webapp
```
>Inside the AppArmor profile, add:
>/home/webapp/static-files/ r,
>/home/webapp/static-files/* r,
>Then save and exit nano.

```bash
# ============================================================
# STAGE 8 — Reload the AppArmor profile
# ============================================================
# Problem:
# We modified the AppArmor profile on disk, but AppArmor is
# still using the old version.
#
# Solution:
# Reload the modified profile.

sudo apparmor_parser -r /etc/apparmor.d/usr.local.bin.webapp


# ============================================================
# STAGE 9 — Final test
# ============================================================
# All the security layers that were blocking the request
# should now be fixed:
#
# 1. TLS certificate/key permissions  → fixed
# 2. Static file permissions          → fixed
# 3. Private CA trust                → fixed
# 4. Certificate hostname            → webapp
# 5. webapp hostname resolution      → /etc/hosts
# 6. AppArmor access                 → allowed
#
# The final request should work without using -k or disabling
# TLS certificate verification.

curl https://webapp:7000/users.html
```

## 1. First: what's the application we have?

We have a web application written in **Go**, named:

```
webapp
```

Running as a Linux service:

```
webapp.service
```

listening on:

```
port 7000
```

Roughly:

```
             Linux Server
┌───────────────────────────────────────────┐
│                                           │
│              webapp                       │
│                │                          │
│                │ listens                  │
│                ▼                          │
│             :7000                         │
│                                           │
└───────────────────────────────────────────┘
```

The application serves static files, e.g.:

```
/home/webapp/static-files/users.html
```

So if a request comes in for:

```
GET /users.html
```

the application looks for `/home/webapp/static-files/users.html` and sends its contents to the client.

---

## 2. But the app uses HTTPS, not HTTP

The lab doesn't want:

```
http://webapp:7000
```

it wants:

```
https://webapp:7000
```

This brings us into **TLS**. For the Go application to serve HTTPS, it needs:

```
server.crt
server.pem
```

located at:

```
/home/webapp/pki/
```

Roughly:

```
/home/webapp/
│
├── pki/
│   ├── CA.crt
│   ├── server.crt
│   └── server.pem
│
└── static-files/
    └── users.html
```

---

## 3. What does each file do?

**`server.crt`** — the **server certificate**. It essentially tells the client: "I'm a server named `webapp`, this is my identity, and this is the CA that signed my certificate."

**`server.pem`** — in this lab, this is the **private key** the server needs to establish a TLS session. It's highly sensitive — no regular user on the system should be able to read it.

**`CA.crt`** — the certificate of the **Certificate Authority** that issued/signed the server's certificate:

```
                 CA
                 │
                 │ signs
                 ▼
          server.crt
                 │
                 │ belongs to
                 ▼
              webapp
```

---

## 4. Now imagine the journey of a `curl` request

We want:

```bash
curl https://webapp:7000/users.html
```

What happens involves several stages:

```
curl
 │
 │ 1. Who is webapp?
 ▼
DNS / /etc/hosts
 │
 │ 2. Connect to IP:7000
 ▼
webapp
 │
 │ 3. TLS handshake
 │
 ├── server.crt
 └── server.pem
 │
 │ 4. Is certificate trusted?
 ▼
CA trust
 │
 │ 5. Does certificate belong to webapp?
 ▼
Hostname verification
 │
 │ 6. Give me /users.html
 ▼
static-files/users.html
 │
 │ 7. Linux permissions?
 ▼
 │
 │ 8. AppArmor?
 ▼
Response
```

**This is the entire lab.** The clues simply walk us through fixing each of these stages, one at a time.

---

## 5. First problem: the app can't even start HTTPS

Start with:

```bash
sudo journalctl -u webapp -ef
```

We find:

```
open /home/webapp/pki/server.crt: permission denied
open /home/webapp/pki/server.pem: permission denied
```

What does this mean? The app tried to:

```
webapp
  │
  ├── read server.crt
  │
  └── read server.pem
```

but Linux said:

```
Permission denied ❌
```

Why? Because of **Linux file permissions**. For example, if the file is:

```
-rw------- root root server.pem
```

the owner is `root`, but the app runs as the user `webapp`:

```
root ──────── can read ✅

webapp ────── cannot ❌
```

The app can't serve HTTPS without access to the key/certificate.

---

## 6. First fix: give `webapp` ownership of the PKI files

The clue:

```bash
chown -vR webapp.webapp /home/webapp/pki
```

i.e., `Owner = webapp`, `Group = webapp`:

```
/home/webapp/pki/
        │
        ├── server.crt → webapp:webapp
        └── server.pem → webapp:webapp
```

Then:

```bash
chmod -v 0600 /home/webapp/pki/server.*
```

i.e.:

```
             owner   group   others
               rw-     ---     ---
```

Only `webapp` can read them — safer, especially for the private key.

---

## 7. Now the app can start HTTPS

After this step:

```
webapp
 │
 ├── server.crt ✅
 └── server.pem ✅
       │
       ▼
     HTTPS
       │
       ▼
     :7000
```

Test it:

```bash
curl https://localhost:7000/users.html
```

But a new error appears. Important thing to understand:

> **A new error appearing after fixing the first one is a good sign.** It means we've gotten past the first layer.

---

## 8. Second problem: our machine doesn't trust the CA

Say the server sends `server.crt` to `curl`. `curl` says: "this certificate was signed by CA X" — then asks: "do I trust this CA?"

Ubuntu has a set of trusted CA certificates, but the CA in this lab is a **private CA**, not one of the system's known public CAs:

```
webapp
   │
   │ server.crt
   ▼
 curl
   │
   │ "who signed this certificate?"
   ▼
 Unknown CA ❌
```

---

## 9. Add the CA to the system

The clue:

```bash
sudo cp -v /home/webapp/pki/CA.crt /usr/local/share/ca-certificates/CA.crt
```

We take `/home/webapp/pki/CA.crt` and place it in `/usr/local/share/ca-certificates/` — the location designated for locally-added CA certificates. Then:

```bash
sudo chmod 0644 /usr/local/share/ca-certificates/CA.crt
```

because this is a **public CA certificate**, not a private key. Then:

```bash
sudo update-ca-certificates
```

so the system refreshes its trust store. Now:

```
curl
 │
 │ "do I trust this CA?"
 ▼
System CA Store
 │
 └── CA.crt ✅
```

---

## 10. But HTTPS has another check: the server name

The system now trusts the CA — but there's still a question: does this certificate actually belong to the server we requested?

We requested:

```
https://localhost:7000
```

but the certificate says:

```
webapp
```

i.e.:

```
curl
 │
 │ "I'm talking to localhost"
 ▼
server certificate
 │
 │ "I am webapp"
 ▼
❌ Mismatch
```

This is called **hostname verification**.

---

## 11. So we inspect the certificate

```bash
echo QUIT | openssl s_client -connect localhost:7000 -showcerts 2>/dev/null | openssl x509 -noout -subject -ext subjectAltName
```

Its purpose: show information about the server's certificate, especially the name it's issued for. We might see:

```
subject=CN = webapp

X509v3 Subject Alternative Name:
    DNS:webapp
```

So the certificate says it belongs to `webapp`, not `localhost`.

---

## 12. The fix: make `webapp` resolve to the server itself

We ultimately want:

```bash
curl https://webapp:7000/users.html
```

So Linux needs to understand: `webapp = 127.0.10.1`. Add:

```bash
echo '127.0.10.1 webapp' >> /etc/hosts
```

Now:

```
webapp
   │
   ▼
127.0.10.1
   │
   ▼
port 7000
   │
   ▼
webapp
```

Everything lines up now:

```
URL:
https://webapp:7000

Certificate:
webapp

/etc/hosts:
webapp → 127.0.10.1
```

---

## 13. Now we reach the final problem: 403 Forbidden

```bash
curl https://webapp:7000/users.html
```

might now return:

```
403 Forbidden
```

Notice the difference: at the start, the app **couldn't even start HTTPS**. Now the app is running, TLS works, the certificate is trusted, and the hostname is correct. But when we request `/users.html`, the app tries to access `/home/webapp/static-files/users.html`, and there's **another security layer** at play.

---

## 14. First security layer: Linux permissions

Already fixed earlier:

```bash
chown -Rv webapp.webapp /home/webapp/static-files
chmod -Rv 0640 /home/webapp/static-files/*
```

i.e., `users.html` owner = `webapp`, with permissions allowing `webapp` to read it. So from Linux's perspective:

```
webapp
   │
   ▼
users.html
   │
   └── Linux permission → ALLOWED ✅
```

But something else is still blocking it.

---

## 15. AppArmor says no

This is the key concept in this lab: **Linux permissions aren't the only protection system.** We also have **AppArmor**, which operates at the level of **the program itself**. So Linux might say:

```
webapp can read the file ✅
```

but AppArmor says:

```
program /usr/local/bin/webapp is not allowed to read this path ❌
```

Resulting in:

```
                 users.html
                     ▲
                     │
             ┌───────┴────────┐
             │                │
       Linux permissions   AppArmor
             │                │
             │                │
             ✅               ❌
```

and the result:

```
403 Forbidden
```

---

## 16. Where's the AppArmor rule?

The clue:

```
/etc/apparmor.d/usr.local.bin.webapp
```

This is the **AppArmor profile** for `webapp`. It contains rules like:

```
webapp is allowed to do X
webapp is allowed to read Y
webapp is not allowed to access Z
```

---

## 17. Add `static-files` to AppArmor

Add:

```
/home/webapp/static-files/ r,
/home/webapp/static-files/* r,
```

Meaning:

- `/home/webapp/static-files/ r,` — allows the program to access the directory.
- `/home/webapp/static-files/* r,` — allows it to read the files inside.

Resulting in:

```
AppArmor
   │
   └── /home/webapp/static-files/
           │
           ├── users.html → read ✅
           ├── index.html → read ✅
           └── ...
```

---

## 18. Why run `apparmor_parser -r`?

After editing `/etc/apparmor.d/usr.local.bin.webapp`, the change exists on disk, but AppArmor needs to load the new profile:

```bash
apparmor_parser -r /etc/apparmor.d/usr.local.bin.webapp
```

i.e., **reload** this AppArmor profile. Now:

```
AppArmor
   │
   └── webapp
         │
         └── static-files → READ ✅
```

---

## 19. Now let's see the whole lab as one journey

This is the most important part. When you run:

```bash
curl https://webapp:7000/users.html
```

the request roughly passes through this sequence:

```
                    curl
                      │
                      │ HTTPS
                      ▼
              ┌───────────────┐
              │ /etc/hosts    │
              │               │
              │ webapp        │
              │      ↓        │
              │ 127.0.10.1    │
              └───────┬───────┘
                      │
                      │ :7000
                      ▼
                ┌──────────┐
                │  webapp  │
                └────┬─────┘
                     │
                     │ TLS
                     ▼
          ┌──────────────────────┐
          │ server.crt           │
          │ server.pem           │
          └──────────┬───────────┘
                     │
                     ▼
              Certificate OK
                     │
                     ▼
                CA trusted
                     │
                     ▼
          hostname = webapp ✓
                     │
                     ▼
             GET /users.html
                     │
                     ▼
 /home/webapp/static-files/users.html
                     │
              ┌──────┴──────┐
              │             │
              ▼             ▼
       Linux permissions  AppArmor
              │             │
              ✓             ✓
              └──────┬──────┘
                     │
                     ▼
                users.html
                     │
                     ▼
                   curl
```

---

## 20. So why did we go in this order?

Because each problem **blocks access to the next stage**.

At first:

```
TLS certificate
      ↓
Permission denied
```

We fix it. Then we hit:

```
CA verification
      ↓
Unknown CA
```

We fix it. Then:

```
Hostname verification
      ↓
localhost ≠ webapp
```

We fix it. Then:

```
Application requests users.html
      ↓
AppArmor
      ↓
403
```

We fix it. And finally:

```
curl https://webapp:7000/users.html
                    ↓
              SUCCESS ✅
```

---

## 🧠 The security concepts SadServers wants to teach here

This lab isn't really about Go at all — it bundles together **4 Linux/system security concepts**:

| Problem                                        | Layer                        | Fix                       |
|-------------------------------------------------|-------------------------------|----------------------------|
| `server.crt` / `server.pem` unreadable          | Linux permissions/ownership   | `chown` + `chmod`          |
| CA not trusted                                  | TLS trust store                | `update-ca-certificates`  |
| `localhost` doesn't match the certificate       | TLS hostname verification      | `/etc/hosts`               |
| `users.html` returns 403                        | AppArmor                       | edit profile + reload      |

**If you want to remember the whole lab in one sentence:**

> We're not fixing a single web server; we're clearing the path for an HTTPS request through a chain of security layers, and each layer was blocking the request for a different reason.
```bash
```
