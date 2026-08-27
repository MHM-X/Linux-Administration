# Lab 01: "Saint John" — What Is Writing to This Log File?

## 📄 Description
A process is continuously appending unwanted entries to `/var/log/messages` (or a specific log file), and the goal is to identify **which process** is responsible for writing to it, then stop it. This lab tests your ability to trace file writers using process and file-descriptor inspection tools.

🔗 **Lab Link:** [SadServers - "Saint John"](https://sadservers.com/scenarios/linux-server-log)

---

## 🪜 Steps

### Step 1: Confirm the log file is actively growing
Before hunting for the culprit, verify the file is really being written to in real time. Running `tail -f` lets you watch new lines appear live, confirming the problem is ongoing and not a one-time event.

![Watching the log file grow in real time](images/lab-01/step1-tail-log.png)

```bash
tail -f /var/log/messages
```

### Step 2: Find which process has the file open
Use `lsof` to list every process that currently holds the log file open. The PID column tells you exactly who's writing to it.

```bash
lsof | grep /var/log/messages
```

### Step 3: Identify and inspect the process
Once you have the PID from the previous step, look it up to see what the process actually is (its command, parent process, and how it started) — for example with `ps -p <PID> -f`. This step is often just investigation and reasoning, so no command or image is required if the PID/process is self-explanatory from the previous step.

---

## ✅ Solution
Summarize the root cause here (which process/service was writing to the file) and the fix or answer required by the lab (e.g., stopping the service, redirecting its output, or renaming the culprit as required by the scenario).
