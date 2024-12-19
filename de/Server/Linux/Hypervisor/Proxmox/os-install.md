---
title: Proxmox Installation auf Hardware
description: 
published: true
date: 2024-09-21T07:41:44.024Z
tags: 
editor: markdown
dateCreated: 2024-07-15T21:31:58.472Z
---

# Erstellen eines Installations-USB-Sticks aus einer Proxmox ISO-Datei mit Rufus

In diesem Artikel wird beschrieben, wie man einen Installations-USB-Stick aus einer Proxmox ISO-Datei mit dem Tool Rufus erstellt. Dieser USB-Stick kann verwendet werden, um Proxmox auf einem Server oder PC zu installieren.

## Voraussetzungen

- Ein USB-Stick mit mindestens 2 GB Speicherplatz
- Ein Computer mit Windows-Betriebssystem
- Die Proxmox ISO-Datei
- Das Tool Rufus

## Schritt 1: Herunterladen der Proxmox ISO-Datei

1. Öffnen Sie Ihren Webbrowser und gehen Sie zu [Proxmox Downloads](https://www.proxmox.com/de/downloads/proxmox-virtual-environment/iso).
2. Laden Sie die neueste Version der Proxmox Virtual Environment ISO-Datei herunter.

## Schritt 2: Herunterladen und Installieren von Rufus

1. Besuchen Sie die [Rufus Website](https://rufus.ie/de/).
2. Laden Sie die neueste Version von Rufus herunter.
3. Führen Sie die heruntergeladene Datei aus, um Rufus zu starten (es ist keine Installation erforderlich).

## Schritt 3: Erstellen des Installations-USB-Sticks

1. Stecken Sie den USB-Stick in einen freien USB-Port Ihres Computers.
2. Starten Sie Rufus.
3. Wählen Sie Ihren USB-Stick unter "Gerät" aus.
4. Klicken Sie auf "Auswahl" neben "Boot-Auswahl" und navigieren Sie zu der heruntergeladenen Proxmox ISO-Datei. Wählen Sie die ISO-Datei aus.
5. Die restlichen Einstellungen können in der Regel auf den Standardwerten belassen werden. Stellen Sie sicher, dass die Partitionierungsschema auf "MBR" und das Dateisystem auf "FAT32" eingestellt ist.
6. Klicken Sie auf "Start", um den Prozess zu beginnen. Rufus wird Sie warnen, dass alle Daten auf dem USB-Stick gelöscht werden. Bestätigen Sie dies, um fortzufahren.

## Schritt 4: Abschließen des Vorgangs

1. Warten Sie, bis Rufus den Vorgang abgeschlossen hat. Dies kann einige Minuten dauern.
2. Sobald der Vorgang abgeschlossen ist, können Sie Rufus schließen und den USB-Stick sicher entfernen.

Ihr Installations-USB-Stick mit der Proxmox ISO ist nun bereit zur Verwendung. Sie können ihn verwenden, um Proxmox auf einem Server oder PC zu installieren, indem Sie von diesem USB-Stick booten.

## Fazit

Das Erstellen eines Installations-USB-Sticks mit Rufus ist ein einfacher Prozess, der nur wenige Minuten dauert. Mit diesem USB-Stick können Sie schnell und einfach Proxmox auf einem beliebigen kompatiblen System installieren.

Viel Erfolg bei der Installation von Proxmox!
