# Lab 26: "Melbourne": WSGI with Gunicorn

## Description
There is a Python WSGI web application file at /home/admin/wsgi.py , the purpose of which is to serve the string "Hello, world!". This file is served by a Gunicorn server which is fronted by an nginx server (both servers managed by systemd). So the flow of an HTTP request is: Web Client (curl) -> Nginx -> Gunicorn -> wsgi.py . The objective is to be able to curl the localhost (on default port :80) and get back "Hello, world!", using the current setup.

Test: curl -s http://localhost returns Hello, world! (serving the wsgi.py file via Gunicorn and Nginx)

🔗 **Lab Link:** [SadServers - "Melbourne": WSGI with Gunicorn](https://sadservers.com/scenario/melbourne)

<br>

## 🪜 Steps

### Step 1:

```bash
```
