# Lab 01: "Saint John" — What Is Writing to This Log File?

## Description
A developer created a testing program that is continuously writing to a log file /var/log/bad.log and filling up disk. You can check for example with tail -f /var/log/bad.log.
This program is no longer needed. Find it and terminate it. Do not delete the log file.

🔗 **Lab Link:** [SadServers - "Saint John"](https://sadservers.com/scenario/saint-john)

<br>

## 🪜 Steps

### Step 1: you can view that there's a process that contentiously write to /var/log/bad.log via:

```bash
tail -f /var/log/messages
```

### Step 2: find the process id that writes to /var/log/bad.log

```bash
# lsof: list open files

lsof /var/log/bad.log
```

### Step 3: Identify the process id and kill it

```bash
kill -9 425
```
