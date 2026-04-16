---
title: "Docker Scenario Interview Questions and Answers"
description: "35 real-world Docker scenario-based interview questions and answers covering containers, networking, volumes, Docker Compose, security, performance troubleshooting, and CI/CD — Beginner to Advanced."
date: 2026-04-07T14:26:30+05:30
author: "DB"
tags: ["Docker", "Containers", "DevOps", "CI/CD", "Interview", "Docker Compose"]
tool: "docker"
level: "All Levels"
question_count: 35
draft: false
---

<div class="qa-list">


{{< qa num="1" q="Your team has a new developer who needs to run a Node.js app without installing Node.js locally. How would you help them using Docker?" level="basic" >}}
You would containerize the Node.js app so the developer only needs Docker installed.

**Step 1 — Create a `Dockerfile` in the project root:**

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

**Step 2 — Build the image:**
```bash
docker build -t my-node-app .
```

**Step 3 — Run the container:**
```bash
docker run -p 3000:3000 my-node-app
```

The developer can now access the app at `http://localhost:3000` without installing Node.js locally.
{{< /qa >}}

{{< qa num="2" q="You ran a container but forgot to name it. Now you want to stop it. What do you do?" level="basic" >}}
List all running containers to find the container ID or auto-generated name:

```bash
docker ps
```

Then stop it using the container ID or name:

```bash
docker stop <container_id_or_name>
```

{{< callout type="tip" title="Pro Tip" >}}
Always name your containers using the `--name` flag to avoid this situation:
```bash
docker run --name my-app -d nginx
```
{{< /callout >}}
{{< /qa >}}

{{< qa num="3" q="A container exited immediately after you ran it. How do you debug what went wrong?" level="basic" >}}
**Step 1 — View container logs:**
```bash
docker logs <container_id>
```

**Step 2 — List exited containers:**
```bash
docker ps -a
```

**Step 3 — Inspect the exit code:**
```bash
docker inspect <container_id> --format='{{.State.ExitCode}}'
```

**Exit code reference:**

| Code | Meaning |
|------|---------|
| `0` | Exited normally |
| `1` | Application error |
| `137` | Killed (OOM or SIGKILL) |
| `125` | Docker daemon error |
| `126` | Permission denied |
| `127` | Command not found |

**Step 4 — Run interactively to debug:**
```bash
docker run -it <image> /bin/sh
```
{{< /qa >}}

{{< qa num="4" q="You built a Docker image and it's 1.5GB. Your manager wants it smaller. What do you do?" level="basic" >}}
Several techniques reduce image size significantly:

**1. Use a smaller base image:**
```dockerfile
# Instead of:
FROM node:18

# Use:
FROM node:18-alpine
```

**2. Use multi-stage builds (most impactful):**
```dockerfile
# Build stage — heavy, has all build tools
FROM node:18-alpine AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

# Production stage — only the output
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

**3. Combine RUN commands to reduce layers:**
```dockerfile
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

**4. Use `.dockerignore` to exclude unnecessary files:**
```
node_modules
.git
*.log
tests/
.env
```

{{< callout type="tip" title="Check image size breakdown" >}}
Use `docker history my-image` to see which layers are biggest and target those first.
{{< /callout >}}
{{< /qa >}}

{{< qa num="5" q="You want to share data between your host machine and a running container. How do you do it?" level="basic" >}}
Use **bind mounts** to share host directories with the container:

```bash
docker run -v /host/path:/container/path my-image
```

**Example — sharing a local `src` folder for live development:**
```bash
docker run -v $(pwd)/src:/app/src -p 3000:3000 my-node-app
```

Changes made on the host are reflected instantly inside the container — perfect for development.

**For production data persistence, use named volumes instead:**
```bash
docker volume create mydata
docker run -v mydata:/app/data my-image
```

| Type | Use Case | Persists After Container Removed? |
|------|----------|----------------------------------|
| Bind mount | Development, sharing host files | Yes (lives on host) |
| Named volume | Production data, databases | Yes (managed by Docker) |
| tmpfs | Sensitive/temporary data in RAM | No |
{{< /qa >}}

{{< qa num="6" q="Your application container needs to connect to a MySQL database container. How do you set this up?" level="intermediate" >}}
Use a **Docker network** so containers can communicate by name:

```bash
# Step 1 — Create a custom network
docker network create myapp-network

# Step 2 — Run MySQL on the network
docker run -d \
  --name mysql-db \
  --network myapp-network \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=appdb \
  mysql:8

# Step 3 — Run the app on the same network
docker run -d \
  --name my-app \
  --network myapp-network \
  -e DB_HOST=mysql-db \
  -e DB_PORT=3306 \
  my-app-image
```

The app container can reach MySQL using the hostname `mysql-db` because they share the same network. Docker's built-in DNS resolves container names automatically.

