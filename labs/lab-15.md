# Lab 15: "Alexandria": The Vanishing Backups

## Description
A critical backup cron job has silently stopped working 3 days ago. The backup script is located at /opt/backup/backup.sh and should create daily backups in /var/backups/daily/, but no new backups have been created recently.

Looking at the backup directory, you can see old backup files from a few days ago, proving the system used to work. However, there are no error emails, no obvious error logs, and the cron service appears to be running normally.

Fix ALL issues preventing the backups from running, so that backups are created successfully and reliably.

Test directory: /var/backups/daily/
Backup script: /opt/backup/backup.sh

🔗 **Lab Link:** [SadServers - "Alexandria": The Vanishing Backups](https://sadservers.com/scenario/alexandria)

<br>

## 🪜 Steps

### Step 1: checking if the cron service is running

```bash
sudo cat /etc/crontab
```

<img width="1141" height="572" alt="image" src="https://github.com/user-attachments/assets/7ed4ca97-3012-46ff-8457-1e3cd52cb3d1" />

```bash
sudo systemctl status cron
```

<img width="1182" height="457" alt="image" src="https://github.com/user-attachments/assets/56788da7-9034-4f53-855f-ca4c36178885" />
>At 13:45:01, cron ran /opt/backup/old_backup.sh as the root user, and all output and errors were ignored.
>This means there’s a Cron job running, but it’s running: `/opt/backup/old_backup.sh` While the laptop description says the correct script is: `/opt/backup/backup.sh` So we have strong suspicion that the cron job is pointing to an >old/wrong script.

### Step 2: After we found out that there’s a file we suspect is running the cron and it’s old, and we want to put the new one, the lab said this is the correct backup script: /opt/backup/backup.sh

First, we specify the location of the cron entry:

```bash
sudo grep -R "/opt/backup/old_backup.sh" /etc/cron.d /etc/crontab /var/spool/cron/crontabs 2>/dev/null
```

<img width="1170" height="118" alt="image" src="https://github.com/user-attachments/assets/5cbae2b8-37e6-4315-a5cc-12ac2f072681" />

