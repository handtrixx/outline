# Private Cloud

# Idee

Server oder virtueller Server.

# Anwendungsfälle

## Coding IDEs

* VS Code Server

## Der virtuelle Desktop

* Webtop

## Foto- und Videoverwaltung

* [Immich](./Private%20Cloud/Immich.md)
* [Open Cloud](./Private%20Cloud/Open%20Cloud.md)

## Online Office

* Collabora in Open Cloud
* Outline

# Installation

Auf dem Server installieren wir nichts außer:


1. ZFS
2. Docker
3. Firewall


:::warning
Sei aber sicher, dass eine Firewall läuft.

:::

Der Rest läuft dann in Docker Containern, welche als Services über Docker Compose komponiert werden.

# Applikationen

Siehe untergeordnete Dokumente.