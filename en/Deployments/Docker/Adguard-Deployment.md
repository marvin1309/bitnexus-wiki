---
title: Adguard Deployment
description: 
published: true
date: 2024-09-21T10:57:16.271Z
tags: 
editor: markdown
dateCreated: 2024-09-21T10:57:16.271Z
---

# Deployment Guide for AdGuard Home with Docker Compose

This guide will walk you through deploying **AdGuard Home**, a network-wide software for blocking ads and tracking, using Docker Compose. It includes important considerations during the installation process to ensure a smooth setup.

---

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Step 1: Set Up Directory Structure](#step-1-set-up-directory-structure)
- [Step 2: Prepare the `docker-compose.yml` File](#step-2-prepare-the-docker-composeyml-file)
- [Step 3: Understand the Configuration](#step-3-understand-the-configuration)
- [Step 4: Start the AdGuard Home Container](#step-4-start-the-adguard-home-container)
- [Step 5: Initial Configuration of AdGuard Home](#step-5-initial-configuration-of-adguard-home)
- [Step 6: Verify the Installation](#step-6-verify-the-installation)
- [Additional Considerations](#additional-considerations)
  - [Security](#security)
  - [Updating AdGuard Home](#updating-adguard-home)
- [Referencing Docker Commands](#referencing-docker-commands)
- [Conclusion](#conclusion)

---

## Introduction

**AdGuard Home** is a network-wide software for blocking ads and tracking. After you set it up, it'll cover **ALL** your home devices, and you don't need any client-side software for that. It's similar to Pi-hole but offers additional features.

This guide will help you deploy AdGuard Home using Docker Compose, allowing for easy management and scalability.

## Prerequisites

- A system with **Docker** and **Docker Compose** installed.
- Basic knowledge of Docker and Docker Compose.
- (Optional) SSL certificates if you plan to run the service over HTTPS.

**Note**: If you haven't installed Docker and Docker Compose, you can follow the [Installation-Docker](/Deployments/Docker/Installation-Docker) guide (recommended).

## Step 1: Set Up Directory Structure

Create directories to store AdGuard Home's configuration and working data:

```bash
mkdir -p /export/docker/adguard/workdir
mkdir -p /export/docker/adguard/confdir
cd /export/docker/adguard
```

This ensures that your configuration and data persist across container restarts and updates.

## Step 2: Prepare the `docker-compose.yml` File

Create a `docker-compose.yml` file in the `~/docker/adguard` directory with the following content:

```yaml
version: '3'
services:
  adguard:
    image: adguard/adguardhome
    container_name: adguard
    hostname: adguard
    restart: unless-stopped
    ports:
      - "53:53/tcp"        # Unencrypted DNS
      - "53:53/udp"        # Unencrypted DNS
      - "67:67/udp"        # DHCP
      # - "68:68/udp"      # DHCP (uncomment if needed)
      - "80:80/tcp"        # Admin interface & DNS over HTTP
      - "443:443/tcp"      # Admin interface & DNS over HTTPS
      - "443:443/udp"      # DNS over HTTPS
      - "3000:3000/tcp"    # Initial setup portal
      - "853:853/tcp"      # DNS over TLS
      - "853:853/udp"      # DNS over QUIC
      - "784:784/udp"      # DNS over QUIC
      - "8853:8853/udp"    # DNS over QUIC
      - "5443:5443/tcp"    # DNSCrypt
      - "5443:5443/udp"    # DNSCrypt
    volumes:
      - "/export/docker/workdir:/opt/adguardhome/work"
      - "/export/docker/confdir:/opt/adguardhome/conf"
      - "/etc/localtime:/etc/localtime:ro"
```

### Important Notes:

- Adjust the port mappings if necessary, especially if ports are already in use on your host machine.
- Uncomment the `68:68/udp` port mapping if you need DHCP relay functionality.
- Ensure that the volume paths point to the correct directories.

## Step 3: Understand the Configuration

### Service Definition

- **image**: Specifies the Docker image to use (`adguard/adguardhome`).
- **container_name**: Names the container `adguard`.
- **hostname**: Sets the container's hostname to `adguard`.
- **restart**: Restarts the container unless it is stopped manually (`unless-stopped`).

### Ports

The `ports` section maps container ports to host ports:

- **DNS Services**:
  - `53:53/tcp` and `53:53/udp`: Standard DNS service ports.
- **DHCP Services**:
  - `67:67/udp`: DHCP server port.
  - `68:68/udp`: DHCP client port (uncomment if needed).
- **Web and API Access**:
  - `80:80/tcp`: HTTP access to the web interface.
  - `443:443/tcp` and `443:443/udp`: HTTPS access and DNS over HTTPS.
- **Initial Setup Portal**:
  - `3000:3000/tcp`: Access to the initial setup portal.
- **Encrypted DNS Services**:
  - `853:853/tcp` and `853:853/udp`: DNS over TLS and QUIC.
  - `784:784/udp`, `8853:8853/udp`: Additional DNS over QUIC ports.
  - `5443:5443/tcp` and `5443:5443/udp`: DNSCrypt.

### Volumes

The `volumes` section mounts host directories into the container:

- `/export/docker/workdir:/opt/adguardhome/work`: Stores runtime data.
- `/export/docker/confdir:/opt/adguardhome/conf`: Stores configuration files.
- `/etc/localtime:/etc/localtime:ro`: Syncs container time with host time.



## Step 4: Start the AdGuard Home Container

From the directory containing your `docker-compose.yml` file, run:

```bash
docker-compose up -d
```

This command will download the AdGuard Home image and start the container in detached mode.

### Referencing Docker Commands

For more information on Docker Compose commands, you can refer to the [Docker Cheat Sheet](/path/to/docker-cheat-sheet) or use:

```bash
docker-compose --help
```

## Step 5: Initial Configuration of AdGuard Home

After starting the container, you'll need to perform the initial setup.

### Access the Setup Portal

1. Open a web browser and navigate to:

   ```
   http://<your-host-ip>:3000
   ```

   Replace `<your-host-ip>` with your host's IP address.

2. Follow the on-screen instructions to configure AdGuard Home.

   - Set up the administrator username and password.
   - Configure upstream DNS servers.
   - Set any additional settings as per your requirements.

### Post-Setup Access

After initial setup, the web interface moves to port `80` or `443` (if SSL is configured):

- HTTP:

  ```
  http://<your-host-ip>
  ```

- HTTPS:

  ```
  https://<your-host-ip>
  ```

## Step 6: Verify the Installation

Check if the container is running:

```bash
docker ps
```

You should see the `adguard` container listed.

### Test DNS Resolution

Set your device's DNS server to the host's IP address and verify that ads are being blocked.

### Check Logs

To view the container logs:

```bash
docker logs adguard
```

For real-time logs:

```bash
docker logs -f adguard
```

## Additional Considerations

### Security

- **Firewall Settings**: Ensure that your firewall allows traffic on the necessary ports.
- **Admin Interface Access**: Restrict access to the admin interface by adjusting AdGuard Home's settings.
- **SSL Configuration**: For secure access over HTTPS, consider setting up SSL certificates.

### Updating AdGuard Home

To update the AdGuard Home Docker image and container:

```bash
docker-compose pull
docker-compose up -d
```

### Backing Up Configuration

Your configuration is stored in the `confdir` directory. Regularly back up this directory to prevent data loss.

### Stopping and Removing the Container

To stop and remove the AdGuard Home container:

```bash
docker-compose down
```

This will stop and remove the container but keep your data intact due to the volume mappings.

## Referencing Docker Commands

For common Docker commands and tips, refer to the [Docker Cheat Sheet](/path/to/docker-cheat-sheet).

- **Viewing Container Logs**: `docker logs adguard`
- **Restarting the Container**: `docker-compose restart adguard`
- **Inspecting the Container**: `docker inspect adguard`

## Conclusion

By following this guide, you should have a fully functional AdGuard Home instance running in a Docker container. This setup provides an efficient way to manage network-wide ad blocking and DNS services.

---

**Note**: Always ensure your Docker environment is secure, especially if the service is exposed to the internet. Regularly update your Docker images and monitor for any security advisories related to the software you are running.

For more detailed Docker commands and best practices, refer to:

- [Docker Installation Guide](/Deployments/Docker/Installation-Docker)
- [Docker Cheat Sheet](/path/to/docker-cheat-sheet)