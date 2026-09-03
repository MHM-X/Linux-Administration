# Lab 37: "Budapest": User Creation

## Description

🔗 **Lab Link:** [SadServers - "Budapest": User Creation](https://sadservers.com/scenario/budapest)

<br>

## 🪜 Steps

### using the script:

```bash
#!/bin/bash

while IFS=';' read -r username password
do
    # Create the user
    useradd "$username"

    # Set the user's password
    echo "$username:$password" | chpasswd

done < user_list.txt
```
