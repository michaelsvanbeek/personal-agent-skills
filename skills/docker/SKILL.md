---
name: docker
description: 'Docker and container best practices with security-first, rootless design. Use when: writing Dockerfiles, building container images, creating docker-compose files, hardening containers, setting up local dev environments, configuring container registries, auditing existing Dockerfiles for security and size, or improving container build pipelines.'
tags:
  - developer
  - operations
---

# Docker and Container Standards

## When to Use

- Writing or reviewing Dockerfiles
- Building container images for deployment
- Creating docker-compose configurations for local dev
- Hardening containers for production
- Setting up CI/CD container pipelines
- Auditing existing Dockerfiles for security issues (running as root, missing health checks, bloated images)
- Reducing image size via multi-stage builds and dependency cleanup
- Reviewing docker-compose files for isolation, volume, and network hygiene

## Core Principle: Minimal, Rootless, Isolated

Every container should run with the **least privilege**, the **smallest attack surface**, and **no capability** beyond what its process requires.

---

## Dockerfile Best Practices

### Multi-Stage Builds

Always use multi-stage builds to separate build dependencies from the runtime image:

```dockerfile
# Stage 1: Build
FROM python:3.12-slim AS builder

WORKDIR /app
COPY pyproject.toml .
RUN pip install --no-cache-dir --target=/deps .

COPY src/ src/

# Stage 2: Runtime
FROM python:3.12-slim AS runtime

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser -d /app -s /sbin/nologin appuser

WORKDIR /app
COPY --from=builder /deps /usr/local/lib/python3.12/site-packages/
COPY --from=builder /app/src src/

USER appuser
EXPOSE 8000
CMD ["python", "-m", "uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Node.js Example

```dockerfile
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json .
RUN npm ci --production=false
COPY . .
RUN npm run build

FROM node:22-alpine AS runtime
RUN addgroup -S appuser && adduser -S appuser -G appuser
WORKDIR /app
COPY --from=builder /app/dist dist/
COPY --from=builder /app/node_modules node_modules/
COPY --from=builder /app/package.json .
USER appuser
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

### Layer Optimization

- Copy dependency manifests first (`package.json`, `pyproject.toml`), install, then copy source. This maximizes layer caching.
- Combine `RUN` commands with `&&` to reduce layers.
- Use `--no-cache-dir` (pip) and `npm ci` to avoid caching download artifacts in the image.
- Order instructions from least to most frequently changing.

### Image Selection

- Use **`-slim`** or **`-alpine`** base images. Never use full OS images.
- Pin image tags to specific versions: `python:3.12.3-slim`, not `python:latest`.
- Prefer official images from Docker Hub or verified publishers.

---

## Rootless Containers

**Never run containers as root.** This is the single most impactful security measure.

### Rules

- Every Dockerfile must include a `USER` instruction that switches to a non-root user before `CMD`/`ENTRYPOINT`.
- Create the user in the Dockerfile:
  ```dockerfile
  # Debian/Ubuntu-based
  RUN groupadd -r appuser && useradd -r -g appuser -d /app -s /sbin/nologin appuser
  
  # Alpine-based
  RUN addgroup -S appuser && adduser -S appuser -G appuser
  ```
- Application files should be owned by root and readable by the app user. Only specific directories (data, logs, tmp) should be writable:
  ```dockerfile
  COPY --chown=root:root src/ src/
  RUN mkdir -p /app/data && chown appuser:appuser /app/data
  ```
- Never use `chmod 777`. Use the minimum permissions needed.

### Rootless Docker Daemon

For production hosts, prefer **rootless Docker mode** or **Podman** (rootless by default):

```bash
# Install rootless Docker
dockerd-rootless-setuptool.sh install

# Or use Podman (drop-in replacement)
podman run --rm -p 8000:8000 myimage
```

---

## Container Security

### Drop Capabilities

Containers inherit Linux capabilities they don't need. Drop all and add back only what's required:

