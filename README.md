# SSH Root-Zugang & Benutzerkonfiguration

Dieser Leitfaden beschreibt, wie du einen sicheren SSH-Zugang für den Root-User einrichtest, SSH-Keys im [SCP](https://servercontrolpanel.de/) hinterlegst, und einen neuen Benutzer mit SSH-Login erstellst.  

***

## Quick Commands (Copy-Paste)

### Root SSH-Key generieren
```bash
ssh-keygen -t ed25519
```
### Root SSH-Pub kopieren
Vom Mac aus (neues Terminal):
```bash
cat ~/root.pub | ssh root@SERVERIP 'cat >> /root/.ssh/authorized_keys'
```
Oder per scp:
```bash
scp ~/root.pub root@SERVERIP:/root/.ssh/authorized_keys
```

### SSH nur Key-Login (sshd_config)
sshd öffnen und bearbeiten
```bash
sudo nano /etc/ssh/sshd_config
# Suche und setze:
PermitRootLogin yes
PubkeyAuthentication yes
PasswordAuthentication yes  # Optional: später auf "no" für Sicherheit
```

Nun systemctl neustarten
```bash
sudo systemctl restart ssh
```

### Admin-User erstellen
```bash
adduser admin
usermod -aG sudo admin
```

### Admin SSH-Key generieren (lokal)
```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_admin -C "user@pangolin-vps"
```

### Admin SSH-Key kopieren
```bash
ssh-copy-id -i ~/.ssh/id_ed25519_admin.pub admin@SERVER_IP
```

### Admin SSH-Rechte setzen (manuell)
```bash
mkdir -p /home/admin/.ssh
nano /home/admin/.ssh/authorized_keys
chown -R admin:admin /home/admin/.ssh
chmod 700 /home/admin/.ssh
chmod 600 /home/admin/.ssh/authorized_keys
```

### System aktualisieren
```bash
apt update && apt upgrade -y
```

***

## Schritt 1 – SSH-Schlüssel für Root generieren

Öffne ein Terminal und generiere einen SSH-Schlüssel:

```bash
ssh-keygen -t ed25519
```

Standardmäßig wird der Schlüssel unter `~/.ssh/id_ed25519` gespeichert. Optional kannst du ein Passwort hinzufügen (empfohlen).

***

## Schritt 2 – SSH Public Key im SCP speichern

1. Gehe zum [Server Control Panel (SCP)](https://servercontrolpanel.de/).  
2. Navigiere zu **SSH Keys**.  
3. Füge hier den **Public Key** ein und speichere ihn.  

***

## Schritt 3 – Server neu installieren

1. Wähle im SCP den gewünschten Server aus.  
2. Gehe zu **Medien → Images**.  
3. Wähle unter **Offizielle Images → Distribution** dein gewünschtes System.  
4. Klicke auf **Panel → Minimal / Minimal image**.  
5. Unter **Partitionen** → *kompletten freien Speicher wählen*.  
6. Unter **SSH Key** → den zuvor angelegten **Root Public Key** wählen.  

***

## Schritt 4 – Firewall-Policys setzen

Erlaube auf dem Server **nur folgende Ports**:

- 22 (SSH)  
- 80 (HTTP)  
- 443 (HTTPS)

***

## Schritt 5 – SSH nur per Key-Login erlauben

Öffne die SSH-Konfigurationsdatei:

```bash
sudo nano /etc/ssh/sshd_config
```

Füge die folgenden Einstellungen hinzu bzw. ändere sie:

```
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

Danach den SSH-Dienst neu starten:

```bash
sudo systemctl restart ssh
```

👉 Jetzt ist der Login **nur noch mit SSH-Key** möglich.

***

## Schritt 6 – SSH-Key für neuen User anlegen

### 1. Benutzer erstellen

```bash
adduser admin
```

Passwort kann gesetzt werden, wird später jedoch nicht genutzt.  
Anschließend Benutzer zu **sudo** hinzufügen:

```bash
usermod -aG sudo admin
```

Testen:

```bash
su - admin
sudo whoami
# Ausgabe: root ✔
```

***

## Schritt 7 – SSH-Key für neuen User anlegen (lokal)

Auf deinem Laptop:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_admin -C "user@pangolin-vps"
```

Ergebnis:

- Private Key: `~/.ssh/id_ed25519_admin`
- Public Key: `~/.ssh/id_ed25519_admin.pub`

***

## Schritt 8 – Public Key auf Server kopieren

### Variante A (empfohlen)
```bash
ssh-copy-id -i ~/.ssh/id_ed25519_admin.pub admin@SERVER_IP
```
👉 Nur dieses eine Mal wird das Passwort genutzt.

### Variante B (manuell)

```bash
mkdir -p /home/admin/.ssh
nano /home/admin/.ssh/authorized_keys
```

➡️ Public Key einfügen und Rechte setzen:

```bash
chown -R admin:admin /home/admin/.ssh
chmod 700 /home/admin/.ssh
chmod 600 /home/admin/.ssh/authorized_keys
```

***

## Schritt 9 – SSH-Login testen

```bash
ssh -i ~/.ssh/id_ed25519_admin admin@SERVER_IP
```

✅ Kein Passwort notwendig  
✅ Login funktioniert  

Nur wenn das klappt → weiter mit Systemvorbereitung.

***

## Mini-Checkliste: Debian 13 vorbereiten

```bash
apt update && apt upgrade -y
```

System ist nun bereit zur Nutzung. 🎉
