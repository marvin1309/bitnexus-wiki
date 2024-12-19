---
title: 02 Docker Cheatsheet
description: 
published: true
date: 2024-09-21T09:38:29.420Z
tags: 
editor: markdown
dateCreated: 2024-09-21T09:37:03.502Z
---

# Docker Cheat Sheet: Essential Commands, Tips, and Tricks

This cheat sheet provides a quick reference to the most commonly used Docker commands, along with tips and tricks to enhance your Docker experience.

---

## Table of Contents

- [Docker Cheat Sheet: Essential Commands, Tips, and Tricks](#docker-cheat-sheet-essential-commands-tips-and-tricks)
  - [Table of Contents](#table-of-contents)
  - [Docker Basics](#docker-basics)
    - [Docker Versions](#docker-versions)
    - [Docker Help](#docker-help)
  - [Images](#images)
    - [Pulling Images](#pulling-images)
    - [Listing Images](#listing-images)
    - [Removing Images](#removing-images)
    - [Building Images](#building-images)
    - [Tagging Images](#tagging-images)
    - [Pushing Images](#pushing-images)
  - [Containers](#containers)
    - [Running Containers](#running-containers)
    - [Listing Containers](#listing-containers)
    - [Starting and Stopping Containers](#starting-and-stopping-containers)
    - [Removing Containers](#removing-containers)
    - [Inspecting Containers](#inspecting-containers)
    - [Logs](#logs)
    - [Executing Commands in Containers](#executing-commands-in-containers)
    - [Copying Files](#copying-files)
  - [Networks](#networks)
    - [Listing Networks](#listing-networks)
    - [Creating Networks](#creating-networks)
    - [Connecting Containers](#connecting-containers)
  - [Volumes](#volumes)
    - [Listing Volumes](#listing-volumes)
    - [Creating Volumes](#creating-volumes)
    - [Removing Volumes](#removing-volumes)
  - [Docker Compose](#docker-compose)
    - [Common Commands](#common-commands)
  - [Tips and Tricks](#tips-and-tricks)
    - [Cleaning Up Resources](#cleaning-up-resources)
    - [Dockerfile Tips](#dockerfile-tips)
    - [Optimizing Images](#optimizing-images)
    - [Security Best Practices](#security-best-practices)
    - [Networking Tips](#networking-tips)
    - [Logging and Monitoring](#logging-and-monitoring)
    - [Docker Compose Tips](#docker-compose-tips)
  - [Additional Resources](#additional-resources)

---

## Docker Basics

### Docker Versions

Check the installed Docker version:

```bash
docker --version
```

Check Docker Compose version:

```bash
docker-compose --version
```

### Docker Help

Get help on Docker commands:

```bash
docker --help
docker <command> --help
```

## Images

### Pulling Images

Pull an image from Docker Hub:

```bash
docker pull <image_name>
```

Example:

```bash
docker pull ubuntu:latest
```

### Listing Images

List all local images:

```bash
docker images
```

### Removing Images

Remove an image:

```bash
docker rmi <image_name_or_id>
```

Force remove an image:

```bash
docker rmi -f <image_name_or_id>
```

### Building Images

Build an image from a Dockerfile:

```bash
docker build -t <image_name>:<tag> .
```

Example:

```bash
docker build -t myapp:latest .
```

### Tagging Images

Tag an image for pushing to a registry:

```bash
docker tag <existing_image> <repository>/<image_name>:<tag>
```

Example:

```bash
docker tag myapp:latest myrepo/myapp:v1.0
```

### Pushing Images

Push an image to a registry:

```bash
docker push <repository>/<image_name>:<tag>
```

Example:

```bash
docker push myrepo/myapp:v1.0
```

## Containers

### Running Containers

Run a container interactively:

```bash
docker run -it <image_name> /bin/bash
```

Run a container in the background (detached):

```bash
docker run -d <image_name>
```

Run a container with port mapping:

```bash
docker run -p <host_port>:<container_port> <image_name>
```

Mount a volume:

```bash
docker run -v <host_dir>:<container_dir> <image_name>
```

Assign a name to a container:

```bash
docker run --name <container_name> <image_name>
```

### Listing Containers

List running containers:

```bash
docker ps
```

List all containers (including stopped):

```bash
docker ps -a
```

### Starting and Stopping Containers

Start a stopped container:

```bash
docker start <container_id_or_name>
```

Stop a running container:

```bash
docker stop <container_id_or_name>
```

Restart a container:

```bash
docker restart <container_id_or_name>
```

Pause and unpause a container:

```bash
docker pause <container_id_or_name>
docker unpause <container_id_or_name>
```

### Removing Containers

Remove a stopped container:

```bash
docker rm <container_id_or_name>
```

Force remove a running container:

```bash
docker rm -f <container_id_or_name>
```

Remove all stopped containers:

```bash
docker container prune
```

### Inspecting Containers

Inspect container details:

```bash
docker inspect <container_id_or_name>
```

Get container IP address:

```bash
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container_id_or_name>
```

### Logs

View container logs:

```bash
docker logs <container_id_or_name>
```

Follow container logs (real-time):

```bash
docker logs -f <container_id_or_name>
```

Tail last N lines:

```bash
docker logs --tail <number_of_lines> <container_id_or_name>
```

### Executing Commands in Containers

Run a command in a running container:

```bash
docker exec -it <container_id_or_name> /bin/bash
```

Run a specific command:

```bash
docker exec -it <container_id_or_name> <command>
```

### Copying Files

Copy files from container to host:

```bash
docker cp <container_id_or_name>:/path/to/file /host/path
```

Copy files from host to container:

```bash
docker cp /host/path <container_id_or_name>:/path/to/file
```

## Networks

### Listing Networks

List all Docker networks:

```bash
docker network ls
```

### Creating Networks

Create a new network:

```bash
docker network create <network_name>
```

Run a container on a specific network:

```bash
docker run --network <network_name> <image_name>
```

### Connecting Containers

Connect a container to a network:

```bash
docker network connect <network_name> <container_id_or_name>
```

Disconnect a container from a network:

```bash
docker network disconnect <network_name> <container_id_or_name>
```

## Volumes

### Listing Volumes

List all Docker volumes:

```bash
docker volume ls
```

### Creating Volumes

Create a new volume:

```bash
docker volume create <volume_name>
```

Use a volume in a container:

```bash
docker run -v <volume_name>:/container/path <image_name>
```

### Removing Volumes

Remove a volume:

```bash
docker volume rm <volume_name>
```

Remove all unused volumes:

```bash
docker volume prune
```

## Docker Compose

### Common Commands

Start services in the background:

```bash
docker-compose up -d
```

Start services and follow logs:

```bash
docker-compose up
```

Stop services:

```bash
docker-compose down
```

Stop and remove containers, networks, images, and volumes:

```bash
docker-compose down --rmi all --volumes
```

List running services:

```bash
docker-compose ps
```

View service logs:

```bash
docker-compose logs
docker-compose logs -f
```

Execute a command in a service container:

```bash
docker-compose exec <service_name> <command>
```

Build or rebuild services:

```bash
docker-compose build
docker-compose build --no-cache
```

Pull service images:

```bash
docker-compose pull
```

Scale services:

```bash
docker-compose up -d --scale <service_name>=<number_of_instances>
```

## Tips and Tricks

### Cleaning Up Resources

Remove all stopped containers:

```bash
docker container prune
```

Remove all unused images:

```bash
docker image prune
```

Remove all unused networks:

```bash
docker network prune
```

Remove all unused volumes:

```bash
docker volume prune
```

Remove all unused data (dangerous):

```bash
docker system prune
```

Force prune (includes stopped containers, unused images, networks, and volumes):

```bash
docker system prune -a --volumes
```

### Dockerfile Tips

- **Use .dockerignore**: To exclude files from the build context, create a `.dockerignore` file.

- **Order Matters**: Arrange your Dockerfile instructions to maximize caching.

- **Use Multi-Stage Builds**: Reduce image size by compiling in one stage and copying artifacts to a smaller final image.

Example:

```dockerfile
# Stage 1: Build
FROM golang:1.16 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Stage 2: Run
FROM alpine:latest
COPY --from=builder /app/myapp /myapp
CMD ["/myapp"]
```

### Optimizing Images

- **Use Alpine Base Images**: Alpine Linux images are smaller.

- **Remove Unnecessary Files**: Clean up cache and temp files during build.

- **Combine RUN Commands**: Reduce layers by combining commands.

Example:

```dockerfile
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
 && rm -rf /var/lib/apt/lists/*
```

### Security Best Practices

- **Run as Non-Root User**: Avoid running containers as root.

Example:

```dockerfile
RUN adduser -D myuser
USER myuser
```

- **Limit Container Resources**: Use flags to limit CPU and memory usage.

```bash
docker run --cpus=".5" --memory="256m" <image_name>
```

- **Use Trusted Images**: Pull images from official or verified sources.

- **Keep Docker Updated**: Regularly update Docker to the latest version.

- **Scan Images for Vulnerabilities**: Use tools like `docker scan` or third-party scanners.

### Networking Tips

- **Use Docker Networks**: Isolate containers by using custom networks.

- **Host Networking**: Use `--network host` to run containers on the host network (use with caution).

- **Expose Only Necessary Ports**: Avoid exposing unnecessary ports to the host.

### Logging and Monitoring

- **Configure Log Rotation**: Prevent logs from consuming disk space.

Example:

```yaml
logging:
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"
```

- **Use Monitoring Tools**: Integrate with tools like Prometheus, Grafana, or ELK Stack.

### Docker Compose Tips

- **Use `.env` Files**: Store environment variables in an `.env` file.

- **Override Configurations**: Use multiple `docker-compose.yml` files for different environments.

Example:

```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

- **Use Variables in Compose Files**: Reference environment variables in your `docker-compose.yml`.

Example:

```yaml
version: '3.8'
services:
  app:
    image: "${DOCKER_REGISTRY}/myapp:${TAG}"
```

## Additional Resources

- **Official Documentation**: [Docker Documentation](https://docs.docker.com/)
- **Dockerfile Reference**: [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- **Docker Compose File Reference**: [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- **Best Practices Guide**: [Docker Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

---

Feel free to bookmark this cheat sheet and refer back to it whenever you need a quick reminder of Docker commands or best practices.