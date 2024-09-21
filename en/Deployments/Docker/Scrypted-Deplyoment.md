---
title: Scrypted Deplyoment
description: 
published: true
date: 2024-09-21T08:45:02.535Z
tags: 
editor: markdown
dateCreated: 2024-09-21T08:45:02.535Z
---

# Installation Guide for Scrypted with Docker Compose

This guide will walk you through setting up Scrypted using Docker Compose, including important considerations during the installation process.

## Prerequisites

- A system with Docker and Docker Compose installed.
- NVIDIA GPU drivers and NVIDIA Docker runtime configured (if using GPU acceleration).
- Basic knowledge of Docker and Docker Compose.

## Step 1: Install Docker and Docker Compose

If you haven't already, install Docker and Docker Compose on your system. You can follow the official installation guides:

- [Docker Engine Installation](https://docs.docker.com/engine/install/)
- [Docker Compose Installation](https://docs.docker.com/compose/install/)

## Step 2: Set Up the Directory Structure

Create a directory to store Scrypted's data:

```bash
mkdir -p ~/export/docker/scrypted
```

## Step 3: Create the `scrypted.env` File

Create an environment file named `scrypted.env` in the same directory as your `docker-compose.yml` file. Add any necessary environment variables required by Scrypted.

```bash
# Example content of scrypted.env
# Replace with actual environment variables as needed
ENV_VAR_1=value1
ENV_VAR_2=value2
```

## Step 4: Configure NVIDIA Docker Runtime (Optional)

If you plan to use GPU acceleration, ensure that:

- NVIDIA GPU drivers are installed on your host system.
- NVIDIA Docker runtime is configured.

Follow the [NVIDIA Container Toolkit Installation Guide](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) for detailed instructions.

## Step 5: Prepare the `docker-compose.yml` File

Create a `docker-compose.yml` file with the following content:

```yaml
version: "3.5"

services:
  scrypted:
    image: koush/scrypted
    container_name: scrypted
    restart: unless-stopped
    env_file: scrypted.env
    network_mode: host
    runtime: nvidia
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
    volumes:
      - ~/export/docker/scrypted:/server/volume
      # Uncomment and modify the following line to mount an additional volume for Scrypted NVR
      # - /path/to/your/nvr:/nvr
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "10"
    labels:
      - "com.centurylinklabs.watchtower.scope=scrypted"
```

### Explanation of Configuration Options

- **image**: Specifies the Docker image to use (`koush/scrypted`).
- **container_name**: Names the container `scrypted`.
- **restart**: Restarts the container unless it is stopped manually.
- **env_file**: Loads environment variables from `scrypted.env`.
- **network_mode**: Sets the container to use the host's network stack.
- **runtime**: Uses the NVIDIA runtime for GPU acceleration.
- **deploy.resources.reservations.devices**: Reserves the GPU device for the container.
- **volumes**: Mounts host directories into the container.
  - `~/export/docker/scrypted:/server/volume`: Persists Scrypted data.
  - Uncomment and modify `- /path/to/your/nvr:/nvr` to add NVR storage.
- **logging**: Configures log rotation to prevent excessive disk usage.
- **labels**: Adds labels for container management tools like Watchtower.

## Step 6: Configure Volumes and Devices (Optional)

### Mount Additional Volumes for NVR

If you plan to use Scrypted's NVR features, you may need to mount additional storage:

```yaml
volumes:
  - ~/export/docker/scrypted:/server/volume
  - /path/to/your/nvr:/nvr
```

Replace `/path/to/your/nvr` with the actual path on your host system where you want to store NVR data.

### Map Hardware Devices

If you need to pass hardware devices (e.g., USB devices, Coral TPU) to the container, add the `devices` section:

```yaml
devices:
  - /dev/dri:/dev/dri  # For hardware-accelerated video decoding
  - /dev/bus/usb:/dev/bus/usb  # To pass through all USB devices
  # - /dev/ttyACM0:/dev/ttyACM0  # Example for a Z-Wave USB serial device
```

## Step 7: Start the Container

Navigate to the directory containing your `docker-compose.yml` file and run:

```bash
docker-compose up -d
```

This command will download the necessary Docker image and start the Scrypted container in detached mode.

## Step 8: Verify the Installation

Check if the container is running:

```bash
docker ps
```

You should see the `scrypted` container listed.

## Step 9: Access Scrypted

Since the container uses the host's network stack (`network_mode: host`), you can access Scrypted via your host machine's IP address and the default port used by Scrypted.

Open a web browser and navigate to:

```
http://<your-host-ip>:<scrypted-port>
```

Replace `<your-host-ip>` with your host's IP address and `<scrypted-port>` with the port Scrypted is configured to use.

## Additional Considerations

### Log Management

The logging configuration in the `docker-compose.yml` file limits the log file size to prevent excessive disk usage:

```yaml
logging:
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "10"
```

### Using Watchtower for Automatic Updates

The label added to the container allows tools like Watchtower to manage updates for the `scrypted` container:

```yaml
labels:
  - "com.centurylinklabs.watchtower.scope=scrypted"
```

### Exposing mDNS Advertiser (Optional)

If you need to expose Avahi (an mDNS advertiser) and it's running on your host machine, you can mount the following sockets:

```yaml
volumes:
  - /var/run/dbus:/var/run/dbus
  - /var/run/avahi-daemon/socket:/var/run/avahi-daemon/socket
```

## Troubleshooting

- **Container Fails to Start**: Check the Docker logs for error messages.
  ```bash
  docker logs scrypted
  ```
- **Cannot Access Scrypted Web Interface**: Ensure that the container is running and that `network_mode: host` is correctly set.
- **GPU Not Detected**: Verify that the NVIDIA drivers and Docker runtime are properly installed and configured.

## Updating Scrypted

To update the Scrypted Docker image and container:

```bash
docker-compose pull
docker-compose up -d
```

## Stopping and Removing the Container

To stop and remove the Scrypted container:

```bash
docker-compose down
```

---

By following this guide, you should have a fully functional Scrypted installation running in a Docker container with the necessary configurations tailored to your setup.