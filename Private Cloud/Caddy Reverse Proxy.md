# Caddy Reverse Proxy

Heute beschreibe ich was ein Reverse Proxy ist und warum er in deiner Docker Landschaft nicht fehlen darf. Nachdem ich mehrere Jahre lang sowohl Traefik, als auch den Nginx Proxy Manager als Reverse Proxy für meine Docker Container genutzt habe, bin ich nun bei Caddy gelandet. Am Nginx Proxy Manager störte mich, dass zur Konfiguration ausschließlich die Web UI zur Verfügung steht. Das hat mich in Backup/Restore Szenarien öfter an die Grenzen gebracht, was zum Schluss dazu führte jedes Mal aufs neue eine Klickorgie zu veranstalten. An Traefik störte mich wiederum die aufgeblähte Konfiguration, sowie die vielen Labels die an jedem zu berücksichtigenden Container ergänzt werden müssen. Caddy bietet mit einer CLI und einer API, die gewünschte Flexibilität und lässt sich auch einfach sichern/wiederherstellen.

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/10fa2b47-ae93-4d57-801c-a5227a3d625e/02_caddy-open-graph.jpg " =1280x640")


\
# compose.yml File

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
      - ./caddy_data:/data
      - ./caddy_config:/config
    networks:
      - caddy

networks:
    caddy:
        external: true
```

# Kommandos

\n#To reload Caddy after making changes to your Caddyfile: #docker exec -w /etc/caddy caddy caddy fmt --overwrite #docker exec -w /etc/caddy caddy caddy reload #docker exec -w /etc/caddy caddy caddy hash-password -p password


## Warum ein Reverse Proxy für Docker Container?

Die Nutzung eines sogenannten Reverse Proxies für die eigene (Docker) Container Landschaft soll zum einen Flexibilität als zum anderen auch erhöhte Sicherheit bieten. Das (virtuelle) Netzwerk der Container wird in verschiedene Zonen aufgeteilt, so dass z.B. Datenbankinstanzen nicht direkt aus dem Internet erreichbar sind. Auch können so z.B. mehrere WordPress Installationen parallel betrieben werden, ohne dass sie sich irgendwie in die „Quere" kommen. Beides wird in folgender Darstellung illustriert:

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/82c747a7-d679-4fe8-9b9e-7fc5c9c13315/01_caddy.png " =1204x755")


Außerdem soll der Zugriff auf die selbst gehosteten Webseiten auschließlich über das verschlüsselte HTTPS Protokoll funktionieren. Die dafür benötigten Zertifikate verwaltet und aktualisiert eine moderne Rerverse Proxy Lösung für uns voll automatisch.

## Wie installiere und betreibe ich Caddy als Reverse Proxy?

Zunächst richten wir und ein dediziertes Netzwerk für Caddy und die Container die öffentlich erreichbar sein sollen unter einem gewünschten Namen (hier „Caddy") mit folgendem Befehl ein: `docker network create caddy` . Diesen Namen siehst nur du in deinen Konfigurationsdateien und in Docker.

Caddy selbst stellen wir ebenfalls über einen Docker Container zur Verfügung, auf dem wir die Ports 80 und 443 exponieren. Eine mögliche Docker Compose Datei „compose.yml" dazu kann so aussehen:

Bevor wir den Container starten, legen wir im gleichen Verzeichnis eine Datei „Caddyfile" an. Diese musst du auf deine Bedürfnisse anpassen und orientiert sich hier am oben gezeigten Beispiel:

```js
{
    # TLS Options
    email deine.email@provider.com
}

(wordpress) {
    header {
        Cache-Control "public, max-age=3600, must-revalidate"
    }
}

quickscot.de, www.quickscot.de {
    import wordpress
    reverse_proxy wordpresscontainer1:80
}

niklas-stephan.de, www.niklas-stephan.de {
    import wordpress
    reverse_proxy wordpresscontainer3:80
}
```

Nun können wir den Container mit folgendem Befehl starten: `docker compose up -d`. Falls irgendetwas nicht funktioniert hilft eine Überprüfung der Logdatei mit: `docker compose logs `.

Bei Problemen hilft auch ein Blick in die Caddy Community, die viele Problembehandlungen und Konfigurationsbeispiele bereitstellt: <https://caddy.community/>.

Falls ihr Änderungen/Ergänzungen an dem Caddyfile vornehmt müsst ihr nicht jedes Mal den kompletten Container durchstarten und könnt so Dowtimes durch folgenden Befehl, der die Konfiguration live neu läd, aktualisieren:

```bash

docker compose exec -w /etc/caddy caddy caddy reload
```


# Quellen