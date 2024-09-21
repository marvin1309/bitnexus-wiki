---
title: 2fAuth Deployment
description: 
published: true
date: 2024-09-21T09:01:31.272Z
tags: 
editor: markdown
dateCreated: 2024-09-21T08:57:47.936Z
---

# Installation Guide for 2FAuth with Docker Compose

This guide will walk you through setting up **2FAuth**, a two-factor authentication service, using Docker Compose. It includes important considerations during the installation process to ensure a smooth setup.

## Prerequisites

- A system with **Docker** and **Docker Compose** installed.
- Basic knowledge of Docker and Docker Compose.
- An email address for the site owner.
- (Optional) SSL certificates if you plan to run the service over HTTPS.

## Step 1: Install Docker and Docker Compose

If you haven't already, install Docker and Docker Compose on your system. You can follow the official installation guides:

- [Docker Engine Installation](https://docs.docker.com/engine/install/)
- [Docker Compose Installation](https://docs.docker.com/compose/install/)

or use my Guide at [Installation-Docker](/en/Deployments/Docker/Installation-Docker) `recommended`

## Step 2: Set Up Directory Structure

Create a directory to store your 2FAuth configuration and data:

```bash
mkdir -p ~/export/docker/2fauth
cd ~/export/docker/2fauth
```

## Step 3: Create the Environment File

Create a file named `2fauth.env` in the directory. This file will contain environment variables needed for the 2FAuth service.

```bash
nano 2fauth.env
```

### Sample `2fauth.env` Content

Below is a sample configuration. Replace placeholder values with your actual settings.

```ini
# Application Settings
APP_NAME=2FAuth
APP_ENV=local
APP_DEBUG=false
APP_URL=https://yourdomain.com
APP_KEY=base64:YourGeneratedAppKeyHere

# Site Owner Email
SITE_OWNER=youremail@domain.com

# Logging Configuration
LOG_CHANNEL=daily
LOG_LEVEL=notice

# Database Configuration
DB_DATABASE="/srv/database/database.sqlite"

# Cache and Session Drivers
CACHE_DRIVER=file
SESSION_DRIVER=file

# Mail Configuration (Optional if you plan to send emails)
MAIL_MAILER=smtp
MAIL_HOST=smtp.yourprovider.com
MAIL_PORT=587
MAIL_USERNAME=your_smtp_username
MAIL_PASSWORD=your_smtp_password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=youremail@domain.com
MAIL_FROM_NAME="2FAuth Support"

# SSL Peer Verification
MAIL_VERIFY_SSL_PEER=true

# API Throttling
THROTTLE_API=60

# Authentication Throttling
LOGIN_THROTTLE=5

# WebAuthn Settings
WEBAUTHN_NAME=2FAuth
WEBAUTHN_USER_VERIFICATION=preferred

# Trusted Proxies (Set '*' to trust all proxies)
TRUSTED_PROXIES='*'

# Outgoing Requests Proxy (Optional)
PROXY_FOR_OUTGOING_REQUESTS=null

# Other Settings (Leave as default unless necessary)
BROADCAST_DRIVER=log
QUEUE_DRIVER=sync
SESSION_LIFETIME=120
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379
```

### Important Notes on Environment Variables

- **APP_KEY**: This is a crucial encryption key. Generate a secure key using the command provided in Step 5.
- **APP_URL**: Must match the external address where your service will be accessible.
- **SITE_OWNER**: Set this to your email address.
- **MAIL Configuration**: If you plan to send emails (e.g., for password resets), configure these settings according to your email provider. If not, you can leave the mail settings commented out or set `MAIL_MAILER=log` to log emails instead of sending them.
- **THROTTLE_API** and **LOGIN_THROTTLE**: Adjust these settings to control API and login rate limits.

## Step 4: Generate the Application Key

Before running the service, you need to generate an application key for encryption.

```bash
docker run --rm -v $(pwd):/app -w /app 2fauth/2fauth:latest php artisan key:generate --show
```

This command will output a key. Copy it and paste it into the `APP_KEY` variable in your `2fauth.env` file.

Example:

```ini
APP_KEY=base64:YourGeneratedAppKeyHere
```

## Step 5: Prepare the `docker-compose.yml` File

Create a `docker-compose.yml` file in the same directory with the following content:

```yaml
version: '3.3'
services:
  2fauth:
    container_name: 2fauth
    image: 2fauth/2fauth:latest
    env_file: 2fauth.env
    volumes:
      - ./database:/srv/database
      - ./storage:/var/www/html/storage
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "8888:8000"
    restart: unless-stopped
```

### Explanation of Configuration Options

- **image**: Specifies the Docker image to use (`2fauth/2fauth:latest`).
- **container_name**: Names the container `2fauth`.
- **env_file**: Loads environment variables from `2fauth.env`.
- **volumes**:
  - `./database:/srv/database`: Persists the SQLite database.
  - `./storage:/var/www/html/storage`: Persists storage files (e.g., logs, sessions).
  - `/etc/localtime:/etc/localtime:ro`: Syncs the container's time with the host.
- **ports**: Maps port `8000` in the container to port `8888` on the host.
- **restart**: Restarts the container unless it is stopped manually.

### Adjusting Ports

If you want the service to be accessible on a different port, change the `ports` mapping:

```yaml
ports:
  - "your_desired_port:8000"
```

## Step 6: Set Up Directories and Permissions

Ensure that the `database` and `storage` directories exist and have the correct permissions:

```bash
mkdir -p database storage
chmod -R 755 database storage
```

## Step 7: Start the Container

Start the 2FAuth service using Docker Compose:

```bash
docker-compose up -d
```

This command will download the necessary Docker image and start the 2FAuth container in detached mode.

## Step 8: Verify the Installation

Check if the container is running:

```bash
docker ps
```

You should see the `2fauth` container listed.

## Step 9: Access the 2FAuth Service

Open a web browser and navigate to:

```
http://<your-host-ip>:8888
```

Replace `<your-host-ip>` with your host's IP address. If you're running this on your local machine, you can use `http://localhost:8888`.

## Step 10: Configure SSL (Optional but Recommended)

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
        proxy_pass http://localhost:8888;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### Update `APP_URL` in `2fauth.env`

Ensure that `APP_URL` is set to your domain with HTTPS:

```ini
APP_URL=https://yourdomain.com
```

## Additional Considerations

### Logging Configuration

The logging settings in `2fauth.env` control how logs are handled.

- **LOG_CHANNEL**: Set to `daily` to rotate logs daily.
- **LOG_LEVEL**: Controls the severity of logs. Common levels are `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, and `emergency`.

### Mail Configuration

If you plan to enable email features (e.g., password resets), configure the mail settings in `2fauth.env`.

```ini
MAIL_MAILER=smtp
MAIL_HOST=smtp.yourprovider.com
MAIL_PORT=587
MAIL_USERNAME=your_smtp_username
MAIL_PASSWORD=your_smtp_password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=youremail@domain.com
MAIL_FROM_NAME="2FAuth Support"
```

### API and Authentication Throttling

Adjust `THROTTLE_API` and `LOGIN_THROTTLE` in `2fauth.env` to control rate limiting.

- **THROTTLE_API**: Maximum number of API calls per minute from the same IP.
- **LOGIN_THROTTLE**: Number of failed login attempts allowed per minute before locking out the IP.

### WebAuthn Settings

Configure WebAuthn settings for enhanced security.

- **WEBAUTHN_NAME**: The name displayed during authentication prompts.
- **WEBAUTHN_USER_VERIFICATION**: Set to `required`, `preferred`, or `discouraged` based on your security needs.

### Trusted Proxies

If you're behind a reverse proxy, set `TRUSTED_PROXIES` in `2fauth.env`.

```ini
TRUSTED_PROXIES='*'
```

### Session and Cache Drivers

By default, the session and cache drivers are set to `file`. For improved performance, you can use `redis` or `memcached` if you have them installed.

```ini
CACHE_DRIVER=redis
SESSION_DRIVER=redis
```

Remember to configure Redis settings accordingly.

## Updating 2FAuth

To update the 2FAuth Docker image and container:

```bash
docker-compose pull
docker-compose up -d
```

## Stopping and Removing the Container

To stop and remove the 2FAuth container:

```bash
docker-compose down
```

## Troubleshooting

- **Container Fails to Start**: Check the Docker logs for error messages.

  ```bash
  docker logs 2fauth
  ```

- **Cannot Access Web Interface**: Ensure that the container is running and that the ports are correctly mapped.

- **SSL Issues**: Verify your SSL certificates and reverse proxy configuration.

- **Database Errors**: Ensure that the `database` directory exists and has the correct permissions.

## Conclusion

By following this guide, you should have a fully functional 2FAuth service running in a Docker container with the necessary configurations tailored to your setup. Always ensure to secure your environment variables and regularly update your services for the latest features and security patches.

---

**Note**: This guide assumes a basic level of familiarity with Docker, Docker Compose, and server administration. Always back up your data before making significant changes to your setup.