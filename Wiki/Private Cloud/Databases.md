# Databases

# MariaDB

## compose.yml file

```yaml
db:
  container_name: YOURCONTAINERNAME
  image: mariadb:10.11
  user: "1000:1000"
  restart: unless-stopped
  command: --transaction-isolation=READ-COMMITTED --log-bin=binlog --binlog-format=ROW --log_bin_trust_function_creators=true
  volumes:
    - ./db:/var/lib/mysql
  environment:
    MARIADB_ROOT_PASSWORD: YOURROOTPASSWORD
    MARIADB_DATABASE: YOURDB
    MARIADB_USER: YOURUSER
    MARIADB_PASSWORD: YOURPASSWORD
    TZ: Europe/Berlin
    PUID: 1000
    PGID: 1000
  networks:
    - default
```

## .env file

```ini

MARIADB_ROOT_PASSWORD=
MARIADB_DATABASE=
MARIADB_USER=
MARIADB_PASSWORD=
TZ=Europe/Berlin

PUID=1000

PGID=1000
```

## Operations

Dump Database

```bash
docker exec DBCONTAINERNAME sh -c 'exec mysqldump -u"root" -p"ROOTPASSWORD" "DBNAME"' > dump.sql
```

Restore Database

```bash
cat dump.sql | docker exec -i DBCONTAINERNAME sh -c 'exec mysql -u"root" -p"ROOTPASSWORD" "DBNAME"'
```

# PSQL

## Compose File

```yaml

services:
  db:
    container_name: YOURCONTAINERNAME
    image: postgres:15-alpine
    user: "1000:1000"
    environment:
      POSTGRES_DB: YOURDB
      POSTGRES_PASSWORD: YOURPASSWORD
      POSTGRES_USER: YOURUSER
      PUID: 1000
      PGID: 1000
      TZ: Europe/Berlin
    restart: unless-stopped
    volumes:
      - ./db:/var/lib/postgresql/data
    networks:
      default:
```