{{< callout type="tip" title="Use Docker Compose for multi-container apps" >}}
For apps with multiple services, Docker Compose handles network creation and linking automatically — no manual `docker network` commands needed.
{{< /callout >}}
{{< /qa >}}

{{< qa num="7" q="You need to run the same container image in dev, staging, and production with different configurations. How do you handle this without rebuilding the image?" level="intermediate" >}}
Use **environment variables** to inject config at runtime, keeping the same image across all environments. This follows the **12-factor app** methodology.

```bash
# Development
docker run -e ENV=dev -e DB_URL=dev-db:5432 my-app

# Staging
docker run -e ENV=staging -e DB_URL=staging-db:5432 my-app

# Production
docker run -e ENV=prod -e DB_URL=prod-db:5432 my-app
```

**Or use an env file per environment:**

```bash
# dev.env
ENV=dev
DB_URL=dev-db:5432
LOG_LEVEL=debug
```

```bash
docker run --env-file dev.env my-app
```

{{< callout type="warn" title="Never bake config into your image" >}}
Rebuilding an image per environment breaks the principle of build once, deploy anywhere. Keep images environment-agnostic and inject config at runtime.
{{< /callout >}}
{{< /qa >}}

{{< qa num="8" q="A container is consuming too much memory and causing issues on the host. How do you limit it?" level="intermediate" >}}
Use Docker resource constraints at container startup:

```bash
# Limit memory to 512MB and CPU to 0.5 cores
docker run \
  --memory=512m \
  --memory-swap=512m \
  --cpus=0.5 \
  my-app
```

**Resource flag reference:**

| Flag | Description |
|------|-------------|
| `--memory` | Max RAM the container can use |
| `--memory-swap` | Max swap (set equal to `--memory` to disable swap) |
| `--cpus` | Number of CPUs (fractional values allowed) |
| `--cpu-shares` | Relative CPU weight (default 1024) |

**Monitor live resource usage:**
```bash
docker stats
docker stats --no-stream   # One-shot snapshot
```
{{< /qa >}}

{{< qa num="9" q="You updated your app code. What's your workflow to update the running container?" level="intermediate" >}}
**Never modify a running container directly.** Follow the immutable infrastructure pattern — replace, don't patch:

```bash
# Step 1 — Build a new image with a version tag
docker build -t my-app:v2.0 .

# Step 2 — Stop and remove the old container
docker stop my-app-container
docker rm my-app-container

# Step 3 — Start a fresh container with the new image
docker run -d --name my-app-container -p 3000:3000 my-app:v2.0
```

{{< callout type="tip" title="Tag images with versions, not just latest" >}}
Always tag with a specific version (`v2.0`, `git SHA`) in addition to `latest`. This gives you a clear history and the ability to roll back instantly by running the previous tag.
{{< /callout >}}
{{< /qa >}}

{{< qa num="10" q="You need to copy a file from inside a running container to your host machine. How?" level="basic" >}}
Use `docker cp`:

```bash
# Copy FROM container TO host
docker cp <container_name>:/path/in/container /path/on/host

# Real example — pull logs from a running container
docker cp my-app:/app/logs/app.log ./app.log
```

**Copy from host TO container:**
```bash
docker cp ./config.json my-app:/app/config.json
```

{{< callout type="info" title="Works on running and stopped containers" >}}
`docker cp` works even if the container is stopped — you don't need to start it just to retrieve a file.
{{< /callout >}}
{{< /qa >}}

{{< qa num="11" q="After several weeks of development, your system is cluttered with stopped containers, unused images, and dangling volumes. How do you clean up?" level="basic" >}}
Docker provides a granular prune system for each resource type:

```bash
# Remove all stopped containers
docker container prune

# Remove dangling images (untagged)
docker image prune

# Remove ALL unused images (not just dangling)
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Nuclear option — clean everything unused at once
docker system prune -a --volumes
```

**Check disk usage before and after:**
```bash
docker system df
```

{{< callout type="warn" title="Be careful with --volumes flag" >}}
`docker system prune --volumes` removes ALL unused volumes including ones with data you may still need. Always run `docker system df` first to see what will be affected.
{{< /callout >}}
{{< /qa >}}

{{< qa num="12" q="You need to build a Docker image for both ARM (Apple M1/M2) and AMD64 architectures. How do you do it?" level="advanced" >}}
Use **Docker Buildx** for multi-platform builds:

```bash
# Step 1 — Create and activate a multi-platform builder
docker buildx create --name multibuilder --use

# Step 2 — Build and push for both platforms in one command
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myrepo/my-app:latest \
  --push .
```

This creates a **manifest list** (multi-arch image) so Docker automatically pulls the correct image for the host's architecture — the same tag works on both Intel Macs and Apple Silicon.

