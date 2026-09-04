# Lab 43: "Stockholm": DNS health check issue

## Description
 The internal status portal on this host should answer on http://127.0.0.1:9167/ with a body containing OK.

It worked until operations ran a package cleanup.

The portal service (stockholm-portal) only runs after a DNS health check at /usr/local/bin/stockholm-dns-check.sh succeeds.

Make the necessary changes so the portal works again.

Do not modify /usr/local/bin/stockholm-dns-check.sh.

Test: The health script /usr/local/bin/stockholm-dns-check.sh runs successfully, stockholm-portal is active, and curl http://127.0.0.1:9167/ returns a response whose body contains OK.

🔗 **Lab Link:** [SadServers - "Stockholm": DNS health check issue](https://sadservers.com/scenario/stockholm)

<br>

## 🪜 Steps

