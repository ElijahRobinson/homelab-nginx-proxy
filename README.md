# Homelab Nginx Proxy

A self-hosted reverse proxy setup for my Proxmox homelab using Nginx Proxy Manager, enabling clean subdomain-based HTTPS access to all internal services via a real domain.

## Stack

| Component | Tool |
|---|---|
| Hypervisor | Proxmox VE 9.1.1 |
| Host | Intel Core i5-6500 @ 3.20GHz, 15.5GB RAM, 93GB Storage |
| Container | LXC (Docker) |
| Reverse Proxy | Nginx Proxy Manager |
| SSL Certificates | Let's Encrypt wildcard via Cloudflare DNS challenge |
| DNS | Cloudflare |

## How It Works

- A wildcard `A` record (`*.idiotproductions.net`) in Cloudflare points to the internal IP of the LXC running NPM (`10.0.0.x`)
- Nginx Proxy Manager routes incoming requests to the correct internal service based on subdomain
- A single wildcard SSL certificate (`*.idiotproductions.net`) covers all subdomains automatically
- All services are **internal only** — the Cloudflare records point to a private IP, so nothing is publicly accessible

## Services Proxied

| Subdomain | Service | Port |
|---|---|---|
| `jellyfin.idiotproductions.net` | Jellyfin | 8096 |
| `jellyseerr.idiotproductions.net` | Jellyseerr | 5055 |
| `sonarr.idiotproductions.net` | Sonarr | 8989 |
| `radarr.idiotproductions.net` | Radarr | 7878 |
| `prowlarr.idiotproductions.net` | Prowlarr | 9696 |

## Setup

### Prerequisites

- Proxmox VE
- LXC container with Docker and Docker Compose installed
- Domain registered and DNS managed by Cloudflare
- Cloudflare API token with `Zone:DNS:Edit` permissions

### Deploy Nginx Proxy Manager

```bash
mkdir -p /opt/nginx-proxy-manager && cd /opt/nginx-proxy-manager
nano docker-compose.yml
```

Paste the following into `docker-compose.yml`:

```yaml
services:
  nginx-proxy-manager:
    image: jc21/nginx-proxy-manager:latest
    container_name: npm
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "81:81"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

```bash
docker compose up -d
```

Access the admin UI at `http://<your-lxc-ip>:81`

Default credentials: `admin@example.com` / `changeme` — change these immediately on first login.

### Wildcard SSL Certificate

1. In NPM go to **Certificates → Add Certificate → Let's Encrypt via DNS**
2. Add both `*.idiotproductions.net` and `idiotproductions.net` as domain names
3. Select **Cloudflare** as the DNS provider
4. Paste your Cloudflare API token in the credentials file:

```
# Cloudflare API token
dns_cloudflare_api_token=YOUR_TOKEN_HERE
```

5. Save — the cert will be issued in ~30 seconds and auto-renews every 90 days

### Cloudflare DNS

Add a single wildcard `A` record in Cloudflare:

| Type | Name | Value | Proxy |
|---|---|---|---|
| A | `*` | `10.0.0.x` (your NPM LXC IP) | DNS only (grey cloud) |

### Adding a New Proxy Host

For each service in NPM (**Hosts → Proxy Hosts → Add Proxy Host**):

- **Domain:** `service.idiotproductions.net`
- **Scheme:** `http`
- **Forward Hostname/IP:** internal IP of the service
- **Forward Port:** service port
- **Websockets:** enabled
- **SSL tab:** select wildcard cert, enable Force SSL and HTTP/2

## Security Notes

- All subdomains resolve to a **private IP** — services are not publicly accessible
- Remote access is handled via **Tailscale** (no open ports on the router)
- Cloudflare API token is scoped to DNS edit on a single zone only
- Never commit your `.env` file or API tokens to this repo
