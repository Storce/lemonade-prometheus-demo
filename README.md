# Lemonade Prometheus Testing Setup

This tutorial sets up a local Prometheus and Grafana stack for testing the
Lemonade Server built-in `/metrics` endpoint.

The setup assumes Lemonade is running on your host machine at
`http://localhost:13305`, while Prometheus and Grafana run in Docker.

## Files

- `docker-compose.localhost.yml`: starts Prometheus and Grafana
- `prometheus.localhost.yml`: configures Prometheus to scrape Lemonade on localhost
- `lemonade-builtin-metrics-dashboard.json`: Grafana dashboard to import
- `debug_metrics.py`: optional helper for watching raw Lemonade metrics

## 1. Start Lemonade

Start your Lemonade instance first. It must be reachable on the host at:

```text
http://localhost:13305
```

Verify the metrics endpoint from your host:

```bash
curl http://localhost:13305/metrics
```

You should see Prometheus-formatted metrics with names that start with
`lemonade_`, for example:

```text
lemonade_server_up
lemonade_loaded_models
lemonade_cpu_usage_percent
```

If Lemonade is configured with `LEMONADE_API_KEY`, Prometheus must also be
configured with a bearer token before scraping will work.

## 2. Start Prometheus and Grafana

From this directory, run:

```bash
docker compose -f docker-compose.localhost.yml up -d
```

This starts:

- Prometheus at `http://localhost:9090`
- Grafana at `http://localhost:3000`

Check that both containers are running:

```bash
docker compose -f docker-compose.localhost.yml ps
```

## 3. Verify Prometheus Can Scrape Lemonade

Open Prometheus:

```text
http://localhost:9090
```

Go to **Status > Targets**.

The `lemonade-localhost` target should be listed as **UP**. The target URL
should point to:

```text
host.docker.internal:13305
```

Prometheus uses `host.docker.internal` so the Docker container can reach the
Lemonade process running on your host machine.

Next, go to the Prometheus query page and run:

```promql
lemonade_server_up
```

The value should be `1`.

## 4. Log In to Grafana

Open Grafana:

```text
http://localhost:3000
```

Use the default local credentials:

```text
Username: admin
Password: admin
```

Grafana may ask you to set a new password. You can set one or skip it for local
testing.

## 5. Add Prometheus as a Grafana Data Source

In Grafana, open the left navigation and go to:

```text
Connections > Data sources
```

Select **Add data source**, then choose **Prometheus**.

Use this URL:

```text
http://prometheus:9090
```

Click **Save & test**.

Grafana should report that the Prometheus data source is working.

Use `http://prometheus:9090`, not `http://localhost:9090`, because Grafana is
running inside Docker and reaches Prometheus by Docker Compose service name.

## 6. Import the Lemonade Dashboard

Open the dashboard import page in Grafana:

```text
Dashboards > New > Import
```

Copy the full contents of:

```text
lemonade-builtin-metrics-dashboard.json
```

Paste the JSON into the import field, then click **Load**.

When Grafana asks for the Prometheus data source, select the Prometheus data
source you created in the previous step.

Click **Import**.

## 7. Navigate to the Dashboard

In Grafana, open:

```text
Dashboards
```

Select the imported Lemonade dashboard.

Panels should begin showing data after Prometheus has scraped Lemonade. This
compose setup uses a `1s` scrape interval in `prometheus.localhost.yml`, so new
data should appear quickly.

If a panel is empty, first verify that the matching metric exists in Prometheus.
For example:

```promql
lemonade_loaded_models
```

Some dashboard panels only show data after you load a model or send requests to
Lemonade.

## 8. Open Apps in Lemonade and Generate Activity

Open your Lemonade UI and navigate to **Apps**.

Use an app or model flow that sends requests through Lemonade. Then return to
Grafana and Prometheus to confirm the metrics change.

Useful Prometheus queries for verification:

```promql
lemonade_server_up
lemonade_loaded_models
lemonade_cpu_usage_percent
lemonade_memory_used_gb
```

Request and token counters may only change after model traffic is generated.

## 9. Optional: Watch Raw Metrics Locally

You can also watch raw Lemonade metrics from the terminal:

```bash
python debug_metrics.py
```

The script polls:

```text
http://localhost:13305/metrics
```

## 10. Stop the Stack

When finished, stop Prometheus and Grafana:

```bash
docker compose -f docker-compose.localhost.yml down
```

To remove the saved Prometheus and Grafana volumes as well:

```bash
docker compose -f docker-compose.localhost.yml down -v
```
