# 📼 Backup: Apple Timemachine

Wie man mit Hilfe eines Raspberry Pi oder ähnlichem mit Docker Compose eine Backup Lösung für seinen Mac aufsetzt, die der original Apple TimeMachine in nichts nachsteht, erkläre ich in diesem Post.

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/f9056c34-679e-4864-b897-898633c4b01b/01_mac_time_machine_logo-56a5d49b5f9b58b7d0de9f0b-1.jpg " =768x512")


## Grundbegriffe, Backup vs. Synchronisation

Mit der sogenannten TimeMachine hatte Apple es wieder einmal geschafft, einen Service der bei anderen wahnsinnig kompliziert einzurichten ist/war, kinderleicht bedienbar zu machen. Leider scheinen Datensicherungen in das lokale Netz aus der Mode gekommen zu sein, so propagiert Apple selbst nur noch das Backup direkt auf einen USB-Speicher oder eben in die eigene iCloud. Dabei haben beide Lösungen größere Nachteile. Für das lokale Backup muss ich immer daran denken, dass externe Speichermedium in den Rechner zu stecken und kommt es z.B. zu einem Kurzschluss während beide Geräte miteinander verbunden sind, sind auch beide zerstört. Dahin ist die Datensicherung, genau dann wenn man sie am meisten braucht. Die Alternativen iCloud und Co. begleiten ebenfalls mehrere Nachteile. Zum Einem vertrauen wir unsere Daten einem externen Anbieter an, der uns zwar versprechen kann dass diese dort sicher und geschützt aufbewahrt werden, aber garantieren kann uns das niemand. Außerdem handelt es sich bei iCloud und ähnlichen zunächst nur um Services zur Dateisynchronisation. D.h. meine Daten werden parallel lokal und eben in der Cloud abgelegt. Das heisst auch, wenn ich eine Datei lokal lösche, passiert das gleiche auch in der Cloud. Und wieder Adieu liebe Dateischerung. Dem entgegen wirk die Dateiversionierung die entweder Standardmäßig oder optional aktiviert werden kann, allerdings habe ich selbst die Erfahrung gemacht, dass diese auch genau dann wenn man sie bräuchte gerade mal nicht funktioniert.

Aus diesen und anderen Gründen hat Apple noch ein „Türchen" offen gelassen und stellt es anderen Herstelleren z.B. von Netzwerkspeichern wie QNAP frei, TimeMachine in ihre Hardwarelsöungen einzubinden, auch wenn Apple selbst keine entsprechende Hardware mehr vertreibt.

Falls ihr aber so wie ich sowieso schon einen Linux Server oder RaspberryPi mit Docker Compose habt, geht es auch noch einfacher: Wir konfigurieren einfach einen Container der als TimeMachine Server im Heimnetz dient!

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/7423b33c-b975-4a23-b83a-406616a531bd/02_timemachinenet2usb.png " =1135x382")


## Server vorbereiten

Zunächst verbinden wir unseren externen Speicher, ich selbst verwende eine externe Festplatte, mit unserem Server. Falls nicht schon geschehen, formatieren wir unseren Speicher noch in einem für Linux nativen Dateisystem.

## Festplatte formatieren

Um eine externe Festplatte in der Linux-Kommandozeile mit dem Dateisystem ext4 zu formatieren, kannst du die folgenden Schritte befolgen:


1. Schließe die externe Festplatte an deinen Computer an. Stelle sicher, dass sie erkannt wird und einen zugewiesenen Gerätepfad hat. Du kannst dies mit dem Befehl lsblk überprüfen, der eine Liste der blockbasierten Geräte anzeigt.
2. Öffne ein Terminal oder eine Konsole, um die Linux-Kommandozeile zu öffnen.
3. Gib den Befehl sudo fdisk -l ein, um eine Liste der erkannten Festplatten und ihrer Partitionen anzuzeigen. Finde den Gerätepfad deiner externen Festplatte in der Liste. Normalerweise wird sie als „/dev/sdX" bezeichnet, wobei „X" für einen Buchstaben steht (z. B. /dev/sdb).
4. Stelle sicher, dass du den richtigen Gerätepfad auswählst und die Daten auf der Festplatte sicher gesichert hast. Das Formatieren einer Festplatte löscht alle darauf befindlichen Daten unwiederbringlich.
5. Gib den folgenden Befehl ein, um das Festplattenformat zu ändern und das Dateisystem ext4 zu erstellen:

