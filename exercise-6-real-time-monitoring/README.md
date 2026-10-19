# Exercise 6 — Real-Time Operations Monitoring and Alerting

Scenario: ZAPPTTO needs real-time delivery metrics, dashboards, and alert rules.

## Contents
- `delivery_metrics.py`: generates metrics every second on port 8000.
- `prometheus.yml`: Prometheus scrape configuration.
- `alert_rules.yml`: warning and critical rules.
- `docker-compose.yml`: starts the app, Prometheus, and Grafana on one Docker network.
- `grafana/`: provisions the Prometheus data source and dashboard.
- `Jenkinsfile`: validates project files, Python syntax, Compose config, and builds the metrics image.
- `screenshots/`: save evidence screenshots here.

## Run on Windows PowerShell
Make sure Docker Desktop is running, extract the ZIP, then run PowerShell in this folder:

```powershell
cd "$HOME\Documents\kubernetes-hands-on\exercise-6-real-time-monitoring"
docker compose up -d --build
docker compose ps
```

First startup may take a few minutes while images download.

## Result 1: Metrics endpoint
Open `http://localhost:8000/metrics` in a browser, or run:

```powershell
curl.exe http://localhost:8000/metrics
docker compose logs --tail 30 delivery-metrics
```

Look for `total_deliveries`, `pending_deliveries`, `on_the_way_deliveries`, `average_delivery_time_sum`, and `average_delivery_time_count`.

## Result 2: Prometheus
Open `http://localhost:9090/targets`. The `delivery_service` target should be **UP**.
Open `http://localhost:9090/graph`, enter `pending_deliveries`, and click Execute. Also try `total_deliveries` and `average_delivery_time_sum / average_delivery_time_count`.
Open `http://localhost:9090/alerts` to view the alert rules and their states.

## Result 3: Grafana dashboard
Open `http://localhost:3000`.
Login: username `admin`, password `admin`.
Open **Dashboards** → **Delivery Monitoring** → **ZAPPTTO Delivery Monitoring**. Wait 15–30 seconds after startup if the dashboard is initially empty. Panels display total, pending, on-the-way deliveries, average delivery time, and time series.

## Result 4: Trigger/observe alerts
- `HighPendingDeliveries`: pending deliveries > 10 for 15 seconds. The simulator generates pending values from 10 to 20, so it should often fire.
- `HighAverageDeliveryTime`: average of observed delivery-time samples > 30 seconds for 15 seconds. Because values are random, this one may or may not fire.

For a repeatable demo of the critical alert, stop the stack and edit `delivery_metrics.py`: change `avg_time = random.uniform(15, 45)` to `avg_time = random.uniform(35, 45)`. Then rebuild with `docker compose up -d --build` and wait at least 30 seconds before refreshing `http://localhost:9090/alerts`. You can restore the original range afterwards.

## Jenkins pipeline (optional)
The `Jenkinsfile` expects a Linux Jenkins agent with Python 3, Docker CLI, Docker Compose plugin, and access to the Docker daemon. Put the project in Git and configure a Pipeline job using **Pipeline script from SCM** and select this `Jenkinsfile`. Its stages check required files, compile-check Python, validate Compose, and build the app image. If Jenkins has no Docker access, the build stage will fail; the Docker Compose monitoring stack can still be run directly as above.

## Screenshots to save
Save these in `screenshots/`:
1. `docker-compose-ps.png` — all three services running.
2. `metrics-endpoint.png` — `/metrics` output.
3. `prometheus-targets.png` — delivery target UP.
4. `prometheus-alerts.png` — alert rules/states.
5. `grafana-dashboard.png` — dashboard with live values.
6. `jenkins-pipeline.png` — successful pipeline, if completing Jenkins.

## Stop the services
After taking screenshots:

```powershell
docker compose down
```

This keeps the Prometheus/Grafana data volumes. To delete those too:

```powershell
docker compose down -v
```

## Troubleshooting
- Port already in use: run `docker ps` and stop the conflicting container, or change the host-side port.
- Container exits: use `docker compose logs delivery-metrics`, `docker compose logs prometheus`, or `docker compose logs grafana`.
- Prometheus target DOWN: check `docker compose ps` and `docker compose logs prometheus`.
- Container name already exists: run `docker compose down` from this folder, then `docker compose up -d --build`.
