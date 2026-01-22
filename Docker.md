# Docker Cheat Sheet & Walkthrough

## Installation (Arch)

```bash
sudo pacman -S docker docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
# Log out and back in for group to take effect
```

Verify it works:

```bash
docker run hello-world
```

---

## Core Concepts

|Term|What it is|
|---|---|
|**Image**|Read-only template (like a snapshot/ISO). You pull or build these.|
|**Container**|Running instance of an image. Ephemeral by default.|
|**Volume**|Persistent storage that survives container destruction.|
|**Network**|Virtual network for containers to communicate.|
|**Dockerfile**|Recipe to build a custom image.|
|**Compose**|Tool to define multi-container apps in YAML.|

---

## Essential Commands

### Images

```bash
docker pull nginx                    # Download image
docker images                        # List local images
docker rmi nginx                     # Remove image
docker build -t myapp .              # Build from Dockerfile in current dir
```

### Containers

```bash
docker run nginx                     # Run (foreground, blocks terminal)
docker run -d nginx                  # Run detached (background)
docker run -d -p 8080:80 nginx       # Map host:container ports
docker run -d --name webserver nginx # Give it a name
docker run --rm nginx                # Auto-remove when stopped

docker ps                            # List running containers
docker ps -a                         # List all (including stopped)
docker stop webserver                # Stop by name or ID
docker start webserver               # Start stopped container
docker restart webserver             # Restart
docker rm webserver                  # Remove (must be stopped)
docker rm -f webserver               # Force remove (kills if running)

docker logs webserver                # View stdout/stderr
docker logs -f webserver             # Follow logs (like tail -f)

docker exec -it webserver bash       # Shell into running container
docker exec webserver cat /etc/nginx/nginx.conf  # Run single command
```

### Volumes

```bash
docker volume create mydata          # Create named volume
docker volume ls                     # List volumes
docker volume rm mydata              # Remove volume

# Mount volume to container
docker run -d -v mydata:/var/lib/mysql mysql

# Bind mount (host path)
docker run -d -v /home/mike/site:/usr/share/nginx/html nginx
```

### Cleanup

```bash
docker system prune                  # Remove unused containers, networks, images
docker system prune -a               # Above + all unused images
docker volume prune                  # Remove unused volumes
```

---

## Dockerfile Basics

Create a file named `Dockerfile` (no extension):

```dockerfile
FROM ubuntu:22.04

# Install packages
RUN apt-get update && apt-get install -y \
    nginx \
    && rm -rf /var/lib/apt/lists/*

# Copy files from build context into image
COPY ./html /usr/share/nginx/html

# Set working directory
WORKDIR /app

# Expose port (documentation, doesn't actually publish)
EXPOSE 80

# Command to run when container starts
CMD ["nginx", "-g", "daemon off;"]
```

Build and run:

```bash
docker build -t my-nginx .
docker run -d -p 8080:80 my-nginx
```

---

## Docker Compose

This is where it gets useful. Define your whole stack in one file.

Create `docker-compose.yml`:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    depends_on:
      - app

  app:
    build: ./app
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/myapp
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=myapp
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### Compose Commands

```bash
docker compose up                    # Start all services (foreground)
docker compose up -d                 # Start detached
docker compose down                  # Stop and remove containers
docker compose down -v               # Above + remove volumes (wipes data!)

docker compose ps                    # List services
docker compose logs                  # View all logs
docker compose logs -f app           # Follow specific service

docker compose build                 # Rebuild images
docker compose pull                  # Pull latest images

docker compose exec app bash         # Shell into running service
docker compose run app npm install   # Run one-off command in new container
```

---

## Practical Walkthrough: Local Dev Stack

Let's set up a simple web project with nginx serving static files and a PostgreSQL database.

### 1. Create project structure

```bash
mkdir -p ~/docker-test/{html,data}
cd ~/docker-test
```

### 2. Create a test page

```bash
echo '<h1>Docker is working</h1>' > html/index.html
```

### 3. Create docker-compose.yml

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: devpass
      POSTGRES_DB: testdb
    volumes:
      - ./data/postgres:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    restart: unless-stopped

  adminer:
    image: adminer
    ports:
      - "8081:8080"
    depends_on:
      - db
```

### 4. Start it

```bash
docker compose up -d
```

### 5. Verify

- http://localhost:8080 — Your nginx site
- http://localhost:8081 — Adminer (web UI for database)
    - Server: `db`
    - User: `dev`
    - Password: `devpass`
    - Database: `testdb`

### 6. Make changes

Edit `html/index.html` — changes appear immediately (bind mount).

### 7. Stop when done

```bash
docker compose down      # Keeps data
docker compose down -v   # Wipes database
```

---

## Tips

**View resource usage:**

```bash
docker stats
```

**Copy files in/out of container:**

```bash
docker cp webserver:/etc/nginx/nginx.conf ./nginx.conf
docker cp ./myfile.txt webserver:/tmp/
```

**Inspect container details:**

```bash
docker inspect webserver
```

**See what's eating disk space:**

```bash
docker system df
```

**Run as non-root in container (more secure):**

```dockerfile
RUN useradd -m appuser
USER appuser
```

---

## Migration to Droplet

When ready to deploy:

1. Push your images to Docker Hub or GitHub Container Registry
2. Copy your `docker-compose.yml` to the droplet
3. Install Docker on the droplet
4. `docker compose up -d`

Or just copy the compose file as documentation and install services directly if you prefer bare metal.