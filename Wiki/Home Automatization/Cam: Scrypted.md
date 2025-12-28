# Cam: Scrypted

```bash
services:
    scrypted:
        environment:
            # Scrypted NVR Storage (Part 2 of 3)

            # Uncomment the next line to configure the NVR plugin to store recordings
            # use the /nvr directory within the container. This can also be configured
            # within the plugin manually.
            # The drive or network share will ALSO need to be configured in the volumes
            # section below.
            - SCRYPTED_NVR_VOLUME=/nvr
            - SCRYPTED_WEBHOOK_UPDATE_AUTHORIZATION=Bearer ${WATCHTOWER_HTTP_API_TOKEN:-env_missing_fallback}
            - SCRYPTED_WEBHOOK_UPDATE=http://localhost:10444/v1/update
            # Avahi can be used for network discovery by passing in the host daemon
            # or running the daemon inside the container. Choose one or the other.
            # Uncomment next line to run avahi-daemon inside the container.
            # See volumes and security_opt section below to use the host daemon.
            - SCRYPTED_DOCKER_AVAHI=true
        # Valid images:
        # ghcr.io/koush/scrypted
        # ghcr.io/koush/scrypted:intel
        # ghcr.io/koush/scrypted:lite
        image: ghcr.io/koush/scrypted:intel
        volumes:
            # The following example would mount the /mnt/media/video path on the host
            # to the /nvr path inside the docker container.
            - /stream:/nvr
            # Uncomment the following lines to use Avahi daemon from the host.
            # Ensure Avahi is running on the host machine:
            # It can be installed with: sudo apt-get install avahi-daemon
            # This is not compatible with running avahi inside the container (see above).
            # Also, uncomment the lines under security_opt
            # - /var/run/dbus:/var/run/dbus
            # - /var/run/avahi-daemon/socket:/var/run/avahi-daemon/socket

            # Default volume for the Scrypted database. Typically should not be changed.
            # The volume will be placed relative to this docker-compose.yml.
            - ./volume:/server/volume
        # Uncomment the following lines to use Avahi daemon from the host
        # Without this, AppArmor will block the container's attempt to talk to Avahi via dbus
        # security_opt:
        #     - apparmor:unconfined
        devices: [
            # hardware accelerated video decoding, opencl, etc.
            "/dev/dri:/dev/dri",
        ]

        container_name: scrypted
        restart: unless-stopped
        network_mode: host
        # logging is noisy and will unnecessarily wear on flash storage.
        # scrypted has per device in memory logging that is preferred.
        # enable the log file if enhanced debugging is necessary.
        #logging:
            #driver: "none"
            # driver: "json-file"
            # options:
            #     max-size: "10m"
            #     max-file: "10"
        labels:
            - "com.centurylinklabs.watchtower.scope=scrypted"

        # Use global DNS servers to avoid issues with some local DNS servers.
        # particularly with npm registry, etc.
        dns:
            - ${SCRYPTED_DNS_SERVER_0:-1.1.1.1}
            - ${SCRYPTED_DNS_SERVER_1:-8.8.8.8}
```