**Verify the manifest:**
```bash
docker buildx imagetools inspect myrepo/my-app:latest
```
{{< /qa >}}

{{< qa num="13" q="Your container's application crashes occasionally. How do you make it automatically restart?" level="intermediate" >}}
Use Docker **restart policies**:

```bash
docker run -d --restart=unless-stopped my-app
```

**Restart policy reference:**

| Policy | Behaviour |
|--------|-----------|
| `no` | Never restart (default) |
| `on-failure` | Restart only on non-zero exit code |
| `on-failure:5` | Restart up to 5 times, then stop |
| `always` | Always restart, even after `docker stop` |
| `unless-stopped` | Always restart unless manually stopped |

{{< callout type="tip" title="Recommended for production" >}}
Use `unless-stopped` for long-running services. It survives host reboots and Docker daemon restarts but respects intentional `docker stop` commands from operators.
{{< /callout >}}
{{< /qa >}}

{{< qa num="14" q="How would you pass secrets (like API keys and DB passwords) securely to a container without hardcoding them in the image or environment variables?" level="advanced" >}}
**Option 1 — Docker Secrets (Swarm mode):**
```bash
# Create a secret
echo "supersecretpassword" | docker secret create db_password -

# Use it in a service
docker service create \
  --secret db_password \
  my-app
```
Inside the container, the secret is available at `/run/secrets/db_password` as a file.

**Option 2 — Mounted secret files (standalone containers):**
```bash
docker run \
  -v /host/secrets/db_pass.txt:/run/secrets/db_password:ro \
  my-app
```

**Option 3 — External secrets managers (recommended for production):**
- **AWS Secrets Manager** — fetch at runtime via IAM role, no credentials in container
- **HashiCorp Vault** — dynamic secrets with lease-based rotation
- **Azure Key Vault** — managed identity access

{{< callout type="danger" title="Avoid ENV variables for secrets" >}}
Never pass secrets as `-e MY_SECRET=value`. Environment variables are visible in `docker inspect` output, appear in process listings, and can leak through application logs.
{{< /callout >}}
{{< /qa >}}

{{< qa num="15" q="You suspect a container was compromised. How do you investigate it without affecting the running system?" level="advanced" >}}
**Step 1 — Gather evidence without touching the container:**
```bash
# Inspect container metadata
docker inspect <container_id>

# Check running processes
docker top <container_id>

# View recent logs
docker logs --tail=200 <container_id>
```

**Step 2 — Check for filesystem changes:**
```bash
docker diff <container_id>
```

`docker diff` output codes:
- `A` = file added
- `C` = file changed
- `D` = file deleted

Unexpected binaries in `/tmp`, new cron jobs, or modified `/etc/passwd` are red flags.

**Step 3 — Check network connections:**
```bash
docker exec <container_id> netstat -tulnp
```

**Step 4 — Export filesystem for forensic analysis:**
```bash
docker export <container_id> > compromised.tar
```

**Step 5 — Quarantine: disconnect from all networks:**
```bash
docker network disconnect mynetwork <container_id>
```

This isolates the container from other services while preserving its state for investigation.
{{< /qa >}}

{{< qa num="16" q="You want your Docker image build to be faster in CI/CD. How do you optimize it?" level="advanced" >}}
**Optimization 1 — Order Dockerfile layers by change frequency:**
```dockerfile
# Rarely changes — cached unless package.json changes
COPY package*.json ./
RUN npm install

# Changes on every push — always rebuilds from here
COPY . .
RUN npm run build
```

**Optimization 2 — Enable BuildKit:**
```bash
DOCKER_BUILDKIT=1 docker build .
```

**Optimization 3 — Use BuildKit cache mounts:**
```dockerfile
# syntax=docker/dockerfile:1
RUN --mount=type=cache,target=/root/.npm \
    npm install
```
The npm cache persists between builds — dramatically speeds up dependency installs.

**Optimization 4 — Use a registry cache backend in CI:**
```bash
docker buildx build \
  --cache-from=type=registry,ref=myrepo/my-app:cache \
  --cache-to=type=registry,ref=myrepo/my-app:cache,mode=max \
  -t myrepo/my-app:latest .
```

**Optimization 5 — Use GitHub Actions layer cache:**
```yaml
- name: Build and push
  uses: docker/build-push-action@v4
  with:
    cache-from: type=gha
    cache-to: type=gha,mode=max
```
{{< /qa >}}

{{< qa num="17" q="You have three containers — frontend, backend, and database. The frontend should only talk to the backend, and the backend should talk to the database. The database should not be reachable from the frontend. How do you architect this?" level="advanced" >}}
Use **multiple isolated networks** to enforce traffic boundaries:

