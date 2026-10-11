# Monitoring Stack

This stack runs Grafana, Prometheus, and Node Exporter using Docker Compose.

## Components

- **Grafana**: dashboards at `http://127.0.0.1:3002` on the host by default.
- **Prometheus**: scrapes metrics every 15 seconds; host port 9090 is bound to localhost only.
- **Node Exporter**: collects Linux host metrics for CPU, memory, filesystem, and network.
- **Provisioning**: Prometheus datasource and the Node Overview dashboard are provisioned from files.

## Local setup

From the repository root:

```bash
cd infrastructure/monitoring
cp .env.example .env
```

Edit `.env` and set a unique, strong password. Never commit `.env`.

Validate and start:

```bash
docker compose config -q
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=100
```

Open `http://127.0.0.1:3002`. Log in with the username and password in `.env`.

## Useful checks

```bash
curl -fsS http://127.0.0.1:9090/-/ready
curl -fsS http://127.0.0.1:3002/api/health
docker compose exec prometheus promtool check config /etc/prometheus/prometheus.yml
```

In Prometheus, check **Status → Targets** and confirm `prometheus` and `node` are `UP`.

## Stop and remove

```bash
docker compose down
```

This keeps named data volumes. To remove the stored metrics and Grafana data as well, use `docker compose down -v` (destructive).

## Remote deployment and Cloudflare Tunnel

The default Grafana binding is `127.0.0.1:3002`, suitable for a host-level `cloudflared` service targeting `http://127.0.0.1:3002`. Do not expose Grafana or Prometheus directly to the public internet. Configure Cloudflare Access before sharing the hostname.

When deploying on a remote server, copy the project files to the server and create the server's `.env` there. Do not copy secrets into Git. Validate firewall, tunnel routing, and authentication separately before enabling external access.

## Notes

- Metrics and Grafana state are stored in named Docker volumes.
- This stack is intended for a single Linux host.
- Image versions are pinned in `docker-compose.yml`; review and update versions deliberately.
