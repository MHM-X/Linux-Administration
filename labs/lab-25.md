# Lab 25: "Oaxaca": Close an Open File

## Description
The file /home/admin/somefile is open for writing by some process. Close this file without killing the process.
Test: lsof /home/admin/somefile returns nothing.

🔗 **Lab Link:** [SadServers - "Oaxaca": Close an Open File](https://sadservers.com/scenario/oaxaca)

<br>

## 🪜 Steps

```bash
# 1. Find which process has the file open
lsof /home/admin/somefile

# 2. Find the current shell's file descriptors
ls -la /proc/$$/fd/

# 3. Close FD 77
exec 77>&-

exec
# Modify the current shell's file descriptors

77
# File descriptor number

>&
# Redirect an output file descriptor

-
# Close the file descriptor

# 4. Verify
lsof /home/admin/somefile
```

<img width="1167" height="382" alt="image" src="https://github.com/user-attachments/assets/89c7b305-0c72-47c9-9105-b56d6416a33c" />
