---
title: "Docker Interview Questions & Answers(2026) Part 02"
description: "40+ Docker interview questions and answers covering architecture, Dockerfile, networking, security, and troubleshooting — Basic to Advanced."
date: 2025-04-26
author: "DB"
tags: ["docker", "containers", "interview", "devops"]
tool: "docker"
level: "All Levels"
question_count: 40
draft: "false"
---

<div class="qa-list">

{{< qa num="1" q="What is Docker and why is it used?" level="basic" >}}
**Ans:**

**Docker** is a platform for building, shipping, and running applications inside **containers** — lightweight, isolated environments that package the application with all its dependencies.

**Why Docker?**
- **Consistency** — "Works on my machine" problem solved. Same container runs everywhere.
- **Isolation** — Each container has its own filesystem, network, and process space
- **Portability** — Run on any OS that has Docker installed
- **Efficiency** — Containers share the host OS kernel (unlike VMs with full OS copies)
- **Speed** — Containers start in milliseconds

```bash
# Run a container
docker run -d -p 80:80 --name webserver nginx

# Check running containers
docker ps

# Check all containers (including stopped)
docker ps -a
```
{{< /qa >}}

{{< qa num="2" q="What is the difference between Docker and Virtual Machines?" level="basic" >}}
**Ans:**

| Feature | Docker Container | Virtual Machine |
|---------|-----------------|-----------------|
| **OS** | Shares host OS kernel | Full guest OS |
| **Size** | MBs | GBs |
| **Startup** | Milliseconds | Minutes |
| **Isolation** | Process-level | Hardware-level |
| **Portability** | High | Lower |
| **Performance** | Near-native | Overhead from hypervisor |
| **Use case** | Microservices, apps | Full OS environments |

```
VM Architecture:              Docker Architecture:
┌──────────────────┐          ┌──────────────────┐
│  App A  │ App B  │          │  App A  │ App B  │
│  Libs   │ Libs   │          │  Libs   │ Libs   │
│ Guest OS│ Guest OS│         │──────────────────│
│──────────────────│          │   Docker Engine   │
│    Hypervisor    │          │──────────────────│
│─────────────────-│          │    Host OS        │
│     Host OS      │          │──────────────────│
│    Hardware      │          │    Hardware       │
└──────────────────┘          └──────────────────┘
```
{{< /qa >}}

{{< qa num="3" q="Explain Docker architecture?" level="basic" >}}
**Ans:**

Docker uses a **client-server architecture**:

```
Docker CLI (client)
      │  REST API
      ▼
Docker Daemon (dockerd) ← server running on host
      │
      ├── Images
      ├── Containers
      ├── Networks
      └── Volumes
```

**Key components:**

| Component | Role |
|-----------|------|
| **Docker Client** (`docker` CLI) | Sends commands to Docker daemon |
| **Docker Daemon** (`dockerd`) | Manages containers, images, networks, volumes |
| **Docker Images** | Read-only templates used to create containers |
| **Docker Containers** | Running instances of images |
| **Docker Registry** | Stores and distributes images (Docker Hub, ECR, etc.) |
| **Docker Networks** | Isolated networking for containers |
| **Docker Volumes** | Persistent storage for containers |

```bash
# Client sends command to daemon
docker run nginx
# Docker client → API call → dockerd → pulls nginx image → creates container
```
{{< /qa >}}

{{< qa num="4" q="What is the use and purpose of a Dockerfile?" level="basic" >}}
**Ans:**

A **Dockerfile** is a text file containing instructions to build a Docker image. It automates the image creation process.

**Why use Dockerfile:**
- Reproducible builds — same Dockerfile = same image every time
- Version-controlled — stored in Git alongside code
- Automation — builds are scripted, not manual

```dockerfile
# Basic Dockerfile example
FROM node:18-alpine          # Base image
WORKDIR /app                 # Set working directory
COPY package*.json ./        # Copy dependency files
RUN npm install              # Install dependencies
COPY . .                     # Copy application code
EXPOSE 3000                  # Document port
CMD ["node", "server.js"]    # Default command to run
```