```bash
# Create two separate networks
docker network create frontend-net
docker network create backend-net

# Frontend: only on frontend-net
docker run -d --name frontend --network frontend-net frontend-image

# Backend: bridge between both networks
docker run -d --name backend backend-image
docker network connect frontend-net backend
docker network connect backend-net backend

# Database: only on backend-net
docker run -d --name database --network backend-net db-image
```

**Result:**

| Connection | Allowed? |
|-----------|---------|
| `frontend` → `backend` | ✅ Yes (share `frontend-net`) |
| `backend` → `database` | ✅ Yes (share `backend-net`) |
| `frontend` → `database` | ❌ No (no shared network) |

The database is completely invisible to the frontend — there is no route to it, even if an attacker compromises the frontend container.
{{< /qa >}}

{{< qa num="18" q="A container needs to expose a port but only to the host machine, not to external networks. How do you do this?" level="intermediate" >}}
Bind to `127.0.0.1` (localhost) instead of `0.0.0.0`:

```bash
docker run -p 127.0.0.1:3000:3000 my-app
```

The app is reachable at `http://localhost:3000` on the host but **not** accessible from other machines on the network.

**Comparison:**

```bash
# Binds to 0.0.0.0 — accessible from any machine on the network ⚠️
docker run -p 3000:3000 my-app

# Binds to localhost only — private to the host machine ✅
docker run -p 127.0.0.1:3000:3000 my-app
```

{{< callout type="warn" title="Default binding is a security risk" >}}
By default, `-p 3000:3000` binds to `0.0.0.0`, meaning the port is open to the entire network. Always use `127.0.0.1:` prefix for internal services like admin panels, metrics endpoints, or dev servers.
{{< /callout >}}
{{< /qa >}}

{{< qa num="19" q="Two containers on different Docker networks need to communicate. How do you connect them?" level="intermediate" >}}
Connect one container to the other's network:

```bash
docker network connect network-b container-a
```

Or create a shared network and connect both:

```bash
docker network create shared-net
docker network connect shared-net container-a
docker network connect shared-net container-b
```

A container can belong to **multiple networks simultaneously**, so connecting to an additional network does not remove it from its existing networks.

**Verify connectivity:**
```bash
# Check which networks container-a is on
docker inspect container-a --format='{{json .NetworkSettings.Networks}}' | jq .

# Test from inside container-a
docker exec container-a ping container-b
```
{{< /qa >}}

{{< qa num="20" q="Your database container crashed and you lost all data. How do you prevent this from happening in the future?" level="intermediate" >}}
Use **named volumes** for persistent data storage that survives container restarts and deletion:

```bash
# Create a named volume
docker volume create pgdata

# Run PostgreSQL with persistent volume
docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:15
```

Even if the container is deleted, the volume persists:
```bash
docker rm postgres        # Container is gone
docker volume ls          # pgdata still exists ✅
```

Recreate the container pointing to the same volume to recover instantly:
```bash
docker run -d --name postgres -v pgdata:/var/lib/postgresql/data postgres:15
```

**Backup the volume to a file:**
```bash
docker run --rm \
  -v pgdata:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/pgdata-backup.tar.gz /data
```

{{< callout type="danger" title="Never store database data in the container layer" >}}
Data written inside a container (not in a volume) is destroyed when the container is removed. Always mount a volume for any stateful service.
{{< /callout >}}
{{< /qa >}}

{{< qa num="21" q="Multiple containers need to read from the same shared configuration files. How do you handle this?" level="intermediate" >}}
Mount the same directory or volume into multiple containers simultaneously:

**Using bind mount (host directory):**
```bash
docker run -d --name app1 -v $(pwd)/config:/etc/config:ro app-image
docker run -d --name app2 -v $(pwd)/config:/etc/config:ro app-image
docker run -d --name app3 -v $(pwd)/config:/etc/config:ro app-image
```

**Using a named volume:**
```bash
docker volume create shared-config

# Pre-populate the volume
docker run --rm -v shared-config:/config -v $(pwd)/config:/src alpine cp -r /src/. /config/

# Mount read-only in all app containers
docker run -d --name app1 -v shared-config:/etc/config:ro app-image
docker run -d --name app2 -v shared-config:/etc/config:ro app-image
```

{{< callout type="tip" title="Always mount shared config as read-only" >}}
The `:ro` flag prevents any container from accidentally modifying shared configuration. Without it, one misbehaving container could corrupt config for all others.
{{< /callout >}}
{{< /qa >}}

{{< qa num="22" q="You have a web app, a Redis cache, and a PostgreSQL database. How do you define all of them in Docker Compose?" level="intermediate" >}}
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://postgres:secret@db:5432/appdb
      - REDIS_URL=redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - app-net
    restart: unless-stopped

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-net

  redis:
    image: redis:7-alpine
    networks:
      - app-net

