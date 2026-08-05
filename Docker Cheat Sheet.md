# Docker Cheat Sheet

> Comprehensive Docker reference covering basics, images, containers, networking, volumes, Docker Compose, and system administration.

---

# Basics

## **Docker**
Docker is a platform for packaging applications and their dependencies into lightweight, portable units called containers. Containers run consistently across environments.

ASCII analogy:

```text
Application + Dependencies
           |
           v
     +-----------+
     | Container |
     +-----------+
           |
           v
      Docker Engine
           |
           v
      Host Machine
```

## **Image**
An image is a read-only blueprint used to create containers.

```text
Image --> Container --> Running Process
```

## **Container**
A container is a running instance of an image.

```bash
docker run nginx          # Create and start a container from nginx image
docker ps                 # List running containers
docker stop <container>   # Stop a running container
```

### Common Docker CLI Structure

```bash
docker <object> <command> [options]   # General command format
```

| Object | Purpose |
|----------|----------|
| image | Manage images |
| container | Manage containers |
| network | Manage networks |
| volume | Manage volumes |
| compose | Multi-container applications |
| system | Docker system operations |

---

# Images

## **Docker Image Lifecycle**

```text
Dockerfile
    |
    v
 docker build
    |
    v
   Image
    |
    v
 docker run
    |
    v
 Container
```

### Pull Images

```bash
docker pull nginx:latest      # Download image from registry
docker pull python:3.12       # Download specific version
```

### List Images

```bash
docker images                 # Show local images
docker image ls               # Alternative syntax
```

### Build Images

```bash
docker build -t myapp:1.0 .   # Build image from current directory
```

### Tag Images

```bash
docker tag myapp:1.0 myrepo/myapp:latest   # Create another tag
```

### Push Images

```bash
docker login                  # Authenticate to registry
docker push myrepo/myapp:latest   # Upload image
```

### Remove Images

```bash
docker rmi myapp:1.0          # Delete image
```

> **Warning:** Removing an image may fail if containers still depend on it.

### Example Dockerfile

```dockerfile
FROM python:3.12-slim                     # Base image
WORKDIR /app                              # Working directory
COPY . .                                  # Copy project files
RUN pip install -r requirements.txt       # Install dependencies
CMD ["python", "app.py"]                # Default command
```

### Useful Image Flags

| Flag | Meaning |
|------|---------|
| -t | Assign tag |
| -f | Specify Dockerfile |
| --no-cache | Ignore build cache |
| -q | Quiet output |

---

# Containers

## **Container States**

```text
Created -> Running -> Stopped -> Removed
```

### Run Containers

```bash
docker run nginx                           # Run container
docker run -d nginx                        # Detached mode
docker run -p 8080:80 nginx                # Port mapping
docker run --name web nginx                # Named container
docker run -it ubuntu bash                 # Interactive shell
```

### Inspect Running Containers

```bash
docker ps                                  # Running containers
docker ps -a                               # All containers
```

### Logs

```bash
docker logs web                            # View logs
docker logs -f web                         # Follow logs
```

### Execute Commands Inside Container

```bash
docker exec -it web bash                   # Interactive shell
```

### Start and Stop

```bash
docker stop web                            # Graceful stop
docker start web                           # Start existing container
docker restart web                         # Restart container
```

### Remove Containers

```bash
docker rm web                              # Delete stopped container
docker rm -f web                           # Force removal
```

> **Warning:** Force removal stops the container immediately and may interrupt active workloads.

### Container Flags

| Flag | Meaning |
|------|---------|
| -d | Detached mode |
| -it | Interactive terminal |
| -p | Publish ports |
| --name | Assign container name |
| -e | Environment variable |
| --rm | Auto-remove after exit |

---

# Network

## **Docker Network**
Docker networks allow containers to communicate securely.

```text
Container A ---- Bridge Network ---- Container B
```

### List Networks

```bash
docker network ls                          # Show networks
```

### Create Network

```bash
docker network create app-net              # Create bridge network
```

### Connect Containers

```bash
docker run -d --name db --network app-net mysql
docker run -d --name api --network app-net myapi
```

### Inspect Network

```bash
docker network inspect app-net             # Detailed configuration
```

### Remove Network

```bash
docker network rm app-net                  # Delete network
```

> **Warning:** Networks cannot be removed while containers are attached.

### Network Types

| Type | Use Case |
|------|----------|
| bridge | Default single-host communication |
| host | Direct host networking |
| none | No networking |
| overlay | Multi-host swarm networking |

---

# Volumes

