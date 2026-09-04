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

## Architecture

```text
systemd (user)
      |
      v
container-nginx.service
      |
      v
    Podman
      |
      v
Nginx Container
      |
      v
   Nginx :80
```

Port mapping:

```text
localhost:8888
      |
      v
Container :80
      |
      v
    Nginx
```

---

## Step 1: Create and Start the Nginx Container

```bash
# Create and run the Nginx container in the background.
# Map host port 8888 to container port 80.
podman run -d --name nginx -p 8888:80 docker.io/library/nginx
```

### Explanation

* `podman run` → Creates and starts a container.
* `-d` → Runs the container in detached/background mode.
* `--name nginx` → Gives the container the name `nginx`.
* `-p 8888:80` → Maps host port `8888` to container port `80`.
* `docker.io/library/nginx` → Uses the Nginx container image.

The application can now be accessed through:

```bash
curl localhost:8888
```

---

## Step 2: Generate a systemd Service

```bash
# Generate a systemd service for the Nginx container.
# The service will recreate and manage the container.
podman generate systemd --name nginx --new --files --restart-policy=always
```

This generates:

```text
container-nginx.service
```

### Important Options

```text
--name nginx
```

Specifies the container to generate the service for.

```text
--new
```

Makes the generated service create the container when the service starts.

```text
--files
```

Creates the service file in the current directory.

```text
--restart-policy=always
```

Configures the container to restart automatically if it stops unexpectedly.

---

## Step 3: Create the systemd User Directory

Because Podman is running in **rootless mode**, the service is managed as a user service.

```bash
# Create the systemd user service directory.
mkdir -p ~/.config/systemd/user
```

Move the generated service:

```bash
# Move the generated service into the systemd user directory.
mv container-nginx.service ~/.config/systemd/user/
```

The service is now located at:

```text
~/.config/systemd/user/container-nginx.service
```

---

## Step 4: Reload systemd

```bash
# Reload systemd so it detects the newly created service.
systemctl --user daemon-reload
```

`daemon-reload` tells systemd to reread its service files.

---

## Step 5: Enable and Start the Service

```bash
# Enable the service to start automatically and start it immediately.
systemctl --user enable --now container-nginx.service
```

Two operations happen here:

```text
enable
```

Makes the service start automatically.

```text
--now
```

Starts the service immediately.

---

## Step 6: Verify the Service

```bash
# Check the status of the Nginx container service.
systemctl --user status container-nginx.service
```

The service should show:

```text
Active: active (running)
```

Check the Nginx web server:

```bash
# Test the Nginx web server.
curl localhost:8888
```

Expected result:

```text
Welcome to nginx!
```

---

## Final Solution

```bash
# Create and run the Nginx container.
podman run -d --name nginx -p 8888:80 docker.io/library/nginx

# Generate a systemd service for the container.
podman generate systemd --name nginx --new --files --restart-policy=always

# Create the systemd user service directory.
mkdir -p ~/.config/systemd/user

# Move the generated service file into the user systemd directory.
mv container-nginx.service ~/.config/systemd/user/

# Reload systemd to recognize the new service.
systemctl --user daemon-reload

# Enable the service and start it immediately.
systemctl --user enable --now container-nginx.service

# Verify that the service is running.
systemctl --user status container-nginx.service

# Verify that Nginx is accessible.
curl localhost:8888
```

## Key Lessons

* **Docker** uses a central daemon (`dockerd`) to manage containers.
* **Podman** is daemonless and can run containers directly.
* Podman supports **rootless containers**, allowing containers to run without root privileges.
* **systemd** can be used to manage Podman containers.
* `systemctl --user` manages services belonging to the current user.
* `daemon-reload` makes systemd reread service files.
* `enable` configures automatic startup.
* `--now` starts the service immediately.
* `-p 8888:80` maps host port `8888` to Nginx's container port `80`.
* A generated systemd service can provide Docker-like lifecycle management for a Podman container.

```
```