volumes:
  pgdata:

networks:
  app-net:
```

**Start all services:**
```bash
docker compose up -d

# View running services
docker compose ps

# View logs for all services
docker compose logs -f
```
{{< /qa >}}

{{< qa num="23" q="Your Docker Compose app works locally but the database isn't ready when the app starts, causing connection errors. How do you fix this?" level="intermediate" >}}
`depends_on` only waits for the **container to start**, not for the service inside it to be ready to accept connections. Fix this with **healthchecks**:

```yaml
services:
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 10
      start_period: 10s   # Grace period before health checks begin

  app:
    build: .
    depends_on:
      db:
        condition: service_healthy   # Wait until health check passes
```

**For MySQL:**
```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 10s
  timeout: 5s
  retries: 5
```

**For Redis:**
```yaml
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
  interval: 5s
  retries: 5
```

{{< callout type="tip" title="Also add retry logic in your application" >}}
Healthchecks handle the startup race condition, but your app should also implement connection retries with exponential backoff for production resilience — the database can become unavailable at any time, not just at startup.
{{< /callout >}}
{{< /qa >}}

{{< qa num="24" q="You want to run different Compose configurations for development vs production. How do you structure this?" level="intermediate" >}}
Use **Compose file overrides** — a base file with environment-specific overrides layered on top:

```yaml
# docker-compose.yml — base shared config
services:
  web:
    image: my-app
    environment:
      - NODE_ENV=production
  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

```yaml
# docker-compose.dev.yml — development overrides
services:
  web:
    build: .                    # Build from source instead of pulling image
    volumes:
      - .:/app                  # Live reload — mount source code
    environment:
      - NODE_ENV=development
    ports:
      - "9229:9229"             # Node.js debugger port
```

```yaml
# docker-compose.prod.yml — production overrides
services:
  web:
    image: myrepo/my-app:latest  # Pull versioned image
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 512m
```

**Run with the appropriate override:**
```bash
# Development
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

Later overrides win on any conflicting keys, and non-conflicting keys are merged.
{{< /qa >}}

{{< qa num="25" q="Your security team says containers should not run as root. How do you implement this?" level="advanced" >}}
Add a dedicated non-root user in your Dockerfile:

```dockerfile
FROM node:18-alpine

# Create a non-root group and user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app
COPY --chown=appuser:appgroup . .
RUN npm install

# Switch to non-root user before CMD
USER appuser

CMD ["node", "index.js"]
```

**Verify the container runs as non-root:**
```bash
docker run my-app whoami
# appuser
```

**Additionally, harden the container at runtime:**
```bash
docker run \
  --read-only \              # Immutable root filesystem
  --no-new-privileges \      # Prevent privilege escalation via setuid
  --cap-drop ALL \           # Drop all Linux capabilities
  --cap-add NET_BIND_SERVICE \ # Add back only what you need
  --security-opt=no-new-privileges \
  my-app
```

{{< callout type="info" title="Use Pod Security Standards in Kubernetes" >}}
If deploying to Kubernetes, enforce non-root using `securityContext.runAsNonRoot: true` and `securityContext.readOnlyRootFilesystem: true` in your Pod spec.
{{< /callout >}}
{{< /qa >}}

{{< qa num="26" q="You want to ensure Docker images in production come only from trusted registries and are verified. How do you implement this?" level="advanced" >}}
**1. Enable Docker Content Trust (image signing):**
```bash
# Enable globally in your shell or CI environment
export DOCKER_CONTENT_TRUST=1

# Push automatically signs the image
docker push myrepo/my-app:latest
```

**2. Pin images to immutable digests (not mutable tags):**
```dockerfile
# Tags can be overwritten at any time — digests cannot
FROM node:18-alpine@sha256:a9e6b0b7a36c5da51d5a0d7c95bb3bba29c3b5e1bd0b5a1e9e9c9c0c0c0c0c0
```

**3. Scan images for vulnerabilities before deploying:**
```bash
# Docker Scout (built into Docker Desktop)
docker scout cves my-app:latest

