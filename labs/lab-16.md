# Lab 16:  "Kortenberg": Can't touch this!

## Description
Is "All I want for Christmas is you" already everywhere?. A bit unrelated, someone messed up the permissions in this server, the admin user can't list new directories and can't write into new files. Fix the issue.
NOTE: Besides solving the problem in your current admin shell session, you need to fix it permanently, as in a new login shell for user "admin" (like the one initiated by the scenario checker) should have the problem fixed as well.

🔗 **Lab Link:** [SadServers - "Kortenberg": Can't touch this!](https://sadservers.com/scenario/kortenberg)

<br>

## 🪜 Steps

### Step 1: Create a file so we could know what the default permissions are
```bash
touch test.txt
```

```bash
ls -l test.txt
```

<img width="566" height="112" alt="Screenshot 2026-08-31 185534" src="https://github.com/user-attachments/assets/fad6b6e6-d36e-4f38-88d5-82700ca5750e" />

Apparently there's a problem with the umask

```bash
umask
```

<img width="516" height="52" alt="Screenshot 2026-08-31 185705" src="https://github.com/user-attachments/assets/8d71bf66-851d-4ac6-ab62-6b45138f0f64" />


We should change it to 0222

```bash
umask 022

# then:
umask
```

<img width="487" height="107" alt="Screenshot 2026-08-31 185743" src="https://github.com/user-attachments/assets/7ff7be20-2e72-48b0-ab56-78c56d1035ee" />


But we need to keep it permanent when reboot so will add it to the profile file `nano ~/.profile` then at the last of the file add `umask 022`

```bash
nano ~/.profile
```

<img width="1175" height="602" alt="Screenshot 2026-08-31 190258" src="https://github.com/user-attachments/assets/42aedc67-7e7d-490e-a0bd-8f40f1276a69" />

>but we found a note: `notice: # the default umask is set in /etc/profile`

So, we just need to change the previous setting that was made in `/etc/profile`

```bash
sudo nano /etc/profile
```

<img width="1173" height="880" alt="Screenshot 2026-08-31 190543" src="https://github.com/user-attachments/assets/1a51dbbd-50d7-44eb-8cd8-0faf264a48b6" />

>just change it from 077 to 022
