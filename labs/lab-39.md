# Lab 39: "Kampala": Strange Script Error

## Description

A developer has been working on Linux deployment scripts on their machine and then transferred the files to a Linux server. However, when they try to execute the scripts, they encounter the mysterious error:

-bash: cannot execute: required file not found

The scripts appear to be syntactically correct, but something is preventing them from executing properly. The developer needs your help to identify and fix the issue so the deployment can proceed.

There are several script files in /home/admin/deploy/ that need to be fixed before the deployment process can work correctly.

Test: All script files in /home/admin/deploy/ should execute without the cannot execute: required file not found error.

🔗 **Lab Link:** [SadServers - "Kampala": Strange Script Error](https://sadservers.com/scenario/kampala)

<br>

## 🪜 Steps

## Step 1 — Examine the Files

First, list the scripts and check their permissions:

```bash
# List the deployment scripts and their permissions
ls -la /home/admin/deploy/
```

Then try executing the scripts to reproduce the error:

```bash
# Try executing the deployment scripts to see the exact error
/home/admin/deploy/*.sh
```

If the files exist and have `x` permissions but still produce the error, investigate the file format.

---

## Step 2 — Understand the Shebang

A Bash script normally starts with:

```bash
#!/bin/bash
```

This is called the **shebang**.

It tells Linux which interpreter should be used to execute the script.

For example:

```text
./script.sh
    │
    ▼
#!/bin/bash
    │
    ▼
/bin/bash
    │
    ▼
Execute the script
```

If the shebang contains unexpected hidden characters, Linux may fail to locate the interpreter.

---

## Step 3 — Check the Line Endings

Linux and Windows use different line-ending formats.

### Linux

Linux normally uses:

```text
LF
```

or:

```text
\n
```

### Windows

Windows normally uses:

```text
CRLF
```

or:

```text
\r\n
```

Therefore:

```text
Linux   → LF
Windows → CR + LF
```

The extra `CR` character can cause problems with Bash scripts transferred from Windows to Linux.

---

## Step 4 — Inspect Hidden Characters

Use `cat -A` to display hidden characters:

```bash
# Display the script including hidden characters and line endings
cat -A /home/admin/deploy/deploy.sh
```

A normal Linux file may look like:

```text
#!/bin/bash$
echo "Hello"$
```

The `$` is **not part of the file**. It is displayed by `cat -A` to indicate the end of a line.

A Windows-style file may look like:

```text
#!/bin/bash^M$
echo "Hello"^M$
```

Here:

```text
^M = Carriage Return (CR)
$  = End of line displayed by cat -A
```

So:

```text
Linux:
#!/bin/bash[LF]

Windows:
#!/bin/bash[CR][LF]
```

---

## Step 5 — Confirm the File Format

The `file` command can also detect Windows line endings:

```bash
# Check the file type and line-ending format
file /home/admin/deploy/deploy.sh
```

If the file contains Windows line endings, the output may indicate:

```text
with CRLF line terminators
```

---

## Root Cause

The deployment scripts were created or modified using **Windows-style CRLF line endings** instead of Linux-style LF line endings.

This causes the shebang:

```bash
#!/bin/bash
```

to effectively contain an extra carriage return character.

Linux may therefore interpret the interpreter path incorrectly, resulting in:

```text
cannot execute: required file not found
```

The problem is not necessarily that the script itself is missing.

---

## Step 6 — Fix the Line Endings

### Method 1 — Using `dos2unix`

If `dos2unix` is available:

```bash
# Convert Windows CRLF line endings to Unix LF line endings
dos2unix /home/admin/deploy/*.sh
```

This removes the Windows carriage return characters.

---

### Method 2 — Using `sed`

If `dos2unix` is not available, use `sed`:

```bash
# Remove carriage return characters from the end of every line
sed -i 's/\r$//' /home/admin/deploy/*.sh
```

The command removes `\r` from the end of each line and converts the files from:

```text
CRLF
```

to:

```text
LF
```

---

## Step 7 — Verify the Fix

Check the hidden characters again:

```bash
# Verify that the carriage return characters are gone
cat -A /home/admin/deploy/deploy.sh
```

The `^M` characters should no longer appear.

Then execute the scripts again:

```bash
# Execute the deployment scripts after fixing the line endings
/home/admin/deploy/*.sh
```

The scripts should now execute normally.

---

## Key Lessons

* Linux normally uses **LF (`\n`)** for line endings.
* Windows normally uses **CRLF (`\r\n`)**.
* `^M` represents a hidden **Carriage Return (CR)** character.
* `$` shown by `cat -A` represents the **end of a line** and is not part of the script.
* The **shebang** (`#!/bin/bash`) tells Linux which interpreter should execute the script.
* Windows line endings can corrupt the shebang and prevent a script from executing.
* `cat -A` is useful for inspecting hidden characters.
* `file` can help identify CRLF line endings.
* `dos2unix` converts Windows line endings to Unix line endings.
* `sed -i 's/\r$//'` can remove carriage returns without installing additional tools.

## Final Concept

```text
Windows
   │
   │ CRLF
   ▼
Deployment Script
   │
   │ transferred to Linux
   ▼
Linux
   │
   ▼
#!/bin/bash^M
   │
   ▼
Incorrect interpreter path
   │
   ▼
❌ cannot execute: required file not found

Fix:
CRLF → LF
   │
   ▼
#!/bin/bash
   │
   ▼
/bin/bash
   │
   ▼
✅ Script executes successfully
```

```
```
