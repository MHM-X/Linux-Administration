# Lab 10: "Yokohama": Linux Users Working Together

## Description
There are four Linux users working together in a project in this server: abe, betty, carlos, debora.

First, they have asked you as the sysadmin, to make it so each of these four users can read the project files of the other users in the /home/admin/shared directory, but none of them can modify a file that belongs to another user. Users should be able modify their own files.

Secondly, they have asked you to modify the file shared/ALL so that any of these four users can write more content to it, but previous (existing) content cannot be altered.

clues:
1. Check the project files permissions with ls -l sharedYou can check the user experience (reading or writing filed) with sudo su - abe

2. For the first part, users need read access to the project files of the other users: sudo chmod 0644 shared/project_*

3. For the second part, we can create a Linux group for the four users, and then make the ALL file owned by the new group

4. As per the previous clue, do: sudo groupadd project; sudo usermod -aG project abe; sudo usermod -aG project betty; sudo usermod -aG project carlos; sudo usermod -aG project debora; sudo chown root:project /home/admin/shared/ALL and test with a user writing to the ALL file

5. Make the ALL file writeable by the group: sudo chmod 0664 /home/admin/shared/ALL

(Next Clue will give the last command needed for the solution).

6. Make the ALL file append-only: sudo chattr +a /home/admin/shared/ALL

🔗 **Lab Link:** [SadServers - "Yokohama": Linux Users Working Together](https://sadservers.com/scenario/yokohama)

<br>

## 🪜 Steps

### Step 1: Thinking
- We need to give all users the permission to read `/home/admin/shared`. But only the owners can modify their own files.
- All four users can add new content to the file `/home/admin/shared/ALL`, but they can't edit the existing content.

### Step 2: Let's first view the current files list

```bash
ls -la /home/admin/shared
```

<img width="682" height="206" alt="image" src="https://github.com/user-attachments/assets/e6f049c0-6434-4db8-b871-8e33e4e45e00" />

### Step 3: We will create a group, the members of it are the 4 users, then set all the project files with this new group and give the read permission for this group

```bash
sudo groupadd project
```

```bash
sudo usermod -aG project abe
sudo usermod -aG project betty
sudo usermod -aG project carlos
sudo usermod -aG project debora
```

```bash
sudo chgrp project /home/admin/shared/project_*
```

```bash
sudo chmod g+r /home/admin/shared/project_*
```

### Step 4:
