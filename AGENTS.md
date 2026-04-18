# AGENTS.md

## Purpose

This repository contains a Docker Compose homelab operations stack. It is not an application codebase. Its main job is to run monitoring, logging, and admin tooling around a small self-hosted environment.

The stack is defined almost entirely under [`docker/`](/mnt/c/Work-Dev/joondev-homelab/docker).

## Stack Summary

Primary services:

- `nginx`: reverse proxy and front door on port `80`
- `grafana`: dashboards and visualization
- `prometheus`: metrics collection
- `loki`: log storage
- `promtail`: log shipping from Docker and host logs
- `node-exporter`: host metrics
- `cadvisor`: container metrics
- `postgres`: database service
- `postgres-exporter`: Postgres metrics
- `portainer`: Docker management UI
- `uptime-kuma`: service availability monitoring
- `dozzle`: live container log viewer
- `elasticsearch`: standalone search/index service

Commented-out services:

- `kibana`: Elasticsearch UI

## Networks

Two Docker bridge networks are defined in [`docker/docker-compose.yml`](/mnt/c/Work-Dev/joondev-homelab/docker/docker-compose.yml:1):

- `monitoring`: observability and admin-plane services
- `application`: app-facing services such as `postgres` and `portainer`

`nginx` is attached to both networks so it can proxy selected UIs.

## Traffic Flow

### Reverse-proxied paths

Configured in [`docker/nginx/nginx.conf`](/mnt/c/Work-Dev/joondev-homelab/docker/nginx/nginx.conf:10):

- `/grafana/` -> `grafana:3000`
- `/prometheus/` -> `prometheus:9090`
- `/portainer/` -> `portainer:9000`
- `/dozzle/` -> `dozzle:8080/dozzle/`

Default `/` returns a plain-text health response.

### Observability flow

- Prometheus scrapes `prometheus`, `node-exporter`, `cadvisor`, `loki`, and `postgres-exporter`
- Promtail discovers Docker containers through `/var/run/docker.sock`
- Promtail ships logs to Loki
- Grafana is provisioned with Prometheus and Loki datasources plus a default dashboard

## Important Files

- [`docker/docker-compose.yml`](/mnt/c/Work-Dev/joondev-homelab/docker/docker-compose.yml:1): source of truth for services, ports, volumes, and networks
- [`docker/nginx/nginx.conf`](/mnt/c/Work-Dev/joondev-homelab/docker/nginx/nginx.conf:1): reverse proxy rules
- [`docker/prometheus/prometheus.yml`](/mnt/c/Work-Dev/joondev-homelab/docker/prometheus/prometheus.yml:1): scrape targets
- [`docker/promtail/promtail-config.yaml`](/mnt/c/Work-Dev/joondev-homelab/docker/promtail/promtail-config.yaml:1): Docker and host log collection
- [`docker/loki/loki-config.yaml`](/mnt/c/Work-Dev/joondev-homelab/docker/loki/loki-config.yaml:1): Loki storage and schema config
- [`docker/grafana/provisioning/datasources/datasources.yaml`](/mnt/c/Work-Dev/joondev-homelab/docker/grafana/provisioning/datasources/datasources.yaml:1): Grafana datasources
- [`docker/grafana/provisioning/dashboards/dashboards.yml`](/mnt/c/Work-Dev/joondev-homelab/docker/grafana/provisioning/dashboards/dashboards.yml:1): dashboard provisioning
- [`docker/grafana/provisioning/dashboards/leviathan-overview.json`](/mnt/c/Work-Dev/joondev-homelab/docker/grafana/provisioning/dashboards/leviathan-overview.json:1): prebuilt dashboard
- [`docker/.env`](/mnt/c/Work-Dev/joondev-homelab/docker/.env:1): runtime secrets and credentials

## Environment Variables

The stack expects these values in [`docker/.env`](/mnt/c/Work-Dev/joondev-homelab/docker/.env:1):

- `POSTGRES_USER`
- `POSTGRES_PASSWORD`
- `POSTGRES_DB`
- `GRAFANA_USER`
- `GRAFANA_PASSWORD`

Do not print or commit secret values.

## Known Risks And Gaps

These came from static inspection of the repository. Runtime validation was not completed in the analysis environment because the `docker` CLI was not available there.

- Several services are exposed directly on host interfaces instead of only through `nginx`, including `grafana`, `cadvisor`, `portainer`, and `postgres`
- `grafana` is configured for `/grafana/`, but `GF_SERVER_ROOT_URL` uses `%(domain)s` without setting `GF_SERVER_DOMAIN`; this may produce bad redirects or asset URLs
- `prometheus` is reverse-proxied under `/prometheus/`, but no `--web.external-url` or `--web.route-prefix` flags are set
- `nginx` depends on `uptime-kuma`, but there is no `location` block for it
- Loki’s ruler points to `http://localhost:9093`, but no Alertmanager service exists in the stack
- `docker/.env` contains live credentials; keep it gitignored and avoid copying those values into tracked files

## Working Rules For Future Changes

- Treat [`docker/docker-compose.yml`](/mnt/c/Work-Dev/joondev-homelab/docker/docker-compose.yml:1) as the primary change surface
- Preserve the split between `monitoring` and `application` networks unless there is a clear routing reason to change it
- Keep secrets in `docker/.env`; do not hardcode credentials into Compose files
- When changing reverse-proxied services, update both Compose and `nginx.conf`
- When adding Grafana dashboards or datasources, prefer provisioning files over manual UI setup
- If Docker is available, validate changes with `docker compose -f docker/docker-compose.yml config` before bringing the stack up
