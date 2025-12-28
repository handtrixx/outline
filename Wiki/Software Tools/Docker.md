# Docker

Docker hilft uns durch sogenannte Container dabei die technische Serverinfrastruktur, die für verschiedenste Projekte benötigt wird, schnell und vom Basissystem isoliert aufzusetzen. Außerdem veringern wir durch den Einsatz von Docker den Wartungsaufwand z.B. für Systemaktualsierungen und Backups. Tausende von Projekten stehen vorgefertigt zur Verfügung, die mit einem einfachen Kommando nur gestartet werden können. In diesem Artikel beschreibe ich wie Docker und das zusätzlich hilfreiche Docker Compose installiert werden. Docker Compose hilft uns dabei ganze Containerlandschaften zur orchestrieren. Ein Beispiel: Bei der Installation von WordPress über Docker werden mehrere Container benötigt, einer für die App und einer für die Datenbank. Wenn wir diese nun alle einzeln starten und evtl. dabei noch bestimmte Parameter übergeben wollen, wird es schnell uünbersichtlich. Mit Compose können wir alle benötigten Parameter und Container in einer strukturierten Datei zusammenfassen und diese Umgebung mit einfachen Befehlen steuern.

## Vorraussetzungen

Wir gehen davon aus, dass wir ein Linuxsystem auf Basis von Debian als Betriebssystem nutzen. Das können z.B. das Raspberry Pi OS, Ubuntu oder eine der vielen anderen Varianten davon sein. Grunsätzlich funktioniert Docker aber auch unter allen anderen Linux Systemen und auch unter Windows oder MacOS.

## Installation der Docker Engine

Wir wollen immer die aktuellsten Versionen nutzen und orientieren uns deshalb an den Anleitungen auf den offiziellen Docker Seiten dazu.


1. Docker installieren: <https://docs.docker.com/engine/install/debian/>
2. Schritte zur komfortableren Nutzung von Docker: <https://docs.docker.com/engine/install/linux-postinstall/>

Wenn man sich nicht von der augeinscheinlich großen Masse der Schritte erschlagen lässt, sehen wir dass wir im Endeffekt nur 3-4 Kommandos per Copy & Pase in unsere Kommandozeile kopieren mussten.

Am besten führen wir abschließend noch einmalig einen Reboot unseres System durch.

## Verwendung von Docker Compose

Der Aufbau einer Docker-Composse Konfiguration ist im endeffeckt immer gleich und basiert auf einer einzelnen Datei, der compose.yml. Diese legt man am besten nicht einfach irgendwo hin, sondern überlegt sich eine Ordnerstruktur, da man ja evtl. viele Umgebungen parallel betreiben und den Überblick behalten möchten. Ein Beispiel:

* wir legen im Verzeichnis /opt/ ein Unterverzeichnis /opt/docker/an.
* Und hier dann wiederum Unterverzeichnisse z.B. nach Projekten oder URLs gegliedert. Also zum Beispiel /opt/docker/wordpress/.
* In dieses Untervzeichnis gehört dann unsere Datei zur Definition der Umgebung, also /opt/docker/wordpress/compose.yml
* Wenn wir beim Beispiel WordPress bleiben bekommen wir auf <https://hub.docker.com/_/wordpress> ein Beispiel für den Inhalt der compose.yml

Dieses Beispiel wollen wir noch etwas für uns anpassen und für den dauerhaften Betrieb optimieren. Das kann dann so aussehen (Achtung die Formatierung mit den freien Zeichen als Tabs ist bindend):

```yaml

services:
  wordpress-app:
    container_name: wordpress-app
    hostname: wordpress-app
    image: wordpress
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: wordpress-db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
      TZ: Europe/Berlin
    volumes:
      - ./wordpress:/var/www/html
    logging:
      options:
        max-size: "10m"
        max-file: "3"

  wordpress-db:
    container_name: wordpress-db
    hostname: wordpress-db
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'      
      TZ: Europe/Berlin
    volumes:
      - ./db:/var/lib/mysql
    logging:
      options:
        max-size: "10m"
        max-file: "3"
```

Zur Erklärung:

* service, container_name, hostname: Nur den Service selbst zu benennen ist Pflicht. Die anderen beiden Variablen setzen wir um unseren Container je nach Zugriffsart schnell finden zu können, wenn wir ihn mal suchen.
* image: Die Anwendung welche wir verwenden möchten, im Standard ein Image von Docker Hub <https://hub.docker.com/search?q=>
* restart: always bedeutet, dass der Container neu gestartet wird, egal warum auch immer er zuvor abgebrochen/abgestürzt ist.
* environment: Verschiedene Umgebungsvariablen die an den Container weitergegben werden. Das können je nach Image ganz unterschiedliche sein.
* volumes: Persistente Daten, die wir z.B. auch für ein Backup berücksichtig bzw. grundsätzlich unter unserer Kontrolle haben und nicht im Container "versteckt" haben wollen. durch ./bestimmen wir, dass es sich dabei um einen Uneterverzeichnis des aktuellen Ordners handelt. Falls nicht vorhanden würde dieser automatisch angelegt werden, besser aber vorher schon selber anlegen (Berechtigungen, etc…).
* logging: Manche Container/Applikationen sind recht freudig im Schreiben von Protokolldateien. Hiermit schränken wir dies auf 3 10MB große Dateien ein, wobei der älteste Eintrag beim überschreiten der Grenze überschrieben wird.
* ports: Falls kein Reverse Proxy zum Einsatz kommt, gibt z.B. 8080:80 an, dass der im Container laufende Port 80 and das Host System unter Port 8080 weitergleitet wird. D.h. wenn wir in unserem Beispiel WordPress aufrufen möchten, dann mit: <http://HOST:8080> . Zur Kommunikation der Container Untereinander also z.B. von App zur DB wird keine Portfreigabe benötigt! Das erhöht die gleichzeitig die Sicherheit unserer Umgebung.

