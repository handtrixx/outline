# Artalk

[Artalk](https://artalk.js.org), eine in GO geschriebene Anwendung, dient dazu eine Kommentarfunktion in WebApps einzubinden und zu verwalten. Das kann entweder durch eine native Integration wie bei [Wiki.js](/Apps-Server/wikijs) oder manuell ober die Artalk API erfolgen. Artalk ist Open-Source und lässt sich, wie im Beispiel hier, in einem Docker Container auf einem Server betreiben.

[artalk.webm 960x720](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/cfead37d-b423-4f83-85ce-d4d4ebcb657f/artalk.webm)


# Compose File

Das zugehörige Docker Compose File `compose.yml` kann so aussehen:

```yaml

services:
  artalk:
    container_name: artalk
    image: artalk/artalk-go
    restart: unless-stopped
    volumes:
      - ./data:/data
    networks:
      default:
      caddy:
    environment:
      - TZ=${TZ}
      - ATK_LOCALE=${TZ}
      - ATK_SITE_DEFAULT=${TZ}
      - ATK_SITE_URL${TZ}
networks:
  caddy:
    external: true
```

# .env File

```ini

TZ=Euorope/Berlin

ATK_LOCALE=en

ATK_SITE_DEFAULT=wiki

ATK_SITE_URL=https://wiki.niklas-stephan.de
```

# Konfiguration

Nach dem hochfahren des Service muss zunächst ein neuer Admin user angelegt werden. Folgender Befehl erledigt das:

```bash

docker exec -it artalk artalk admin
```

# Alternativen zur Artalk

* <https://commento.io>
* <https://disqus.com>