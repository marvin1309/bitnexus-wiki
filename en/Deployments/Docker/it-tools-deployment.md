---
title: IT-Tools Deployment
description: 
published: true
date: 2024-09-21T09:31:05.905Z
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

## Step 7: Configure SSL (Optional but Recommended)

If you plan to run the service over HTTPS, you should set up SSL certificates.

### Using a Reverse Proxy with SSL

It's common to use a reverse proxy like **NGINX** or **Traefik** to handle SSL termination.

#### Example with NGINX

1. Install NGINX on your host machine.
2. Obtain SSL certificates using **Let's Encrypt**.
3. Configure NGINX to proxy requests to the Docker container.

**Sample NGINX Configuration:**

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### Update `docker-compose.yml` for Internal Networking (Optional)

If you're using a reverse proxy, you might not want to expose the container port to the host network. In that case, you can remove the `ports` section and use a Docker network.

```yaml
version: '3.9'

services:
  it-tools:
    image: 'corentinth/it-tools:latest'
    container_name: it-tools
    restart: unless-stopped
    networks:
      - it-tools-network

networks:
  it-tools-network:
```

Update your NGINX configuration to connect to the container using the Docker network:

```nginx
proxy_pass http://it-tools:80;
```

Ensure that your reverse proxy is part of the same Docker network or can resolve the container name.

## Additional Considerations

### Updating IT-Tools

To update the IT-Tools Docker image and container:

```bash
docker-compose pull
docker-compose up -d
```

### Automatic Restarts

The `restart: unless-stopped` policy ensures that the container restarts automatically if it crashes or if the Docker daemon restarts.

### Data Persistence

The IT-Tools container does not require data persistence as it serves static content. However, if future versions require storing data, you may need to map volumes accordingly.

### Environment Variables

If the IT-Tools image supports environment variables for configuration, you can pass them using the `environment` section in `docker-compose.yml`. Refer to the image documentation for supported variables.

```yaml
environment:
  - ENV_VAR_NAME=env_var_value
```

### Logging

Docker captures the container's stdout and stderr streams. To view logs:

```bash
docker logs it-tools
```

For log rotation and management, you can configure logging drivers in `docker-compose.yml`:

```yaml
logging:
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"
```

### Securing the Service

- **Authentication**: If IT-Tools supports authentication mechanisms, consider enabling them to restrict access.
- **Firewall**: Ensure that only necessary ports are open on your host machine.
- **Monitoring**: Implement monitoring to track the health and performance of the service.

## Troubleshooting

- **Container Fails to Start**: Check the Docker logs for error messages.

  ```bash
  docker logs it-tools
  ```

- **Cannot Access Web Interface**:
  - Ensure that the container is running and that the ports are correctly mapped.
  - Check if any firewall rules are blocking the connection.
  - Verify that your reverse proxy (if used) is correctly configured.

- **SSL Issues**: Verify your SSL certificates and reverse proxy configuration.

## Stopping and Removing the Container

To stop and remove the IT-Tools container:

```bash
docker-compose down
```

This command stops and removes the container but keeps the images and volumes. To remove all images and volumes (use with caution):

```bash
docker-compose down --rmi all --volumes
```

## Conclusion

By following this guide, you should have a fully functional IT-Tools service running in a Docker container. This setup provides an easy way to deploy and manage IT-Tools with minimal configuration.

---

**Note**: Always ensure your Docker environment is secure, especially if the service is exposed to the internet. Regularly update your Docker images and monitor for any security advisories related to the software you are running.