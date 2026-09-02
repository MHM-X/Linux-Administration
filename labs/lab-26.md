# Lab 26: "Melbourne": WSGI with Gunicorn

## Description
There is a Python WSGI web application file at /home/admin/wsgi.py , the purpose of which is to serve the string "Hello, world!". This file is served by a Gunicorn server which is fronted by an nginx server (both servers managed by systemd). So the flow of an HTTP request is: Web Client (curl) -> Nginx -> Gunicorn -> wsgi.py . The objective is to be able to curl the localhost (on default port :80) and get back "Hello, world!", using the current setup.

Test: curl -s http://localhost returns Hello, world! (serving the wsgi.py file via Gunicorn and Nginx)

🔗 **Lab Link:** [SadServers - "Melbourne": WSGI with Gunicorn](https://sadservers.com/scenario/melbourne)

<br>

## 🪜 Steps

### 1. What does the lab require?

We have a Python app at:

```
/home/admin/wsgi.py
```

Its purpose is to return:

```
Hello, world!
```

But the app isn't handled directly by Nginx. The architecture is:

```
curl
  │
  │ HTTP :80
  ▼
Nginx
  │
  │ Unix Socket
  ▼
Gunicorn
  │
  │ loads
  ▼
wsgi.py
  │
  ▼
Hello, world!
```

We need:

```bash
curl -s http://localhost
```

to return:

```
Hello, world!
```

### 2. First thing: check Nginx

```bash
sudo systemctl status nginx
```

Initially:

```
Active: inactive (dead)
```

Nginx was stopped. So:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

The difference:

- `start` — runs Nginx now.
- `enable` — makes Nginx start automatically on boot.

Then:

```bash
sudo systemctl status nginx
```

Now:

```
Active: active (running)
```

### 3. Test the app from outside

```bash
curl -s http://localhost
```

Returned:

```
502 Bad Gateway
```

This matters a lot. A 502 here means: Nginx is running, but it couldn't get a valid response from the backend it depends on. So the problem isn't reaching Nginx itself:

```
curl
  ↓
Nginx ✅
  ↓
??? ❌
```

### 4. Find out where Nginx sends the request

```bash
sudo nginx -T 2>/dev/null | grep -nE "listen |location |proxy_pass|upstream"
```

Output:

```
listen 80;

location / {
    proxy_pass http://unix:/run/gunicorn.socket;
}
```

Important discovery: Nginx listens on `:80`, and for requests to `/`, it proxies to `/run/gunicorn.socket`:

```
curl
  │
  │ :80
  ▼
Nginx
  │
  │ /run/gunicorn.socket
  ▼
Gunicorn
```

### 5. Check Gunicorn

```bash
sudo systemctl status gunicorn
```

It was:

```
Active: active (running)
```

And, importantly, its command line:

```
/usr/local/bin/gunicorn --bind unix:/run/gunicorn.sock wsgi
```

Here's the likely bug: Nginx wants `/run/gunicorn.socket`, but Gunicorn is actually running on `/run/gunicorn.sock`. Notice:

```
gunicorn.socket   ❌
              ↑
gunicorn.sock     ✅
```

A one-character difference: `.socket` vs `.sock`. So Nginx is trying to connect to one place while Gunicorn is listening on another.

### 6. Confirm the socket

```bash
sudo ls -l /run/gunicorn*
```

Output:

```
srw-rw-rw- 1 root root 0 ... /run/gunicorn.sock
```

So `/run/gunicorn.sock` exists, and `/run/gunicorn.socket` does not — confirming Nginx was pointed at the wrong socket.

### 7. Understand the Gunicorn service line

The systemd unit had:

```
[Service]
User=admin
Group=admin
WorkingDirectory=/home/admin
ExecStart=/usr/local/bin/gunicorn \
          --bind unix:/run/gunicorn.sock \
          wsgi
Restart=on-failure
```

This means that when systemd starts Gunicorn, it effectively runs:

```
/usr/local/bin/gunicorn \
    --bind unix:/run/gunicorn.sock \
    wsgi
```

In other words: start Gunicorn, have it accept connections on the Unix socket `/run/gunicorn.sock`, and load a Python module named `wsgi`.

### 8. What does `wsgi` mean here?

We have the file:

```
/home/admin/wsgi.py
```

containing:

```python
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html'), ('Content-Length', '0'), ])
    return [b'Hello, world!']
```

When Gunicorn sees `wsgi`, it interprets it as the Python module `wsgi.py`. Since `WorkingDirectory=/home/admin`, it looks for it at `/home/admin/wsgi.py`.

### 9. Test Gunicorn directly

Instead of going through Nginx, we test the backend directly:

```bash
curl -v --unix-socket /run/gunicorn.sock http://localhost/
```

Result:

```
Connected to localhost (/run/gunicorn.sock)
```

then:

```
HTTP/1.1 200 OK
Server: gunicorn
```

So: Gunicorn ✅, `wsgi.py` ✅, Unix socket ✅.

But we also saw:

```
Content-Length: 0
```

This is an important finding. The code sets:

```
'Content-Length', '0'
```

telling the client the response size is zero — even though it actually returns:

```
[b'Hello, world!']
```

That's a bug in the application itself.

### 10. So there were two separate problems

**Problem 1 — socket mismatch:**

```
Nginx    → /run/gunicorn.socket   ❌ (wrong)
Gunicorn → /run/gunicorn.sock     ✅ (correct)
```

Both need to point to the same socket. Nginx's config should be:

```
proxy_pass http://unix:/run/gunicorn.sock;
```

**Problem 2 — wrong `Content-Length` in `wsgi.py`:**

```
'Content-Length', '0'
```

while the app returns `Hello, world!`, which is 13 bytes in ASCII — so the header should say `13`, not `0`.

### 11. Full picture of the lab

Once fixed, the path should be:

```
                    HTTP :80
curl ──────────────────────────► Nginx
                                  │
                                  │ Unix Socket
                                  │ /run/gunicorn.sock
                                  ▼
                              Gunicorn
                                  │
                                  │ loads
                                  ▼
                             /home/admin/wsgi.py
                                  │
                                  ▼
                          "Hello, world!"
```

### 12. Where does systemd fit in?

We have two services:

```
nginx.service
gunicorn.service
```

systemd is responsible for running and managing both:

```
systemd
 ├── nginx.service
 │      └── Nginx
 │
 └── gunicorn.service
        └── Gunicorn
               └── wsgi.py
```

So we can inspect each one individually:

```bash
sudo systemctl status nginx
sudo systemctl status gunicorn
```

And see how a service is configured to run:

```bash
sudo systemctl cat gunicorn
```