```bash
# Build image from Dockerfile
docker build -t myapp:1.0 .

# Build with no cache
docker build --no-cache -t myapp:1.0 .
```
{{< /qa >}}

{{< qa num="5" q="What are Dockerfile instructions? Explain key ones." level="basic" >}}
**Ans:**

| Instruction | Purpose |
|-------------|---------|
| `FROM` | Base image to build on |
| `RUN` | Execute commands during build (creates a layer) |
| `CMD` | Default command when container starts (can be overridden) |
| `ENTRYPOINT` | Main process; CMD becomes its arguments |
| `COPY` | Copy files from host to image |
| `ADD` | Like COPY, but also extracts tarballs and supports URLs |
| `EXPOSE` | Documents which port the app listens on |
| `ENV` | Set environment variables |
| `ARG` | Build-time variables |
| `WORKDIR` | Set working directory |
| `USER` | Set user to run as |
| `VOLUME` | Create a mount point |
| `LABEL` | Add metadata |
| `HEALTHCHECK` | Define health check command |

```dockerfile
FROM ubuntu:22.04
LABEL maintainer="dev@example.com"
ARG APP_VERSION=1.0
ENV APP_ENV=production
WORKDIR /app
COPY . .
RUN apt-get update && apt-get install -y python3
EXPOSE 8080
USER appuser
HEALTHCHECK --interval=30s CMD curl -f http://localhost:8080/health
ENTRYPOINT ["python3"]
CMD ["app.py"]
```
{{< /qa >}}

{{< qa num="6" q="Difference between CMD, ENTRYPOINT, and RUN in Docker." level="basic" >}}
**Ans:**

| Instruction | When it runs | Purpose | Overridable? |
|-------------|-------------|---------|--------------|
| `RUN` | **Build time** | Execute commands to build the image (install packages, etc.) | N/A |
| `CMD` | **Runtime** | Default arguments/command when container starts | Yes (`docker run img custom-cmd`) |
| `ENTRYPOINT` | **Runtime** | Fixed main process of the container | Only with `--entrypoint` flag |

**Practical example:**
```dockerfile
FROM nginx
RUN apt-get update && apt-get install -y curl   # Build time only

ENTRYPOINT ["nginx"]         # Always runs nginx
CMD ["-g", "daemon off;"]    # Default args (can be overridden)
```

```bash
# CMD can be overridden:
docker run myimage echo "hello"   # Overrides CMD

# ENTRYPOINT stays, CMD becomes args:
docker run myimage -c /etc/nginx/nginx.conf   # "-c ..." overrides CMD only
```
{{< /qa >}}

{{< qa num="7" q="What are main Docker network types?" level="basic" >}}
**Ans:**

| Network | Description | Use Case |
|---------|-------------|---------|
| **bridge** | Default. Containers get private IPs, communicate via bridge interface | Same-host container communication |
| **host** | Container uses host's network namespace directly | High performance, no port mapping needed |
| **none** | No network interface. Complete isolation | Security-sensitive containers |
| **overlay** | Multi-host networking (Docker Swarm/K8s) | Distributed apps across nodes |
| **macvlan** | Container gets its own MAC/IP on physical network | Legacy apps expecting direct network access |

```bash
# List networks
docker network ls

# Create custom bridge network
docker network create --driver bridge mynetwork

# Run container on custom network
docker run -d --network mynetwork --name app nginx

# Inspect network
docker network inspect mynetwork

# On same network, containers can resolve each other by name:
docker run -it --network mynetwork alpine ping app
```
{{< /qa >}}

