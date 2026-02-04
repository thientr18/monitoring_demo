# Monitoring Demo Stack

A complete monitoring solution using Prometheus, Grafana, Node Exporter, Blackbox Exporter, and Alertmanager to monitor system metrics, website availability, and send alerts.

## Architecture

This demo sets up a full monitoring stack with the following components:

- **Prometheus**: Metrics collection and time-series database
- **Grafana**: Visualization and dashboards
- **Node Exporter**: System and hardware metrics
- **Blackbox Exporter**: HTTP/HTTPS endpoint monitoring
- **Alert Manager**: Alert routing and notification management

## Prerequisites

- Docker installed on your system
- Ports available: 3000, 9090, 9093, 9100, 9115

## Quick Start

### 1. Create Docker Network

```bash
docker network create demo_network
```

### 2. Launch Grafana

```bash
docker run -d --name=grafana -p 3000:3000 --network demo_network grafana/grafana
```

- **Access**: http://localhost:3000
- **Default credentials**: admin/admin

### 3. Launch Prometheus

```bash
docker run --name prometheus -d -p 9090:9090 --network demo_network -v ${PWD}/prometheus:/etc/prometheus prom/prometheus
```

**Note**: If the volume mount fails, replace `${PWD}` with your absolute path:
- Get your current path:
  - Windows (PowerShell): `pwd` or `Get-Location`
  - Windows (CMD): `cd`
  - Linux/Mac: `pwd`
- Then use the full path:
  - Windows: `-v D:\your\path\monitoring_demo\prometheus:/etc/prometheus`
  - Linux/Mac: `-v /your/path/monitoring_demo/prometheus:/etc/prometheus`

- **Access**: http://localhost:9090

### 4. Launch Node Exporter

```bash
docker run -d --name host-node-exporter --net="demo_network" -p 9100:9100 prom/node-exporter
```

- **Access**: http://localhost:9100/metrics

### 5. Launch Blackbox Exporter

```bash
docker run -d -p 9115:9115 --name host-blackbox --network demo_network prom/blackbox-exporter
```

- **Access**: http://localhost:9115

### 6. Launch Alertmanager

```bash
docker run --name host-alertmanager -d -p 9093:9093 --network demo_network -v ${PWD}/alertmanager:/etc/alertmanager prom/alertmanager
```

**Note**: If the volume mount fails, replace `${PWD}` with your absolute path:
- Get your current path:
  - Windows (PowerShell): `pwd` or `Get-Location`
  - Windows (CMD): `cd`
  - Linux/Mac: `pwd`
- Then use the full path:
  - Windows: `-v your\path\monitoring_demo\alertmanager:/etc/alertmanager`
  - Linux/Mac: `-v /your/path/monitoring_demo/alertmanager:/etc/alertmanager`

- **Access**: http://localhost:9093

## Monitoring Targets

### System Metrics (Node Exporter)
- CPU usage and load average
- Memory and disk usage
- Network statistics
- System uptime

### Website Availability (Blackbox Exporter)
The following websites are monitored for HTTP 200 status:
- https://www.google.com
- https://www.youtube.com
- https://www.facebook.com

### Alert Rules
Configured alerts in `prometheus/rules/alert-rules.yml`:
- **AlwaysFiringTest**: Test alert that always fires (for testing)
- **HighRAMUsage**: Triggers when RAM usage > 80% for 5 seconds

## Configuration

### Project Structure

```
monitoring_demo/
├── prometheus/
│   ├── prometheus.yml          # Main Prometheus configuration
│   └── rules/
│       └── alert-rules.yml     # Alerting rules
└── alertmanager/
    └── alertmanager.yml        # Alertmanager configuration
```

### Prometheus Jobs

The Prometheus configuration includes 4 scrape jobs:

1. **prometheus**: Self-monitoring (1m interval)
2. **node**: System metrics from Node Exporter
3. **blackbox**: HTTP endpoint monitoring
4. **blackbox_exporter**: Blackbox Exporter self-monitoring

### Alertmanager Configuration

Edit `alertmanager/alertmanager.yml` to configure alert receivers (email, Slack, webhook, etc.).

Default configuration uses console webhook receiver for testing.

## Grafana Setup

1. Access Grafana at http://localhost:3000
2. Add Prometheus as a data source:
   - URL: `http://prometheus:9090`
3. Import dashboards:
   - Node Exporter Full (ID: 1860)
   - Blackbox Exporter (ID: 13659)

## Useful Commands

### Check Container Status
```bash
docker ps
```

### View Logs
```bash
docker logs prometheus
docker logs grafana
docker logs host-node-exporter
docker logs host-blackbox
docker logs host-alertmanager
```

### Restart Containers (after config changes)
```bash
docker restart prometheus
docker restart host-alertmanager
```

### Stop All Containers
```bash
docker stop grafana prometheus host-node-exporter host-blackbox host-alertmanager
```

### Remove All Containers
```bash
docker rm grafana prometheus host-node-exporter host-blackbox host-alertmanager
docker network rm demo_network
```

## Sample Prometheus Queries

### CPU Usage
```promql
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

### Memory Usage
```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
```

### Website Up/Down Status
```promql
probe_success{job="blackbox"}
```

### HTTP Response Time
```promql
probe_http_duration_seconds{job="blackbox"}
```

### Active Alerts
```promql
ALERTS{alertstate="firing"}
```

## Monitoring URLs

- **Prometheus**: http://localhost:9090
  - Alerts: http://localhost:9090/alerts
  - Targets: http://localhost:9090/targets
- **Grafana**: http://localhost:3000
- **Alertmanager**: http://localhost:9093
- **Node Exporter**: http://localhost:9100/metrics
- **Blackbox Exporter**: http://localhost:9115

## Troubleshooting

- **Prometheus can't reach targets**: Ensure all containers are on the `demo_network`
- **Grafana can't connect to Prometheus**: Use `http://prometheus:9090` (container name, not localhost)
- **Volume mount fails**: Update the path in the Docker command to match your workspace location
- **Alert rules not loading**: Check the `prometheus/rules/alert-rules.yml` file exists and syntax is correct
- **Alerts not firing**: Check Prometheus alerts page and verify Alertmanager is running
- **Alertmanager not receiving alerts**: Verify Prometheus configuration points to `host-alertmanager:9093`