# Lemonade Prometheus Demo

Local Prometheus and Grafana setup for Lemonade Server metrics.

## Requirements

- Lemonade Server running on the host at `http://localhost:13305`
- Docker Compose

## Run

```bash
curl http://localhost:13305/metrics
docker compose -f docker-compose.localhost.yml up -d
```

Open:

- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3000` (`admin` / `admin`)

Prometheus scrapes `host.docker.internal:13305/metrics`. In Prometheus,
**Status > Targets** should show `lemonade-localhost` as `UP`.

## Grafana

Add a Prometheus data source:

```text
http://prometheus:9090
```

Then import `lemonade-builtin-metrics-dashboard.json` from **Dashboards > New >
Import**.

```promql
lemonade_server_up
lemonade_loaded_models
lemonade_cpu_usage_percent
lemonade_memory_used_gb
```

Some metrics appear only after loading a model or sending traffic.

## Notes

- `metrics.md` documents the available Lemonade metrics.
- `python debug_metrics.py` watches raw `lemonade_` metrics locally.
- If Lemonade uses `LEMONADE_API_KEY`, Prometheus needs a bearer token.

## Stop

Stop containers:

```bash
docker compose -f docker-compose.localhost.yml down
```

Stop containers and remove saved Prometheus/Grafana data:

```bash
docker compose -f docker-compose.localhost.yml down -v
```