# Trivy (popular open-source scanner)
trivy image my-app:latest
```

**4. Use a private registry with access control:**
- **AWS ECR** — IAM-based access, integrated with ECS/EKS
- **Harbor** — self-hosted, built-in Trivy scanning, policy enforcement
- **JFrog Artifactory** — enterprise registry with audit trails

{{< callout type="tip" title="Block unverified images at the cluster level" >}}
In Kubernetes, use **Admission Controllers** (OPA Gatekeeper or Kyverno) to reject pods that use images from untrusted registries or without verified digests.
{{< /callout >}}
{{< /qa >}}

{{< qa num="27" q="A developer accidentally pushed an image with hardcoded credentials to Docker Hub. What do you do?" level="advanced" >}}
**Immediate response — move fast:**

**Step 1 — Revoke the exposed credentials immediately**
- Change passwords, rotate API keys, revoke tokens — do this before anything else

**Step 2 — Delete the image from Docker Hub**
- Docker Hub → Repository → Tags → Delete the affected tag
- Note: Docker Hub does not support private image deletion via CLI — use the web UI

**Step 3 — Push a clean replacement image**
```bash
# Build without secrets
docker build -t myrepo/my-app:latest .
docker push myrepo/my-app:latest
```

**Step 4 — Audit who pulled the image**
- Check Docker Hub pull statistics and access logs
- Review cloud provider access logs for any use of the exposed credentials

**Step 5 — Scan all other images for secrets**
```bash
trufflehog docker --image myrepo/my-app:latest
```

**Prevention going forward:**
```bash
# .dockerignore — exclude secret files from build context
.env
*.pem
secrets/
credentials.json

