---
title: SSH Authentication mit Public-Keys
description: 
published: true
date: 2024-09-20T10:03:24.902Z
tags: 
editor: markdown
dateCreated: 2024-09-20T10:03:24.902Z
---

### SSH-Konfiguration & Einrichtung für Anfänger

In diesem Tutorial lernst du, wie du SSH (Secure Shell) auf deinem Raspberry Pi oder einem anderen Linux-basierten System einrichtest und konfigurierst. Diese Anleitung richtet sich an absolute Anfänger, daher wird jeder Schritt ausführlich erklärt.

---

### 1. **Installieren von `sudo`, falls nicht vorhanden**

Manche Linux-Distributionen haben `sudo` nicht vorinstalliert. Sudo ermöglicht es dir, Befehle mit Administratorrechten auszuführen. Um es zu installieren, verwende folgenden Befehl:

```bash
apt-get install sudo -y
```

Nachdem `sudo` installiert ist, solltest du deinen Benutzer der `sudo`-Gruppe hinzufügen, damit er Befehle mit erhöhten Rechten ausführen kann:

```bash
sudo adduser admin
```

Um den Benutzer zur `sudo`-Gruppe hinzuzufügen:

```bash
sudo usermod -aG sudo admin
```

Überprüfen, ob der Benutzer in der `sudo`-Gruppe ist:

```bash
groups admin
```

Damit wird dir angezeigt, in welchen Gruppen der Benutzer ist. Du solltest `sudo` in der Liste sehen.

Um sicherzustellen, dass alles korrekt konfiguriert ist, bearbeite die sudoers-Datei:

```bash
sudo visudo
```

Falls der Eintrag `%sudo` fehlt, füge die folgende Zeile hinzu, damit alle Benutzer in der `sudo`-Gruppe volle Berechtigungen haben:

```bash
%sudo   ALL=(ALL:ALL) ALL
```

Wenn du nicht bei jedem Befehl dein Passwort eingeben möchtest, kannst du das wie folgt konfigurieren:

```bash
admin ALL=(ALL) NOPASSWD:ALL
```

Nun kann der Benutzer `admin` Befehle mit `sudo` ausführen, ohne jedes Mal ein Passwort eingeben zu müssen.

---

### 2. **SSH-Schlüssel erzeugen**

SSH-Schlüssel bieten eine sicherere Möglichkeit, sich in eine SSH-Sitzung einzuloggen, verglichen mit der Verwendung eines Passworts. Um einen SSH-Schlüssel zu erzeugen, benutze folgenden Befehl:

```bash
ssh-keygen -t rsa -b 4096 -C "admin"
```

- `-t rsa`: Gibt den Typ des Schlüssels an (hier RSA).
- `-b 4096`: Erzeugt einen 4096-Bit-Schlüssel, der sicherer ist.
- `-C "admin"`: Fügt einen Kommentar hinzu, um den Schlüssel zu identifizieren.

Du kannst auch **PuTTYgen** auf Windows verwenden, um einen SSH-Schlüssel zu generieren. Lade es von [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) herunter und folge einem Leitfaden zum Erstellen des Schlüssels (Link zum Tutorial wird noch erstellt).

---

### 3. **SSH-Schlüssel-Authentifizierung konfigurieren**

Um die SSH-Schlüssel-basierte Anmeldung für deinen `admin`-Benutzer zu ermöglichen:

1. Wechsel zum Benutzer `admin`:

   ```bash
   su admin
   ```

2. Erstelle das Verzeichnis `.ssh` und setze die korrekten Berechtigungen:

   ```bash
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   ```

3. Füge deinen öffentlichen SSH-Schlüssel zur Datei `authorized_keys` hinzu:

   ```bash
   echo "DEIN_SSH_PUBLIC_KEY" >> ~/.ssh/authorized_keys
   ```

   Zum Beispiel:

   ```bash
   echo "ssh-rsa AAAAB3Nza... user@hostname" >> ~/.ssh/authorized_keys
   ```

4. Setze die richtigen Berechtigungen:

   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

