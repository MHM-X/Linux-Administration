# Lab 06: "Bata": Find in /proc

## Description
A spy has left a password in a file in /proc/sys . The contents of the file start with "secret:" (without the quotes).

Find the file and save the word after "secret:" to the file /home/admin/secret.txt with a newline at the end (e.g. if the file contents were "secret:password" do: echo "password" > /home/admin/secret.txt).

🔗 **Lab Link:** [SadServers -  "Bata": Find in /proc](https://sadservers.com/scenario/bata)

<br>

## 🪜 Steps

### Step 1: searching for the secret using grep

```bash
grep -RIs '^secret:' /proc/sys 2>/dev/null
```

### Step 2: using a grep command to solve the lab

```bash
grep -RIs '^secret:' /proc/sys 2>/dev/null | cut -d: -f3 > /home/admin/secret.txt

# -d: → اجعل : هو delimiter
# -f3 →خذ الحقل الثالث
# -R → recursive
# -I → ignore binary
# -s → suppress error messages
```

Or:

```bash
grep -RIs '^secret:' /proc/sys 2>/dev/null | awk -F: '{print $3}' > /home/admin/secret.txt
```

Or:

```bash
# find /proc/sys -type f -exec grep -El '^secret:' {} 2>/dev/null \;

# find finds the files → -exec gives each file to grep

# find        = Search for files and directories
# /proc/sys   = The directory where the search starts
# -type f     = Search for regular files only
#
# -exec       = Execute a command for each file found by find
# grep -E     = Use extended regular expressions
# -l          = Print only the names of files that contain a match
# '^secret:'  = Match lines that start with "secret:"
# {}          = Placeholder for the current file found by find
# 2>/dev/null = Hide error messages
# \;          = Marks the end of the -exec command
```

Or:

```bash
# find /proc/sys -type f -print0 | xargs -0 grep -El '^secret:' 2>/dev/null

# find finds the files → xargs collects them → grep checks them

# -print0     = Print file names separated by a NULL character
# |           = Pipe the output of find to the next command
# xargs       = Takes the input and passes it as arguments to another command
# -0          = Tell xargs that the input is NULL-separated
# grep -E     = Use extended regular expressions
# -l          = Print only the names of matching files
# '^secret:'  = Match lines that start with "secret:"
# 2>/dev/null = Hide error messages
```
