# Lab 29: "Paris": Where is my webserver?

## Description
A developer put an important password on his webserver localhost:5000 . However, he can't find a way to recover it. This scenario is easy to to once you realize the one "trick".

Find the password and save it in /home/admin/mysolution , for example: echo "somepassword" > ~/mysolution

Test: md5sum ~/mysolution returns d8bee9d7f830d5fb59b89e1e120cce8e

🔗 **Lab Link:** [SadServers - "Paris": Where is my webserver?](https://sadservers.com/scenario/paris)

<br>

## 🪜 Steps

## 1. Check the Web Server

First, we checked whether anything was listening on port `5000`:

```bash
ss -lntp | grep :5000
```

The output showed:

```text
LISTEN 0 128 127.0.0.1:5000 0.0.0.0:*
```

This confirmed that a web server was listening on `localhost:5000`.

---

## 2. Test the Web Server

We sent a request using `curl`:

```bash
curl http://localhost:5000
```

The server responded:

```text
Unauthorized
```

So the server was reachable, but it did not provide the password.

---

## 3. Inspect the HTTP Request

To understand what `curl` was sending, we used verbose mode:

```bash
curl -v http://localhost:5000
```

Among the request headers, we found:

```text
> GET / HTTP/1.1
> Host: localhost:5000
> User-Agent: curl/8.14.1
> Accept: */*
```

The important part was:

```text
User-Agent: curl/8.14.1
```

The **User-Agent** is an HTTP request header that identifies the client making the request.

By default, `curl` identifies itself as:

```text
curl/8.14.1
```

Since the scenario specifically mentioned the User-Agent, we suspected that the server might return a different response for another User-Agent.

The scenario included an important clue:

> The user agent of the client you are using against the web server may play a role here.
---

## 4. Change the User-Agent

`curl` provides the `-A` option to specify a custom User-Agent:

```bash
curl -A "Mozilla/5.0" http://localhost:5000
```

This tells `curl` to send:

```text
User-Agent: Mozilla/5.0
```

instead of:

```text
User-Agent: curl/8.14.1
```

The server then responded:

```text
Welcome! Password is FDZPmh5AX3oiJt
```

The password was:

```text
FDZPmh5AX3oiJt
```

---

## 5. Save the Password

We saved the password in the required file:

```bash
echo "FDZPmh5AX3oiJt" > ~/mysolution
```

Since `~` represents `/home/admin`, this created:

```text
/home/admin/mysolution
```

We verified the contents:

```bash
cat ~/mysolution
```

Output:

```text
FDZPmh5AX3oiJt
```

---

## 6. Verify the Solution

Finally, we verified the MD5 checksum:

```bash
md5sum ~/mysolution
```

The expected result was:

```text
d8bee9d7f830d5fb59b89e1e120cce8e
```

## Root Cause

The web server was checking the **User-Agent HTTP header**.

When the request used the default `curl` User-Agent:

```text
User-Agent: curl/8.14.1
```

the server returned:

```text
Unauthorized
```

When we changed the User-Agent to:

```text
User-Agent: Mozilla/5.0
```

the server returned the password.

### Key Commands

```bash
ss -lntp | grep :5000

curl http://localhost:5000

curl -v http://localhost:5000

curl -A "Mozilla/5.0" http://localhost:5000

echo "FDZPmh5AX3oiJt" > ~/mysolution

md5sum ~/mysolution
```

### What I Learned

* `localhost` refers to the current machine.
* Port `5000` was hosting the web server.
* `curl` sends HTTP requests from the terminal.
* `User-Agent` is an HTTP header that identifies the client.
* `curl -A` allows us to customize the User-Agent.
* A web server can use the User-Agent value to change its response.
