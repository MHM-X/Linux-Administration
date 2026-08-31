# Lab 14: "Cairo": Time for a Timer

## Description
A critical health check script at /opt/scripts/health.sh is supposed to run every 10 seconds. This check is triggered by a systemd timer.
The script's job is to check the local Nginx server and write its status (e.g., "STATUS: OK") to the log file at /var/log/health.log.
The log file is not being updated, and it appears the health check is failing.

Find out why the health check system is broken and fix it. The check will pass once the /var/log/health.log file is being correctly updated by the timer with a STATUS: OK message.

🔗 **Lab Link:** [SadServers - "Cairo": Time for a Timer](https://sadservers.com/scenario/cairo)

<br>

## 🪜 Steps

### Step 1: Find the timer

```bash
systemctl list-timers --all
```

<img width="1181" height="247" alt="Screenshot 2026-08-30 234920" src="https://github.com/user-attachments/assets/004c020d-9ca6-4805-b1bd-cde60b115aa2" />

>Notice that the list of timers doesn't include: health.timer Even though the lab description says there's a systemd timer responsible for running health.sh.
This means there are two possibilities: 1- The timer has a different name than health.timer. -2- Or the timer isn't loaded/doesn't exist in the current systemd list.

```bash
sudo find /etc/systemd /usr/lib/systemd -name '*health*.timer' 2>/dev/null
```

<img width="1156" height="50" alt="Screenshot 2026-08-30 235031" src="https://github.com/user-attachments/assets/06edf0ac-71a7-4e70-aef1-00059b9d60e4" />

>The file exists: `/etc/systemd/system/health.timer` But systemctl list-timers --all didn't show it, which suggests that the timer isn't loaded/active right now.
But systemctl list-timers --all didn't show it, which suggests that the timer isn't loaded/active right now.
Now we want to check its status:

```bash
sudo systemctl status health.timer
```

<img width="936" height="160" alt="Screenshot 2026-08-30 235129" src="https://github.com/user-attachments/assets/53be4c0b-1aa3-4bf5-903d-167a03dc2f77" />

```bash
# to start the timer:
sudo systemctl start health.timer

# And to make it run automatically after the server restarts too:
sudo systemctl enable health.timer

# And you can do both at the same time with a single command:
sudo systemctl enable --now health.timer
```

<img width="1176" height="82" alt="Screenshot 2026-08-30 235403" src="https://github.com/user-attachments/assets/a98e0a21-a652-4351-8368-f3af023a1ae0" />

- Ensuring that it's now working

```bash
systemctl list-timers --all
sudo systemctl status health.timer
```
<img width="1181" height="325" alt="Screenshot 2026-08-30 235414" src="https://github.com/user-attachments/assets/8b10fb37-a34e-48a3-b094-8f70ecd00f1a" />
<img width="1172" height="235" alt="Screenshot 2026-08-30 235444" src="https://github.com/user-attachments/assets/7022e593-1373-4061-92f8-aa9bf31d1803" />

---
>But when we check our solution, we find that it still don't write `ok` on `/var/log/health.log`

<img width="850" height="160" alt="Screenshot 2026-08-30 235627" src="https://github.com/user-attachments/assets/216c5ff3-31ce-46bf-971a-dbcef6113f12" />

---

### There is still another problem with the firewall

 Check the iptables rules with: sudo iptables -L -n , this lists the rules in the default filter table (there are other tables that won't list their rules with this command).

```bash
sudo iptables -L -n
```

<img width="1172" height="817" alt="Screenshot 2026-08-31 000406" src="https://github.com/user-attachments/assets/4407d226-4053-45a4-945f-f51e03315ef6" />

The command shows a DROP policy for 127.0.0.1 when the destination is port tcp:80 , easier to see with: sudo iptables -L OUTPUT -n --line-numbers

```bash
 sudo iptables -L OUTPUT -n --line-numbers
```

<img width="1167" height="187" alt="image" src="https://github.com/user-attachments/assets/748c44aa-0559-4716-8784-3a79e653b241" />

>`1    DROP    tcp    --    0.0.0.0/0    127.0.0.1    tcp dpt:80` means:
If there's any outgoing TCP connection from this server, from any source, going to 127.0.0.1 on port 80, drop it.

Delete the IPv4 rule by line number: sudo iptables -D OUTPUT 1. Although we are not going to reboot this instance for this exercise, it's good practice to save the iptable state with sudo sh -c 'iptables-save > /etc/iptables/rules.v4' (you can do the same with ip6tables).

```bash
sudo iptables -D OUTPUT 1
```

<img width="1157" height="117" alt="image" src="https://github.com/user-attachments/assets/406cce46-eea2-425d-9087-e85aefed886a" />

Although we are not going to reboot this instance for this exercise, it's good practice to save the iptable state with sudo sh -c 'iptables-save > /etc/iptables/rules.v4' (you can do the same with ip6tables).

---

### Now we solved the two problems and when we run `cat var/log/health.log` we will see STATUS: OK

<img width="780" height="855" alt="Screenshot 2026-08-31 000633" src="https://github.com/user-attachments/assets/41a1a753-17c3-401d-8419-a85c0f5ffef3" />
