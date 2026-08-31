# Lab 19: "Bologna": counting ELB 5xx errors

## Description
Operations handed you a classic AWS Elastic Load Balancer access log at /home/admin/elb.log. Each line is one request. Fields are space-separated; the quoted HTTP request starts at field 12, so the numeric fields before it are fixed-width columns.

Field 8 is the ELB status code and field 9 is the backend status code returned by the target instance. Count how many log lines have a backend status code in the 5xx range (500 through 599). Write that integer — digits only — to /home/admin/solution.txt. For example: echo 42 > ~/solution.txt

The log mixes successful responses, redirects, client errors, and server errors; only backend 5xx responses count toward your answer.

Test: The MD5 checksum of your answer file md5sum /home/admin/solution.txt is b73ce398c39f506af761d2277d853a92 (we also accept the correct count with a trailing newline in the file).

🔗 **Lab Link:** [SadServers - "Bologna": counting ELB 5xx errors](https://sadservers.com/scenario/bologna)

<br>

## 🪜 Steps

```bash
awk '$9 >= 500 && $9 <= 599 {count++} END {print count}' /home/admin/elb.log > /home/admin/solution.txt
```
