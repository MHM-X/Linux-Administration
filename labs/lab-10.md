# Lab 10: "Yokohama": Linux Users Working Together

## Description
There are four Linux users working together in a project in this server: abe, betty, carlos, debora.

First, they have asked you as the sysadmin, to make it so each of these four users can read the project files of the other users in the /home/admin/shared directory, but none of them can modify a file that belongs to another user. Users should be able modify their own files.

Secondly, they have asked you to modify the file shared/ALL so that any of these four users can write more content to it, but previous (existing) content cannot be altered.

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
