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

### Step 1: 

```bash
```
