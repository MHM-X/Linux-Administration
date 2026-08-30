# Lab 13: "Nuuk": More SSH Troubles

## Description
SSH seems broken in this server. The user admin has an id_ed25519 SSH key pair in their ~/.ssh directory with the public key in ~/.ssh/authorized_keys but ssh 127.0.0.1 won't work.
Test: You can ssh locally, i.e. ssh admin@127.0.0.1 works.

🔗 **Lab Link:** [SadServers - "Nuuk": More SSH Troubles](https://sadservers.com/scenario/nuuk)

<br>

## 🪜 Steps

### Step 1: Insuring that ssh service is working

```bash
sudo systemctl status ssh
```

<img width="1135" height="472" alt="image" src="https://github.com/user-attachments/assets/c5cc61f8-2625-4936-9c0a-53e42fe802ec" />

### Step 2: Insuring that ssh service is working on the port 22

```bash
sudo ss -tlnp | grep ':22'
```

<img width="1007" height="68" alt="image" src="https://github.com/user-attachments/assets/91f6c7a8-274b-4c81-aa88-0d4224827d66" />

### Step 3: Trying to connect to know what's wrong

```bash
ssh admin@127.0.0.1
```

<img width="1175" height="155" alt="image" src="https://github.com/user-attachments/assets/951e324e-94d1-4a22-9080-de4bd79875b0" />

> The important line is: `hostkeys_find_by_key_hostfile: hostkeys_foreach failed for /home/admin/.ssh/known_hosts: Permission denied`
It means that SSH is trying to reach: `/home/admin/.ssh/known_hosts` But he doesn't have the authority to read/write on it.


### Step 3: Configuring the permissions

```bash
sudo ls -ld /home/admin/.ssh
```

```bash
chmod 700 /home/admin/.ssh
```

<img width="883" height="113" alt="image" src="https://github.com/user-attachments/assets/99ef86f6-bd43-49e4-8bdc-ced83598dbe1" />

### Step 4: Testing

```bash
ssh admin@127.0.0.1
```

<img width="1177" height="370" alt="image" src="https://github.com/user-attachments/assets/f9618751-e42a-40d2-86d4-c6a6d7d95ecf" />
