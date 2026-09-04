# Lab 41: "Podgorica": Docker to Podman migration

## Description

You have been tasked with migrating this future web server from using Docker (which uses a daemon) to rootless Podman.
There is already an Nginx Podman image on the server, and your objective is to manage the container created from it using systemd, so the it starts automatically on reboot and continues running unless explicity stopped (the same behaviour expected from a Docker-managed container).
Create a systemd service named container-nginx.service that manages the Podman Nginx container. Enable and start this service.

NOTES: Although a quadlet file solution should be valid, the check script is still not accounting for it.

There is no need to reboot the VM, although if you want you could reboot it from the command line with /sbin/shutdown -r now and refresh or reopen the web console.

Test: The checker script will test if the container-nginx.service is active and enabled, and if it can stop and start the service. It will also verify that curl localhost:8888 returns the default "Welcome to nginx" web page.

🔗 **Lab Link:** [SadServers - "Podgorica": Docker to Podman migration](https://sadservers.com/scenario/podgorica)

<br>

## 🪜 Steps

### Step 1: 

```bash
```
