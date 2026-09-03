# Lab 34: "Tukaani": XZ LZMA Library Compromised

## Description
Description: (You can learn about Linux Libraries before starting this scenario).

The Linux shared library liblzma.so has been compromised (the real compromised XZ Utils liblzma has not been used). The liblzma.so at the path /usr/lib/x86_64-linux-gnu/liblzma.so.5.2.5 is the good one. Consider the same library liblzma.so.5.2.5 at other paths as compromised or malicious (ideally we would have used other real versions with different checksums).

Find all instances of this "malicious" liblzma library (remember, it's the same library but in different directory locations) and make it so none of the running processes use it, while the applications "webapp" and "jobapp" (both of which managed by systemd) still run properly (eg, stopping those applications is not a solution).

Test: lsof | grep liblzma.so.5 returns only the liblzma in the path: /usr/lib/x86_64-linux-gnu/liblzma.so.5.2.5

🔗 **Lab Link:** [SadServers - "Tukaani": XZ LZMA Library Compromised](https://sadservers.com/scenario/tukaani)

<br>

## 🪜 Steps

# How We Detected the Problem

We started by checking which processes were using `liblzma`:

```bash
sudo lsof | grep liblzma.so.5
```

We found processes such as:

```text
gotty
sadagent
bash
```

using:

```text
/opt/.trash/liblzma.so.5
```

This was suspicious because the lab told us that the legitimate library was located somewhere else.

### What is `lsof`?

`lsof` means **List Open Files**.

In Linux, shared libraries loaded into a process can appear in `lsof`, so it is useful for finding which processes are currently using a specific library.

---

# First Problem: `/etc/ld.so.preload`

The first major problem was a system-wide preload configuration.

We checked:

```bash
cat /etc/ld.so.preload
```

and found:

```text
/opt/.trash/liblzma.so.5
```

This explained why many unrelated processes were using the malicious library.

## What is `/etc/ld.so.preload`?

Linux uses a **Dynamic Linker/Loader** to load shared libraries required by programs.

Normally, if a program needs:

```text
liblzma.so.5
```

the Dynamic Linker searches the normal library locations and loads the appropriate library.

`/etc/ld.so.preload` is different.

It contains a list of shared libraries that the Dynamic Linker should **load before the normal libraries** for dynamically linked programs.

So if it contains:

```text
/opt/.trash/liblzma.so.5
```

the system effectively says:

> Load this library first whenever a suitable process starts.

That is why programs such as `gotty`, `bash`, and other processes were loading the malicious library.


> `/etc/ld.so.preload` does not perform the general job of loading libraries; the Dynamic Linker does that. `/etc/ld.so.preload` only gives the Dynamic Linker a list of additional libraries to load forcibly in advance. In this lab, that list contained the malicious library, so we removed the file. Normal applications can still load their required libraries through the Dynamic Linker after `/etc/ld.so.preload` is removed.

### Important clarification

`/etc/ld.so.preload` is **not responsible for loading all libraries**.

The **Dynamic Linker** does that normally.

`/etc/ld.so.preload` only provides additional libraries that should be **forced/preloaded**.

Therefore, deleting `/etc/ld.so.preload` does **not** stop Linux from loading required libraries.

After removing the malicious preload entry, programs can still load their normal libraries through the Dynamic Linker.

## How We Fixed It

We removed the malicious preload configuration:

```bash
sudo rm /etc/ld.so.preload
```

We then rebooted the server:

```bash
sudo reboot now
```

### Why did we reboot?

Some processes had already loaded the malicious library into memory.

Deleting `/etc/ld.so.preload` prevents **new processes** from being forced to load it, but it does not magically remove a library that is already loaded inside a running process.

After reboot, all processes start again without the malicious preload configuration.

---

# Second Problem: `webapp` and `jobapp`

After rebooting, we checked again:

```bash
sudo lsof | grep liblzma.so.5
```

We discovered that some applications were still loading a `liblzma` library from a bad location.

This told us that the `/etc/ld.so.preload` problem was not the only way the malicious library was being loaded.

We then inspected the systemd services.

---

## `webapp`

We checked its service configuration:

```bash
systemctl cat webapp
```

We found:

```ini
Environment="LD_LIBRARY_PATH=/opt/.trash/"
```

### What is `LD_LIBRARY_PATH`?

`LD_LIBRARY_PATH` tells the Dynamic Linker:

> Search these directories when looking for shared libraries.

Because it contained:

```text
/opt/.trash/
```

`webapp` could find and load the malicious:

```text
/opt/.trash/liblzma.so.5
```

### Fix

We opened the service file:

```bash
sudo vim /etc/systemd/system/webapp.service
```

and removed:

```ini
Environment="LD_LIBRARY_PATH=/opt/.trash/"
```

Then we reloaded systemd's configuration:

```bash
sudo systemctl daemon-reload
```

and restarted the application:

```bash
sudo systemctl restart webapp
```

---

# `jobapp`

We then inspected:

```bash
systemctl cat jobapp
```

We found:

```ini
EnvironmentFile=/opt/.trash/.jobapp.env
```

We inspected that environment file:

```bash
cat /opt/.trash/.jobapp.env
```

and found:

```text
APP_CONFIG_DB_NAME="jobapp"
APP_CONFIG_USER="dev"
LD_PRELOAD="/opt/.trash/liblzma.so.5"
DB_CONFIG_PRELOAD="true"
```

The important line was:

```text
LD_PRELOAD="/opt/.trash/liblzma.so.5"
```

## What is `LD_PRELOAD`?

`LD_PRELOAD` is an environment variable that tells the Dynamic Linker:

> Load this specific shared library before other libraries.

So `jobapp` was explicitly being forced to load:

```text
/opt/.trash/liblzma.so.5
```

### Fix

We opened the systemd service:

```bash
sudo vim /etc/systemd/system/jobapp.service
```

and removed:

```ini
EnvironmentFile=/opt/.trash/.jobapp.env
```

We did **not** need to delete the `.jobapp.env` file itself. We simply stopped `jobapp` from loading that environment file.

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl restart jobapp
```

---

# The Two Main Problems

There were actually **two different mechanisms** causing the malicious library to be loaded.

### Problem 1 — System-wide preload

```text
/etc/ld.so.preload
        ↓
/opt/.trash/liblzma.so.5
```

This affected many processes across the system.

**Solution:**

```bash
sudo rm /etc/ld.so.preload
sudo reboot now
```

---

### Problem 2 — Application-specific configuration

`webapp`:

```text
LD_LIBRARY_PATH=/opt/.trash/
```

**Solution:** remove the `LD_LIBRARY_PATH` configuration.

`jobapp`:

```text
EnvironmentFile=/opt/.trash/.jobapp.env
        ↓
LD_PRELOAD=/opt/.trash/liblzma.so.5
```

**Solution:** remove the `EnvironmentFile` configuration.

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl restart webapp
sudo systemctl restart jobapp
```
