# 🐳 Docker Cheatsheet

> A comprehensive, beginner-to-advanced Docker reference guide. Bookmark it, fork it, share it!

---

## 📚 Table of Contents

1. [What is Docker?](#-what-is-docker)
2. [Installation](#-installation)
3. [Docker Architecture](#-docker-architecture)
4. [Docker CLI Basics](#-docker-cli-basics)
5. [Images](#-images)
6. [Containers](#-containers)
7. [Volumes](#-volumes)
8. [Networks](#-networks)
9. [Dockerfile Reference](#-dockerfile-reference)
10. [Docker Compose](#-docker-compose)
11. [Registry & Hub](#-registry--hub)
12. [System & Maintenance](#-system--maintenance)
13. [Environment Variables](#-environment-variables)
14. [Logs & Monitoring](#-logs--monitoring)
15. [Docker Contexts](#-docker-contexts)
16. [Security Tips](#-security-tips)
17. [Common Use Cases](#-common-use-cases)
18. [Quick Reference Card](#-quick-reference-card)

---

## 🐋 What is Docker?

Docker is an open-source platform that enables developers to **build, ship, and run applications in containers** — lightweight, portable, self-sufficient environments that bundle code with all its dependencies.

| Concept | Description |
|---|---|
| **Image** | A read-only blueprint/template used to create containers |
| **Container** | A running instance of an image |
| **Dockerfile** | A script with instructions to build a Docker image |
| **Registry** | A storage/distribution system for Docker images (e.g., Docker Hub) |
| **Volume** | Persistent storage that survives container restarts |
| **Network** | Virtual network that allows containers to communicate |

---

## 🛠 Installation

### Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER   # Run Docker without sudo
```

### macOS / Windows

Download and install **[Docker Desktop](https://www.docker.com/products/docker-desktop/)**.

### Verify Installation

```bash
docker --version
docker info
docker run hello-world
```

---

## 🏗 Docker Architecture

```
┌────────────────────────────────────────────┐
│              Docker Client (CLI)           │
│         docker build / run / pull          │
└────────────────┬───────────────────────────┘
                 │ REST API
┌────────────────▼───────────────────────────┐
│            Docker Daemon (dockerd)         │
│  ┌─────────┐ ┌──────────┐ ┌────────────┐  │
│  │ Images  │ │Containers│ │  Volumes   │  │
│  └─────────┘ └──────────┘ └────────────┘  │
└────────────────────────────────────────────┘
                 │
┌────────────────▼───────────────────────────┐
│          Docker Registry (Hub)             │
└────────────────────────────────────────────┘
```

---

## 🖥 Docker CLI Basics

```bash
docker version          # Show Docker version info
docker info             # Display system-wide info
docker help             # List all commands
docker <command> --help # Help for a specific command
```

---

## 🖼 Images

### Pulling & Listing

```bash
docker pull <image>               # Pull image from registry
docker pull nginx:1.25            # Pull specific tag
docker images                     # List all local images
docker images -a                  # Include intermediate images
docker image ls                   # Same as docker images
```

### Building

```bash
docker build -t myapp:1.0 .                  # Build from current directory
docker build -t myapp:1.0 -f Dockerfile.dev . # Use custom Dockerfile
docker build --no-cache -t myapp .           # Build without cache
docker build --build-arg ENV=prod -t myapp . # Pass build arguments
```

### Inspecting & Tagging

```bash
docker inspect <image>              # Detailed image info (JSON)
docker history <image>              # Show image layer history
docker tag myapp:1.0 myapp:latest   # Tag an image
docker image prune                  # Remove dangling images
docker rmi <image>                  # Remove an image
docker rmi $(docker images -q)      # Remove all images
```

---

## 📦 Containers

### Running Containers

```bash
docker run <image>                        # Run a container
docker run -d <image>                     # Run in detached (background) mode
docker run -it <image> bash               # Interactive terminal
docker run --name mycontainer <image>     # Assign a name
docker run -p 8080:80 <image>             # Map host:container port
docker run -P <image>                     # Map all exposed ports randomly
docker run --rm <image>                   # Auto-remove on exit
docker run -e ENV_VAR=value <image>       # Set environment variable
docker run -v /host/path:/container/path <image>  # Bind mount volume
docker run --network mynet <image>        # Attach to a network
docker run --restart always <image>       # Restart policy
```

### Restart Policies

| Policy | Description |
|---|---|
| `no` | Never restart (default) |
| `always` | Always restart |
| `on-failure` | Restart only on non-zero exit code |
| `unless-stopped` | Restart unless explicitly stopped |

### Managing Containers

```bash
docker ps                     # List running containers
docker ps -a                  # List all containers (including stopped)
docker start <container>      # Start a stopped container
docker stop <container>       # Gracefully stop a container
docker restart <container>    # Restart a container
docker kill <container>       # Force kill a container
docker rm <container>         # Remove a stopped container
docker rm -f <container>      # Force remove running container
docker rm $(docker ps -aq)    # Remove all stopped containers
```

### Interacting with Containers

```bash
docker exec -it <container> bash          # Open shell in running container
docker exec <container> <command>         # Run a command in container
docker attach <container>                 # Attach to container's main process
docker cp <container>:/path ./local       # Copy file from container
docker cp ./local <container>:/path       # Copy file to container
docker diff <container>                   # Show changed files in container
docker top <container>                    # Show running processes
docker stats                              # Live resource usage stats
docker stats <container>                  # Stats for specific container
```

### Container Info

```bash
docker inspect <container>              # Detailed container info
docker port <container>                 # Show port mappings
docker logs <container>                 # Show logs
docker logs -f <container>              # Follow (tail) logs
docker logs --tail 100 <container>      # Last 100 lines
docker logs --since 10m <container>     # Logs from last 10 minutes
```

---

## 💾 Volumes

Volumes provide **persistent storage** that outlives container lifecycles.

### Volume Types

| Type | Syntax | Description |
|---|---|---|
| **Named Volume** | `-v myvolume:/data` | Managed by Docker, best for persistence |
| **Bind Mount** | `-v /host/path:/container/path` | Maps host directory directly |
| **tmpfs Mount** | `--tmpfs /tmp` | Stored in host memory only |

### Volume Commands

```bash
docker volume create myvolume           # Create a named volume
docker volume ls                        # List all volumes
docker volume inspect myvolume          # Inspect a volume
docker volume rm myvolume               # Remove a volume
docker volume prune                     # Remove all unused volumes

# Run container with volume
docker run -v myvolume:/app/data <image>

# Read-only mount
docker run -v myvolume:/app/data:ro <image>
```

---

## 🌐 Networks

### Network Drivers

| Driver | Description |
|---|---|
| `bridge` | Default. Isolated network on the host (for standalone containers) |
| `host` | Removes network isolation; container uses host's network |
| `none` | Disables all networking |
| `overlay` | Multi-host networking (Docker Swarm) |
| `macvlan` | Assigns a MAC address to the container |

### Network Commands

```bash
docker network create mynet                         # Create bridge network
docker network create --driver host mynet           # Create host network
docker network ls                                   # List networks
docker network inspect mynet                        # Inspect a network
docker network rm mynet                             # Remove a network
docker network prune                                # Remove all unused networks

docker network connect mynet <container>            # Connect container to network
docker network disconnect mynet <container>         # Disconnect container

# Run with network
docker run --network mynet <image>

# Containers on same bridge network can communicate by name
docker run --network mynet --name app1 <image>
docker run --network mynet --name app2 <image>
# app2 can reach app1 via http://app1
```

---

## 📄 Dockerfile Reference

A `Dockerfile` is a text file with instructions to build an image **layer by layer**.

### Instruction Reference

| Instruction | Description | Example |
|---|---|---|
| `FROM` | Base image | `FROM node:20-alpine` |
| `LABEL` | Add metadata | `LABEL version="1.0"` |
| `RUN` | Execute command during build | `RUN apt-get install -y curl` |
| `CMD` | Default command at runtime (overridable) | `CMD ["node", "app.js"]` |
| `ENTRYPOINT` | Fixed command at runtime | `ENTRYPOINT ["nginx", "-g"]` |
| `COPY` | Copy files from host to image | `COPY . /app` |
| `ADD` | Like COPY but supports URLs and tar extraction | `ADD app.tar.gz /app` |
| `WORKDIR` | Set working directory | `WORKDIR /app` |
| `ENV` | Set environment variable | `ENV PORT=3000` |
| `ARG` | Build-time variable | `ARG VERSION=latest` |
| `EXPOSE` | Document exposed port | `EXPOSE 3000` |
| `VOLUME` | Create a mount point | `VOLUME ["/data"]` |
| `USER` | Set user for subsequent instructions | `USER node` |
| `HEALTHCHECK` | Define a health check | `HEALTHCHECK CMD curl -f http://localhost/` |
| `ONBUILD` | Trigger instruction in child images | `ONBUILD RUN npm install` |
| `STOPSIGNAL` | Set stop signal | `STOPSIGNAL SIGTERM` |

### Example: Node.js App

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
ENV NODE_ENV=production
EXPOSE 3000
USER node
HEALTHCHECK --interval=30s --timeout=5s \
  CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "server.js"]
```

### Example: Python App

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1
EXPOSE 8000
CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:8000"]
```

### .dockerignore

```
node_modules
.git
.env
*.log
dist
__pycache__
```

---

## 🐙 Docker Compose

Docker Compose lets you define and run **multi-container applications** using a YAML file.

### Key Commands

```bash
docker compose up                 # Start all services
docker compose up -d              # Start in detached mode
docker compose up --build         # Build images before starting
docker compose down               # Stop and remove containers/networks
docker compose down -v            # Also remove volumes
docker compose ps                 # List running services
docker compose logs               # View all logs
docker compose logs -f app        # Follow logs for 'app' service
docker compose exec app bash      # Shell into service
docker compose run app npm test   # Run one-off command
docker compose build              # Build/rebuild services
docker compose pull               # Pull latest images
docker compose restart            # Restart all services
docker compose stop               # Stop without removing
docker compose start              # Start stopped services
docker compose config             # Validate and view config
```

### docker-compose.yml Reference

```yaml
version: "3.9"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - NODE_ENV=production
    image: myapp:latest
    container_name: my_app
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    env_file:
      - .env
    volumes:
      - ./src:/app/src
      - app_data:/app/data
    networks:
      - backend
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:16
    container_name: my_db
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    networks:
      - backend

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    networks:
      - backend

volumes:
  db_data:
  app_data:

networks:
  backend:
    driver: bridge
```

---

## 🗄 Registry & Hub

```bash
# Login / Logout
docker login                          # Login to Docker Hub
docker login registry.example.com    # Login to private registry
docker logout

# Push & Pull
docker tag myapp:1.0 username/myapp:1.0   # Tag for Hub
docker push username/myapp:1.0            # Push to Docker Hub
docker pull username/myapp:1.0            # Pull from Docker Hub

# Save & Load (offline transfer)
docker save -o myapp.tar myapp:1.0        # Export image to tar
docker load -i myapp.tar                  # Import image from tar

# Export/Import container (not recommended for images)
docker export <container> > mycontainer.tar
docker import mycontainer.tar myapp:imported
```

---

## 🧹 System & Maintenance

```bash
# Disk Usage
docker system df                    # Show Docker disk usage
docker system df -v                 # Verbose breakdown

# Cleanup
docker system prune                 # Remove stopped containers, unused networks, dangling images
docker system prune -a              # Also remove unused images
docker system prune -a --volumes    # Also remove volumes (⚠️ destructive)

docker container prune              # Remove all stopped containers
docker image prune                  # Remove dangling images
docker image prune -a               # Remove all unused images
docker volume prune                 # Remove unused volumes
docker network prune                # Remove unused networks
```

---

## 🔑 Environment Variables

```bash
# Single variable
docker run -e MY_VAR=hello <image>

# Multiple variables
docker run -e VAR1=a -e VAR2=b <image>

# From a file
docker run --env-file .env <image>
```

### Sample `.env` file

```env
NODE_ENV=production
DB_HOST=localhost
DB_PORT=5432
SECRET_KEY=supersecret
```

---

## 📊 Logs & Monitoring

```bash
# Logs
docker logs <container>                    # All logs
docker logs -f <container>                 # Follow logs in real time
docker logs --tail 50 <container>          # Last 50 lines
docker logs --since "2024-01-01" <container>  # Since date
docker logs --until "2024-12-31" <container>  # Until date
docker logs -t <container>                 # Include timestamps

# Stats (live CPU/Memory/Network usage)
docker stats                               # All running containers
docker stats <container1> <container2>     # Specific containers
docker stats --no-stream                   # One-time snapshot

# Events
docker events                              # Real-time event stream
docker events --since 10m                  # Events from last 10 minutes
docker events --filter type=container      # Filter by type
```

---

## 🌍 Docker Contexts

Contexts allow you to switch between different Docker environments (local, remote, cloud).

```bash
docker context ls                                    # List contexts
docker context use <context>                         # Switch context
docker context create myremote --docker "host=ssh://user@remote-host"
docker context inspect <context>                     # Inspect context
docker context rm <context>                          # Remove context
```

---

## 🔒 Security Tips

| Tip | Description |
|---|---|
| Use official images | Always prefer verified/official base images |
| Use specific tags | Avoid `latest`; pin to specific versions like `node:20.11` |
| Run as non-root | Use `USER` in Dockerfile to avoid running as root |
| Scan images | `docker scout cves <image>` or use Trivy/Snyk |
| Use `.dockerignore` | Prevent secrets and unnecessary files from entering the image |
| Read-only filesystem | `docker run --read-only <image>` |
| Limit resources | `docker run --memory 512m --cpus 1 <image>` |
| Use secrets | Use Docker secrets or env files instead of hardcoding credentials |
| Minimal base images | Prefer `alpine` or `distroless` to reduce attack surface |
| No privileged mode | Avoid `--privileged` unless absolutely necessary |

```bash
# Scan for vulnerabilities
docker scout cves <image>

# Resource limits
docker run --memory="256m" --cpus="0.5" <image>

# Read-only with writable tmpfs
docker run --read-only --tmpfs /tmp <image>
```

---

## 💡 Common Use Cases

### Run a temporary database

```bash
docker run -d --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  postgres:16
```

### Run a web server

```bash
docker run -d --name nginx \
  -p 80:80 \
  -v $(pwd)/html:/usr/share/nginx/html:ro \
  nginx:alpine
```

### Debug a running container

```bash
docker exec -it <container> sh
docker exec -it <container> bash
```

### Use Docker as a build environment

```bash
docker run --rm -v $(pwd):/app -w /app node:20 npm run build
```

### Multi-stage build for smaller images

```dockerfile
FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN go build -o app .

FROM gcr.io/distroless/base
COPY --from=builder /app/app /app
ENTRYPOINT ["/app"]
```

---

## ⚡ Quick Reference Card

### Images

| Command | Description |
|---|---|
| `docker pull <img>` | Pull image |
| `docker build -t name .` | Build image |
| `docker images` | List images |
| `docker rmi <img>` | Remove image |
| `docker tag src dst` | Tag image |
| `docker push <img>` | Push to registry |

### Containers

| Command | Description |
|---|---|
| `docker run <img>` | Run container |
| `docker run -d <img>` | Run detached |
| `docker run -it <img> sh` | Interactive shell |
| `docker ps` | List running |
| `docker ps -a` | List all |
| `docker stop <c>` | Stop container |
| `docker start <c>` | Start container |
| `docker rm <c>` | Remove container |
| `docker exec -it <c> sh` | Shell into container |
| `docker logs -f <c>` | Follow logs |
| `docker stats` | Live stats |

### Volumes

| Command | Description |
|---|---|
| `docker volume create v` | Create volume |
| `docker volume ls` | List volumes |
| `docker volume rm v` | Remove volume |
| `docker volume prune` | Remove unused |

### Networks

| Command | Description |
|---|---|
| `docker network create n` | Create network |
| `docker network ls` | List networks |
| `docker network connect n c` | Connect to network |
| `docker network rm n` | Remove network |

### Compose

| Command | Description |
|---|---|
| `docker compose up -d` | Start services |
| `docker compose down` | Stop & remove |
| `docker compose ps` | List services |
| `docker compose logs -f` | Follow logs |
| `docker compose exec s sh` | Shell into service |
| `docker compose build` | Build images |

### Cleanup

| Command | Description |
|---|---|
| `docker system prune -a` | Clean everything unused |
| `docker container prune` | Remove stopped containers |
| `docker image prune -a` | Remove unused images |
| `docker volume prune` | Remove unused volumes |

---

## 📖 Resources

- <a href="https://docs.docker.com/" target="_blank">Official Docker Documentation</a>
- <a href="https://hub.docker.com/" target="_blank">Docker Hub</a>
- <a href="https://labs.play-with-docker.com/" target="_blank">Play with Docker (Browser Playground)</a>
- <a href="https://docs.docker.com/compose/" target="_blank">Docker Compose Docs</a>
- <a href="https://docs.docker.com/develop/develop-images/dockerfile_best-practices/" target="_blank">Dockerfile Best Practices</a>
- <a href="https://docs.docker.com/engine/security/" target="_blank">Docker Security Guide</a>
---

<p align="center">
  ⭐ If this helped you, please give it a star! <br/>
  🐛 Found an error? Open an issue or submit a PR!
</p>
