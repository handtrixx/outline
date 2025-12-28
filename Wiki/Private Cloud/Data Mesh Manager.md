# Data Mesh Manager

# Docker Compose file

```yaml

services:
  app:
    image: datameshmanager/datamesh-manager-ce:latest
    restart: unless-stopped
    environment:
      - APPLICATION_HOST_WEB=http://localhost:8081
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/postgres
      - SPRING_DATASOURCE_USERNAME=YOURUSER
      - SPRING_DATASOURCE_PASSWORD=YOURPASSWORD
      - SPRING_MAIL_HOST=mailhog
      - SPRING_MAIL_PORT=1025
    #ports:
    #  - '8081:8080'
    networks:
      - default
      - caddy
    depends_on:
      - postgres
      - mailhog
  postgres:
    image: pgvector/pgvector:pg16
    restart: unless-stopped
    environment:
      - POSTGRES_USER=YOURUSER
      - POSTGRES_PASSWORD=YOURPASSWORD
    volumes:
      - ./postgres:/var/lib/postgresql/data
    networks:
      - default
  mailhog:
    # Test SMTP server, does not actually send mails
    # View messages in the web UI http://localhost:8025
    image: mailhog/mailhog:latest
    restart: unless-stopped
    #ports:
    #  - "8025:8025"
    networks:
      - default
      - caddy

networks:
  caddy:
    external: true
```