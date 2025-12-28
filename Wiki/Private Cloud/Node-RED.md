# 🤖 Node-RED

No-, Low-, oder All-Code Prozessautomatisierung.

[https://www.youtube.com/watch?v=ksGeUD26Mw0&t=4s](https://www.youtube.com/watch?v=ksGeUD26Mw0&t=4s)


# Installation


## Volumes

```bash
mkdir data
```

## compose.yml

```bash
name: nodered
services:
  node-red:
    container_name: node
    hostname: node
    image: nodered/node-red:latest
    user: "1000:1000"
    restart: unless-stopped
    environment:
      - TZ=Europe/Berlin
    networks:
      - node-red-net
      - caddy
    volumes:
      - ./data:/data

volumes:
  node-red-data:

networks:
  node-red-net:
  caddy:
    external: true
```

## Variablen


## Reverse Proxy

Für Caddy:

```javascript
node.handtrixxx.com {
	reverse_proxy node:1880
}
```


## Password

Im Standard läuft Node-RED ungeschützt, was natürlich nicht geht, wenn wir es in unserer Private Cloud betreiben wollen. Das Passwort muss als Hashwert in die `settings.js` Datei im "data" Volume eingetragen werden.

Dieses generieren wir vom Host aus mit dem Befehl:

```bash
docker exec -i node node-red admin hash-pw
```

Dann werden wir aufgefordert das Passwort einzugeben und erhalten dafür den Hashwert.

Nun kommentieren wir den entsprechenden Eintrag in der `settings.js` aus und setzten ihn wie folgt:

```javascript
    adminAuth: {
        type: "credentials",
        users: [{
            username: "WUNSCHNAME",
            password: "HASHWERT",
            permissions: "*"
        }]
    },
```

Nach dem speichern starten wir den Container noch neu, damit diese Einstellungen wirksam sind.

# Alternativen

* \

# Quellen / Links

* <https://nodered.org>