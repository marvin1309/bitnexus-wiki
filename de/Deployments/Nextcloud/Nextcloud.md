---
title: Nextcloud Deployment
description: 
published: true
date: 2024-09-21T07:41:50.768Z
tags: 
editor: markdown
dateCreated: 2024-09-21T06:59:07.899Z
---

# Header
# Nextcloud Setup mit Docker Compose

## Einführung

In diesem Tutorial zeige ich Ihnen, wie Sie Nextcloud mit Docker Compose einrichten. Nextcloud ist eine leistungsstarke, selbstgehostete Alternative zu Cloud-Diensten wie Google Drive.

## Voraussetzungen

- Docker und Docker Compose installiert
- Grundlegende Kenntnisse im Umgang mit der Kommandozeile

## Schritt-für-Schritt Anleitung

### Schritt 1: Docker Compose Datei erstellen

Erstellen Sie eine Datei namens `docker-compose.yml` mit folgendem Inhalt:

```yaml
version: '3.8'
services:
  nextcloud:
    container_name: nextcloud
    restart: always
    image: nextcloud:latest
    ports:
      - "9765:80"
    env_file: nextcloud.env
    volumes:
      - /export/docker/nextcloud/html:/var/www/html
      - /mnt/nextcloud:/mnt/data
```

### Schritt 2: Umgebungsvariablen konfigurieren

Erstellen Sie eine Datei namens `nextcloud.env` und fügen Sie die notwendigen Umgebungsvariablen hinzu:

```env
MYSQL_PASSWORD=yourpassword
MYSQL_DATABASE=nextcloud
MYSQL_USER=nextcloud
MYSQL_HOST=mysql
```

### Schritt 3: Datenbank einrichten

Erstellen Sie ein Shell-Skript `createdb.sh`, um die Datenbank einzurichten:

```bash
#!/bin/bash
docker exec -it nextcloud_db mysql -u root -p -e "CREATE DATABASE nextcloud;"
docker exec -it nextcloud_db mysql -u root -p -e "CREATE USER 'nextcloud'@'%' IDENTIFIED BY 'yourpassword';"
docker exec -it nextcloud_db mysql -u root -p -e "GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'%';"
docker exec -it nextcloud_db mysql -u root -p -e "FLUSH PRIVILEGES;"
```

### Schritt 4: Docker Compose starten

Führen Sie den folgenden Befehl aus, um Nextcloud zu starten:

```bash
docker-compose up -d
```

### Abschluss

Ihre Nextcloud-Instanz sollte nun laufen. Sie können darauf zugreifen, indem Sie Ihren Browser öffnen und `http://localhost:9765` eingeben.
