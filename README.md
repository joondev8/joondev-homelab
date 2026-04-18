# Joondev Homelab

This repository contains a Docker Compose-based homelab operations stack. It is centered on monitoring, logging, dashboards, and a few admin utilities rather than application code.

## What It Runs

- `nginx` as the front door on port `80`
- `grafana` for dashboards
- `prometheus` for metrics collection
- `loki` for log storage
- `promtail` for log shipping
- `node-exporter` for host metrics
- `cadvisor` for container metrics
- `postgres` plus `postgres-exporter`
- `portainer`, `uptime-kuma`, and `dozzle` for container and service operations
- `elasticsearch` as a standalone search/index service

## Repository Layout

```text
.
├── AGENTS.md
├── README.md
└── docker
    ├── .env.example
    ├── docker-compose.yml
    ├── grafana/
    ├── loki/
    ├── nginx/
    ├── prometheus/
    └── promtail/
```

Important paths:

- [`docker/docker-compose.yml`](/mnt/c/Work-Dev/joondev-homelab/docker/docker-compose.yml:1): main stack definition
- [`docker/nginx/nginx.conf`](/mnt/c/Work-Dev/joondev-homelab/docker/nginx/nginx.conf:1): reverse-proxy rules
- [`docker/prometheus/prometheus.yml`](/mnt/c/Work-Dev/joondev-homelab/docker/prometheus/prometheus.yml:1): Prometheus scrape targets
- [`docker/promtail/promtail-config.yaml`](/mnt/c/Work-Dev/joondev-homelab/docker/promtail/promtail-config.yaml:1): log collection config
- [`docker/loki/loki-config.yaml`](/mnt/c/Work-Dev/joondev-homelab/docker/loki/loki-config.yaml:1): Loki storage config
- [`docker/grafana/provisioning/`](/mnt/c/Work-Dev/joondev-homelab/docker/grafana/provisioning): Grafana datasources and dashboards

## Quick Start

1. Create `docker/.env` using the keys listed in [`docker/.env.example`](/mnt/c/Work-Dev/joondev-homelab/docker/.env.example:1).
2. Validate the stack:

```bash
docker compose -f docker/docker-compose.yml config
```

3. Start the stack:

```bash
docker compose -f docker/docker-compose.yml up -d
```

4. Access the main entrypoints through `nginx`:

- `http://<host>/grafana/`
- `http://<host>/prometheus/`
- `http://<host>/portainer/`
- `http://<host>/dozzle/`

## Notes

- The repository now ignores `docker/.env` so credentials are less likely to be committed accidentally.
- [`AGENTS.md`](/mnt/c/Work-Dev/joondev-homelab/AGENTS.md:1) contains an agent-oriented summary of the stack and known issues.

## Current Gaps

From static review, a few parts of the stack still need tightening:

- Some services are exposed directly on host ports in addition to the reverse proxy
- Grafana subpath configuration should be made explicit
- Prometheus should be configured for reverse-proxy use under `/prometheus/`
- `uptime-kuma` is declared in Compose but not routed in `nginx`
