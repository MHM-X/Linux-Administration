# Lab 09: "Tokamachi": Troubleshooting a Named Pipe

## Description
There's a process reading from the named pipe /home/admin/namedpipe.

If you run this command that writes to that pipe:

/bin/bash -c 'while true; do echo "this is a test message being sent to the pipe" > /home/admin/namedpipe; done' &

And check the reader log with tail -f reader.log

You'll see that after a minute or so it works for a while (the reader receives some messages) and then it stops working (no more received messages are printed to the reader log or it takes a long time to process one). Troubleshoot and fix (for example changing the writer command) so that the writer keeps sending the messages and the reader is able to read all of them.

🔗 **Lab Link:** [SadServers - "Tokamachi": Troubleshooting a Named Pipe](https://sadservers.com/scenario/tokamachi)

<br>

## 🪜 Steps

### Step 1: Thinking

The problem is that the writer sends messages to the named pipe much faster than the reader can read them. The pipe buffer eventually fills up, causing the writer to block and the reader to become slow or stop receiving messages.

The solution is to add a delay between writes so the writer does not overwhelm the reader.

Look at the reader process command — how often does it read from the pipe? Compare that to how often the writer sends. What happens to the named pipe's buffer when the writer is much faster than the reader?

What should we do?

Kill the writer process you started (kill %1 if it's your only background job, or for example: kill $(pgrep -f namedpipe)). Re-launch it with a delay between writes longer than the reader's 2s interval; for example sleep 3:
/bin/bash -c 'while true; do echo "this is a test message being sent to the pipe" > /home/admin/namedpipe; sleep 3; done' &

### Step 2: Starting the writer

```bash
/bin/bash -c 'while true; do echo "this is a test message being sent to the pipe" > /home/admin/namedpipe; done' &
```

<img width="1177" height="70" alt="Screenshot 2026-08-29 181046" src="https://github.com/user-attachments/assets/b5c78c34-4241-4fe9-9695-a7f571a79457" />

`[1] 817` is the process id for this script

### Step 2: View what happens!

```bash
tail -f reader.log
```

<img width="946" height="137" alt="Screenshot 2026-08-29 180128" src="https://github.com/user-attachments/assets/3fff4981-ccfb-4d0c-b31e-dd2f355cc3e6" />

### Step 3: View the process that we started and killing it.

```bash
ps aux | grep -E 'namedpipe|reader'
```

<img width="1177" height="140" alt="Screenshot 2026-08-29 181143" src="https://github.com/user-attachments/assets/23fa62a4-0c0a-43b8-a9fc-2b0a58929e4b" />


```bash
kill 817
```