## Nützliche Befehle

Aus dem Verzeichnis heraus, in dem wir unsere compose.yml angelegt haben können wir unsere Umgebung nun mit `docker compose up -d` starten. Wenn wir alle Container aus der Compose Datei herunterfahren möchten, geht das mit `docker compose down`. Mit dem Befehl `docker compose pull` ziehen wir die aktuellste Version unserer Images von Docker Hub. Nach dem Pull ist ein erneutes `docker compose up -d` erforderlich um die Container in der neuen Version zu starten. Die persistenten Daten aus den definierten Volumes bleiben dabei natürlich erhalten. Gefährlich wird es nur bei größeren Versionssprüngen innerhalb der Anwendung. Dann gibt es entweder die Möglichkeit die Umgebung herunterzufahren, das letzte geladene Image zu löschen und dann wieder zu starten. Alternativ lässt sich in der compose.yml auch eine version des Images fest einstellen. Damit verzichtet man in der Regel aber auch Sicherheitsupdates.

Warum um Himmels willen sollte man Containern feste IP-Adresse zuweisen? Eine absolut berechtigte Frage auf die die Antwort lautet: Am besten nie! Denn: Docker kommt mit einer Art eingebauten DNS- und DHCP-Server und verwaltet die IP-Adresse wunderbar selbst. Leider gibt es trotzdem Spezialfälle in denen man sich das Leben vereinfachen kann, wenn man IPs händisch setzt. So ein Spezialfall wäre z.B. gegeben, wenn die Applikation die in einem Container betrieben wird nicht mit Hostnamen sondern nur mit IP-Adressen umgehen kann, um auf einen weiteren Container zuzugreifen.

Früher war das ein relativ kompliziertes unterfangen, mit den aktuellen Docker (Compose) Versionen ist aber auch das kinderleicht.

## IP-Adressen laufender Container herausfinden

Falls du bereits diverse Container innerhalb eines Netzes am laufen hast, wäre es gefährlich nur einigen davon eine feste IP-Adresse zuzuordnen. Warum? Weil, es dir dann z.B. nach einem Reboot deines Servers passieren könnte, dass sich die automatisch zugewiesene IP eines Containers mit einer von dir fest zugeordneten eines anderen Containers überschneidet. Das Resultat wäre, dass der zweite Container nicht starten kann, da die IP ja bereits vergeben ist.

Um zu erfahren welche IP-Adresse einem Container aktuell zugewiesen ist, hilft uns das Kommando

```bash

docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' CONTAINERNAMEoderID
```

Falls man die IP-Adressen aller laufenden Container herausfinden möchte, ist dieses Kommando sehr nützlich:

```bash

docker inspect $(docker ps -q ) --format='{{ printf "%-50s" .Name}} {{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}'
```

## Ein Netz einrichten

In unserem Beispiel wollen wir allen Containern die in einem für unseren Caddy Reverse Proxy erstellen Netz liegen feste IP-Adressen vergeben. Falls ihr noch kein solches Netz habt, könnt ihr dieses über den Befehl

```bash

docker network create --subnet=172.28.0.0/16 proxy
```

anlegen. Nachdem dies erfolgt ist, können wir nun in den Docker Compose Definitionen der einzelnen Container die IP-Adressen 172.20.0.X verwenden. Wobei X für die Zahlen 1-255 steht.

## IP-Zuweisung mit Docker Compose

Beginnen wir mit der Zuweisung einer festen IP für unseren Caddy Reverse Proxy:

```yaml

services:
  caddy:
    container_name: caddy
    image: caddy:latest
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./site:/srv
      - ./caddy_data:/data
      - ./caddy_config:/config
    logging:
        options:
          max-size: "10m"
          max-file: "3"
    networks:
        proxy:
          ipv4_address: 172.28.0.1

volumes:
  caddy_data:
  caddy_config:
networks:
    proxy:
        external: true
```

Nach dem speichern und anschließendem Neustart der Umgebung, ist die IP fest zugeordnet. Entscheidend sind die Blöcke:

```yaml
    networks:
        proxy:
          ipv4_address: 172.28.0.1
```

sowie

```yaml

networks:
    proxy:
        external: true
```

Nun verfahren wir genauso mit allen anderen Containern mit einer jeweils individuellen IP und haben unser vorhaben erfolgreich beendet.

## Fazit

Auch wenn die Zuweisung fester IP-Adressen relativ einfach ist, erscheint das Ganze als etwas „unsauber". So widerspricht die Vorgehensweise doch dem Prinzip, dass ich die Definition eines Docker Containers nehmen und jederzeit auf einem anderen Server/Host starten kann. Das ist so nun nicht mehr möglich, weil zunächst manuell ein neues Netz angelegt werden muss. Anderseits ist die manuelle Erstellung des Netzes sowieso erforderlich, wenn man eine Art DMZ (eigenes Netz) für seinen Reverse Proxy definiert. Nur der manuell vergebene Subnetzbereich ist hier der zusätzliche Aufwand. So bleibt die Verwendung fester IP-Adressen als ein Spezialfall der nur in bestimmten Szenarien Sinn macht.