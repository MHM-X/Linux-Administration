# Lab 02: "Saskatoon": counting IPs

## Description
There's a web server access log file at /home/admin/access.log. The file consists of one line per HTTP request, with the requester's IP address at the beginning of each line (first column).

Find what's the IP address that has the most requests in this file (there's no tie; the IP is unique). Write the solution into a file /home/admin/highestip.txt. For example, if your solution is "1.2.3.4", you can do echo "1.2.3.4" > /home/admin/highestip.txt

NOTE: The solution IP shows 482 times, ie grep -c -F -f highestip.txt access.log returns 482, if your solution has a different (lower) number you got the wrong most common IP.

🔗 **Lab Link:** ["Saskatoon": counting IPs](https://sadservers.com/scenario/saskatoon)

<br>

## 🪜 Steps

### Step 1: Using awk, we counted the number of occurrences for each IP address. Then, we sorted the results in descending order, selected the IP address with the highest number of requests, and saved it to /home/admin/highestip.txt.

```bash
awk '{count[$1]++} END {for (ip in count) printf "%d %s\n", count[ip], ip}' /home/admin/access.log | sort -nr | head -1 | awk '{printf "%s\n", $2}' > /home/admin/highestip.txt
```
