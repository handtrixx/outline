# Open Cloud

# Installation

## Docker Compose File

Es gbit auch eine offizielle Anleitung hier: <https://github.com/opencloud-eu/opencloud-compose> . Die gefällt mir aber aus verschiedenen Gründen nicht, so dass ich alles auf diese compose und die darauf folgend .env Dateien summiert habe:

```yaml

services:
  opencloud:
    container_name: ocserver
    image: opencloudeu/opencloud-rolling:latest
    networks:
      opencloud-net:
    entrypoint:
      - /bin/sh
    command: ["-c", "opencloud init || true; opencloud server"]
    ports:
      - "9200:9200"
    environment:
      OC_ADD_RUN_SERVICES: notifications
      OC_URL: https://${OC_DOMAIN:-cloud.opencloud.test}
      OC_LOG_LEVEL: ${LOG_LEVEL:-info}
      OC_LOG_COLOR: "${LOG_PRETTY:-false}"
      OC_LOG_PRETTY: "${LOG_PRETTY:-false}"
      PROXY_TLS: "false"
      OC_INSECURE: "${INSECURE:-false}"
      PROXY_ENABLE_BASIC_AUTH: "${PROXY_ENABLE_BASIC_AUTH:-false}"
      IDM_ADMIN_PASSWORD: "${ADMIN_PASSWORD:-admin}"
      IDM_CREATE_DEMO_USERS: "${DEMO_USERS:-false}"
      NOTIFICATIONS_SMTP_HOST: "${SMTP_HOST}"
      NOTIFICATIONS_SMTP_PORT: "${SMTP_PORT}"
      NOTIFICATIONS_SMTP_SENDER: "${SMTP_SENDER:-OpenCloud notifications <notifications@${OC_DOMAIN:-cloud.opencloud.test}>}"
      NOTIFICATIONS_SMTP_USERNAME: "${SMTP_USERNAME}"
      NOTIFICATIONS_SMTP_PASSWORD: "${SMTP_PASSWORD}"
      NOTIFICATIONS_SMTP_INSECURE: ""
      NOTIFICATIONS_SMTP_AUTHENTICATION: ""
      NOTIFICATIONS_SMTP_ENCRYPTION: "${SMTP_TRANSPORT_ENCRYPTION:-none}"
      FRONTEND_ARCHIVER_MAX_SIZE: "10000000000"
      PROXY_CSP_CONFIG_FILE_LOCATION: /etc/opencloud/csp.yaml
      OC_PASSWORD_POLICY_BANNED_PASSWORDS_LIST: banned-password-list.txt
      #PROXY
      PROXY_HTTP_ADDR: "0.0.0.0:9200"
      #TIKA
      SEARCH_EXTRACTOR_TYPE: tika
      SEARCH_EXTRACTOR_TIKA_TIKA_URL: http://tika:9998
      FRONTEND_FULL_TEXT_SEARCH_ENABLED: "true"
      #COLLABORA
      COLLABORA_DOMAIN: ${COLLABORA_DOMAIN:-collabora.opencloud.test}
      NATS_NATS_HOST: 0.0.0.0
      GATEWAY_GRPC_ADDR: 0.0.0.0:9142
      FRONTEND_APP_HANDLER_SECURE_VIEW_APP_ADDR: eu.opencloud.api.collaboration.CollaboraOnline
      GRAPH_AVAILABLE_ROLES: "b1e2218d-eef8-4d4c-b82d-0f1a1b48f3b5,a8d5fe5e-96e3-418d-825b-534dbdf22b99,fb6c3e19-e378-47e5-b277-9732f9de6e21,58c63c02-1d89-4572-916a-870abc5a1b7d,2d00ce52-1fc2-4dbc-8b95-a73b73395f5a,1c996275-f1c9-4e71-abdf-a42f6495e960,312c0871-5ef7-4b3a-85b6-0e4074c64049,aa97fe03-7980-45ac-9e50-b325749fd7e6"
    volumes:
      - ./config/opencloud/csp.yaml:/etc/opencloud/csp.yaml
      - ./config/opencloud/banned-password-list.txt:/etc/opencloud/banned-password-list.txt
      - ${OC_CONFIG_DIR:-opencloud-config}:/etc/opencloud
      - ${OC_DATA_DIR:-opencloud-data}:/var/lib/opencloud
      - ${OC_APPS_DIR:-./config/opencloud/apps}:/var/lib/opencloud/web/assets/apps
    restart: always
  radicale:
    container_name: ocradicale
    image: opencloudeu/radicale:latest
    networks:
      opencloud-net:
    restart: always
    volumes:
      - ./config/radicale/config:/etc/radicale/config
      - ${RADICALE_DATA_DIR:-radicale-data}:/var/lib/radicale
  tika:
    container_name: octika
    image: apache/tika:latest-full
    networks:
      opencloud-net:
    restart: always
  collaboration:
    container_name: occollaboration
    image: opencloudeu/opencloud-rolling:latest
    networks:
      opencloud-net:
    depends_on:
      opencloud:
        condition: service_started
      collabora:
        condition: service_healthy
    entrypoint:
      - /bin/sh
    command: [ "-c", "opencloud collaboration server" ]
    ports:
      - "9300:9300"
    environment:
      COLLABORATION_GRPC_ADDR: 0.0.0.0:9301
      COLLABORATION_HTTP_ADDR: 0.0.0.0:9300
      MICRO_REGISTRY: "nats-js-kv"
      MICRO_REGISTRY_ADDRESS: "opencloud:9233"
      COLLABORATION_WOPI_SRC: https://${WOPISERVER_DOMAIN:-wopiserver.opencloud.test}
      COLLABORATION_APP_NAME: "CollaboraOnline"
      COLLABORATION_APP_PRODUCT: "Collabora"
      COLLABORATION_APP_ADDR: https://${COLLABORA_DOMAIN:-collabora.opencloud.test}
      COLLABORATION_APP_ICON: https://${COLLABORA_DOMAIN:-collabora.opencloud.test}/favicon.ico
      COLLABORATION_APP_INSECURE: "${INSECURE:-true}"
      COLLABORATION_CS3API_DATAGATEWAY_INSECURE: "${INSECURE:-true}"
      COLLABORATION_LOG_LEVEL: ${LOG_LEVEL:-info}
      OC_URL: https://${OC_DOMAIN:-cloud.opencloud.test}
    volumes:
      - ${OC_CONFIG_DIR:-opencloud-config}:/etc/opencloud
    restart: always
  collabora:
    container_name: occollabora
    image: collabora/code:25.04.4.2.1
    networks:
      opencloud-net:
    ports:
      - "9980:9980"
    environment:
      aliasgroup1: https://${WOPISERVER_DOMAIN:-wopiserver.opencloud.test}:443
      DONT_GEN_SSL_CERT: "YES"
      extra_params: |
        --o:ssl.enable=${COLLABORA_SSL_ENABLE:-true} \
        --o:ssl.ssl_verification=${COLLABORA_SSL_VERIFICATION:-true} \
        --o:ssl.termination=true \
        --o:welcome.enable=false \
        --o:net.frame_ancestors=${OC_DOMAIN:-cloud.opencloud.test}
      username: ${COLLABORA_ADMIN_USER:-admin}
      password: ${COLLABORA_ADMIN_PASSWORD:-admin}
    cap_add:
      - MKNOD
    restart: always
    entrypoint: ['/bin/bash', '-c']
    command: ['coolconfig generate-proof-key && /start-collabora-online.sh']
    healthcheck:
      test: ["CMD", "bash", "-c", "exec 3<>/dev/tcp/127.0.0.1/9980 && echo -e 'GET /hosting/discovery HTTP/1.1\r\nHost: localhost:9980\r\n\r\n' >&3 && head -n 1 <&3 | grep '200 OK'"]
networks:
  opencloud-net:
```

