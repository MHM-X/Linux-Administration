# Lab 35: "Hanoi": Find the Multitasking Users

## Description
The Hanoi office has a Linux server with a large number of user accounts and groups. The system administrators need to identify which users belong to multiple groups for better access management.

Given two files, `users.txt` and `groups.txt`, create a new file `/home/admin/multi-group-users.txt` containing the usernames of users who belong to more than one group, one username per line, sorted alphabetically.

The `users.txt` file contains a list of usernames, one per line. The `groups.txt` file contains group names and their members, in the format `group_name:user1,user2,user3`.

Test: Running md5sum /home/admin/multi-group-users.txt returns dc0ae86caae7125d21df03a0ab29d8ae


🔗 **Lab Link:** [SadServers - "Hanoi": Find the Multitasking Users](https://sadservers.com/scenario/hanoi)

<br>

## 🪜 Steps

```bash
# Extract usernames from groups.txt, split them into separate lines,
# count how many groups each user belongs to, keep users with more than one group,
# sort the result alphabetically, and save it to multi-group-users.txt.
cut -d: -f2 groups.txt | tr ',' '\n' | sort | uniq -c | awk '$1 > 1 {print $2}' | sort > /home/admin/multi-group-users.txt
```
