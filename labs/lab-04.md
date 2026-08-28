# Lab 04: "Taipei": Come a-knocking

## Description
There is a web server on port :80 protected with Port Knocking [Port Knocking](https://en.wikipedia.org/wiki/Port_knocking). Find the one "knock" needed (sending a SYN to a single port, not a sequence) so you can curl localhost.

🔗 **Lab Link:** [SadServers - "Taipei": Come a-knocking](https://sadservers.com/scenario/taipei)

<br>

## 🪜 Steps

### Step 1: What is Port Knocking?

**Port Knocking** is a security technique where a server keeps certain network ports closed or inaccessible and opens access to a protected service only after receiving a specific sequence of connection attempts, or "knocks," on predefined ports.

The main purpose of **Port Knocking** is to hide or protect services from unauthorized access, making a port appear closed until the correct knocking sequence is received.

as you see in the picture the port 80 isn't responding:-

<img width="822" height="47" alt="Screenshot 2026-08-28 153618" src="https://github.com/user-attachments/assets/9e979fdb-52eb-4641-b957-d99125d69f7e" />




Let's check the status of the ports currently available:

```bash
ss -tln

# ss = Socket Statistics
# -t = TCP
# -l = Listening
# -n = Numeric
```

<img width="1175" height="212" alt="Screenshot 2026-08-28 153503" src="https://github.com/user-attachments/assets/c5bfd383-2e98-4c11-a23d-aaf72fd0f713" />

### Step 2: The Solution 

#### using a simple bash script

```bash
#!/bin/bash

# Loop through all ports from 1 to 65535
for port in {1..65535}; do

    # Send a knock to the current port
    knock localhost "$port"

done
```

#### using nmap

```bash
nmap -p- localhost

# nmap      = Network Mapper, a tool used to scan networks and ports
# -p-       = Scan all TCP ports from 1 to 65535
# localhost = Scan the local machine (127.0.0.1)
```

<img width="787" height="307" alt="Screenshot 2026-08-28 155540" src="https://github.com/user-attachments/assets/ef8cdca1-963b-4563-b34a-fdedfdb8e535" />

#### Verifying Our solution
```bash
echo $(curl localhost)| md5sum
```

<img width="941" height="162" alt="Screenshot 2026-08-28 155808" src="https://github.com/user-attachments/assets/106cbe02-3bc8-458d-b9c0-b1ac9cfb9bdc" />

