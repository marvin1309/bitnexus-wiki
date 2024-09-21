---
title: IT-Tools Deployment
description: 
published: true
date: 2024-09-21T09:33:39.016Z
tags: 
editor: markdown
dateCreated: 2024-09-21T09:29:49.886Z
---

# Installation Guide for IT-Tools with Docker Compose

This guide will walk you through setting up **IT-Tools**, a collection of handy online tools for developers and IT professionals, using Docker Compose. It includes important considerations during the installation process to ensure a smooth setup.

## Prerequisites

- A system with **Docker** and **Docker Compose** installed.
- Basic knowledge of Docker and Docker Compose.
- (Optional) SSL certificates if you plan to run the service over HTTPS.

## Step 1: Install Docker and Docker Compose

If you haven't already, install Docker and Docker Compose on your system. You can follow the official installation guides:

- [Docker Engine Installation](https://docs.docker.com/engine/install/)
- [Docker Compose Installation](https://docs.docker.com/compose/install/)

or use my Guide at [Installation-Docker](/Deployments/Docker/Installation-Docker) `recommended`

## Step 2: Set Up Directory Structure

Create a directory to store your IT-Tools configuration and data (optional, since this container doesn't require persistent storage):

```bash
mkdir -p /export/docker/it-tools
cd /export/docker/it-tools
```

## Step 3: Prepare the `docker-compose.yml` File

Create a `docker-compose.yml` file in the directory with the following content:

```yaml
version: '3.9'

services:
  it-tools:
    image: 'corentinth/it-tools:latest'
    container_name: it-tools
    ports:
      - '8080:80'
    restart: unless-stopped
```

### Explanation of Configuration Options

- **version**: Specifies the version of the Docker Compose file format.
- **services**: Defines the services to be run.
  - **it-tools**: The name of the service.
    - **image**: Specifies the Docker image to use (`corentinth/it-tools:latest`).
    - **container_name**: Names the container `it-tools`.
    - **ports**: Maps port `80` in the container to port `8080` on the host.
      - `'8080:80'`: Format `host_port:container_port`.
    - **restart**: Restarts the container unless it is stopped manually (`unless-stopped`).

### Adjusting Ports

If you want the service to be accessible on a different port, change the `ports` mapping:

```yaml
ports:
  - 'your_desired_port:80'
```

For example, to use port `80` on the host:

```yaml
ports:
  - '80:80'
```

## Step 4: Start the Container

Start the IT-Tools service using Docker Compose:

```bash
docker-compose up -d
```

This command will download the necessary Docker image and start the IT-Tools container in detached mode.

## Step 5: Verify the Installation

Check if the container is running:

```bash
docker ps
```

You should see the `it-tools` container listed.

## Step 6: Access the IT-Tools Service

Open a web browser and navigate to:

```
http://<your-host-ip>:8080
```

Replace `<your-host-ip>` with your host's IP address. If you're running this on your local machine, you can use `http://localhost:8080`.

You should see the IT-Tools web interface, providing various tools for your use.


## Conclusion

By following this guide, you should have a fully functional IT-Tools service running in a Docker container. This setup provides an easy way to deploy and manage IT-Tools with minimal configuration.

---

**Note**: Always ensure your Docker environment is secure, especially if the service is exposed to the internet. Regularly update your Docker images and monitor for any security advisories related to the software you are running.