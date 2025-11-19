# my-homelab

Self-hosted homelab stack using Docker and Docker Compose. Includes container lifecycle management, reverse proxying + SSL, dashboard, private cloud storage, monitoring, and automated updates.

## Stack Overview

| Service | Purpose |
| --- | --- |
| Portainer | Visual Docker management UI |
| Nginx Proxy Manager | Reverse proxy with Let's Encrypt automation |
| Homer | Central homelab dashboard |
| Nextcloud | Self-hosted file, calendar, and sync suite |
| Uptime Kuma | Monitoring and status page |
| Watchtower | Automatic container updates |
| Docker Socket Proxy | Restricts socket access for Portainer/Watchtower |

All persistent volumes live under `./data/` for easy backup/migration.

## Prerequisites

- Linux host with Docker Engine 24+ and Docker Compose plugin
- Exposed ports 8088/8443 (Nginx Proxy Manager), 81 (NPM admin), 8080 (Homer), 8081 (Nextcloud), 9443 (Portainer), 3001 (Uptime Kuma)
- Sufficient disk space for Nextcloud data

## First-Time Setup

```bash
# Clone your repo and enter it
cd /home/mdimamhosen/Code/server
# (or git clone <repo> my-homelab && cd my-homelab)

# Ensure folder structure exists
mkdir -p docker/portainer docker/nextcloud docker/uptime-kuma \
         docker/nginx-proxy-manager docker/homer config data

# Bring the stack online
docker compose up -d
```

Docker will pull images, create containers, and initialize data directories automatically on first run. To view progress: `docker compose logs -f`.

## Daily Use

- Dashboard: http://localhost:8080
- Portainer: https://localhost:9443 (accept self-signed cert, create admin user)
- Nginx Proxy Manager: http://localhost:81 (default `admin@example.com / changeme` -> change immediately)
- Nextcloud: http://localhost:8081 (create admin and storage path prompt uses `/var/www/html/data`)
- Uptime Kuma: http://localhost:3001 (create admin)

Once DNS for your domains points to this server, proxy them through Nginx Proxy Manager (HTTP on port 8088, HTTPS on 8443) for TLS.

## Managing the Stack

```bash
# Start containers
docker compose up -d

# Stop containers
docker compose down

# Inspect logs for a service
docker compose logs -f nextcloud

# Update every container to the newest image
docker compose pull
docker compose up -d

# Remove unused images left after updates
docker image prune
```

Watchtower also runs nightly at 03:00 UTC to pull newer images automatically; review its logs occasionally (`docker compose logs watchtower`).

## Backups & Maintenance

- Back up the entire `data/` directory routinely. This contains Portainer state, reverse-proxy config/SSL certs, Nextcloud files, Uptime Kuma data, and Homer configuration.
- Keep Docker Engine updated on the host.
- Monitor disk usage, especially `data/nextcloud`.
- When adding new services, follow the same pattern: mount persistent volumes under `./data/<service-name>`.
- Use Portainer for visual health checks/upgrades; use Uptime Kuma to alert you if services fail.

## Extending

- Add a database (MariaDB/Postgres) for Nextcloud if you anticipate heavy use (update environment + volumes accordingly).
- Add backup containers (e.g., Borg, Restic) targeting `./data/`.
- Use Homer shortcuts to publish documentation, SSH links, or metrics dashboards.

## Troubleshooting

- `docker compose ps` to confirm containers are healthy.
- `docker compose logs <service>` for detailed errors.
- If ports clash, adjust the host side of the mappings in `docker-compose.yml`.
- Ensure DNS A records point to your public IP before requesting certificates in Nginx Proxy Manager.

Happy self-hosting!
# homelab
