# MediaStack

A self-hosted media stack running on Docker Compose. Covers the full pipeline from content discovery and requesting, through downloading, to serving — with indexer management and quality profiles on top.

## Stack

| Service | Role | Port |
|---|---|---|
| [Plex](https://hub.docker.com/r/linuxserver/plex) | Media server | 32400 |
| [Jellyfin](https://jellyfin.org/docs/general/administration/installing#docker) | Media server (alternative) | 8096 |
| [Seerr](https://github.com/seerr-team/seerr) | Media request frontend | 5055 |
| [Sonarr](https://docs.linuxserver.io/images/docker-sonarr) | TV library manager | 8989 |
| [Radarr](https://docs.linuxserver.io/images/docker-radarr) | Movie library manager | 7878 |
| [Prowlarr](https://docs.linuxserver.io/images/docker-prowlarr) | Indexer manager | 9696 |
| [qBittorrent](https://docs.linuxserver.io/images/docker-qbittorrent) | Download client | 8080 |
| [Profilarr](https://github.com/santiagosayshey/Profilarr) | Quality profile manager | 6868 |
| [Pulsarr](https://jamcalli.github.io/Pulsarr/) | Plex watchlist sync | 3003 |
| [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr) | Cloudflare bypass proxy | 8191 |
| [Tailscale](https://tailscale.com/) | VPN / remote access | — |

## Requirements

- Docker + Docker Compose
- A host machine on your LAN (the `.env` assumes a static local IP)
- A [Tailscale](https://tailscale.com/) account and auth key (if you want remote access)
- A Plex claim token if using Plex (optional)

## Setup

**1. Clone the repo**

```bash
git clone https://github.com/youruser/mediastack.git
cd mediastack
```

**2. Create the `.env` file**

Copy the example and fill in your values:

```bash
cp env.txt .env
```

The minimum you need to set:

```env
TIMEZONE=Region/City
LOCAL_DOCKER_IP=10.0.0.xxx        # your host's LAN IP
TAILSCALE_AUTHKEY=tskey-...       # from tailscale.com/admin/settings/keys
```

Everything else defaults to the values already in `env.txt` and should work out of the box for most home setups.

**3. Create the data directories**

```bash
mkdir -p mediastack/{movies,tv,downloads,appdata}
```

**4. Start the stack**

```bash
docker compose up -d
```

## Directory Layout

```
.
├── docker-compose.yaml
├── .env
└── mediastack/
    ├── movies/
    ├── tv/
    ├── downloads/
    └── appdata/
        ├── jellyfin/
        ├── plex/
        ├── sonarr/
        ├── radarr/
        ├── prowlarr/
        ├── qbittorrent/
        ├── profilarr/
        ├── pulsarr/
        ├── seerr/
        └── tailscale/
```

## Bookmarks

Import `bookmarks.html` into your browser for quick access to all service UIs. Update the IP addresses in the file to match your `LOCAL_DOCKER_IP` before importing.

## Network

All containers run on a dedicated Docker bridge network (`mediastack`, default subnet `172.28.10.0/24`). Ports are bound to the host so you can reach each service at `http://<host-ip>:<port>`.

Tailscale is configured as an exit node and advertises both your LAN subnet and the Docker subnet, so you can reach everything remotely without opening firewall ports.

## Notes

- Sonarr and Radarr both depend on qBittorrent and Prowlarr being up before they start.
- Plex and Jellyfin share the same `movies` and `tv` volume mounts — you don't need to store media twice.
- `PLEX_CLAIM` can be left blank if you're not using Plex or have already claimed the server.
