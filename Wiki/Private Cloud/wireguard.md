# 🔒 wireguard

Ziel ist es mittels wireguard einen (Home-)server für bestimmte Applikationen wie z.B. HomeAssistant über ein VPN von einem öffentlichem Server aus erreichbar zu machen.

# Setup

## Private Server

Zur Bequemlichkeit installieren wir die Wireguard-Tools auf beiden servern:

```bash
sudo apt install wireguard-tools
```

Nun erstellen wir ein Verzeichnis mit einem beliebigen Namen für unsere Konfigrationsdateien. Z.B.:

```yaml
mkdir /data/docker/wireguard && cd /data/docker/wireguard
```

und noch Unterverzeichnise für die spezifische Config und die keys

```yaml
mkdir keys && mkdir config && cd keys
```

Im keys Verzeichis folgendes ausführen:

```bash
# Generate private key for private server
wg genkey > private-server-private.key

# Generate public key for private server
wg pubkey < private-server-private.key > private-server-public.key

# Generate private key for public server
wg genkey > public-server-private.key

# Generate public key for public server
wg pubkey < public-server-private.key > public-server-public.key
```

Zurück in Hauptverzeichnis dann die compose.yml anlegen

```yaml
services:
  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wg
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
      - SERVERURL=vpn.handtrixxx.com
      - SERVERPORT=51820
      - PEERS=1
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0/24
    volumes:
      - ./config:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
    network_mode: "host"
```

Im config verzeichnis legen wir die datei wg0.conf

```bash
[Interface]
PrivateKey = <contents of private-server-private.key>
Address = 10.0.0.2/32
ListenPort = 51820
DNS = 1.1.1.1

[Peer]
PublicKey = <contents of public-server-public.key>
Endpoint = public.example.com:51820
AllowedIPs = 10.0.0.1/32
PersistentKeepalive = 25
```

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Make persistent by adding to /etc/sysctl.conf:

net.ipv4.ip_forward=1

```bash
sudo iptables -A FORWARD -i wg0 -o lo -j ACCEPT
sudo iptables -A FORWARD -i lo -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o lo -j MASQUERADE

sudo iptables -A FORWARD -i wg0 -o eno1 -j ACCEPT
sudo iptables -A FORWARD -i eno1 -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o eno1 -j MASQUERADE
```


\
sudo apt install iptables-persistent


sudo netfilter-persistent save


\
## Public Server

Open UDP port 51820 inbound.

Dort ebenfalls ein Verzeichnis und die Untervzereichnisse für die Conatinerumgebung anlegen

```bash
mkdir config
```

compose.yml

```bash
services:
  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Berlin
    volumes:
      - ./config:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```

wg0.conf

```bash
[Interface]
PrivateKey = <contents of public-server-private.key>
Address = 10.0.0.1/32
ListenPort = 51820
DNS = 1.1.1.1

[Peer]
PublicKey = <contents of private-server-public.key>
AllowedIPs = 10.0.0.2/32
```


## Caddy

```bash
home.handtrixxx.com {
    reverse_proxy http://172.19.0.1:8123
}
```