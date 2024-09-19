---
title: Proxmox-LXC-Pi-hole-EASY
description: 
published: true
date: 2024-09-19T23:11:40.104Z
tags: 
editor: markdown
dateCreated: 2024-09-19T23:10:45.385Z
---

# PiHole Installation in einem LXC Container auf einem Proxmox Server

In diesem Tutorial lernst du, wie du PiHole in einem LXC Container auf einem Proxmox Server installierst. Es wird vorausgesetzt, dass du bereits Screenshots der Installation gemacht hast und der Proxmox Server bereits läuft.

## Schritt 1: Herunterladen des Debian 12 LXC Templates

1. Öffne die Proxmox Web GUI.
2. Gehe zu **local storage** unter deiner Proxmox Node.
3. Wähle den Reiter **CT Templates** und lade das Debian 12 Template herunter.

![screenshot_2024-09-19_190901.png](/proxmox/lxc/screenshot_2024-09-19_190901.png)

![screenshot_2024-09-19_190959.png](/proxmox/lxc/screenshot_2024-09-19_190959.png)

## Schritt 2: Erstellen des LXC Containers

1. Rechtsklicke auf deine Proxmox Node und wähle **Create CT**.

![screenshot_2024-09-19_191036.png](/proxmox/lxc/screenshot_2024-09-19_191036.png)

2. Gib in der **General** Ansicht die folgenden Informationen ein:
   - **CT ID**: (eine eindeutige ID für den Container)
   - **Hostname**: pihole
   - **Password**: Definiere ein Passwort für den Container

![screenshot_2024-09-19_191118.png](/proxmox/lxc/screenshot_2024-09-19_191118.png)

3. In der **Template** Ansicht wähle das heruntergeladene **Debian 12** Template auf **local** aus.

![screenshot_2024-09-19_191147.png](/proxmox/lxc/screenshot_2024-09-19_191147.png)

4. In der **Disks** Ansicht:
   - **Disk Storage**: Wähle **local-lvm** aus.
   - **Disk Size**: Setze die Größe auf 10GB.

![screenshot_2024-09-19_191207.png](/proxmox/lxc/screenshot_2024-09-19_191207.png)

5. In der **CPU** Ansicht:
   - **CPU Cores**: Wähle 1 Kern.

![screenshot_2024-09-19_191224.png](/proxmox/lxc/screenshot_2024-09-19_191224.png)

6. In der **Memory** Ansicht:
   - **Memory (RAM)**: Setze den Wert auf 1024 MB.



## Schritt 3: Netzwerkeinstellungen konfigurieren

1. In der **Network** Ansicht wähle:
   - **IPv4**: Setze eine statische IP, z.B. `10.1.100.190/24`.
     - **Hinweis**: Wähle eine IP-Adresse außerhalb des DHCP-Ranges deines Routers.
		  ![screenshot_2024-09-19_191732.png](/proxmox/lxc/screenshot_2024-09-19_191732.png)

   - **Bridge**: Wähle **vmbr0** (Standardbridge).

![screenshot_2024-09-19_192742.png](/proxmox/lxc/screenshot_2024-09-19_192742.png)


2. Optional: In der **DNS** Ansicht kannst du einen **DNS Server** eintragen, z.B. `8.8.8.8`. Später kannst du den Upstream-DNS-Server in PiHole konfigurieren.

![screenshot_2024-09-19_191358.png](/proxmox/lxc/screenshot_2024-09-19_191358.png)

![screenshot_2024-09-19_191420.png](/proxmox/lxc/screenshot_2024-09-19_191420.png)


3. Klicke auf **Finish** und erstelle den Container.

## Schritt 4: Starten und Einrichten des Containers

1. Starte den erstellten LXC Container.

2. Öffne die Shell des Containers.

![screenshot_2024-09-19_191506.png](/proxmox/lxc/screenshot_2024-09-19_191506.png)

3. Führe folgende Befehle aus, um den Container zu aktualisieren und `curl` zu installieren:

   ```bash
   apt update && apt upgrade -y && apt install curl
   ```
![screenshot_2024-09-19_192518.png](/proxmox/lxc/screenshot_2024-09-19_192518.png)


## Schritt 5: Installation von PiHole

1. Besuche die [offizielle Installationsseite von PiHole](https://docs.pi-hole.net/main/basic-install/).

![screenshot_2024-09-19_191529.png](/proxmox/lxc/screenshot_2024-09-19_191529.png)

2. Kopiere den aktuellen Installationsbefehl und führe ihn im LXC Container aus:

   ```bash
   curl -sSL https://install.pi-hole.net | bash
   ```

![screenshot_2024-09-19_191551.png](/proxmox/lxc/screenshot_2024-09-19_191551.png)

## Schritt 6: Konfiguration während der Installation

Während der Installation wirst du durch mehrere Konfigurationsschritte geführt:






1. **Netzwerkadapter auswählen**: Falls du mehrere Netzwerkadapter hast, wähle den richtigen.
   ![screenshot_2024-09-19_205223.png](/proxmox/lxc/screenshot_2024-09-19_205223.png)
2. **Upstream DNS**: Wähle den Upstream DNS-Server (z.B. Google `8.8.8.8`).
	 ![screenshot_2024-09-19_205311.png](/proxmox/lxc/screenshot_2024-09-19_205311.png)
3. **Blocklists**: Installiere die Standard Blockliste von Steven Black.
   ![screenshot_2024-09-19_205327.png](/proxmox/lxc/screenshot_2024-09-19_205327.png)
4. **Admin Interface**: Installiere das Admin Interface.
   ![screenshot_2024-09-19_205353.png](/proxmox/lxc/screenshot_2024-09-19_205353.png)
5. **Webserver**: Installiere den Webserver.
   ![screenshot_2024-09-19_205405.png](/proxmox/lxc/screenshot_2024-09-19_205405.png)
6. **Query Logging**: Aktiviere das Query Logging.
   ![screenshot_2024-09-19_205418.png](/proxmox/lxc/screenshot_2024-09-19_205418.png)
7. **Anzeigemodus für Logs**: Wähle **Show Everything** (zeigt alle Logs an).
   ![screenshot_2024-09-19_205432.png](/proxmox/lxc/screenshot_2024-09-19_205432.png)



Nach Abschluss der Installation wird dir ein Passwort für das Admin Interface angezeigt. Du kannst dieses später mit dem folgenden Befehl ändern:

![screenshot_2024-09-20_004009.png](/proxmox/lxc/screenshot_2024-09-20_004009.png)


```bash
pihole -a -p

```

## Schritt 7: Zugriff auf das PiHole Admin Interface

Das Admin Interface von PiHole erreichst du unter:

```
http://10.1.100.190/admin
```

![screenshot_2024-09-20_004225.png](/proxmox/lxc/screenshot_2024-09-20_004225.png)

## Schritt 8: DNS-Server im Router konfigurieren

Damit PiHole als DNS-Server für dein Netzwerk fungiert, ändere die DNS-Servereinstellungen in deinem Router. Setze die statische IP deines PiHole Containers als DNS-Server für den DHCP-Bereich deines Netzwerks.

```bash
10.1.100.190
```

Damit ist die Installation von PiHole in einem LXC Container auf einem Proxmox Server abgeschlossen!
``