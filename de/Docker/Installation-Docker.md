---
title: Installation-Docker
description: 
published: true
date: 2024-09-21T01:45:52.432Z
tags: 
editor: markdown
dateCreated: 2024-09-20T23:48:13.793Z
---

# Tutorial zur Installation von Docker, Docker Compose und Samba mit einem speziellen Export-Verzeichnis

Dieses Tutorial beschreibt, wie man Docker, Docker Compose sowie ein Samba-Share auf einem Debian-basierten System installiert. Alle Docker-Container sollen ihre Daten in einem `/export/docker`-Verzeichnis speichern, das per Samba freigegeben wird. Zudem wird ein Benutzer für die Verwaltung des Speichers erstellt.

---

## Disclaimer für Proxmox LXC Container

Wenn du Docker in einem Proxmox LXC Container installierst, müssen bestimmte Vorbereitungen getroffen werden. Dies wird in einem zukünftigen Tutorial behandelt.

---

## Voraussetzungen
- Ein Debian-basiertes System (z.B. Debian, Ubuntu, Proxmox LXC Container)
- Root- oder `sudo`-Zugriff

---

## Schritt 1: Verzeichnisstruktur erstellen

Wir beginnen mit dem Anlegen eines Verzeichnisses für alle Docker-Mounts.

```bash
sudo mkdir -p /export/docker
```

Dies erstellt das Verzeichnis `/export/docker`, in dem später alle Mountpunkte der Docker-Container gespeichert werden. Es wird auch für Docker selbst verwendet.

---

## Schritt 2: Benutzer für die Speicherverwaltung erstellen

Wir erstellen einen speziellen Benutzer `storagesvc`, der Lese- und Schreibzugriff auf das Samba-Share und Docker-Verzeichnisse erhält.

```bash
sudo useradd -M -d /export/docker -s /usr/sbin/nologin storagesvc
sudo passwd storagesvc
```

Der Benutzer `storagesvc` wird ohne Heimatverzeichnis und Login-Shell erstellt, da er nur für die Verwaltung des Speicherplatzes benötigt wird. Danach setzen wir ein Passwort für den Benutzer.

---

## Schritt 3: Installation von Docker

Wir installieren Docker gemäß der offiziellen Anleitung von Docker. Führe die folgenden Befehle aus, um Docker auf deinem System zu installieren:

### Docker installieren (Debian-basierte Systeme)
```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

# Docker's offizieller GPG-Schlüssel hinzufügen
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Repository für Docker hinzufügen
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Docker-Verzeichnis ändern
Um Docker so zu konfigurieren, dass es seine Daten im Verzeichnis `/export/docker/docker-data` speichert, bearbeiten wir die Datei `daemon.json`.

```bash
sudo mkdir -p /export/docker/docker-data
sudo nano /etc/docker/daemon.json
```

Füge den folgenden Inhalt hinzu:

```json
{
  "data-root": "/export/docker/docker-data"
}
```

Speichere die Datei und starte Docker neu:

```bash
sudo systemctl restart docker
```

---

## Schritt 4: Docker Compose installieren

Docker Compose ist in den neueren Versionen von Docker als Plugin enthalten. Um sicherzustellen, dass es korrekt installiert ist, kannst du den folgenden Befehl ausführen:

```bash
docker compose version
```

Falls Docker Compose nicht installiert ist, kannst du es wie folgt nachinstallieren:

```bash
sudo apt install docker-compose-plugin
```

---

Zum testen hier eine Hello World Docker Container

```
version: '3'

services:
  hello-world:
    image: hello-world


```

## Schritt 5: Portainer (optional) installieren

Portainer ist eine grafische Oberfläche zur Verwaltung von Docker-Containern. Es kann wie folgt installiert werden, wenn es sich um die Haupt-Node oder erste Node eines Docker-Clusters handelt:

```bash
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name=portainer \
  --restart=always -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data portainer/portainer-ce:latest
```

Weitere Details findest du in der offiziellen Anleitung: [Portainer Installation](https://docs.portainer.io/start/install-ce/server/docker/linux).

---

## Schritt 6: Samba installieren und konfigurieren

Jetzt installieren wir Samba und richten den Samba-Share ein, sodass der Benutzer `storagesvc` Lese- und Schreibrechte auf das Verzeichnis `/export/docker` erhält.

### Samba installieren

```bash
sudo apt install samba
```

### Samba konfigurieren

Bearbeite die Samba-Konfigurationsdatei:

```bash
sudo nano /etc/samba/smb.conf
```

Füge am Ende der Datei folgenden Abschnitt hinzu:

```ini
[docker]
   path = /export/docker
   browseable = yes
   read only = no
   writable = yes
   force user = storagesvc
   force group = storagesvc
   create mask = 7777
   directory mask = 7777
```

Speichere die Datei und starte Samba neu:

```bash
sudo systemctl restart smbd
```

### Samba-Benutzer für `storagesvc` erstellen

Erstelle einen Samba-Benutzer und setze ein Passwort:

```bash
sudo smbpasswd -a storagesvc
```

---

## Schritt 7: Fertigstellung

Du hast nun Docker, Docker Compose und Samba auf deinem System installiert und konfiguriert. Das Verzeichnis `/export/docker` kann von Docker für die Speicherung der Containerdaten und von Samba für den Zugriff über das Netzwerk verwendet werden.

Weitere Schritte wie die Einrichtung eines VSCode-Servers, der auf die Samba-Shares zugreift, folgen in einem separaten Tutorial.

---

## Quellen

- Docker Installationsseite: [Docker Installationsanleitung](https://docs.docker.com/engine/install/debian/)
- Portainer Installationsseite: [Portainer Installationsanleitung](https://docs.portainer.io/start/install-ce/server/docker/linux)
- Credits: [Setup und Install Docker in einem Proxmox LXC Container](https://thehomelab.wiki/books/promox-ve/page/setup-and-install-docker-in-a-promox-lxc-container)

---

Ende des Tutorials.