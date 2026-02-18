# Docker + Host Visibility Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Enable Glances to display all running Docker containers and real host system metrics (CPU, RAM, disk) instead of only the container's own system info.

**Architecture:** Add the `docker` Python package to the image so Glances can communicate with the Docker daemon via the already-mounted socket. Add read-only host path mounts (`/proc`, `/sys`, `/etc/os-release`) and environment variables (`HOST_PROC`, `HOST_SYS`) so Glances reads host-level metrics instead of container-scoped ones.

**Tech Stack:** Alpine Linux, Python 3, pip, Glances, Docker Python SDK, Docker Compose v3.8

---

### Task 1: Update Dockerfile to include Docker Python package

**Files:**
- Modify: `Dockerfile`

**Step 1: Edit the pip install line to add `docker` package**

Change:
```dockerfile
&& pip3 install --no-cache-dir --break-system-packages glances[web] \
```

To:
```dockerfile
&& pip3 install --no-cache-dir --break-system-packages glances[web] docker \
```

Full updated Dockerfile for reference:
```dockerfile
FROM alpine:latest

RUN apk add --no-cache \
    python3 \
    py3-pip \
    gcc \
    python3-dev \
    musl-dev \
    linux-headers \
    && pip3 install --no-cache-dir --break-system-packages glances[web] docker \
    && apk del gcc python3-dev musl-dev linux-headers

EXPOSE 61208

CMD ["glances", "-w", "-B", "0.0.0.0"]
```

**Step 2: Verify the change looks correct**

Run: `cat Dockerfile`
Expected: pip install line includes both `glances[web]` and `docker`

**Step 3: Commit**

```bash
git add Dockerfile
git commit -m "feat: add docker python package for container visibility"
```

---

### Task 2: Update docker-compose.yml with host mounts and env vars

**Files:**
- Modify: `docker-compose.yml`

**Step 1: Update docker-compose.yml**

Replace the entire file with:
```yaml
version: '3.8'

services:
  glances:
    build: .
    container_name: glances-server
    restart: unless-stopped
    ports:
      - "61208:61208"
    environment:
      - TZ=UTC
      - HOST_PROC=/host/proc
      - HOST_SYS=/host/sys
    pid: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /etc/os-release:/etc/os-release:ro
    privileged: false
```

**Step 2: Verify the change looks correct**

Run: `cat docker-compose.yml`
Expected: 4 volume mounts and 3 environment variables (TZ, HOST_PROC, HOST_SYS)

**Step 3: Commit**

```bash
git add docker-compose.yml
git commit -m "feat: mount host proc/sys for real host metrics and add env vars"
```

---

### Task 3: Build and verify locally

**Step 1: Build the image**

Run: `docker compose build --no-cache`
Expected: Build completes successfully, no errors during pip install

**Step 2: Start the container**

Run: `docker compose up -d`
Expected: Container starts, `docker compose ps` shows it as running

**Step 3: Check Glances web UI**

Open browser at: `http://localhost:61208`

Verify:
- "Docker" section is visible and lists running containers
- CPU/RAM/disk metrics reflect the host machine (not just container limits)

**Step 4: Check container logs for errors**

Run: `docker compose logs glances`
Expected: No errors about missing docker module or permission denied on /host/proc

**Step 5: Stop the container**

Run: `docker compose down`

---

### Task 4: Final commit and push

**Step 1: Verify git status is clean**

Run: `git status`
Expected: nothing to commit (all changes committed in Tasks 1 and 2)

**Step 2: Push to remote**

Run: `git push`
Expected: Branch pushed, GitHub Actions workflow triggers to build and push Docker image
