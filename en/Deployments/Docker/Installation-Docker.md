---
title: Installation-Docker
description: 
published: true
date: 2024-09-21T09:29:29.731Z
tags: 
editor: markdown
dateCreated: 2024-09-21T07:51:59.059Z
---

# Tutorial zur Installation von Docker, Docker Compose und Samba mit einem speziellen Export-Verzeichnis

Dieses Tutorial beschreibt, wie man Docker, Docker Compose sowie ein Samba-Share auf einem Debian-basierten System installiert. Alle Docker-Container sollen ihre Daten in einem `/export/docker`-Verzeichnis speichern, das per Samba freigegeben wird. Zudem wird ein Benutzer für die Verwaltung des Speichers erstellt.

---

## Disclaimer für Proxmox LXC Container

Wenn du Docker in einem Proxmox LXC Container installierst, müssen bestimmte Vorbereitungen getroffen werden.

---

## Voraussetzungen
- Ein Debian-basiertes System (z.B. Debian, Ubuntu, Proxmox LXC Container)
- Root- oder `sudo`-Zugriff

---


## Vorbereitung 1: Erstellung eines LXC Containers auf Proxmox

1. **Erstellen des Containers**:
   - Der Container muss **privilegiert** erstellt werden.

2. **Anpassen der Features nach der Installation**:
   - Öffne in der Proxmox-UI die **Optionen** des LXC-Containers und passe die Features wie folgt an:
     - **NFS** aktivieren
     - **SMB** aktivieren
     - **Nesting** aktivieren

3. **Bearbeiten der LXC-Konfigurationsdatei**:
   - Öffne die Konfigurationsdatei des Containers:
     ```bash
     nano /etc/pve/lxc/<container_id>.conf
     ```
   - Füge am Ende der Datei folgende Zeile hinzu:
     ```bash
     lxc.cap.drop:

     ```

---

So sollte die Struktur klarer und leichter verständlich sein. Wenn du noch Fragen hast oder weitere Informationen benötigst, lass es mich wissen!
  
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

Hier ist der erweiterte Abschnitt mit einer Anleitung, wie man das Docker-Verzeichnis wechselt, inklusive der Umstellung und dem Kopieren der bestehenden Daten, ohne Datenverlust:

---

### Docker-Verzeichnis ändern

Wenn du Docker so konfigurieren möchtest, dass es seine Daten in einem anderen Verzeichnis speichert, zum Beispiel im Verzeichnis `/export/docker/docker-data`, ( für Zukünftige Tutorials relevant ) kannst du dies folgendermaßen vornehmen:

1. **Erstelle das neue Verzeichnis**:
   ```bash
   sudo mkdir -p /export/docker/docker-data
   ```

2. **Bearbeite die `daemon.json`-Datei**:
   - Öffne die Konfigurationsdatei von Docker:
     ```bash
     sudo nano /etc/docker/daemon.json
     ```
   - Füge den folgenden Inhalt hinzu (oder passe den bestehenden Eintrag an):
     ```json
     {
       "data-root": "/export/docker/docker-data"
     }
     ```
   - Speichere die Datei.

3. **Kopiere die existierenden Docker-Daten in das neue Verzeichnis**:
   - Um Datenverlust zu vermeiden, kopierst du das bestehende Verzeichnis (standardmäßig unter `/var/lib/docker/`) in das neue Verzeichnis. Verwende dafür den folgenden `cp`-Befehl:
     ```bash
     sudo systemctl stop docker
     sudo cp -a /var/lib/docker/* /export/docker/docker-data/
     ```
     - Der Parameter `-a` sorgt dafür, dass alle Rechte, Besitzer und Timestamps der Dateien erhalten bleiben.
     - **Wichtig:** Docker muss gestoppt sein, um Datenbeschädigungen während des Kopierens zu vermeiden.

4. **Docker neu starten**:
   - Sobald die Daten kopiert sind, kannst du Docker mit dem neuen Verzeichnis starten:
     ```bash
     sudo systemctl start docker
     ```

5. **Überprüfen der neuen Konfiguration**:
   - Stelle sicher, dass Docker das neue Verzeichnis korrekt verwendet:
     ```bash
     docker info | grep "Docker Root Dir"
     ```
   - Die Ausgabe sollte das neue Verzeichnis, z.B. `/export/docker/docker-data`, anzeigen.

### Wichtige Hinweise:
- Achte darauf, dass genügend Speicherplatz im neuen Verzeichnis verfügbar ist.
- Du kannst das alte Verzeichnis (`/var/lib/docker/`) nach erfolgreichem Testen der neuen Konfiguration löschen, um Speicherplatz freizugeben:
   ```bash
   sudo rm -rf /var/lib/docker/
   ```

---

Dieser Abschnitt führt dich durch den gesamten Prozess: vom Ändern des Docker-Verzeichnisses bis zum Kopieren der bestehenden Daten ohne Datenverlust. Wenn du noch zusätzliche Schritte brauchst oder Anpassungen möchtest, lass es mich wissen!

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

eine Datei erstellen mit Namen docker-compose.yaml

```
version: '3'

services:
  hello-world:
    image: hello-world


```

Container starten mit folgendem befehl im gleichen Verzeichniss.

```
docker compose up -d

```

---

## Schritt 5: Portainer (optional) installieren

Portainer ist eine grafische Oberfläche zur Verwaltung von Docker-Containern. Es kann wie folgt installiert werden, wenn es sich um die Haupt-Node oder erste Node eines Docker-Clusters handelt:

```bash
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name=portainer \
  --restart=always -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data portainer/portainer-ce:latest
```

### Zugriff auf die Web-UI

Nach der Installation kann die Weboberfläche von Portainer im Browser über die IP-Adresse des Servers erreicht werden. Standardmäßig wird Port **9443** für den Zugriff verwendet.

```bash
https://<server-ip>:9443
```

Wenn der Zugriff über Port 9443 nicht funktioniert, stelle sicher, dass:
1. Der Port in der Firewall freigegeben ist:
   ```bash
   sudo ufw allow 9443/tcp
   ```
2. Der Docker-Container läuft:
   ```bash
   docker ps
   ```

Weitere Details findest du in der offiziellen Anleitung: [Portainer Installation](https://docs.portainer.io/start/install-ce/server/docker/linux).

---

Jetzt sollte alles klar verständlich sein, inklusive der Hinweise zur Erreichbarkeit der Web-UI.

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