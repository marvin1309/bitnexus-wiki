---
title: Home-Assistent Deployment
description: 
published: true
date: 2024-09-21T11:07:12.536Z
tags: 
editor: markdown
dateCreated: 2024-09-21T11:06:17.682Z
---

# Deployment Guide for Home Assistant with Docker Compose

This guide will walk you through deploying **Home Assistant**, an open-source home automation platform, using Docker Compose. It includes important considerations during the installation process to ensure a smooth setup.

---

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Step 1: Set Up Directory Structure](#step-1-set-up-directory-structure)
- [Step 2: Create the `homeassistant.env` File](#step-2-create-the-homeassistantenv-file)
- [Step 3: Prepare the `docker-compose.yml` File](#step-3-prepare-the-docker-composeyml-file)
- [Step 4: Understand the Configuration](#step-4-understand-the-configuration)
- [Step 5: Start the Home Assistant Container](#step-5-start-the-home-assistant-container)
- [Step 6: Initial Configuration of Home Assistant](#step-6-initial-configuration-of-home-assistant)
- [Additional Considerations](#additional-considerations)
  - [Accessing Hardware Devices](#accessing-hardware-devices)
  - [Updating Home Assistant](#updating-home-assistant)
  - [Security Considerations](#security-considerations)
- [Referencing Docker Commands](#referencing-docker-commands)
- [Conclusion](#conclusion)

---

## Introduction

**Home Assistant** is an open-source platform designed to help you manage and automate your smart home devices from a single, user-friendly interface. Running Home Assistant in a Docker container allows for easy installation, management, and portability.

This guide will help you deploy Home Assistant using Docker Compose, utilizing the LinuxServer.io Docker image, and includes steps for configuring environment variables in a separate `env_file`.

## Prerequisites

- A system with **Docker** and **Docker Compose** installed.
- Basic knowledge of Docker and Docker Compose.
- (Optional) Hardware devices (e.g., USB devices) that you want to integrate with Home Assistant.

**Note**: If you haven't installed Docker and Docker Compose, you can follow the [Installation-Docker](/Deployments/Docker/Installation-Docker) guide (recommended).

## Step 1: Set Up Directory Structure

Create a directory to store Home Assistant's configuration data:

```bash
mkdir -p /export/docker/homeassistant
cd /export/docker/homeassistant
```

This directory will be used to persist Home Assistant's configuration across container restarts and updates.

## Step 2: Create the `homeassistant.env` File

Create a file named `homeassistant.env` in the `/export/docker/homeassistant` directory. This file will contain environment variables needed for Home Assistant.

```bash
nano homeassistant.env
```

### Sample `homeassistant.env` Content

```ini
# Timezone
TZ=Europe/Berlin

# User and Group IDs
PUID=1000
PGID=1000

# Optional umask for file permissions (default is 022)
UMASK=022

# (Optional) Set the home directory (not typically required)
# HOME=/config
```

**Important Notes:**

- **TZ**: Set this to your local timezone (e.g., `Europe/Berlin`).
- **PUID** and **PGID**: Set these to match your user and group IDs on the host system. You can find your UID and GID with the `id` command.
  ```bash
  id
  ```
- **UMASK**: Optional. Sets default file permissions. The default is `022`, which translates to `755` for directories and `644` for files.

## Step 3: Prepare the `docker-compose.yml` File

Create a `docker-compose.yml` file in the same directory with the following content:

```yaml
version: '3.8'

services:
  home-assistant:
    container_name: homeassistant
    image: lscr.io/linuxserver/homeassistant:latest
    env_file: homeassistant.env
    volumes:
      - /export/docker/homeassistant/config:/config
      - /etc/localtime:/etc/localtime:ro
      - /run/dbus:/run/dbus:ro
      # Uncomment and adjust the following line to map USB devices (e.g., Z-Wave sticks)
      # - /dev/ttyUSB0:/dev/ttyUSB0
    network_mode: host
    restart: unless-stopped
    privileged: true
    security_opt:
      - seccomp:unconfined
```

### Adjusting the Configuration

- **Volumes**:
  - `/export/docker/homeassistant/config:/config`: Maps the `config` directory to persist Home Assistant's configuration data.
  - `/etc/localtime:/etc/localtime:ro`: Syncs the container's time with the host.
  - `/run/dbus:/run/dbus:ro`: Required for certain integrations that use D-Bus.
  - **Mapping USB Devices**: If you have USB devices (e.g., Z-Wave or Zigbee sticks), you can map them by uncommenting and adjusting the device mapping.
- **Network Mode**:
  - `network_mode: host`: Allows Home Assistant to access the host's network directly, which is necessary for certain integrations like mDNS, UPnP, etc.
- **Privileged Mode and Security Options**:
  - `privileged: true` and `security_opt` settings are required for full hardware access and certain integrations.

**Note**: When using `network_mode: host`, you should not include the `ports` section, as port mapping is not supported in host network mode.

## Step 4: Understand the Configuration

### Service Definition

- **image**: Specifies the Docker image to use (`lscr.io/linuxserver/homeassistant:latest`).
- **container_name**: Names the container `homeassistant`.
- **env_file**: Loads environment variables from `homeassistant.env`.
- **volumes**: Maps host directories and devices into the container.
- **network_mode**: Set to `host` to allow Home Assistant to access the host network.
- **restart**: Restarts the container unless it is stopped manually (`unless-stopped`).
- **privileged**: Grants extended privileges to the container.
- **security_opt**: Disables default security features to allow full hardware access.

### Environment Variables

From the LinuxServer.io documentation, the available environment variables are:

- `PUID` (default `1000`): User ID to run the application as.
- `PGID` (default `1000`): Group ID to run the application as.
- `TZ`: Timezone setting, e.g., `Europe/Berlin`.
- `UMASK` (optional): Set the default file creation mask (e.g., `022`).

### Additional Environment Variables

If you require additional environment variables for specific integrations or configurations, you can add them to the `homeassistant.env` file.

## Step 5: Start the Home Assistant Container

From the directory containing your `docker-compose.yml` file, run:

```bash
docker-compose up -d
```

This command will download the Home Assistant image and start the container in detached mode.

### Referencing Docker Commands

For more information on Docker Compose commands, you can refer to the [Docker Cheat Sheet](#docker-cheat-sheet-essential-commands-tips-and-tricks) or use:

```bash
docker-compose --help
```

## Step 6: Initial Configuration of Home Assistant

After starting the container, you'll need to perform the initial setup.

### Access Home Assistant

1. Open a web browser and navigate to:

   ```
   http://<your-host-ip>:8123
   ```

   Replace `<your-host-ip>` with your host's IP address.

2. Follow the on-screen instructions to set up your Home Assistant instance.

   - Create an account.
   - Configure basic settings.
   - Set up integrations and devices.

## Additional Considerations

### Accessing Hardware Devices

If you have hardware devices (e.g., USB sticks for Z-Wave, Zigbee, or Bluetooth), you need to map them into the container.

#### Identify Your Device

List your connected devices:

```bash
ls /dev/serial/by-id/
```

This will display devices like:

```
usb-0658_0200-if00
usb-FTDI_FT232R_USB_UART_A9WR37H8-if00-port0
```

#### Map the Device

In your `docker-compose.yml`, add the device mapping:

```yaml
volumes:
  - /export/docker/homeassistant/config:/config
  - /etc/localtime:/etc/localtime:ro
  - /run/dbus:/run/dbus:ro
  - /dev/serial/by-id/usb-0658_0200-if00:/dev/zwave
```

This maps the device to `/dev/zwave` inside the container.

**Note**: Adjust the device path according to your hardware.

### Updating Home Assistant

To update the Home Assistant Docker image and container:

```bash
docker-compose pull
docker-compose up -d
```

### Security Considerations

- **Privileged Mode**: Running a container in privileged mode grants it extended permissions. Ensure your host system is secure.
- **Network Mode**: Using `network_mode: host` exposes the container to your network directly. Be cautious with exposed services.
- **User IDs**: Use appropriate `PUID` and `PGID` to match your host user for proper file permissions.

### Backing Up Configuration

Your configuration is stored in the `config` directory. Regularly back up this directory to prevent data loss.

### Stopping and Removing the Container

To stop and remove the Home Assistant container:

```bash
docker-compose down
```

This will stop and remove the container but keep your data intact due to the volume mappings.

## Referencing Docker Commands

For common Docker commands and tips, refer to the [Docker Cheat Sheet](#docker-cheat-sheet-essential-commands-tips-and-tricks).

- **Viewing Container Logs**: `docker logs homeassistant`
- **Restarting the Container**: `docker-compose restart homeassistant`
- **Inspecting the Container**: `docker inspect homeassistant`

## Conclusion

By following this guide, you should have a fully functional Home Assistant instance running in a Docker container. This setup provides an efficient way to manage and automate your smart home devices.

---

**Note**: Always ensure your Docker environment is secure, especially if the service is exposed to the internet. Regularly update your Docker images and monitor for any security advisories related to the software you are running.

For more detailed Docker commands and best practices, refer to:

- [Docker Installation Guide](/Deployments/Docker/Installation-Docker)
- [Docker Cheat Sheet](/en/Deployments/Docker/Docker-Cheatsheet)