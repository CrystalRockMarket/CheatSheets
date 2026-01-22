# Docker Compose

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Key Concepts](#key-concepts)
4. [Docker Compose File](#docker-compose-file)
5. [Command Reference](#command-reference)
6. [Advanced Usage](#advanced-usage)
7. [Networking](#networking)
8. [Volume Management](#volume-management)
9. [Environment Variables](#environment-variables)
10. [Troubleshooting](#troubleshooting)
11. [Quick Reference](#quick-reference)

---

## Introduction

Docker Compose is a tool for defining and running multi-container Docker applications. It uses YAML files to configure application services and provides commands to start, stop, and rebuild all services with a single command.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Docker Compose                           │
│                    (docker-compose.yml)                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Service   │  │   Service   │  │   Service   │         │
│  │    Web      │──│    DB       │──│   Redis     │         │
│  │  (Container)│  │ (Container) │  │ (Container) │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│        │               │                │                   │
│        └───────────────┴────────────────┘                   │
│                         │                                   │
│              ┌──────────┴──────────┐                       │
│              │   Shared Network    │                       │
│              └─────────────────────┘                       │
│                         │                                   │
│              ┌──────────┴──────────┐                       │
│              │  Shared Volumes     │                       │
│              └─────────────────────┘                       │
└─────────────────────────────────────────────────────────────┘
```

### When to Use Docker Compose

- Development environments
- Automated testing
- Single-host deployments
- CI/CD pipelines
- Staging environments

---

## Installation

### Linux (Ubuntu/Debian)

```bash
# Install Docker Compose plugin (recommended)
sudo apt update
sudo apt install docker-compose-plugin

# Or standalone binary
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### macOS

```bash
# Included with Docker Desktop
brew install docker-compose
```

### Windows

```bash
# Included with Docker Desktop
choco install docker-compose
```

### Verify Installation

```bash
docker compose version
# Docker Compose version v2.24.0

docker-compose --version
# docker-compose version 1.29.2, build unknown
```

---

## Key Concepts

### Services

A service is a container definition that includes:
- Container image or build configuration
- Port mappings
- Volume mounts
- Environment variables
- Dependencies
- Restart policy

### Networks

Docker Compose creates a bridge network by default, allowing containers to communicate by service name.

### Volumes

Named volumes persist data independently of container lifecycle.

### Profiles

Profiles allow defining groups of services that can be started together.

---

## Docker Compose File

### Basic Structure

```yaml
version: '3.8'  # File version

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Version Compatibility

| Compose File Version | Docker Engine Version |
|---------------------|----------------------|
| 3.9                 | 20.10+               |
| 3.8                 | 19.03+               |
| 3.7                 | 18.06+               |
| 3.6                 | 18.02+               |
| 3.5                 | 17.12+               |
| 2.x                 | 1.13+                |

### Service Definition

```yaml
services:
  myapp:
    # Build from Dockerfile
    build:
      context: ./app
      dockerfile: Dockerfile
      args:
        - BUILD_ENV=production

    # Or use existing image
    image: myapp:latest

    # Container name
    container_name: myapp_container

    # Restart policy
    restart: unless-stopped

    # Environment variables
    environment:
      - NODE_ENV=production
      - DB_HOST=db
      - DB_PORT=5432
    env_file:
      - .env.production

    # Port mapping
    ports:
      - "3000:3000"
      - "3001:3001"

    # Volume mounts
    volumes:
      - ./src:/app/src
      - app_data:/data

    # Dependencies
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

    # Health check
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 5s

    # Networks
    networks:
      - frontend
      - backend

    # Labels
    labels:
      - "com.docker.compose.project=mysite"
      - "com.docker.compose.service=web"

    # Logging
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

### Named Volumes

```yaml
volumes:
  # Empty volume
  postgres_data:

  # Volume with driver options
  app_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /data/app

  # External volume (pre-created)
  external_volume:
    external: true
```

### Networks

```yaml
networks:
  # Default bridge network
  default:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
          gateway: 172.28.0.1

  # Overlay network for swarm
  overlay_network:
    driver: overlay
    attachable: true

  # External network
  external_network:
    external: true
```

---

## Command Reference

### Up/Down

```bash
# Start all services (creates network, volumes)
docker compose up -d

# Start with build
docker compose up -d --build

# Start specific services
docker compose up -d web db

# Start and recreate containers
docker compose up -d --force-recreate

# Start with no color output
docker compose up -d --no-color

# Start in foreground (attach)
docker compose up

# Stop and remove containers, networks
docker compose down

# Stop, remove, and delete volumes
docker compose down -v

# Stop, remove, networks, and images
docker compose down --rmi all

# Stop, remove, everything including named volumes
docker compose down -v --rmi all
```

### Build

```bash
# Build all images
docker compose build

# Build specific service
docker compose build web

# Build with no cache
docker compose build --no-cache

# Build with build arguments
docker compose build --build-arg BUILD_ENV=production
```

### Logs

```bash
# View logs (follow)
docker compose logs -f

# View logs for specific service
docker compose logs -f web

# View last N lines
docker compose logs --tail 100

# View logs with timestamps
docker compose logs -t

# Since duration
docker compose logs --since 1h
```

### Exec

```bash
# Execute command in running container
docker compose exec web sh

# Execute with specific user
docker compose exec -u root web ls /app

# Execute without TTY
docker compose exec web python manage.py migrate
```

### Scale

```bash
# Scale service (deprecated in v2.x, use replicas)
docker compose up -d --scale web=3

# For v3.x, use deploy.replicas
docker compose up -d
```

### Pull

```bash
# Pull all images
docker compose pull

# Pull specific service
docker compose pull db

# Pull with parallel
docker compose pull --parallel

# Pull missing images only
docker compose pull --quiet
```

### Config

```bash
# Validate and show merged config
docker compose config

# Show full resolved config
docker compose config --full

# Convert to canonical format
docker compose config > docker-compose.yml

# Print version
docker compose version
```

### Restart

```bash
# Restart all services
docker compose restart

# Restart specific service
docker compose restart web

# Restart with timeout
docker compose restart -t 30
```

### Start/Stop

```bash
# Start stopped services
docker compose start

# Stop running services
docker compose stop

# Stop with timeout
docker compose stop -t 60

# Pause services
docker compose pause

# Unpause services
docker compose unpause
```

### PS

```bash
# List running containers
docker compose ps

# List all containers (including stopped)
docker compose ps -a

# Show only service names
docker compose ps --services

# Filter by status
docker compose ps --filter "status=running"
```

### Top

```bash
# Show running processes
docker compose top

# For specific service
docker compose top web
```

### Events

```bash
# Stream container events
docker compose events

# With JSON output
docker compose events --json
```

### Port

```bash
# Print public port for service port
docker compose port web 80
# 0.0.0.0:8080
```

### Kill

```bash
# Force stop containers
docker compose kill

# Signal specific service
docker compose kill -s SIGINT web
```

### rm

```bash
# Remove stopped service containers
docker compose rm

# Remove without confirmation
docker compose rm -f

# Remove volumes too
docker compose rm -v
```

---

## Advanced Usage

### Multiple Compose Files

```bash
# Base config
docker compose -f docker-compose.yml up -d

# Override for development
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d

# Override for production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### extends Keyword

```yaml
# docker-compose.base.yml
services:
  webapp:
    build: .
    environment:
      - DEBUG=1
    depends_on:
      - db

# docker-compose.yml
services:
  webapp:
    extends:
      file: docker-compose.base.yml
      service: webapp
    ports:
      - "8000:8000"
    environment:
      - DEBUG=0
```

### Multiple Compose Files Precedence

Order matters (later files override earlier):
1. docker-compose.yml
2. docker-compose.override.yml (auto-loaded)
3. CLI -f files

### Profiles

```yaml
services:
  web:
    image: nginx
    profiles:
      - frontend

  db:
    image: postgres
    profiles:
      - backend

  redis:
    image: redis
    profiles:
      - cache

  # No profile - always started
  monitoring:
    image: prometheus
```

```bash
# Start only frontend profile
docker compose --profile frontend up -d

# Start multiple profiles
docker compose --profile frontend --profile cache up -d
```

### init Containers

```yaml
services:
  web:
    image: nginx
    init: true

    init_config:
      # Init image to use
      image: busybox
      # Command to run
      command: /bin/sh -c "echo init done"
```

### Long Syntax for Options

```yaml
services:
  web:
    image: nginx
    command: ["nginx", "-g", "daemon off;"]
    entrypoint: /bin/sh
    hostname: webhost
    domainname: example.com
    user: 1000:1000
    working_dir: /app
    privileged: true
    read_only: true
    tty: true
    stdin_open: true
```

---

## Networking

### Service Discovery

Containers can communicate using service names as hostnames.

```yaml
services:
  web:
    image: nginx
    ports:
      - "80:80"

  api:
    image: node
    environment:
      - DB_HOST=db  # Service name!
      - DB_PORT=5432
    depends_on:
      - db

  db:
    image: postgres
```

### Custom Networks

```yaml
services:
  web:
    image: nginx
    networks:
      - frontend

  api:
    image: node
    networks:
      - frontend
      - backend

  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.21.0.0/16
```

### Network Aliases

```yaml
services:
  web:
    image: nginx
    networks:
      app_network:
        aliases:
          - webserver
          - www
```

### External Network

```yaml
networks:
  external_network:
    external: true
```

### DNS Settings

```yaml
services:
  web:
    image: nginx
    dns:
      - 8.8.8.8
      - 8.8.4.4

    dns_search:
      - example.com

    domainname: example.com
```

---

## Volume Management

### Named Volumes

```yaml
services:
  db:
    image: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Bind Mounts

```yaml
services:
  web:
    image: nginx
    volumes:
      # Read-write bind mount
      - ./html:/usr/share/nginx/html:ro

      # Read-only bind mount
      - ./config:/etc/nginx:ro

      # Bind mount with propagation
      - ./data:/data:rw,shared
```

### tmpfs Mounts

```yaml
services:
  web:
    image: nginx
    tmpfs:
      - /tmp
      - /run:size=10M,uid=1000
```

### Volume Driver

```yaml
volumes:
  data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /external/storage

  s3_data:
    driver: rexray/s3fs
    driver_opts:
      s3bucket: mybucket
```

### Volume Configuration File

```yaml
volumes:
  data:
    driver: local

# docker-compose.volumes.yml
volumes:
  data:
    driver_opts:
      type: none
      o: bind
      device: /data/$(PWD)
```

---

## Environment Variables

### Environment File

```bash
# .env
POSTGRES_PASSWORD=secretpassword
REDIS_HOST=redis
API_KEY=abc123
DEBUG=true
```

```yaml
services:
  web:
    image: node
    env_file:
      - .env
    environment:
      - NODE_ENV=${NODE_ENV:-development}
      # Literal value
      - SERVICE_NAME=${SERVICE_NAME?SERVICE_NAME must be set}
```

### Variable Substitution

```yaml
services:
  web:
    image: nginx
    ports:
      - "${WEB_PORT:-8080}:80"
    environment:
      - APP_NAME=${APP_NAME:-myapp}
      - DB_URL=postgresql://user:pass@${DB_HOST:-db}:5432/${DB_NAME:-app}
```

### .env File Location

- Current directory
- Parent directories (up to filesystem root)
- Explicit path with --env-file

### Environment Variable Precedence

1. Compose file
2. Shell environment variables
3. .env file
4. Docker CLI environment

---

## Troubleshooting

### Debug Commands

```bash
# Validate compose file
docker compose config

# Show all service configurations
docker compose config --services

# Check service status
docker compose ps

# View service logs
docker compose logs web

# Execute into container
docker compose exec web sh

# Inspect container
docker compose exec web cat /etc/hosts
```

### Common Issues

**Port Conflicts**

```bash
# Check what's using the port
sudo lsof -i :80

# Use different port
docker compose up -d -p 8080:80
```

**Volume Permission Issues**

```bash
# Fix ownership
docker compose exec web chown -R 1000:1000 /data

# Or run as root
docker compose exec -u root web chown -R 1000:1000 /data
```

**Container Won't Start**

```bash
# Check logs
docker compose logs web

# Check health status
docker compose ps

# Inspect container
docker inspect myapp_web_1
```

**Network Issues**

```bash
# List networks
docker network ls

# Inspect network
docker network inspect myapp_default

# Check DNS resolution
docker compose exec web nslookup db
```

**Image Pull Failures**

```bash
# Pull manually
docker compose pull

# Check authentication
docker login

# Use different registry
docker compose pull --quiet
```

### Reset Everything

```bash
# Complete reset
docker compose down -v --rmi all --remove-orphans
docker system prune -a -f
docker volume prune -f
```

---

## Quick Reference

### Command Summary

| Command | Description |
|---------|-------------|
| `docker compose up -d` | Start all services |
| `docker compose down` | Stop all services |
| `docker compose logs -f` | Follow logs |
| `docker compose exec <svc> sh` | Shell into service |
| `docker compose ps` | List containers |
| `docker compose restart <svc>` | Restart service |
| `docker compose build` | Build images |
| `docker compose pull` | Pull images |
| `docker compose config` | Validate config |
| `docker compose logs --tail 100` | Last 100 log lines |

### File Reference

| File | Purpose |
|------|---------|
| `docker-compose.yml` | Main compose file |
| `docker-compose.override.yml` | Auto-loaded overrides |
| `docker-compose.dev.yml` | Development overrides |
| `docker-compose.prod.yml` | Production overrides |
| `.env` | Environment variables |

### Service Keys

| Key | Type | Description |
|-----|------|-------------|
| `image` | string | Docker image name |
| `build` | mapping | Build configuration |
| `ports` | list | Port mappings |
| `volumes` | list | Volume mounts |
| `environment` | list/dict | Environment vars |
| `env_file` | list | Env files to load |
| `depends_on` | list | Service dependencies |
| `networks` | list | Network assignments |
| `restart` | string | Restart policy |
| `healthcheck` | mapping | Health check config |
| `profiles` | list | Profile assignments |

### Restart Policies

| Policy | Behavior |
|--------|----------|
| `no` | Never restart |
| `always` | Always restart |
| `on-failure` | Restart on failure |
| `unless-stopped` | Restart unless stopped |

### Common Patterns

**Web App + DB + Cache**

```yaml
services:
  web:
    image: nginx
    ports: ["80:80"]
    depends_on:
      api:
        condition: service_healthy

  api:
    build: ./api
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
    environment:
      - DB_HOST=db
      - REDIS_HOST=redis

  db:
    image: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis

volumes:
  postgres_data:
```

**Multi-Environment**

```yaml
# docker-compose.yml
services:
  web:
    image: ${IMAGE_TAG:-nginx:latest}

# docker-compose.override.yml (dev)
services:
  web:
    ports: ["8080:80"]
    environment:
      - DEBUG=1

# docker-compose.prod.yml (prod)
services:
  web:
    ports: ["80:80", "443:443"]
    environment:
      - DEBUG=0
```

---

## See Also

- [Docker Documentation](https://docs.docker.com/)
- [Compose Specification](https://compose-spec.io/)
- [Docker Compose CLI](https://docs.docker.com/compose/reference/)
