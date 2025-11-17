# Immich

![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/51f857ad-8901-4959-8dc8-2a4aa4e30b0c/immich.webp " =2420x1560")

Immich ist Google Fotos auf deinem eigenem Server.

# Installation

## Volumes

```bash
mkdir db
mkdir library
mkdir model-cache
```

## compose.yml

```yaml
name: immich
services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    devices:
      - /dev/dri:/dev/dri
    volumes:
      # Do not edit the next line. If you want to change the media storage location on your system, edit the value of UPLOAD_LOCATION in the .env file
      - ${UPLOAD_LOCATION}:/data
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    #ports:
    #  - '2283:2283'
    depends_on:
      - redis
      - database
    restart: unless-stopped
    healthcheck:
      disable: false
    networks:
      - caddy
      - default

  immich-machine-learning:
    container_name: immich_machine_learning
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
    group_add:
      - video
    devices:
      - /dev/dri:/dev/dri
      - /dev/kfd:/dev/kfd
    volumes:
      - ./model-cache:/cache
    env_file:
      - .env
    restart: always
    healthcheck:
      disable: false
    networks:
      - default

  redis:
    container_name: immich_redis
    image: docker.io/valkey/valkey:8@sha256:81db6d39e1bba3b3ff32bd3a1b19a6d69690f94a3954ec131277b9a26b95b3aa
    healthcheck:
      test: redis-cli ping || exit 1
    restart: always
    networks:
      - default

  database:
    container_name: immich_postgres
    image: ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0@sha256:bcf63357191b76a916ae5eb93464d65c07511da41e3bf7a8416db519b40b1c23
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
    user: "1000:1000"
    volumes:
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    shm_size: 128mb
    restart: always
    networks:
      - default
networks:
    caddy:
        external: true
```


## .env

```yaml
# You can find documentation for all the supported env variables at https://docs.immich.app/install/environment-variables

# The location where your uploaded files are stored
UPLOAD_LOCATION=./library

# The location where your database files are stored. Network shares are not supported for the database
DB_DATA_LOCATION=./db

# To set a timezone, uncomment the next line and change Etc/UTC to a TZ identifier from this list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List
TZ=Europe/Berlin

# The Immich version to use. You can pin this to a specific version like "v2.1.0"
IMMICH_VERSION=v2

# Connection secret for postgres. You should change it to a random password
# Please use only the characters `A-Za-z0-9`, without special characters or spaces
DB_PASSWORD=YOURPW

# The values below this line do not need to be changed
###################################################################################
DB_USERNAME=YOURDBUSER
DB_DATABASE_NAME=YOURDBNAME

PUID=1000
PGID=1000
```

## Reverse Proxy

Für Caddy:

```go
photos.handtrixxx.com {
	reverse_proxy immich-server:2283
}
```


# Client Setup


Siehe [Immich Client](/doc/immich-T9TJXI9IKn)

# Alternativen

* Google Fotos
* Open Cloud
* Nextcloud

# Quellen / Links

* <https://docs.immich.app/overview/quick-start>
* <https://github.com/immich-app/immich>
* <https://immich.app>
* <https://get.immich.app/android>
* <https://get.immich.app/ios>