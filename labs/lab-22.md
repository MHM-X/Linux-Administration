# Lab 22: "Nara": No ls on this system

## Description
Common directory-listing tools on this host are missing or refuse to run. A document named shosoin.tag was misplaced somewhere under /home/admin/records.

Write its full absolute path (one line) to /home/admin/solution.txt, for example: echo "/home/admin/records/some/dir/shosoin.tag" > /home/admin/solution.txt

NOTE: There are at least 9 different ways to find the solution in this server (shown in the clues).

Test: md5sum /home/admin/solution.txt returns 8d3b739ebccb41c7c39e608d7a3e0bd6 (the solution without the trailing newline is also accepted).


🔗 **Lab Link:** [SadServers - "Nara": No ls on this system](https://sadservers.com/scenario/nara)

<br>

## 🪜 Steps

>Since `ls` and `find` were unavailable, we used alternative ways to search recursively for `shosoin.tag` under `/home/admin/records`. First, Bash globbing with `echo *` or `printf '%s\n' *` can list the contents of the current directory, allowing us to move into subdirectories manually. Second, enabling `globstar` with `shopt -s globstar` allows `**` to recursively match files and directories at any depth. Third, commands such as `tree`, `du -a`, `tar`, and `rsync` can also walk through the directory tree and reveal the file path.
