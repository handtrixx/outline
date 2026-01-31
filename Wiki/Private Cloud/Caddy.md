# Caddy

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
    name: caddy
    driver: bridge
    ipam:
      config:
        - subnet: 172.19.0.0/16
          gateway: 172.19.0.1
```

# Kommandos

\nTo reload Caddy after making changes to your Caddyfile: 


```javascript
docker exec -w /etc/caddy caddy caddy fmt --overwrite
docker exec -w /etc/caddy caddy caddy reload
docker exec -w /etc/caddy caddy caddy hash-password -p password
```


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
docker exec -w /etc/caddy caddy caddy reload
```


# Caddyfile

```json
{
	# TLS Options
	email niklas.stephan@gmail.com
}

(wordpress) {
	header {
		Cache-Control "public, max-age=7200, must-revalidate"
	}
}

(mail-backend) {
	reverse_proxy 172.19.0.1:8085
}

(trusted_proxy_list) {
	trusted_proxies 172.19.0.0/24
}

cloud.niklas-stephan.de {
	reverse_proxy 172.19.0.1:9200
}

docs.niklas-stephan.de {
	reverse_proxy docsniklas-stephande-outline-1:80
}

collabora.niklas-stephan.de {
	reverse_proxy 172.19.0.1:9980
}
wopiserver.niklas-stephan.de {
	reverse_proxy 172.19.0.1:9300
}

docker.handtrixxx.com {
	reverse_proxy arcane:3552
}

node.handtrixxx.com {
	reverse_proxy node:1880
}

ac-malchsee.de, www.ac-malchsee.de {
	import wordpress
	reverse_proxy malchsee-app:80
}

frau-eule.de, www.frau-eule.de {
	import wordpress
	reverse_proxy fraueulede-app:80
}

new.frau-eule.de {
	import wordpress
	reverse_proxy fraueule-app:80
}

handtrixxx.com, www.handtrixxx.com {
	reverse_proxy handtrixxx:80
}

auth.handtrixxx.com {
	reverse_proxy authentik-server:9000
}

home.handtrixxx.com {
	reverse_proxy wg-easy:8123
}

music.handtrixxx.com {
	reverse_proxy wg-easy:8095
}

keys.handtrixxx.com {
	reverse_proxy vaultwarden:80
}

dav.handtrixxx.com {
	reverse_proxy radicale:5232
}

mail.handtrixxx.com,
mail.niklas-stephan.de,
mail.quickscot.de,
mail.quicksilvers.de,
mail.ac-malchsee.de,
mail.frau-eule.de {
	import mail-backend
}

autodiscover.mail.handtrixxx.com,
autoconfig.mail.handtrixxx.com,
autodiscover.handtrixxx.com,
autoconfig.handtrixxx.com {
	import mail-backend
}

webmail.handtrixxx.com {
	reverse_proxy roundcube:80
}

stalwart.handtrixxx.com {
	reverse_proxy stalwart:8080
}

photos.handtrixxx.com {
	reverse_proxy immich-server:2283
}

search.handtrixxx.com {
	reverse_proxy search-searxng:8080
}

start.handtrixxx.com {
	reverse_proxy heimdall:80
}

desktop.handtrixxx.com {
	reverse_proxy webtop:3000
}

vpn.handtrixxx.com {
	reverse_proxy wg-easy:51821
}

niklas-stephan.de, www.niklas-stephan.de {
	reverse_proxy app:3000
}

quickscot.de, www.quickscot.de, quicksilvers.de, www.quicksilvers.de {
	import wordpress
	reverse_proxy quickscot-static:80
}

stats.niklas-stephan.de, stats.ac-malchsee.de, stats.frau-eule.de, stats.handtrixxx.com, stats.quickscot.de {
	reverse_proxy matomo-web:80 {
		header_up X-Real-IP {remote_host}
		import trusted_proxy_list
	}
}

cdfox.bbraun.io {
	handle_path /ext/drawio/* {
		@index_html path /ext/drawio/index.html
		handle @index_html {
			respond @unauthorized 401
		}
		reverse_proxy drawio:80
	}
	reverse_proxy cdfox:3000
}

```