```bash

sudo mkfs.ext4 /dev/sdX
```

Ersetze „/dev/sdX" durch den tatsächlichen Gerätepfad deiner externen Festplatte. 6. Der Befehl wird dich fragen, ob du fortfahren möchtest, da er alle Daten auf der Festplatte löschen wird. Bestätige mit „y" und drücke die Eingabetaste. 7. Der Formatierungsvorgang beginnt und kann je nach Größe der Festplatte einige Zeit in Anspruch nehmen. 8. Sobald der Vorgang abgeschlossen ist, erhältst du eine Bestätigungsmeldung.

Deine externe Festplatte sollte nun erfolgreich mit dem ext4-Dateisystem formatiert sein und bereit für die Verwendung unter Linux sein.

## Festplatte/Speicher dauerhaft mounten

Als nächstens wollen wir erreichen, dass der Speicher bzw. die Festplatte bei jedem Neustart des Systems automatisch „gemountet" (an das System angehangen wird).

Um eine mit ext4 formatierte Festplatte in der /etc/fstab-Datei zu mounten, kannst du die folgenden Schritte befolgen:


1. Öffne ein Terminal oder eine Konsole, um die Linux-Kommandozeile zu öffnen.
2. Gib den Befehl `sudo blkid` ein, um eine Liste der erkannten Festplatten und ihrer UUIDs anzuzeigen. Finde die UUID deiner ext4-formatierten Festplatte in der Liste. Die UUID sieht in etwa so aus: UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.
3. Erstelle einen Ordner, der als Mountpunkt für die Festplatte dienen soll. Du kannst dies mit dem Befehl sudo `mkdir <mountpunkt>` tun, wobei <mountpunkt> der Pfad zu dem gewünschten Ordner ist. Zum Beispiel: `sudo mkdir /mnt/backup` .
4. Öffne die /etc/fstab Datei in einem Texteditor mit Root-Berechtigungen. Zum Beispiel: `sudo nano /etc/fstab` .
5. Füge eine neue Zeile am Ende der /etc/fstab Datei hinzu, um die Festplatte zu mounten. Die Syntax lautet: `UUID=<UUID> <mountpunkt> ext4 defaults 0 2` . Ersetze <UUID> durch die UUID deiner Festplatte und <mountpunkt> mit dem Pfad zum zuvor erstellten Ordner, z.B. `UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx /mnt/backup ext4 defaults 0 2`.
6. Speichere die Änderungen und schließe den Texteditor.
7. Um die Festplatte sofort zu mounten, ohne den Computer neu zu starten, gib den Befehl `sudo mount -a` ein. Dadurch werden die Einträge in der /etc/fstab Datei gelesen und die entsprechenden Festplatten gemountet.

Die Festplatte wird nun jedes Mal automatisch beim Systemstart gemountet, indem die Informationen in der /etc/fstab-Datei verwendet werden. Du kannst auf die Dateien und Ordner in der Festplatte über den angegebenen Mountpunkt zugreifen.

## Docker Container mit Compose einrichten

Falls ihr Docker und Docker Compose noch nicht installiert habt, ist das vorgehen hierzu hier beschrieben. Wir nutzen das Image <a href="<https://hub.docker.com/r/mbentley/timemachine>" target="_blank">mbentley/timemachine</a> vom Docker Hub um die Funktionalität der TimeMachine so nachzustellen, wie es auch die Anbieter von professionellen Backuplösungen tun.