```yaml
# docker-compose.yml
services:
  app:
    cap_drop:
      - ALL
    cap_add: []  # add specific caps only if truly needed
    security_opt:
      - no-new-privileges:true
```

### Read-Only Filesystem

Mount the container filesystem as read-only. Provide writable volumes only for directories that need writes:

```yaml
services:
  app:
    read_only: true
    tmpfs:
      - /tmp
    volumes:
      - app-data:/app/data
```

### Resource Limits

Always set memory and CPU limits to prevent a runaway container from starving the host:

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: "1.0"
        reservations:
          memory: 256M
          cpus: "0.5"
```

### Network Isolation

- Place services on **internal networks** that are not exposed to the host unless needed.
- Only expose ports that must be reachable externally.
- Use service names for inter-container communication, not host networking.

```yaml
networks:
  backend:
    internal: true  # not reachable from host
  frontend:
    # reachable from host

services:
  api:
    networks: [backend, frontend]
    ports: ["8000:8000"]
  db:
    networks: [backend]  # only reachable by api, not from host
```

### Image Scanning

Scan images for known vulnerabilities before deploying:

```bash
# Docker Scout (built-in)
docker scout cves myimage:latest

# Trivy (open source)
trivy image myimage:latest

# Grype
grype myimage:latest
```

- Run scans in CI on every image build.
- Block deployment if critical/high severity CVEs are found.
- Rebuild images regularly to pick up base image security patches.

---

## .dockerignore

Always include a `.dockerignore` to exclude unnecessary files from the build context:

```
.git
.gitignore
.env
.env.*
node_modules
__pycache__
*.pyc
.pytest_cache
.mypy_cache
.ruff_cache
dist
build
.serverless
*.md
.vscode
.idea
docker-compose*.yml
Dockerfile*
```

---

## Docker Compose for Local Development

### Structure

```yaml
version: "3.9"

services:
  app:
    build:
      context: .
      target: runtime
    ports:
      - "8000:8000"
    environment:
      - ENVIRONMENT=dev
    env_file:
      - .env
    volumes:
      - ./src:/app/src  # hot-reload in dev
    depends_on:
      db:
        condition: service_healthy
    cap_drop: [ALL]
    security_opt: [no-new-privileges:true]
    read_only: true
    tmpfs: [/tmp]

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 5s
      timeout: 3s
      retries: 5
    cap_drop: [ALL]
    cap_add: [CHOWN, SETUID, SETGID, FOWNER, DAC_READ_SEARCH]
    networks: [backend]

networks:
  backend:
    internal: true

volumes:
  db-data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### Best Practices

- Use `healthcheck` on service dependencies. Use `depends_on` with `condition: service_healthy`.
- Use Docker secrets or env_file for credentials. Never put secrets directly in the compose file.
- Mount source code as a volume for hot-reload in development. Do not do this in production images.
- Use named volumes for persistent data (databases, uploads).

---

## Image Naming and Tagging

```
registry/organization/service:tag
```

- Tag with **semantic version** and **git SHA**: `myapp:1.2.3`, `myapp:abc1234`.
- Always tag the latest stable build as `myapp:latest` in addition to the version tag.
- Never deploy `:latest` in production — always use a specific version or SHA tag.
- Use a consistent registry per organization.

---

## Health Checks

Every container should include a health check:

```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1
```

Use a lightweight check (dedicated `/health` endpoint) that verifies the process is responsive, not a full dependency check.

---

## Compose Audit Checklist

Use this checklist when reviewing docker-compose files (homelab, production, or development):

### Security Audit