## **Volume**
Volumes store persistent data outside container lifecycles.

```text
Container
    |
    v
 Volume
    |
    v
 Persistent Data
```

### Create Volume

```bash
docker volume create db-data               # Create volume
```

### List Volumes

```bash
docker volume ls                           # View volumes
```

### Mount Volume

```bash
docker run -d \
  -v db-data:/var/lib/mysql \
  mysql                                     # Persist database files
```

### Inspect Volume

```bash
docker volume inspect db-data              # View metadata
```

### Remove Volume

```bash
docker volume rm db-data                   # Delete volume
```

> **Warning:** Removing a volume permanently deletes stored data.

### Bind Mount vs Volume

| Feature | Bind Mount | Volume |
|----------|----------|----------|
| Managed by Docker | No | Yes |
| Host Path Required | Yes | No |
| Portability | Lower | Higher |
| Recommended for Production | Sometimes | Yes |

---

# Compose

## **Docker Compose**
Compose defines and runs multi-container applications using YAML.

```text
compose.yaml
     |
     v
 docker compose up
     |
     v
Multiple Connected Containers
```

### Example compose.yaml

```yaml
services:
  web:
    image: nginx                # Web server
    ports:
      - "8080:80"              # Port mapping

  db:
    image: mysql:8              # Database service
    environment:
      MYSQL_ROOT_PASSWORD: root # Root password
```

### Compose Commands

```bash
docker compose up -d           # Start services
docker compose down            # Stop and remove services
docker compose ps              # Service status
docker compose logs            # View logs
docker compose restart         # Restart services
```

### Common Compose Flags

| Flag | Meaning |
|------|---------|
| -d | Detached mode |
| -f | Specify compose file |
| --build | Rebuild images |
| --remove-orphans | Remove old services |

### Remove Stack

```bash
docker compose down -v         # Remove containers and volumes
```

> **Warning:** The -v flag deletes attached volumes and stored data.

---

# System

## **Docker System Management**
System commands help track resource usage and clean unused objects.

### View Information

```bash
docker info                    # Detailed engine info
docker version                 # Client/server versions
```

### Disk Usage

```bash
docker system df               # Docker disk consumption
```

### Cleanup Unused Resources

```bash
docker image prune             # Remove dangling images
docker container prune         # Remove stopped containers
docker volume prune            # Remove unused volumes
docker network prune           # Remove unused networks
docker system prune            # General cleanup
```

> **Warning:** Prune commands delete unused resources and may not be reversible.

### Aggressive Cleanup

```bash
docker system prune -a         # Remove all unused images/resources
```

> **Warning:** This can delete images that are not currently in use.

### Common Errors

| Error | Cause | Fix |
|---------|---------|---------|
| Port already allocated | Port in use | Use another port or stop process |
| Container name exists | Duplicate name | Remove or rename container |
| Image not found | Missing image/tag | Pull correct image |
| Permission denied | Docker permissions | Add user to docker group |
| Volume in use | Active attachment | Stop container first |

---

# Real-World Workflows

## Scenario 1: Deploy a Web Application

```bash
docker build -t myweb:1.0 .          # Build image
docker run -d \
  --name myweb \
  -p 8080:80 \
  myweb:1.0                          # Run application

docker logs -f myweb                 # Monitor logs
```

## Scenario 2: Run Application with Database

```bash
docker network create app-net        # Shared network

docker volume create mysql-data      # Persistent storage

docker run -d \
  --name mysql \
  --network app-net \
  -v mysql-data:/var/lib/mysql \
  mysql:8

docker run -d \
  --name api \
  --network app-net \
  myapi:latest
```

## Scenario 3: Full Stack with Compose

```bash
docker compose up -d                 # Start application stack

docker compose ps                    # Check status

docker compose logs -f               # Monitor services

docker compose down                  # Stop stack
```

---

# Quick Reference

| Task | Command |
|------|---------|
| List images | docker images |
| Pull image | docker pull nginx |
| Build image | docker build -t app . |
| Remove image | docker rmi image |
| Run container | docker run image |
| Run in background | docker run -d image |
| List running containers | docker ps |
| List all containers | docker ps -a |
| View logs | docker logs container |
| Enter container | docker exec -it container bash |
| Stop container | docker stop container |
| Restart container | docker restart container |
| Remove container | docker rm container |
| Create network | docker network create net |
| List networks | docker network ls |
| Create volume | docker volume create data |
| List volumes | docker volume ls |
| Start compose stack | docker compose up -d |
| Stop compose stack | docker compose down |
| Clean unused resources | docker system prune |