5. Wechsle wieder aus dem `admin`-Benutzer heraus:

   ```bash
   exit
   ```

Jetzt ist die SSH-Schlüssel-Authentifizierung für den Benutzer `admin` eingerichtet.

---

### 4. **Bearbeiten der SSH-Konfiguration**

Um die Sicherheit zu verbessern und die Schlüssel-basierte Anmeldung zu erlauben, müssen wir die SSH-Konfiguration anpassen. Öffne die Konfigurationsdatei:

```bash
sudo nano /etc/ssh/sshd_config
```

Ändere folgende Einstellungen:

1. Deaktiviere den Root-Login zur Sicherheit:

   ```bash
   PermitRootLogin no
   ```

2. Erlaube die Authentifizierung mit öffentlichen Schlüsseln:

   ```bash
   PubkeyAuthentication yes
   ```

3. Belasse die Passwortauthentifizierung für die Tests aktiviert, aber nur **für die Erstkonfiguration**. Deaktiviere sie später für mehr Sicherheit:

   ```bash
   PasswordAuthentication yes
   ```

   > **Wichtig**: Sobald die SSH-Schlüssel-Authentifizierung funktioniert, setze diese Einstellung auf `no`, um Passwortanmeldungen zu verhindern.

4. Stelle sicher, dass die folgenden Zeilen für X11-Weiterleitung und PAM-Authentifizierung korrekt gesetzt sind:

   ```bash
   UsePAM yes
   X11Forwarding yes
   ```

5. Speichere die Datei (`Ctrl + O`) und verlasse den Editor (`Ctrl + X`).

---

### 5. **SSH-Dienst neustarten**

Nachdem du die Konfiguration geändert hast, starte den SSH-Dienst neu, um die Änderungen zu übernehmen:

```bash
sudo service ssh restart
```

Um zu überprüfen, ob der Dienst ordnungsgemäß läuft, verwende:

```bash
sudo service ssh status
```

Du solltest sehen, dass SSH aktiv ist und läuft.

---

### Erklärungen für absolute Anfänger

1. **Was ist SSH?**
   SSH (Secure Shell) ist ein Protokoll, das es dir ermöglicht, sicher über das Internet oder ein lokales Netzwerk eine Verbindung zu einem anderen Computer herzustellen. Die Kommunikation zwischen dir und dem Server wird verschlüsselt, was sie sicher vor Angreifern macht.

2. **Was sind SSH-Schlüssel?**
   SSH-Schlüssel sind ein Paar kryptografischer Schlüssel (ein privater und ein öffentlicher Schlüssel), die eine sicherere Methode zur Anmeldung auf einem Server bieten, verglichen mit Passwörtern. Der öffentliche Schlüssel wird auf dem Server gespeichert, und nur derjenige, der den entsprechenden privaten Schlüssel besitzt, kann sich einloggen.

3. **Warum sollte man den Root-Login deaktivieren?**
   Der Root-Benutzer ist der mächtigste Benutzer in einem Linux-System. Wenn man den direkten Root-Login per SSH erlaubt, ist das ein Sicherheitsrisiko, da Angreifer oft versuchen, sich direkt als Root-Benutzer einzuloggen. Durch das Deaktivieren des Root-Logins erschwerst du es unautorisierten Benutzern, die Kontrolle über dein System zu übernehmen.

4. **Was ist PAM?**
   PAM (Pluggable Authentication Modules) ist ein System, das in Linux die Authentifizierung regelt. Es ermöglicht verschiedene Authentifizierungsmethoden, einschließlich Passwort- und Schlüssel-basierter Logins.

---

### Abschließende Gedanken

Nachdem du diese Schritte ausgeführt hast, ist deine SSH-Einrichtung sicherer, und du kannst dich mithilfe deines SSH-Schlüssels einloggen, ohne ein Passwort eingeben zu müssen. Denke daran, die Passwortauthentifizierung zu deaktivieren, sobald du sicher bist, dass die SSH-Schlüssel-Authentifizierung funktioniert.

Wenn du weitere Erklärungen oder zusätzliche Schritte benötigst, stehe ich gerne zur Verfügung!