{{< qa num="8" q="If a Docker image is built and you manually install missing packages inside a container, how do you convert it into a new Docker image?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Find the running container ID
docker ps

# Step 2: Run the container and install packages
docker exec -it <container_id> bash
apt-get install -y curl vim

# Step 3: Commit the container to a new image
docker commit <container_id> myimage:v2

# With author and message:
docker commit \
  --author "Dev Team" \
  --message "Added curl and vim" \
  <container_id> myimage:v2

# Verify the new image
docker images | grep myimage
docker run myimage:v2 which curl   # Test curl is available

# Step 4 (Best Practice): Update your Dockerfile instead!
# RUN apt-get install -y curl vim
# docker build -t myimage:v2 .
```

> **Note:** `docker commit` is useful for quick fixes but should always be followed by updating the Dockerfile for reproducibility.
{{< /qa >}}

{{< qa num="9" q="One of your containers keeps restarting in production. How do you identify the root cause?" level="intermediate" >}}
**Ans:**

```bash
# Step 1: Check restart count and status
docker ps -a
# Look at RESTARTS column and STATUS

# Step 2: Check container logs (recent entries)
docker logs <container_name>

# Step 3: Check logs with timestamps
docker logs --timestamps <container_name>

# Step 4: Follow logs in real-time during restart
docker logs -f <container_name>

# Step 5: Check exit code
docker inspect <container_name> \
  --format '{{.State.ExitCode}} {{.State.Error}}'
# Exit 1 = app error, 137 = OOMKilled, 139 = segfault, 143 = SIGTERM

# Step 6: Check events
docker events --filter container=<container_name>

# Step 7: Check resource constraints
docker stats <container_name>

# Step 8: Check inside the container
docker exec -it <container_name> sh
```

**Common causes:**
- Application crash (check exit code 1 → app logs)
- OOMKilled (exit code 137 → increase memory limit)
- Port already in use
- Missing environment variables or config
{{< /qa >}}

{{< qa num="10" q="How do you optimize Docker image size?" level="intermediate" >}}
**Ans:**

```dockerfile
# Tip 1: Use slim/alpine base images
FROM node:18-alpine     # 180MB vs FROM node:18 → 1.1GB

# Tip 2: Multi-stage builds (build in one stage, copy only artifacts)
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
# Final image only contains built files + nginx, no node_modules

# Tip 3: Combine RUN commands (each RUN = one layer)
# BAD (3 layers):
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*

# GOOD (1 layer):
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# Tip 4: Use .dockerignore
echo "node_modules\n.git\n*.log\ndist" > .dockerignore

# Tip 5: Don't copy unnecessary files
COPY src/ ./src/   # Instead of COPY . .
```

```bash
# Check image size
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Analyze layers
docker history myimage:1.0
```
{{< /qa >}}

{{< qa num="11" q="How do you scan Docker images for vulnerabilities before deployment?" level="intermediate" >}}
**Ans:**

```bash
# Method 1: Docker Scout (built into Docker CLI)
docker scout cves myimage:1.0
docker scout recommendations myimage:1.0

# Method 2: Trivy (most popular, free, open-source)
# Install
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh

# Scan image
trivy image myimage:1.0

# Scan with specific severity
trivy image --severity HIGH,CRITICAL myimage:1.0

# Output as JSON for CI/CD
trivy image --format json -o results.json myimage:1.0

# Method 3: Snyk
snyk container test myimage:1.0

# Method 4: AWS ECR Scan (if using ECR)
aws ecr start-image-scan \
  --repository-name my-repo \
  --image-id imageTag=1.0

aws ecr describe-image-scan-findings \
  --repository-name my-repo \
  --image-id imageTag=1.0

# Integrate into CI/CD pipeline (Jenkins/GitHub Actions):
# Run trivy after docker build, fail pipeline if CRITICAL vulns found
trivy image --exit-code 1 --severity CRITICAL myimage:1.0
```
{{< /qa >}}

{{< qa num="12" q="How would you manage secrets in Docker containers securely?" level="intermediate" >}}
**Ans:**

```bash
# BAD (Never do this):
ENV DB_PASSWORD=mysecret123          # Visible in image layers
docker run -e DB_PASSWORD=mysecret   # Visible in docker inspect

# Method 1: Docker Secrets (Docker Swarm)
echo "mysecret" | docker secret create db_password -
docker service create \
  --secret db_password \
  --name myapp myimage
# Secret available at: /run/secrets/db_password inside container

# Method 2: Environment variables from a file (for compose)
# .env file (not committed to git):
DB_PASSWORD=mysecret123

# docker-compose.yml:
# env_file:
#   - .env

# Method 3: AWS Secrets Manager / SSM at runtime
# In entrypoint script:
export DB_PASSWORD=$(aws secretsmanager get-secret-value \
  --secret-id prod/db-password \
  --query SecretString --output text)

# Method 4: Vault (HashiCorp) Agent Sidecar
# Vault agent auto-renews and injects secrets into container

# Method 5: Kubernetes Secrets (when in K8s)
kubectl create secret generic db-secret --from-literal=password=mysecret
# Mount as volume or envFrom in pod spec
```
{{< /qa >}}

{{< qa num="13" q="How do you handle a vulnerable dependency detected in a Docker image after deployment?" level="advanced" >}}
**Ans:**

```bash
# Immediate mitigation:

# Step 1: Confirm and assess the vulnerability
trivy image myimage:1.0 --severity CRITICAL

# Step 2: Check if the vulnerable package is actually used
docker run myimage:1.0 dpkg -l | grep <package-name>

# Step 3: If critical — isolate/rollback immediately
# Kubernetes rollback:
kubectl rollout undo deployment/myapp

# Docker Swarm rollback:
docker service rollback myapp-service

# Step 4: Fix the vulnerability
# Update base image:
# FROM node:18-alpine → FROM node:18.20-alpine (patched version)

# Update specific package:
RUN apt-get update && apt-get install -y --only-upgrade <vulnerable-package>

# Step 5: Rebuild and rescan
docker build -t myimage:1.1 .
trivy image --exit-code 1 --severity CRITICAL myimage:1.1

# Step 6: Secure the CI/CD pipeline going forward
# - Add trivy scan gate in Jenkins/GitHub Actions
# - Set up automated base image update PRs (Renovate/Dependabot)
# - Schedule weekly image rescans even for deployed images
```
{{< /qa >}}

{{< qa num="14" q="What is containerization?" level="basic" >}}
**Ans:**

**Containerization** is the process of packaging an application and all its dependencies (code, runtime, libraries, config) into a single portable unit called a **container**.

**How it works:**
- Uses Linux kernel features: **namespaces** (isolation) and **cgroups** (resource limits)
- Containers share the host OS kernel — no Guest OS overhead
- Each container has its own: filesystem, network stack, process tree

```bash
# Namespaces used:
# PID namespace → isolated process IDs
# NET namespace → isolated networking
# MNT namespace → isolated filesystem
# UTS namespace → isolated hostname
# IPC namespace → isolated inter-process communication
# USER namespace → isolated user IDs

# cgroups control:
# CPU usage
# Memory limits
# I/O bandwidth
```

**Benefits:** Consistency, portability, density (many containers per host), fast startup.
{{< /qa >}}

{{< qa num="15" q="How do you verify deployed app files on an EC2 container host?" level="basic" >}}
**Ans:**

```bash
# Check running containers
docker ps

# Inspect container details (image, mounts, env)
docker inspect <container_name>

# Access container filesystem
docker exec -it <container_name> sh
ls -la /app/
cat /app/config.json

# Check mounted volumes
docker inspect <container_name> \
  --format '{{json .Mounts}}' | python3 -m json.tool

# Copy file from container to host for inspection
docker cp <container_name>:/app/config.json /tmp/config.json

# Check image that the container is running
docker inspect <container_name> --format '{{.Config.Image}}'

# Check container logs for startup output
docker logs <container_name> --tail 50
```
{{< /qa >}}

{{< qa num="16" q="How do you clear old build artifacts before deployment?" level="intermediate" >}}
**Ans:**

```bash
# Remove stopped containers
docker container prune -f

# Remove dangling images (untagged, not used)
docker image prune -f

# Remove all unused images (not just dangling)
docker image prune -a -f

# Remove unused volumes
docker volume prune -f

# Remove unused networks
docker network prune -f

# Nuclear option — remove everything unused
docker system prune -a -f --volumes

# Check disk usage before/after
docker system df

# In Jenkins pipeline — clean before build:
pipeline {
  stages {
    stage('Clean') {
      steps {
        sh 'docker system prune -f'
        sh 'docker rmi myapp:old || true'
      }
    }
  }
}
```
{{< /qa >}}

{{< qa num="17" q="What is the difference between docker stop and docker kill?" level="basic" >}}
**Ans:**

| Command | Signal sent | Behavior |
|---------|-------------|---------|
| `docker stop` | SIGTERM, then SIGKILL after timeout | Graceful shutdown. App can clean up. |
| `docker kill` | SIGKILL by default | Immediate force kill. No cleanup. |

```bash
# Graceful stop (allows 10s for cleanup by default)
docker stop mycontainer

# Graceful stop with custom timeout
docker stop --time 30 mycontainer

# Force kill immediately
docker kill mycontainer

# Send specific signal
docker kill --signal SIGHUP mycontainer
```

**Best practice:** Always use `docker stop` first. Use `docker kill` only if the container is unresponsive.
{{< /qa >}}

{{< qa num="18" q="How do you validate environment variables for app startup in Docker?" level="intermediate" >}}
**Ans:**

```bash
# Method 1: Check env vars inside running container
docker exec mycontainer env
docker exec mycontainer printenv DB_HOST

# Method 2: Inspect from outside
docker inspect mycontainer --format '{{json .Config.Env}}'

# Method 3: Validate at entrypoint script level
#!/bin/bash
# entrypoint.sh
required_vars=("DB_HOST" "DB_PORT" "APP_SECRET")
for var in "${required_vars[@]}"; do
  if [ -z "${!var}" ]; then
    echo "ERROR: Required env var $var is not set"
    exit 1
  fi
done
exec "$@"

# Dockerfile:
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["node", "server.js"]

# Method 4: Docker Compose env validation
# Pass from .env file and validate with compose healthcheck
```
{{< /qa >}}

{{< qa num="19" q="How do you roll back a failed Docker deployment?" level="intermediate" >}}
**Ans:**

```bash
# Method 1: Docker (standalone) — switch back to previous image tag
docker stop myapp
docker rm myapp
docker run -d --name myapp myimage:1.0   # Previous version

# Method 2: Docker Compose
# Revert docker-compose.yml to old image version
# image: myapp:1.0 (instead of 1.1)
docker-compose up -d

# Method 3: Docker Swarm
docker service rollback myapp-service
# Or update to previous image:
docker service update --image myimage:1.0 myapp-service

# Method 4: Kubernetes (if using K8s)
kubectl rollout undo deployment/myapp
kubectl rollout undo deployment/myapp --to-revision=3   # Specific revision

# Method 5: Scripted rollback
PREV_IMAGE=$(docker inspect myapp --format '{{.Config.Image}}')
# Store previous image tag in a file or environment variable before each deploy
docker stop myapp && docker rm myapp
docker run -d --name myapp $PREV_IMAGE
```

**Best practice:** Always tag images with version numbers, never just `latest` in production.
{{< /qa >}}

{{< qa num="20" q="What is a multi-stage Dockerfile and when do you use it?" level="intermediate" >}}
**Ans:**

A **multi-stage build** uses multiple `FROM` statements in a single Dockerfile to produce a minimal final image by copying only the necessary artifacts from build stages.

**Use case:** Compile/build in a fat image, ship only the binary in a slim image.

```dockerfile
# Stage 1: Build (uses full Go/Node/Maven image)
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp ./cmd/

# Stage 2: Final image (tiny, secure)
FROM alpine:3.18
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/myapp .
EXPOSE 8080
CMD ["./myapp"]
```

```bash
# Build
docker build -t myapp:1.0 .

# Result: Final image is ~15MB instead of ~800MB
docker images myapp
```

**Benefits:**
- No build tools (gcc, npm, go) in production image
- Smaller attack surface
- Faster image pulls
{{< /qa >}}

</div>
