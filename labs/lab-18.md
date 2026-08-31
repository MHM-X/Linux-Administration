# Lab 18: "Porto": Port audit without net tools

## Description
The security team removed common network recon utilities from this host. Your job is to determine which TCP ports on localhost (127.0.0.1) are accepting connections.

The ports to check are listed in /home/admin/ports-to-scan.txt (one port per line).

Write your results to /home/admin/port-audit.txt with one line per port, sorted by port number (ascending), using this format:


PORT STATUS
where STATUS is exactly open or closed (lowercase).

A template file /home/admin/port-audit.txt is available with values per port "open|closed", delete the separator and the incorrect value per port or re-create the file.

The following are not available on this system (removed or restricted): ss, netstat, nmap, nc, telnet, curl, wget, lsof, tcpdump, openssl, fuser.

NOTE: you don't have root (superuser) access.
Root (sudo) Access: False

Test: The file /home/admin/port-audit.txt exists and correctly reports whether each port in /home/admin/ports-to-scan.txt is open or closed on 127.0.0.1.


🔗 **Lab Link:** [SadServers - "Porto": Port audit without net tools](https://sadservers.com/scenario/porto)

<br>

## 🪜 Steps

### Step 1: 

```bash
```