Um den Docker-Container „mbentley/timemachine:smb" mit Docker Compose zu starten, erstelle bitte eine compose.yml-Datei mit den entsprechenden Konfigurationen. Hier ist ein Beispiel, das auf dem Image basiert und einen Dienst für den Container definiert:

```yaml

version: "3.3"
services:
  timemachine:
    image: mbentley/timemachine:smb
    container_name: timemachine
    hostname: timemachine
    restart: unless-stopped
    ports:
      - 137:137/udp
      - 138:138/udp
      - 139:139
      - 445:445
    logging:
      options:
        max-size: "10m"
        max-file: "3"
    environment:
      TZ: 'Europe/Berlin'
      CUSTOM_SMB_CONF: "false"
      CUSTOM_USER: "false"
      DEBUG_LEVEL: "1"
      HIDE_SHARES: "no"
      EXTERNAL_CONF: ""
      MIMIC_MODEL: "TimeCapsule8,119"
      TM_USERNAME: "timemachine"
      TM_GROUPNAME: "timemachine"
      TM_UID: "1000"
      TM_GID: "1000"
      PASSWORD: "DEINPASSWORT"
      SET_PERMISSIONS: "false"
      SHARE_NAME: "TimeMachine"
      SMB_INHERIT_PERMISSIONS: "no"
      SMB_NFS_ACES: "yes"
      SMB_PORT: "445"
      SMB_VFS_OBJECTS: "acl_xattr fruit streams_xattr"
      VOLUME_SIZE_LIMIT: "1 T"
      WORKGROUP: "WORKGROUP"
    volumes:
      - /mnt/backup/timemachine:/opt/timemachine
```

Speichere die Datei als compose.yml. Dieses Beispiel verwendet den Port 445 für den SMB-Zugriff. Es bindet auch das Verzeichnis /mnt/backup/timemachine auf dem Host mit dem Verzeichnis /opt/timemachine im Container, so dass die Backups auf der externene Festplatte gespeichert werden.

Führe dann den folgenden Befehl aus, um den Container zu starten: `docker compose up -d`. Dadurch wird der Container im Hintergrund gestartet. Die Option -d stellt sicher, dass der Dienst im Hintergrund (detached mode) läuft.

Jetzt läuft der Container „mbentley/timemachine:smb" und ist über die konfigurierten Ports erreichbar. Du kannst dann eine Time Machine-Verbindung zu deinem Docker-Host mit dem entsprechenden SMB-Protokoll herstellen und das Verzeichnis /mnt/backup/timemachine verwenden, um deine Backups zu speichern.

## Backup auf dem Mac konfigurieren

Nun haben wir alle vorbereitenden Schritte auf unserem Server abgeschlossen und widmen uns unserem zu sicherenden Mac. Ich habe die Erfahrung gemacht, dass sich ein Backup am leichtesten einrichten lässt, wenn man sich zuvor mit dem Netzlaufwerk auf dem Server verbindet. Um dich mit einem SMB-Laufwerk auf deinem Mac zu verbinden, befolge diese Schritte:


