# ARR Stack with Docker

This project implements the **ARR Stack** using Docker. The ARR Stack is a set of self-hosted applications designed to automate the process of searching, indexing, downloading, organizing, and streaming multimedia content such as movies, TV series, books, and music.

## 🧩 Stack Components

- **Gluetun**  
  Provides a secure VPN tunnel to safely download torrents.
  
- **Prowlarr**  
  Acts as an indexer manager, supplying indexers to applications like Sonarr and Radarr. It routes search and download requests to torrent providers.
  
- **qBittorrent**  
  Torrent download client.

- **Radarr / Sonarr / Readarr / Lidarr**  
  Applications responsible for monitoring, organizing, and maintaining media files:
  - **Radarr** – Movies
  - **Sonarr** – TV series
  - **Readarr** – Books
  - **Lidarr** – Music

- **Jellyfin**  
  Media server that allows you to stream and consume the downloaded content from any device.

## 🖼️ Architecture Diagram

> _Insert the architecture diagram here: `Arr Stack.svg`_

## 🚀 Implementation Details

All services are containerized and deployed using Docker. They communicate internally over a shared network, with **Gluetun** acting as a VPN gateway to ensure secure traffic for torrent downloads.

## 📦 Prerequisites

- [Docker](https://docs.docker.com/get-docker/) installed on your system.
- Create a `.env` file in the root directory of the project with the following content:

```dotenv
# Mount point (path where services will store data)
STACK_PATH=<path-to-your-storage-directory>

# Timezone
TZ=Etc/UTC

# User/Group IDs for file permissions
PUID=1000
PGID=1000

# VPN Settings (adjust according to your VPN provider)
VPN_TYPE=wireguard
VPN_PROVIDER=<your-vpn-provider>
WIREGUARD_PRIVATE_KEY=<your-wireguard-private-key>
SERVER_COUNTRIES=
```
💡 For VPN configuration details (e.g., ProtonVPN), refer to the Gluetun wiki.


If you're using Docker on Windows, note that volumes are mounted under:
`~/mnt/host/<drive-letter>`

## ⚙️ Docker Compose Setup
All services are defined and parameterized in the docker-compose.yml file using variables from the .env file. 
You migth need to modify the gluetun enviroment in the docker-compose.yml file depending on you vpn provider, all the vpn providers supported with its docker-compose snippets can bi found at [gluetun's GitHub Repo](https://github.com/qdm12/gluetun-wiki/blob/main/setup/providers/protonvpn.md)

```yml
gluetun:
  image: qmcgaw/gluetun
  container_name: gluetun
  cap_add:
    - NET_ADMIN
  devices:
    - /dev/net/tun:/dev/net/tun
  ports:
    - 8085:8085   # qBittorrent
    - 8989:8989   # Sonarr
    - 7878:7878   # Radarr
    - 9696:9696   # Prowlarr
  environment:
    <<: *common-variables
    VPN_SERVICE_PROVIDER: ${VPN_PROVIDER}
    VPN_TYPE: ${VPN_TYPE}
    WIREGUARD_PRIVATE_KEY: ${WIREGUARD_PRIVATE_KEY}
    SERVER_COUNTRIES: ${SERVER_COUNTRIES}
    VPN_PORT_FORWARDING: "on"
  healthcheck:
    test: ping -c 1 www.google.com || exit 1
    interval: 120s
    timeout: 20s
    retries: 5
    start_period: 2m
```

## ▶️ Running the Stack
To start the services:
```bash
docker compose up -d
```

To verify that the containers are running:
```bash
docker container ls
```

Once everything is up, you can check the ip adress where the service are running and access them from the browser using their respective ports:

- xx.xx.xx.xx:8085 -> qBittorrent
- xx.xx.xx.xx:8989 -> Sonarr
- xx.xx.xx.xx:7878 -> Radarr
- xx.xx.xx.xx:9696 -> Prowlarr

To stop the services:
```bash
docker compose down
```
## ⚙️ Configure the Stack
You will need to setup a few things in the UI of each of the services to have them all up and running. I am not going to cover that here, but you can find a bunch of guides to do so googling for "ARR Stack"