# Pre-commit hook to detect secrets
pip install detect-secrets
detect-secrets scan > .secrets.baseline
```

{{< callout type="danger" title="Deleting the tag is not enough" >}}
Even after deleting the image from Docker Hub, anyone who pulled the image before deletion still has it locally. Treat all exposed credentials as permanently compromised and rotate them immediately.
{{< /callout >}}
{{< /qa >}}

{{< qa num="28" q="Your container is running slow. How do you diagnose and fix performance issues?" level="advanced" >}}
**Step 1 — Monitor in real time:**
```bash
docker stats <container_id>
# Shows: CPU%, MEM usage/limit, NET I/O, BLOCK I/O
```

**Step 2 — Inspect resource limits:**
```bash
docker inspect <container_id> | grep -A 10 "HostConfig"
```

**Step 3 — Profile inside the container:**
```bash
docker exec -it <container_id> top         # CPU and process list
docker exec -it <container_id> df -h       # Disk space
docker exec -it <container_id> free -m     # Memory breakdown
docker exec -it <container_id> iostat 1 5  # Disk I/O
```

**Step 4 — Check for CPU throttling:**
```bash
docker exec <container_id> cat /sys/fs/cgroup/cpu/cpu.stat
# throttled_time > 0 means container is CPU-constrained
```

**Common fixes:**

| Symptom | Fix |
|---------|-----|
| High CPU% | Increase `--cpus`, optimise app code |
| Memory near limit | Increase `--memory`, fix memory leak |
| Slow disk I/O | Use named volumes instead of bind mounts on Docker Desktop |
| High NET I/O | Check for chatty connections, enable keep-alive |
| Image too large | Multi-stage build, alpine base image |
{{< /qa >}}

{{< qa num="29" q="A container that was running fine is suddenly unable to connect to the internet. How do you troubleshoot?" level="intermediate" >}}
**Step 1 — Test basic connectivity from inside the container:**
```bash
docker exec -it <container_id> ping 8.8.8.8
docker exec -it <container_id> curl https://google.com
```

**Step 2 — Check DNS resolution:**
```bash
docker exec -it <container_id> nslookup google.com
docker exec -it <container_id> cat /etc/resolv.conf
```

**Step 3 — Inspect the container's network configuration:**
```bash
docker inspect <container_id> | grep -A 20 "Networks"
docker network inspect <network_name>
```

**Step 4 — Check Docker daemon DNS config:**
```bash
cat /etc/docker/daemon.json
```

**Common causes and fixes:**

| Cause | Fix |
|-------|-----|
| No network attached | `docker network connect bridge <container>` |
| DNS not resolving | Add `"dns": ["8.8.8.8"]` to `/etc/docker/daemon.json` |
| Host firewall blocking | Check `iptables -L` and `ufw status` on host |
| Network marked `--internal` | Recreate network without `--internal` flag |
| Docker daemon restarted and reset `iptables` | Restart the container |

```bash
# Restart Docker daemon after changing daemon.json
sudo systemctl restart docker
```
{{< /qa >}}

{{< qa num="30" q="Your Docker build fails midway with 'no space left on device.' What do you do?" level="intermediate" >}}
**Step 1 — Check Docker's disk usage:**
```bash
docker system df
# Shows: images, containers, volumes, build cache sizes
```

**Step 2 — Clean up aggressively:**
```bash
# Remove stopped containers, unused images, unused networks, build cache
docker system prune -a --volumes
```

**Step 3 — Check host disk space:**
```bash
df -h
du -sh /var/lib/docker/*   # Find what's using space in Docker's directory
```

**Step 4 — Optimize Dockerfile to avoid the problem recurring:**
```dockerfile
# Clean package caches in the SAME RUN layer to avoid bloating intermediate layers
RUN apt-get update && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*

# Use multi-stage builds to discard build-time dependencies
FROM golang:1.21 AS builder
RUN go build -o app .

FROM alpine:3.18
COPY --from=builder /app .   # Final image has no Go toolchain
```

**Step 5 — Set up periodic cleanup on CI servers:**
```bash
# Add to crontab: clean Docker resources weekly
0 3 * * 0 docker system prune -af --volumes
```
{{< /qa >}}

{{< qa num="31" q="You need to set up a CI/CD pipeline that builds, tests, and pushes a Docker image on every git push. How do you design it?" level="advanced" >}}
**GitHub Actions example — build, test, push on every push to main:**

```yaml
# .github/workflows/docker.yml
name: Build and Push Docker Image

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Build image and run tests
        run: |
          docker build -t my-app:test .
          docker run --rm my-app:test npm test

      - name: Build and push final image
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: |
            myrepo/my-app:latest
            myrepo/my-app:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

**Best practices:**
- Tag images with both `latest` and the commit SHA for full traceability
- Run tests before pushing — fail fast if they don't pass
- Use GitHub Actions GHA cache for faster builds
- Store Docker Hub credentials as encrypted repository secrets — never hardcode them

{{< callout type="tip" title="Add vulnerability scanning before push" >}}
Insert a `docker scout cves` or `trivy image` step between the test and push steps to block images with critical CVEs from reaching production.
{{< /callout >}}
{{< /qa >}}

{{< qa num="32" q="Your production Docker container needs to be updated with zero downtime. How do you achieve this with plain Docker (no Kubernetes)?" level="advanced" >}}
Use a **blue-green deployment** strategy with a reverse proxy (Nginx or Traefik) in front:

```bash
#!/bin/bash
# deploy.sh

NEW_IMAGE="my-app:v2"
OLD_CONTAINER="my-app-blue"
NEW_CONTAINER="my-app-green"

# Step 1 — Pull the new image
docker pull $NEW_IMAGE

# Step 2 — Start the green (new) container on a different port
docker run -d \
  --name $NEW_CONTAINER \
  -p 3001:3000 \
  $NEW_IMAGE

# Step 3 — Wait and health check the new container
sleep 10
if curl -sf http://localhost:3001/health; then
  echo "Green container is healthy. Switching traffic..."

  # Step 4 — Update Nginx upstream to point to new container
  # sed -i 's/3000/3001/' /etc/nginx/conf.d/app.conf && nginx -s reload

  # Step 5 — Remove the old container
  docker stop $OLD_CONTAINER
  docker rm $OLD_CONTAINER

  # Step 6 — Rename green to blue (canonical name)
  docker rename $NEW_CONTAINER my-app-blue
  echo "✅ Deployment successful!"
else
  echo "❌ Health check failed. Rolling back..."
  docker stop $NEW_CONTAINER
  docker rm $NEW_CONTAINER
fi
```

{{< callout type="info" title="Use Docker Swarm for rolling updates without scripting" >}}
`docker service update --image my-app:v2 my-service` performs rolling updates natively with configurable `--update-parallelism` and automatic rollback on health check failures.
{{< /callout >}}
{{< /qa >}}

{{< qa num="33" q="You are running a stateful application (e.g. PostgreSQL) in Docker. How do you perform a backup and restore strategy?" level="advanced" >}}
**Backup — SQL dump:**
```bash
# Dump a single database to a timestamped file
docker exec postgres pg_dump -U postgres mydb > backup_$(date +%Y%m%d_%H%M%S).sql

# Dump all databases
docker exec postgres pg_dumpall -U postgres > full-backup-$(date +%Y%m%d).sql
```

**Backup — full volume snapshot:**
```bash
# Spin up a temporary container to tar the volume
docker run --rm \
  -v pgdata:/source:ro \
  -v $(pwd)/backups:/backup \
  alpine tar czf /backup/pgdata-$(date +%Y%m%d).tar.gz -C /source .
```

**Restore — from SQL dump:**
```bash
docker exec -i postgres psql -U postgres mydb < backup_20260407.sql
```

**Restore — from volume snapshot:**
```bash
docker run --rm \
  -v pgdata:/target \
  -v $(pwd)/backups:/backup \
  alpine tar xzf /backup/pgdata-20260407.tar.gz -C /target
```

**Automate with a host cron job:**
```bash
# /etc/cron.d/postgres-backup
0 2 * * * root docker exec postgres pg_dumpall -U postgres \
  > /backups/full-backup-$(date +\%Y\%m\%d).sql
```

{{< callout type="tip" title="Test your restores regularly" >}}
A backup you have never tested is not a backup — it is an assumption. Schedule a monthly restore drill to a test container to verify your backups are actually valid.
{{< /callout >}}
{{< /qa >}}

{{< qa num="34" q="Your team is using Docker in production and a container keeps getting OOM killed. How do you handle this?" level="advanced" >}}
**Step 1 — Confirm it is OOM killed:**
```bash
docker inspect <container_id> | grep -i oom
# "OOMKilled": true

# Check kernel logs for more detail
dmesg | grep -i "oom\|killed"
```

**Step 2 — Monitor current memory usage:**
```bash
docker stats --no-stream <container_id>
```

**Step 3 — Choose a fix based on root cause:**

**Option A — Increase the memory limit (quick fix):**
```bash
docker run --memory=2g --memory-swap=2g my-app
```

**Option B — Fix an application memory leak (proper fix):**
- Generate a heap dump and analyse with language-specific profilers
- Implement pagination instead of loading all records at once
- Tune garbage collection settings (e.g. for JVM: `-Xmx1g -XX:+UseG1GC`)

**Option C — Control eviction priority with OOM score:**
```bash
# Critical service — less likely to be killed when host is under pressure
docker run --oom-score-adj=-500 critical-service

# Low priority batch job — kill this first
docker run --oom-score-adj=500 batch-job
```

{{< callout type="warn" title="--memory-swap gotcha" >}}
If you set `--memory=512m` without setting `--memory-swap`, Docker defaults to `--memory-swap=1024m` (double). Set `--memory-swap` equal to `--memory` to disable swap entirely, or explicitly higher to allow it.
{{< /callout >}}
{{< /qa >}}

{{< qa num="35" q="You need to run Docker inside a Docker container (DinD) for CI/CD purposes. What are the approaches and risks?" level="advanced" >}}
**Approach 1 — Docker-in-Docker (true DinD):**
```bash
docker run --privileged docker:dind
```
The inner container runs its own Docker daemon. Requires `--privileged` flag which grants near-full host access — a significant security risk.

**Approach 2 — Docker socket mount (recommended for most CI use cases):**
```bash
docker run -v /var/run/docker.sock:/var/run/docker.sock docker:cli
```
The inner container talks to the **host's Docker daemon** directly. No privilege escalation needed. Simpler and faster, but the container has full control over all containers on the host.

**Approach 3 — Kaniko (best for Kubernetes/restricted environments):**
```yaml
# Build Docker images without any Docker daemon access
- name: Build with Kaniko
  image: gcr.io/kaniko-project/executor
  args:
    - --context=dir:///workspace
    - --dockerfile=Dockerfile
    - --destination=myrepo/my-app:latest
```
Kaniko builds images entirely in userspace — no daemon, no privileges, works inside unprivileged containers and Kubernetes pods.

**Approach 4 — Podman (daemonless, rootless):**
```bash
podman build -t my-app .
podman run my-app
```
Podman is a drop-in Docker replacement that runs without a daemon and supports rootless containers natively.

**Risk comparison:**

| Approach | Security Risk | Speed | Complexity |
|----------|--------------|-------|------------|
| DinD (`--privileged`) | 🔴 High | Fast | Medium |
| Socket mount | 🟡 Medium | Fast | Low |
| Kaniko | 🟢 Low | Medium | Medium |
| Rootless Podman | 🟢 Low | Fast | High |

{{< callout type="tip" title="Recommendation" >}}
Use **socket mount** for simple CI pipelines. Use **Kaniko** when running on Kubernetes or when you need to eliminate any risk of container escape. Avoid true DinD in production CI unless you have a compelling reason.
{{< /callout >}}
{{< /qa >}}

</div>

---

## Quick Reference Cheatsheet

```bash
# Container lifecycle
docker run -d --name app -p 8080:80 nginx     # Run detached with name
docker start / stop / restart app              # Control a container
docker rm -f app                               # Force remove running container

# Debugging
docker logs -f app                             # Follow logs in real time
docker exec -it app /bin/sh                    # Open a shell inside container
docker inspect app                             # Full metadata JSON
docker stats                                   # Live CPU/memory/network usage
docker top app                                 # Processes inside container
docker diff app                                # Filesystem changes vs image

# Images
docker build -t myapp:v1 .                    # Build from Dockerfile
docker pull / push myrepo/myapp:v1            # Registry operations
docker images                                  # List local images
docker rmi myapp:v1                           # Remove image
docker history myapp:v1                       # Show layer breakdown

# Networks
docker network create mynet                    # Create a network
docker network connect mynet app               # Connect container to network
docker network ls / inspect / rm               # Manage networks

# Volumes
docker volume create mydata                    # Create a named volume
docker volume ls / inspect / rm                # Manage volumes
docker run -v mydata:/app/data myapp           # Mount volume in container

# Cleanup
docker system prune -a --volumes               # Remove everything unused
docker system df                               # Show disk usage breakdown
```
