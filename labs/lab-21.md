# Lab 21: "Bergen": Port already in use

## Description
There's an application at /home/admin/standalone that needs to run successfully but currently it fails.

Fix the environment so the binary can run without errors, without changing the binary itself, and without breaking the web app served on port :80.

Test: Running /home/admin/standalone prints OK and curl http://localhost:80 returns hello SadServers.


🔗 **Lab Link:** [SadServers - "Bergen": Port already in use](https://sadservers.com/scenario/bergen)

<br>

## 🪜 Steps

### Step 1: What's the problem?

```bash
/home/admin/standalone
```

<img width="698" height="43" alt="image" src="https://github.com/user-attachments/assets/38de2d69-d5dd-42d6-b31b-cf20a148f4c9" />

### Step 2: What's the service op port 80
```bash
sudo ss -lntp | grep ':8000'

# sudo  → Run the command with root privileges
# ss    → Display network socket and connection information
# -l    → Show only listening sockets
# -n    → Show IP addresses and port numbers numerically
# -t    → Show TCP sockets only
# -p    → Show the process using the socket
# |     → Pass the output of ss to the next command
# grep  → Search for matching text
# ':8000' → Search specifically for port 8000
```

<img width="1123" height="66" alt="image" src="https://github.com/user-attachments/assets/28b34189-3f07-44b9-86e3-deb5b4f22221" />


```bash
ps -fp 852

# ps   → Display information about running processes
# -f   → Show full process information
# -p   → Select a specific process by its PID
# 850  → The PID of the process we want to inspect
```
>It's a Django server
<img width="1171" height="117" alt="image" src="https://github.com/user-attachments/assets/c8cda733-c961-4d06-9c83-178534463c2b" />

```bash
sudo nginx -T 2>/dev/null | grep -n '8000'
```

<img width="897" height="85" alt="image" src="https://github.com/user-attachments/assets/55991e9b-3372-4565-a865-64fe87ce8cf8" />
>So, now we found that the problem is that the Nginx server runs as a Reverse proxy and redirects the traffic to the Django server.
>But, the Django server also listening to port 80, we need to change it to another port like: `8001` cuz the Nginx is listening to it then, changing the Nginx settings to make it redirect the traffic to `8001`

### Step 3: Applying the edits

```bash
sudo systemctl edit --full django.service
```

<img width="1168" height="870" alt="image" src="https://github.com/user-attachments/assets/821494bd-5242-44e0-a3b2-40dcee54d43f" />
>Change `ExecStart=/usr/bin/python3 manage.py runserver 0.0.0.0:8000` to `ExecStart=/usr/bin/python3 manage.py runserver 0.0.0.0:8001`

```bash
sudo vim /etc/nginx/sites-enabled/bergen
```
<img width="1012" height="325" alt="image" src="https://github.com/user-attachments/assets/65c635c1-2958-4315-92d7-3ca2ea81cbf8" />
>Change the link from 8000 to 8001

### Step 4: Restarting

```bash
sudo systemctl daemon-reload
sudo systemctl restart django
sudo systemctl reload nginx
```

<img width="707" height="106" alt="image" src="https://github.com/user-attachments/assets/019bddfd-55b6-4662-958b-2565225a04c7" />