## Env File

```
INSECURE=true

OC_DOMAIN=opencloud.domain.de

ADMIN_PASSWORD=DEINPASSWORT

OC_CONFIG_DIR=/pfad/auf/host/config

OC_DATA_DIR=/pfad/auf/host/opencloud

SMTP_HOST=mail.domain.com

SMTP_PORT=465

SMTP_SENDER=opencloud@domain.de

SMTP_USERNAME=cloud@domain.de

SMTP_PASSWORD=DEINPASSWORT

SMTP_TRANSPORT_ENCRYPTION=ssltls

COLLABORA_DOMAIN=collabora.domain.de

WOPISERVER_DOMAIN=wopiserver.domain.de

COLLABORA_ADMIN_USER=DEINUSER

COLLABORA_ADMIN_PASSWORD=DEINPASSWORT

COLLABORA_SSL_ENABLE=false

COLLABORA_SSL_VERIFICATION=false

RADICALE_DATA_DIR=/pfad/auf/server/radicale
```

# Caddy Routen

Angenommen der Caddy Container liegt im Docker Netzwerk 172.19.0.x:

```Go

cloud.domain.de {
	reverse_proxy 172.19.0.1:9200
}
collabora.domain.de {
	reverse_proxy 172.19.0.1:9980
}
wopiserver.domain.de {
	reverse_proxy 172.19.0.1:9300
}
```

# Konfiguration und Wartung

Dateiberechtigungen für den Storage: `chmod 644`

# Alternativen

nextcloud

owncloud