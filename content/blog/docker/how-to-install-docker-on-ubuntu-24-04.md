---
title: "How to Install and Configure Docker on Ubuntu 24.04"
description: "A complete step-by-step guide to installing Docker on Ubuntu 24.04 LTS — covering installation, post-install setup, Docker Compose, firewall configuration, and production best practices."
date: 2026-04-08T19:44:56+05:30
author: "DB"
tags: ["Docker", "Ubuntu", "Linux", "Containers", "DevOps", "Docker Compose"]
series: "Linux Web Server Setup"
level: "All Levels"
draft: false
---

## Overview

Docker is an open-source containerization platform that lets you package applications and their dependencies into lightweight, portable containers. In this guide, you will learn how to install Docker Engine on **Ubuntu 24.04 LTS**, configure it for everyday use, install Docker Compose, and apply essential production hardening steps.

By the end of this tutorial, you will have a fully functional Docker environment ready for development and production workloads.

---

## Prerequisites

Before you begin, make sure you have:

- A server or local machine running **Ubuntu 24.04 LTS**
- A non-root user with `sudo` privileges
- A stable internet connection
- Basic familiarity with the Linux terminal

---

## Step 1 — Update the System

Always start by updating your package index and upgrading existing packages to ensure a clean base.

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Step 2 — Install Required Dependencies

Install the packages Docker needs to fetch and verify packages over HTTPS.

```bash
sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

---

## Step 3 — Add Docker's Official GPG Key

Create the directory for storing keyrings, then download and save Docker's GPG key.

```bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

---

## Step 4 — Add the Docker Repository

Add the official Docker `apt` repository to your sources list.

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update the package index to include the new repository.

```bash
sudo apt update
```

---

## Step 5 — Install Docker Engine

Install Docker Engine, the CLI, containerd, and the Buildx and Compose plugins in one command.

```bash
sudo apt install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin
```

---

## Step 6 — Verify the Installation

Check that Docker Engine is installed correctly by running the classic `hello-world` image.

```bash
sudo docker run hello-world
```

You should see a message confirming that Docker is working correctly.

Also verify the installed version:

```bash
docker --version
```

Expected output (version may differ):

```
Docker version 26.x.x, build xxxxxxx
```

---

## Step 7 — Manage Docker as a Non-Root User

By default, Docker requires `sudo`. To run Docker commands as your regular user, add your user to the `docker` group.

```bash
sudo usermod -aG docker $USER
```

Apply the group change without logging out:

```bash
newgrp docker
```

Verify you can run Docker without `sudo`:

```bash
docker run hello-world
```

> **Security Note:** Users in the `docker` group have privileges equivalent to the `root` user. Only add trusted users to this group.

---

## Step 8 — Enable Docker to Start on Boot

Enable the Docker and containerd services to start automatically on system boot.

```bash
sudo systemctl enable docker
sudo systemctl enable containerd
```

Check the current status of the Docker service:

```bash
sudo systemctl status docker
```

You should see `active (running)` in the output.

---

## Step 9 — Configure the Firewall (UFW)

If UFW (Uncomplicated Firewall) is enabled on your server, Docker manages its own iptables rules and may bypass UFW by default. It is good practice to be aware of this behaviour.

Check if UFW is active:

```bash
sudo ufw status
```

Allow any ports your containers expose as needed. For example, to allow HTTP and HTTPS traffic:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw reload
```

---

## Step 10 — Install Docker Compose (Standalone, Optional)

The `docker-compose-plugin` installed in Step 5 provides the `docker compose` (v2) command. If you need the older standalone `docker-compose` (v1) binary for legacy projects, install it as follows.

```bash
sudo apt install -y docker-compose
```

Verify:

```bash
docker compose version
```

---

## Step 11 — Test Docker Compose

Create a simple test to confirm Docker Compose is working correctly.

```bash
mkdir ~/compose-test && cd ~/compose-test
```

Create a `compose.yml` file:

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
```

Start the service:

```bash
docker compose up -d
```

Visit `http://your-server-ip:8080` in your browser. You should see the default Nginx welcome page served from inside a Docker container.

Stop and clean up:

```bash
docker compose down
```

---

## Step 12 — Configure Docker Daemon (Production Hardening)

For production environments, create a `/etc/docker/daemon.json` file to configure logging, storage drivers, and resource limits.

```bash
sudo nano /etc/docker/daemon.json
```

Add the following recommended settings:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2",
  "live-restore": true,
  "userland-proxy": false
}
```

| Option | Purpose |
|---|---|
| `log-driver` | Use JSON log files (default, easy to rotate) |
| `max-size` / `max-file` | Prevent logs from consuming all disk space |
| `storage-driver` | `overlay2` is the recommended driver for Ubuntu |
| `live-restore` | Keeps containers running when Docker daemon restarts |
| `userland-proxy` | Disabling it improves network performance |

Restart Docker to apply changes:

```bash
sudo systemctl restart docker
```

---

## Common Docker Commands Reference

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Pull an image from Docker Hub
docker pull ubuntu:24.04

# Run a container interactively
docker run -it ubuntu:24.04 bash

# Stop a running container
docker stop <container_id>

# Remove a container
docker rm <container_id>

# Remove an image
docker rmi <image_id>

# View disk usage by Docker
docker system df

# Remove unused containers, networks, images
docker system prune -a
```

---

## Troubleshooting

### Docker daemon is not running

```bash
sudo systemctl start docker
sudo systemctl status docker
```

### Permission denied when running Docker

Make sure your user is in the `docker` group and you have applied the group change:

```bash
groups $USER        # Check groups
newgrp docker       # Apply without logout
```

### Cannot connect to the Docker daemon

Check if the socket exists:

```bash
ls -la /var/run/docker.sock
```

If it is missing, restart the service:

```bash
sudo systemctl restart docker
```

---

## Uninstalling Docker

If you need to completely remove Docker from your system:

```bash
# Remove Docker packages
sudo apt purge -y docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin

# Remove all Docker data (images, containers, volumes)
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd

# Remove the Docker repository
sudo rm /etc/apt/sources.list.d/docker.list
sudo rm /etc/apt/keyrings/docker.gpg
```

---

## Conclusion

You have successfully installed and configured Docker on Ubuntu 24.04 LTS. Here is a summary of what was covered:

- Installed Docker Engine from the official Docker repository
- Configured Docker to run without `sudo`
- Enabled Docker to start automatically on boot
- Installed and tested Docker Compose
- Applied production-grade daemon hardening

Docker is now ready to run containers on your Ubuntu server. As a next step, explore Docker networking, named volumes, and multi-container applications using Docker Compose.

---

## Further Reading

- [Official Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Compose Reference](https://docs.docker.com/compose/)
- [Docker Security Best Practices](https://docs.docker.com/engine/security/)