| Check | What to look for |
|-------|-----------------|
| **No root** | Every service has `user:` set or the image runs as non-root by default. If unsure, add `user: "1000:1000"`. |
| **Capabilities dropped** | Every service has `cap_drop: [ALL]`. Only add back specific caps with `cap_add:` when required. |
| **No new privileges** | Every service has `security_opt: [no-new-privileges:true]`. |
| **Read-only filesystem** | Every service has `read_only: true` with explicit `tmpfs:` and writable `volumes:` for dirs that need writes. |
| **No privileged mode** | No service uses `privileged: true`. Find the specific capability needed instead. |
| **No host networking** | No service uses `network_mode: host`. Use bridge networks with explicit port mapping. |
| **No host PID/IPC** | No service uses `pid: host` or `ipc: host`. |
| **Secrets not in env** | Passwords and tokens use Docker secrets (`secrets:`) or `_FILE` env conventions, not plaintext `environment:`. |
| **Limited ports** | Only expose ports that need external access. Use internal networks for service-to-service communication. |
| **Resource limits** | Every service has `deploy.resources.limits` for memory and CPU. |
| **Pinned images** | Image tags use specific versions, not `:latest`. |

### Reliability Audit

| Check | What to look for |
|-------|-----------------|
| **Health checks** | Every service has a `healthcheck:` with appropriate `interval`, `timeout`, `retries`. |
| **Dependency ordering** | Services use `depends_on:` with `condition: service_healthy`, not just service name. |
| **Restart policy** | Every service has `restart: unless-stopped` or `restart: on-failure`. |
| **Named volumes** | Persistent data uses named volumes, not bind mounts to host paths (except dev hot-reload). |
| **Logging config** | Services have `logging:` configured with `max-size` and `max-file` to prevent disk fill. |
| **Backup strategy** | Volumes containing data (databases, configs) have a documented backup method. |

### Network Audit

| Check | What to look for |
|-------|-----------------|
| **Internal networks** | Backend databases and services are on `internal: true` networks. |
| **Segmentation** | Frontend and backend on separate networks. Only proxy/gateway bridges them. |
| **No default network** | If using multiple networks, don't rely on the default network — name them explicitly. |

### Example: Hardened Homelab Service

```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin:10.9.4
    user: "1000:1000"
    read_only: true
    cap_drop: [ALL]
    security_opt: [no-new-privileges:true]
    tmpfs:
      - /tmp
    volumes:
      - jellyfin-config:/config
      - jellyfin-cache:/cache
      - /media/library:/media:ro  # media is read-only
    ports:
      - "8096:8096"
    networks: [frontend]
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 4G
          cpus: "2.0"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8096/health"]
      interval: 30s
      timeout: 5s
      retries: 3
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

networks:
  frontend:

volumes:
  jellyfin-config:
  jellyfin-cache:
```

### Common Homelab Fixes

| Service Type | Typical issue | Fix |
|-------------|--------------|-----|
| Media servers | Run as root | Set `user: "1000:1000"` and `chown` data dirs |
| Databases | No resource limits | Add memory limit (Postgres: 1-2G, Redis: 256M-1G) |
| Reverse proxies | `privileged: true` for port 80/443 | Use `cap_add: [NET_BIND_SERVICE]` instead |
| Download clients | Need host network | Use bridge + explicit port mapping where possible |
| Monitoring | Mount Docker socket | Use read-only mount: `/var/run/docker.sock:/var/run/docker.sock:ro` and consider socket proxy |

### Docker Socket Security

If a service requires the Docker socket (monitoring tools, Portainer, Traefik):

- **Never** mount the socket without understanding the implications — socket access = root access to the host.
- Mount read-only: `/var/run/docker.sock:/var/run/docker.sock:ro`
- Prefer a **socket proxy** (e.g., Tecnativa/docker-socket-proxy) that filters API calls:

```yaml
services:
  docker-proxy:
    image: tecnativa/docker-socket-proxy:0.2
    environment:
      CONTAINERS: 1
      NETWORKS: 0
      VOLUMES: 0
      IMAGES: 0
      EXEC: 0
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks: [backend]
    cap_drop: [ALL]
    security_opt: [no-new-privileges:true]
    read_only: true

  traefik:
    environment:
      DOCKER_HOST: tcp://docker-proxy:2375
    networks: [backend, frontend]
    # no docker socket mount needed
```
