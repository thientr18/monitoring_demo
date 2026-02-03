# Monitoring Demo Stack

A complete monitoring solution using Prometheus, Grafana, Node Exporter, and Blackbox Exporter to monitor system metrics and website availability.

## Architecture

This demo sets up a full monitoring stack with the following components:

- **Prometheus**: Metrics collection and time-series database
- **Grafana**: Visualization and dashboards
- **Node Exporter**: System and hardware metrics
- **Blackbox Exporter**: HTTP/HTTPS endpoint monitoring

## Prerequisites

- Docker installed on your system
- Ports available: 3000, 9090, 9100, 9115

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
docker run --name prometheus -d -p 9090:9090 --network demo_network -v ${PWD}/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

**Note**: Replace `${PWD}` with your absolute path if needed:
- Windows: `-v D:\path\to\your\project\prometheus.yml:/etc/prometheus/prometheus.yml`
- Linux/Mac: `-v /path/to/your/project/prometheus.yml:/etc/prometheus/prometheus.yml`

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
- https://cybersoft.edu.vn

## Configuration

### Prometheus Jobs

The Prometheus configuration includes 4 scrape jobs:

1. **prometheus**: Self-monitoring (1m interval)
2. **node**: System metrics from Node Exporter
3. **blackbox**: HTTP endpoint monitoring
4. **blackbox_exporter**: Blackbox Exporter self-monitoring

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
```

### Stop All Containers
```bash
docker stop grafana prometheus host-node-exporter host-blackbox
```

### Remove All Containers
```bash
docker rm grafana prometheus host-node-exporter host-blackbox
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

## Troubleshooting

- **Prometheus can't reach targets**: Ensure all containers are on the `demo_network`
- **Grafana can't connect to Prometheus**: Use `http://prometheus:9090` (container name, not localhost)
- **Volume mount fails**: Update the path in the Prometheus command to match your workspace location

## Next Steps

- Add alerting rules in Prometheus
- Configure Alertmanager for notifications
- Create custom Grafana dashboards
- Add more exporters (MySQL, Redis, etc.)
- Set up authentication and security