1. Öffne den Finder auf deinem Mac.
2. Klicke in der Menüleiste auf „Gehe zu" und wähle „Mit Server verbinden…" (oder verwende die Tastenkombination „Cmd + K").
3. Gib die Adresse des SMB-Laufwerks in das Eingabefeld ein. Die Adresse kann entweder die IP-Adresse des Servers oder sein Netzwerkname (falls verfügbar) sein. Das Format der Adresse lautet `smb://<adresse>`. Zum Beispiel: `smb://192.168.0.100` oder `smb://meinserver`.
4. Klicke auf „Verbinden".
5. Es wird ein Dialogfenster angezeigt, in dem du deine Anmeldeinformationen eingeben musst. Gib den Benutzernamen und das Passwort für den Zugriff auf das SMB-Laufwerk, so wie in der Docker Compose Datei konfiguriert, ein. Du hast auch die Möglichkeit, das Kästchen „Anmeldeinformationen im Schlüsselbund speichern" anzukreuzen, um die Anmeldeinformationen für zukünftige Verbindungen zu speichern.
6. Klicke auf „Verbinden".
7. Wenn die Anmeldeinformationen korrekt sind und der Zugriff gewährt wurde, wird das SMB-Laufwerk im Finder angezeigt. Du kannst nun auf die Dateien und Ordner des Laufwerks zugreifen und diese bearbeiten.

Das SMB-Laufwerk wird nach der Verbindung im Abschnitt „Freigaben" im Finder angezeigt. Du kannst auch ein Lesezeichen für das SMB-Laufwerk erstellen, indem du es zum Finder-Sidebar ziehst. Dadurch wird der Zugriff in Zukunft vereinfacht.

Wenn du die Verbindung trennen möchtest, kannst du das SMB-Laufwerk einfach aus der Seitenleiste des Finders ziehen oder mit der rechten Maustaste auf das Laufwerk klicken und „Verbindung trennen" auswählen.

Bitte beachte, dass für eine erfolgreiche Verbindung das SMB-Protokoll auf dem SMB-Laufwerk aktiviert sein muss und die Netzwerkeinstellungen des Macs korrekt konfiguriert sein sollten.

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/42eb81d9-c622-433b-8b69-fb62a1eb4bc3/03_bildschirmfoto-2023-05-17-um-16.38.32.png " =808x330")


Um ein regelmäßiges Backup auf einen Netzwerkspeicher (Network Attached Storage, NAS) unter macOS einzurichten, kannst du Time Machine verwenden. Folge diesen Schritten:


 1. Stelle sicher, dass dein Netzwerkspeicher ordnungsgemäß mit deinem Netzwerk verbunden ist und zugänglich ist.
 2. Gehe zu „Systemeinstellungen" auf deinem Mac und klicke auf „Time Machine".
 3. Klicke auf „Time Machine aktivieren".
 4. Klicke auf „Weitere Optionen".
 5. Wähle die Option „Backup-Datenträger auswählen".
 6. Wähle im Finder den Netzwerkspeicher aus, den du für das Backup verwenden möchtest, und klicke auf „Auswählen".
 7. Time Machine wird das Backup-Volume überprüfen und formatieren, falls erforderlich.
 8. Nachdem das Volume ausgewählt wurde, kehre zu den Time Machine-Einstellungen zurück.
 9. Aktiviere das Häkchen bei „Automatische Backups".
10. Du kannst auch den Zeitplan für die Backups anpassen, indem du das Häkchen bei „Automatische Backups" deaktivierst und dann bestimmte Zeiten festlegst, zu denen die Backups stattfinden sollen.
11. Du kannst außerdem die Einstellungen für „Ausgeschlossene Elemente" anpassen, um bestimmte Ordner oder Dateien von den Backups auszuschließen.

Sobald du die oben genannten Schritte abgeschlossen hast, wird Time Machine regelmäßig automatische Backups auf deinem Netzwerkspeicher durchführen. Beachte, dass die Geschwindigkeit der Backups von der Netzwerkverbindung abhängt und möglicherweise länger dauern kann als bei einem direkt angeschlossenen Speichergerät.

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/8dfbdf6c-db07-43e4-9a7e-b21daa938733/03_bildschirmfoto-2023-05-17-um-16.38.32.png " =808x330")


Es ist auch wichtig sicherzustellen, dass der Netzwerkspeicher ordnungsgemäß konfiguriert ist und mit dem Mac kompatibel ist, um die bestmögliche Time Machine-Unterstützung zu gewährleisten. Konsultiere die Dokumentation deines NAS-Herstellers für weitere Informationen zur Einrichtung der Time Machine-Unterstützung.