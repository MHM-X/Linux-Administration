# Lab 27: "Lisbon": etcd SSL cert troubles

## Description
Description: There's an etcd server running on https://localhost:2379 , get the value for the key "foo", ie etcdctl get foo or curl https://localhost:2379/v2/keys/foo

Test: etcdctl get foo returns bar.

🔗 **Lab Link:** [SadServers - "Lisbon": etcd SSL cert troubles](https://sadservers.com/scenario/lisbon)

<br>

## 🪜 Steps

### 1. What's required?

We have an etcd service running on:

```
https://localhost:2379
```

with a key named:

```
foo
```

whose value should be:

```
bar
```

The test is:

```bash
etcdctl get foo
```

which must return:

```
bar
```

But running it gave:

```
certificate has expired or is not yet valid
```

### 2. First question: is etcd even running?

```bash
sudo systemctl status etcd
```

Result:

```
Active: active (running)
```

So etcd itself is up. It was also clear from the process that it uses HTTPS:

```
--cert-file /etc/ssl/certs/localhost.crt
--key-file /etc/ssl/certs/localhost.key
--listen-client-urls=https://localhost:2379
```

So the intended path is:

```
etcdctl
   │
   │ HTTPS
   ▼
localhost:2379
   │
   ▼
etcd
```

### 3. What's wrong with the certificate?

```bash
sudo openssl x509 \
  -in /etc/ssl/certs/localhost.crt \
  -noout -dates
```

Output:

```
notBefore=Dec 31 00:02:48 2022 GMT
notAfter=Jan 30 00:02:48 2023 GMT
```

At first glance it looks like the certificate has simply expired. **But that's the real trap in this lab** — the issue wasn't that the certificate needed renewal.

### 4. The server's clock was wrong

The actual error said:

```
current time 2027-09-01...
is after 2023-01-30...
```

The server thought the date was **2027**, while the certificate is only valid until **2023**. If the clock had been correct at the time, the certificate would still have been valid.

First fix:

```bash
sudo date -s "2023-01-01"
```

i.e., set the server's date back to a point within the certificate's validity window. Then retry:

```bash
etcdctl get foo
```

### 5. But a second problem remained!

This was the harder part of the lab. Even after fixing the clock, an `iptables` rule was silently redirecting the connection.

Check the NAT table:

```bash
sudo iptables -t nat -L --line-numbers
```

Found:

```
Chain OUTPUT

num  target
1    REDIRECT ... tcp dpt:2379 redir ports 443
```

This is a critical rule. It means:

```
any TCP traffic
headed to port 2379
        ↓
REDIRECT
        ↓
port 443
```

So instead of:

```
etcdctl
   ↓
localhost:2379
   ↓
etcd
```

what was actually happening was:

```
etcdctl
   ↓
localhost:2379
   ↓
iptables
   ↓
REDIRECT
   ↓
localhost:443
   ↓
Nginx
```

Not the intended path at all.

### 6. Why was the rule in `OUTPUT`?

Because `etcdctl` runs on the same server — the traffic originates locally:

```
etcdctl
   ↓
OUTPUT
   ↓
2379
```

That's why the rule showed up in the `OUTPUT` chain, not `PREROUTING`.

### 7. Delete the offending rule

Instead of flushing the entire NAT table with:

```bash
sudo iptables -t nat -F
```

we identified the exact rule — `OUTPUT`, rule number `1` — and deleted only that one:

```bash
sudo iptables -t nat -D OUTPUT 1
```

Breaking it down:

- `-t nat` — use the NAT table
- `-D` — delete
- `OUTPUT` — from the OUTPUT chain
- `1` — delete rule number 1

This is safer than `-F`, since it removes only the problem rule.

### 8. Final test

After fixing both problems:

**Problem 1 — wrong server clock:**
```bash
sudo date -s "2023-01-01"
```

**Problem 2 — port redirect (2379 → 443):**
```bash
sudo iptables -t nat -D OUTPUT 1
```

Then:

```bash
etcdctl get foo
```

Expected result:

```
bar
```

### Full picture of the lab

The intended path:

```
              HTTPS
etcdctl ─────────────────► etcd
                            │
                         :2379
                            │
                            ▼
                           foo
                            │
                            ▼
                           bar
```
