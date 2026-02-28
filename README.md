# Homelab Nginx Proxy

> ⚠️ **This repository has been archived.** NPM has been migrated into the consolidated homelab infrastructure stack. See [homelab-infrastructure](https://github.com/TechAsura/homelab-infrastructure) for the current setup.

---

A self-hosted reverse proxy setup for my Proxmox homelab using Nginx Proxy Manager, enabling clean subdomain-based HTTPS access to all internal services via a real domain.

This was the initial standalone deployment of NPM before it was consolidated into the broader mediastack LXC alongside Sonarr, Radarr, Prowlarr, Jellyseerr, and other services.

## Stack

| Component | Tool |
|-----------|------|
| Hypervisor | Proxmox VE |
| Container | LXC (Docker) |
| Reverse Proxy | Nginx Proxy Manager |
| SSL Certificates | Let's Encrypt wildcard via Cloudflare DNS challenge |
| DNS | Cloudflare |

## How It Worked

- A wildcard `A` record (`*.idiotproductions.net`) in Cloudflare points to the internal IP of the LXC running NPM
- Nginx Proxy Manager routes incoming requests to the correct internal service based on subdomain
- A single wildcard SSL certificate (`*.idiotproductions.net`) covers all subdomains automatically
- All services are internal only — the Cloudflare records point to a private IP, so nothing is publicly accessible

## Why It Was Migrated

Running NPM in its own dedicated LXC worked fine, but as the homelab grew it made more sense to consolidate all infrastructure services into a single mediastack LXC managed by one Docker Compose file. This reduces overhead and makes the whole stack easier to manage and document.

## Current Setup

NPM is now defined in the mediastack `docker-compose.yml` in [homelab-infrastructure](https://github.com/TechAsura/homelab-infrastructure), alongside the rest of the media and infrastructure stack.
