# Prometheus Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Configuration Basics](#3-configuration-basics)
4. [PromQL Query Language](#4-promql-query-language)
5. [Prometheus Components](#5-prometheus-components)
6. [Exporters](#6-exporters)
7. [Service Discovery](#7-service-discovery)
8. [Recording Rules](#8-recording-rules)
9. [Alerting](#9-alerting)
10. [Integration with Grafana](#10-integration-with-grafana)
11. [Performance Tuning](#11-performance-tuning)
12. [Troubleshooting](#12-troubleshooting)
13. [Quick Reference](#13-quick-reference)

---

## 1. Introduction

### What is Prometheus?
Prometheus is an open-source systems monitoring and alerting toolkit with a powerful query language (PromQL) for time series data.

### Key Features
| Feature | Description |
|---------|-------------|
| Multi-dimensional data model | Time series identified by metrics and key-value pairs |
| PromQL | Flexible query language for time series analysis |
| No reliance on distributed storage | Single server nodes are autonomous |
| Pull-based metrics collection | HTTP pull model for flexibility |
| Service discovery | Automatic target discovery |
| Alerting | Built-in Alertmanager integration |
| Visualization | Native expression browser and Grafana integration |

### Architecture Components
```
┌─────────────────────────────────────────────────────────────┐
│                      Prometheus Server                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  Storage    │  │  Retrieval  │  │   HTTP Server      │ │
│  │  (TSDB)     │←→│  (Scraper)  │→│   (API/UI)         │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│                                                             │
│  ┌─────────────┐  ┌─────────────────────────────────────┐ │
│  │   Rules     │  │       Query Engine (PromQL)         │ │
│  │ & Alerts    │  │                                     │ │
│  └─────────────┘  └─────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
          │                    │
          │ Pull Metrics       │ Alert Rules
          ▼                    ▼
┌──────────────┐      ┌────────────────┐
│   Targets    │      │  Alertmanager  │
│ (Exporters)  │      │  (External)    │
└──────────────┘      └────────────────┘
                              │
                              ▼
                      ┌──────────────┐
                      │  Alert Routes│
                      │  (Email,Slack│
                      │   PagerDuty) │
                      └──────────────┘
```

### Core Concepts
| Term | Description |
|------|-------------|
| **Time Series** | Data points indexed by timestamp |
| **Metric** | Named time series with labeled dimensions |
| **Label** | Key-value pair for dimensional metrics |
| **Sample** | A single data point (timestamp + value) |
| **Exporter** | Process that exposes metrics for scraping |
| **Target** | Instance being monitored |
| **Job** | Collection of targets with same purpose |
| **PromQL** | Prometheus Query Language |

---

## 2. Installation

### Binary Installation
```bash
# Download Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.0/prometheus-2.47.0.linux-amd64.tar.gz
tar xvf prometheus-2.47.0.linux-amd64.tar.gz
cd prometheus-2.47.0.linux-amd64

# Run Prometheus
./prometheus --config.file=prometheus.yml

# As a service
sudo useradd --system --no-create-home --shell /bin/false prometheus
sudo cp prometheus /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
```

### Docker Installation
```bash
# Run Prometheus container
docker run \
    -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus

# With persistent storage
docker run \
    -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    -v /path/to/data:/prometheus \
    prom/prometheus

# Docker Compose
cat <<EOF > docker-compose.yml
version: '3'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'
EOF
```

### Kubernetes Installation
```bash
# Using Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack

# Using manifests
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/monitoring.coreos.com_prometheuses.yaml
```

### Verify Installation
```bash
# Check Prometheus version
prometheus --version

# Check configuration
promtool check config /path/to/prometheus.yml

# Access web interface
# http://localhost:9090

# Check metrics endpoint
curl http://localhost:9090/metrics

# Check API endpoints
curl http://localhost:9090/api/v1/status/config
curl http://localhost:9090/api/v1/query?query=up
```

---

## 3. Configuration Basics

### Main Configuration File
```yaml
# /etc/prometheus/prometheus.yml
# Global configuration
global:
  scrape_interval: 15s      # Default scrape interval
  evaluation_interval: 15s  # Rule evaluation interval
  external_labels:
    cluster: 'production'
    env: 'prod'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093
      timeout: 10s
      api_version: v2

# Load rules and recording rules from specified directories
rule_files:
  - /etc/prometheus/rules/*.yml
  - /etc/prometheus/rules/*.yaml

# Scrape configurations
scrape_configs:
  # Prometheus self-monitoring
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
    metrics_path: /metrics
    scheme: http
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        regex: '(.*):\\d+'
        replacement: '${1}'

  # Node Exporter
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
    metrics_path: /metrics
    scheme: http

  # Blackbox Exporter
  - job_name: 'blackbox'
    static_configs:
      - targets: ['blackbox-exporter:9115']
    metrics_path: /probe
    params:
      module: [http_2xx]
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance

  # Kubernetes Service Discovery
  - job_name: 'kubernetes-service'
    kubernetes_sd_configs:
      - role: service
        api_server: https://kubernetes.default.svc
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\\d+)?;(\\d+)
        replacement: $1:$2
        target_label: __address__
```

### Configuration Options

#### Global Configuration
```yaml
global:
  scrape_interval: 15s          # How frequently to scrape targets
  scrape_timeout: 10s           # Timeout for individual scrapes
  evaluation_interval: 15s      # How frequently to evaluate rules
  external_labels:              # Labels added to all time series
    cluster: 'production'
    env: 'prod'
    region: 'us-west-2'
```

#### Scrape Configuration
```yaml
scrape_configs:
  - job_name: '<job_name>'
    # How frequently to scrape
    scrape_interval: 15s
    # Timeout for individual scrape
    scrape_timeout: 10s
    # Metrics path (default: /metrics)
    metrics_path: /metrics
    # Protocol scheme (http, https)
    scheme: https
    # DNS resolution
    dns_sd_configs:
      - names:
          - prometheus.io
        refresh_interval: 30s
    # Static targets
    static_configs:
      - targets:
          - localhost:9090
        labels:
          label1: value1
    # HTTP authorization
    basic_auth:
      username: prometheus
      password_file: /etc/prometheus/secret/password
    # TLS configuration
    tls_config:
      ca_file: /etc/prometheus/ca.crt
      cert_file: /etc/prometheus/cert.crt
      key_file: /etc/prometheus/key.key
      insecure_skip_verify: false
    # Proxy URL
    proxy_url: http://proxy.example.com:8080
```

#### Service Discovery
```yaml
# Kubernetes Service Discovery
scrape_configs:
  - job_name: 'kubernetes-service'
    kubernetes_sd_configs:
      - role: service
        api_server: https://kubernetes.default.svc
        namespaces:
          names:
            - default
            - production
        selectors:
          role: service
          label: "app=prometheus"
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_name]
        target_label: service

  - job_name: 'kubernetes-pod'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_ip]
        target_label: pod_ip

  - job_name: 'kubernetes-node'
    kubernetes_sd_configs:
      - role: node

  - job_name: 'kubernetes-endpoints'
    kubernetes_sd_configs:
      - role: endpoints
```

#### Relabel Configuration
```yaml
scrape_configs:
  - job_name: 'example'
    static_configs:
      - targets: ['localhost:8080']
        labels:
          env: production
    relabel_configs:
      # Keep only targets with specific label
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true

      # Replace/rewrite labels
      - source_labels: [__address__]
        target_label: instance
        regex: '(.+):\\d+'
        replacement: '${1}'

      # Map labels
      - source_labels: [__meta_kubernetes_service_name]
        target_label: service_name

      # Remove labels
      - regex: __meta_kubernetes_.*
        action: labeldrop

      # Label mapping
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
        replacement: $1

      # Hashbmod for load balancing
      - target_label: __address__
        replacement: blackbox-exporter:9115
```

---

## 4. PromQL Query Language

### Basic Queries

#### Instant Queries
```promql
# Select all time series with metric name
up

# Select specific metric with labels
up{job="prometheus"}

# Select with multiple labels
up{job="prometheus", env="production"}

# Select with regex matching
up{job=~"prometheus|node-exporter"}

# Select with not equal
up{job!="test"}

# Select with label existence
up{job=~".+"}

# Select without label
up{env=""}
```

#### Range Queries
```promql
# Last 5 minutes
up[5m]

# Last 1 hour
up[1h]

# Last 24 hours
up[24h]

# Last 7 days
up[7d]

# Last 30 minutes with 1-minute resolution
up[30m:1m]

# Last 1 hour with 30-second resolution
rate(http_requests_total[1h:30s])
```

### Vector Types

#### Instant Vector
```promql
# Returns a single sample for each time series
up
http_requests_total{job="webserver"}

# With labels
container_memory_usage_bytes{container="nginx"}
```

#### Range Vector
```promql
# Returns a range of samples over time
up[5m]
rate(http_requests_total[5m])

# Combined with functions
increase(http_requests_total[1h])
```

#### Scalar
```promql
# Single numeric value
100

# Expression resulting in scalar
http_requests_total{job="webserver"} / 2
```

### Common Functions

#### Aggregation Functions
```promql
# Sum all values
sum(http_requests_total)

# Sum by labels
sum by (job) (http_requests_total)
sum without (instance) (http_requests_total)

# Average
avg(cpu_usage_seconds_total)

# Minimum
min(kube_pod_container_resource_requests_cpu_cores)

# Maximum
max(kube_pod_container_resource_limits_memory_bytes)

# Count
count(up)

# Count by labels
count by (job) (up)

# Top 10 largest values
topk(10, http_requests_total)

# Bottom 5 smallest values
bottomk(5, http_requests_total)
```

#### Rate Functions
```promql
# Calculate per-second rate
rate(http_requests_total[5m])

# Calculate increase over time
increase(http_requests_total[1h])

# Calculate rate of counter resets
irate(http_requests_total[5m])

# Calculate per-second rate of histogram
rate(http_request_duration_seconds_bucket[5m])

# Calculate increase with extrapolation
increase(http_requests_total[1h])
```

#### Time Functions
```promql
# Current timestamp
time()

# Time since epoch in seconds
timestamp(up)

# Subquery for time-based calculations
max_over_time(http_requests_total[1h:5m])

# Minimum over time range
min_over_time(http_requests_total[5m])

# Maximum over time range
max_over_time(http_requests_total[5m])

# Average over time range
avg_over_time(http_requests_total[5m])

# Sum over time range
sum_over_time(http_requests_total[5m])

# Count samples in range
count_over_time(http_requests_total[5m])

# Quantile over time range
quantile_over_time(0.95, http_request_duration_seconds[5m])

# Change rate
delta(cpu_seconds_total[5m])

# Derivative
deriv(http_requests_total[5m])

# Holt-Winters smoothing
holt_winters(http_requests_total[5m], 0.2, 0.2)
```

#### Mathematical Functions
```promql
# Absolute value
abs(up - 1)

# Absolute difference
absdiff(http_requests_total[5m])

# Ceiling
ceil(http_request_duration_seconds)

# Floor
floor(http_request_duration_seconds)

# Round
round(http_request_duration_seconds, 0.01)

# Square root
sqrt(http_request_duration_seconds)

# Power
pow(http_request_duration_seconds, 2)

# Logarithm (natural)
ln(http_request_duration_seconds)

# Logarithm (base 2)
log2(http_request_duration_seconds)

# Logarithm (base 10)
log10(http_request_duration_seconds)

# Clamp value
clamp(http_request_duration_seconds, 0, 10)

# Clamp minimum
clamp_min(http_request_duration_seconds, 0.001)

# Clamp maximum
clamp_max(http_request_duration_seconds, 30)

# Exponential
exp(http_request_duration_seconds)

# Histogram quantiles
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Histogram average
histogram_avg(rate(http_request_duration_seconds_bucket[5m]))

# Histogram sum
histogram_sum(rate(http_request_duration_seconds_bucket[5m]))

# Histogram count
histogram_count(rate(http_request_duration_seconds_bucket[5m]))
```

#### Label Functions
```promql
# Remove labels
sum without (instance) (http_requests_total)

# Keep only specific labels
sum by (job, env) (http_requests_total)

# Label matching
label_join(up{job="prometheus"}, "combined", "-", "job", "instance")

# Label replacement
label_replace(up, "service", "$1", "job", "(.*)")

# Drop labels
drop_common_labels(http_requests_total)

# Keep only matching series
keep_common_labels(http_requests_total)
```

#### Type Conversion
```promql
# Convert to float
float(http_requests_total)

# Convert to int
int(http_requests_total)

# Convert Unix timestamp to time
timestamp_to_time(time())

# Convert time to Unix timestamp
time_to_timestamp(time())
```

### Operators

#### Arithmetic Operators
```promql
# Addition
http_requests_total + 100

# Subtraction
http_requests_total - 50

# Multiplication
rate(http_requests_total[5m]) * 100

# Division
rate(http_requests_total[5m]) / 100

# Modulo
http_requests_total % 10

# Power
http_requests_total ^ 2

# Equality
http_requests_total == 100

# Not equal
http_requests_total != 100

# Greater than
http_requests_total > 100

# Greater than or equal
http_requests_total >= 100

# Less than
http_requests_total < 100

# Less than or equal
http_requests_total <= 100
```

#### Logical Operators
```promql
# And (intersection)
up{job="prometheus"} and up{env="production"}

# Or (union)
up{job="prometheus"} or up{job="node"}

# Unless (complement)
up{job="prometheus"} unless up{env="test"}
```

#### Aggregation Operators
```promql
# Sum
sum(http_requests_total)

# Min
min(http_requests_total)

# Max
max(http_requests_total)

# Avg
avg(http_requests_total)

# Group
group(http_requests_total)

# Stddev (standard deviation)
stddev(http_request_duration_seconds)

# Stdvar (variance)
stdvar(http_request_duration_seconds)

# Count
count(http_requests_total)

# Count values
count_values("value", http_requests_total)

# Quantile
quantile(0.95, http_request_duration_seconds)
```

### Common Query Patterns

#### CPU Usage
```promql
# CPU usage percentage
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Per-core CPU usage
100 - (rate(node_cpu_seconds_total{mode="idle"}[5m]) * 100)
```

#### Memory Usage
```promql
# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Memory usage by type
node_memory_MemTotal_bytes - node_memory_MemFree_bytes
```

#### Disk Usage
```promql
# Disk usage percentage
(1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100

# Disk I/O
rate(node_disk_read_bytes_total[5m])
rate(node_disk_written_bytes_total[5m])
```

#### Network Usage
```promql
# Network bytes per second
rate(node_network_receive_bytes_total[5m])
rate(node_network_transmit_bytes_total[5m])

# Network errors
rate(node_network_receive_errs_total[5m])
rate(node_network_transmit_errs_total[5m])
```

#### HTTP Request Metrics
```promql
# Requests per second
rate(http_requests_total[5m])

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])

# Latency percentiles
histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))

# Requests by status code
sum by (status) (rate(http_requests_total[5m]))
```

#### Container Metrics
```promql
# Container CPU usage
rate(container_cpu_usage_seconds_total{image!=""}[5m])

# Container memory usage
container_memory_usage_bytes{image!=""}

# Container network
rate(container_network_rx_bytes_total{image!=""}[5m])
rate(container_network_tx_bytes_total{image!=""}[5m])
```

#### Kubernetes Metrics
```promql
# Pod CPU usage
rate(container_cpu_usage_seconds_total{pod!=""}[5m])

# Pod memory usage
container_memory_working_set_bytes{pod!=""}

# Pod restarts
increase(kube_pod_container_status_restarts_total[1h])

# Node status
kube_node_status_condition{condition="Ready"}

# Persistent Volume claims
kube_persistentvolumeclaim_status_phase
```

---

## 5. Prometheus Components

### Prometheus Server
```bash
# Command line flags
prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus \
    --storage.tsdb.retention=15d \
    --web.enable-lifecycle \
    --web.enable-admin-api \
    --query.max-concurrency=10 \
    --query.timeout=2m \
    --web.route-prefix=/ \
    --web.external-url=http://prometheus.example.com

# Storage flags
--storage.tsdb.path=/var/lib/prometheus
--storage.tsdb.min-block-duration=2h
--storage.tsdb.max-block-duration=2h
--storage.tsdb.retention.time=15d
--storage.tsdb.retention.size=100GB
--storage.tsdb.wal-compression
--no-storage.tsdb.allow-overlapping-blocks
```

### Promtool
```bash
# Check configuration
promtool check config /etc/prometheus/prometheus.yml

# Check rules
promtool check rules /etc/prometheus/rules/*.yml

# Test query
promtool query instant --time=2023-01-01T00:00:00Z 'up'

# Query range
promtool query range --start=2023-01-01T00:00:00Z --end=2023-01-01T01:00:00Z --step=1m 'up'

# Format rules
promtool rules print /etc/prometheus/rules/*.yml
```

### PromQL Engine
```bash
# Start interactive PromQL
promql

# Run PromQL from command line
promql --engine.max-samples=10000000 'rate(http_requests_total[5m])'
```

---

## 6. Exporters

### Node Exporter
```bash
# Install Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar xvf node_exporter-1.6.1.linux-amd64.tar.gz
./node_exporter

# Run as service
cat <<EOF > /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

# Enable collectors
./node_exporter --collector.disable-defaults --collector.cpu --collector.meminfo --collector.diskstats --collector.filesystem --collector.loadavg --collector.netdev --collector.stat --collector.time --collector.uname --collector.vmstat --collector.tcpstat --collector.drbd --collector.nfs --collector.zfs --collector.btrfs --collector.hwmon --collector.thermal_zone --collector.pressure --collector.bonding --collector.bcache --collector.buddyinfo --collector.conntrack --collector.cpufreq --collector.entropy --collector.filefd --collector.fragmentation --collector.ipvs --collector.irq --collector.meminfo_numa --collector.netdev --collector.netstat --collector.sockstat --collector.softirqs --collector.stat --collector.tcpstat --collector.textfile --collector.time --collector.uname --collector.vmstat --collector.xfs --collector.zoneinfo

# Docker
docker run -d \
    --net="host" \
    --pid="host" \
    -v "/":/rootfs:ro \
    -v "/sys:/sys:ro" \
    -v "/dev:/dev:ro" \
    prom/node-exporter \
    --path.rootfs=/rootfs \
    --collector.disable-defaults \
    --collector.cpu \
    --collector.meminfo \
    --collector.diskstats \
    --collector.filesystem
```

### MySQL Exporter
```bash
# Install MySQL Exporter
docker run -d \
    -p 9104:9104 \
    -e DATA_SOURCE_NAME="user:password@(mysql-server:3306)/" \
    prom/mysqld-exporter

# my.cnf configuration
[client]
user=mysql_exporter
password=password

# Run standalone
./mysqld_exporter --config.my-cnf=/etc/.my.cnf
```

### PostgreSQL Exporter
```bash
# Install PostgreSQL Exporter
docker run -d \
    -p 9187:9187 \
    -e DATA_SOURCE_NAME="postgresql://postgres:password@postgres-server:5432/postgres?sslmode=disable" \
    prom/postgres_exporter

# With custom queries
DATA_SOURCE_NAME="postgresql://postgres:password@localhost:5432/postgres?sslmode=disable"
./postgres_exporter --config.file=/etc/postgres_exporter/queries.yml
```

### Redis Exporter
```bash
# Run Redis Exporter
docker run -d \
    -p 9121:9121 \
    -e REDIS_ADDR="redis://redis-server:6379" \
    oliver006/redis_exporter

# With password
docker run -d \
    -p 9121:9121 \
    -e REDIS_ADDR="redis://:password@redis-server:6379" \
    oliver006/redis_exporter
```

### Blackbox Exporter
```bash
# Create configuration
cat <<EOF > /etc/prometheus/blackbox.yml
modules:
  http_2xx:
    prober: http
    timeout: 10s
    http:
      valid_status_codes: [200, 301, 302]
      method: GET
      preferred_ip_protocol: "ip4"
      ip_protocol_fallback: false

  http_post_2xx:
    prober: http
    http:
      method: POST
      headers:
        Content-Type: application/json
      body: '{"key": "value"}'

  tcp_connect:
    prober: tcp
    timeout: 5s

  dns_udp:
    prober: dns
    dns:
      transport_protocol: udp
      query_type: A

  icmp:
    prober: icmp
    timeout: 5s
    icmp:
      preferred_ip_protocol: "ip4"

# Run Blackbox Exporter
docker run -d \
    -p 9115:9115 \
    -v /etc/prometheus/blackbox.yml:/etc/blackbox_exporter/config.yml \
    prom/blackbox-exporter
```

### SNMP Exporter
```bash
# Generate SNMP configuration
snmp_exporter --gen-config > snmp.yml

# Edit snmp.yml with your device MIBs

# Run SNMP Exporter
docker run -d \
    -p 9116:9116 \
    -v /etc/prometheus/snmp.yml:/etc/snmp_exporter/snmp.yml \
    prom/snmp_exporter
```

### Kafka Exporter
```bash
# Run Kafka Exporter
docker run -d \
    -p 9308:9308 \
    -e KAFKA_BROKERS="kafka-server:9092" \
    danielqsj/kafka_exporter
```

### JMX Exporter
```bash
# Run JMX Exporter with Java application
java -javaagent:/path/to/jmx_prometheus_javaagent-0.17.2.jar=8080:/path/to/config.yaml -jar your-app.jar

# Example config.yaml
lowercaseOutputName: true
rules:
  - pattern: ".*"
```

### GPU Exporter (DCGM)
```bash
# Run DCGM Exporter
docker run -d \
    --runtime=nvidia \
    -p 9400:9400 \
    -e DCGM_EXPORTER_KEEPALIVE_INTERVAL=30s \
    nvcr.io/nvidia/k8s/dcgm-exporter:3.1.7-3.1.7-ubuntu20.04
```

---

## 7. Service Discovery

### Static Service Discovery
```yaml
scrape_configs:
  - job_name: 'static'
    static_configs:
      - targets:
          - localhost:9090
          - node-exporter:9100
          - cadvisor:8080
        labels:
          env: production
```

### DNS Service Discovery
```yaml
scrape_configs:
  - job_name: 'dns'
    dns_sd_configs:
      - names:
          - prometheus.io
          - grafana.io
        refresh_interval: 30s
      - type: A
        port: 80
```

### Consul Service Discovery
```yaml
scrape_configs:
  - job_name: 'consul'
    consul_sd_configs:
      - server: 'consul.example.com:8500'
        services:
          - prometheus
          - node-exporter
        tags:
          - production
        token: ${CONSUL_TOKEN}
        scheme: https
        refresh_interval: 30s
    relabel_configs:
      - source_labels: [__meta_consul_service]
        target_label: service
```

### EC2 Service Discovery
```yaml
scrape_configs:
  - job_name: 'ec2'
    ec2_sd_configs:
      - region: us-west-2
        access_key: ${AWS_ACCESS_KEY}
        secret_key: ${AWS_SECRET_KEY}
        profile: "profile"
        filters:
          - name: "tag:Prometheus"
            values: ["true"]
          - name: "instance-state-name"
            values: ["running"]
        port: 9100
        refresh_interval: 60s
    relabel_configs:
      - source_labels: [__meta_ec2_instance_id]
        target_label: instance_id
      - source_labels: [__meta_ec2_availability_zone]
        target_label: zone
      - source_labels: [__meta_ec2_tag_Name]
        target_label: name
      - source_labels: [__meta_ec2_tag_Environment]
        target_label: env
```

### Azure Service Discovery
```yaml
scrape_configs:
  - job_name: 'azure'
    azure_sd_configs:
      - subscription_id: ${AZURE_SUBSCRIPTION_ID}
        tenant_id: ${AZURE_TENANT_ID}
        client_id: ${AZURE_CLIENT_ID}
        client_secret: ${AZURE_CLIENT_SECRET}
        resource_group: production
        port: 9100
    relabel_configs:
      - source_labels: [__meta_azure_machine]
        target_label: machine
      - source_labels: [__meta_azure_resource_group]
        target_label: resource_group
```

### GCE Service Discovery
```yaml
scrape_configs:
  - job_name: 'gce'
    gce_sd_configs:
      - project: my-project
        zone: us-central1-a
        filter: name~'.*prometheus.*'
        credentials_file: /path/to/gce-credentials.json
    relabel_configs:
      - source_labels: [__meta_gce_instance_name]
        target_label: instance_name
```

### Kubernetes Service Discovery
```yaml
scrape_configs:
  # Service discovery for pods
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
        api_server: https://kubernetes.default.svc
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    relabel_configs:
      # Keep pods with prometheus.io/scrape annotation
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      # Use custom metrics path
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      # Use custom port
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      # Keep specific namespace
      - source_labels: [__meta_kubernetes_namespace]
        action: keep
        regex: production
      # Add pod name as label
      - source_labels: [__meta_kubernetes_pod_name]
        target_label: pod

  # Service discovery for services
  - job_name: 'kubernetes-services'
    kubernetes_sd_configs:
      - role: service
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - source_labels: [__meta_kubernetes_service_name]
        target_label: service

  # Service discovery for endpoints
  - job_name: 'kubernetes-endpoints'
    kubernetes_sd_configs:
      - role: endpoints
    relabel_configs:
      - source_labels: [__meta_kubernetes_endpoints_name]
        target_label: endpoint

  # Service discovery for nodes
  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - source_labels: [__meta_kubernetes_node_name]
        target_label: node
```

---

## 8. Recording Rules

### Rule File Structure
```yaml
# /etc/prometheus/rules/server-rules.yml
groups:
  - name: recording_rules
    interval: 30s
    rules:
      # Recording rule for CPU usage
      - record: node_cpu_usage
        expr: |
          100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

      # Recording rule for memory usage
      - record: node_memory_usage
        expr: |
          (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

      # Recording rule for disk usage
      - record: node_disk_usage
        expr: |
          (1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100

      # Recording rule for HTTP request rate
      - record: http_request_rate
        expr: |
          sum by (job, handler, status) (rate(http_requests_total[5m]))

      # Recording rule for HTTP latency
      - record: http_request_latency_p99
        expr: |
          histogram_quantile(0.99, sum by (job, handler, le) (rate(http_request_duration_seconds_bucket[5m])))

      # Recording rule for errors per second
      - record: http_error_rate
        expr: |
          sum by (job) (rate(http_requests_total{status=~"5.."}[5m]))

      # Recording rule for combined metrics
      - record: instance:cpu_utilization:rate5m
        expr: |
          100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

      - record: instance:memory_utilization:ratio
        expr: |
          node_memory_MemTotal_bytes{job="node"} / (1024 * 1024 * 1024)

      - record: instance:disk_io_utilization:rate
        expr: |
          (rate(node_disk_read_bytes_total[5m]) + rate(node_disk_written_bytes_total[5m])) / 1024 / 1024

      # Recording rule for SLO calculations
      - record: service_availability:ratio
        expr: |
          sum by (service) (rate(http_requests_total{status!~"5.."}[5m]))
          /
          sum by (service) (rate(http_request_total[5m]))

      # Recording rule for rate of change
      - record: http_requests_per_minute
        expr: |
          increase(http_requests_total[1m]) / 60
```

### Nested Recording Rules
```yaml
groups:
  - name: nested_rules
    rules:
      # Base metric
      - record: http_requests:total
        expr: sum by (job) (http_requests_total)

      # Derived from base metric
      - record: http_requests:rate5m
        expr: rate(http_requests:total[5m])

      # Further derived
      - record: http_requests:rate5m_by_status
        expr: sum by (job, status) (rate(http_requests_total[5m]))
```

### Federation
```yaml
# Global Prometheus scraping regional Prometheus
scrape_configs:
  - job_name: 'federate'
    scrape_interval: 30s
    honor_labels: true
    metrics_path: /federate
    params:
      match[]:
        - '{__name__=~"up|jobs:.*"}'
        - '{job="production"}'
    static_configs:
      - targets:
          - prometheus-regional-1:9090
          - prometheus-regional-2:9090
```

---

## 9. Alerting

### Alert Rules
```yaml
# /etc/prometheus/rules/alert-rules.yml
groups:
  - name: alerts
    rules:
      # High CPU alert
      - alert: HighCPUUsage
        expr: |
          100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is above 80% (current value: {{ $value | printf \"%.2f\" }}%)"
          runbook_url: "https://example.com/runbooks/high-cpu"

      # High memory alert
      - alert: HighMemoryUsage
        expr: |
          (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is above 90% (current value: {{ $value | printf \"%.2f\" }}%)"

      # Disk space alert
      - alert: DiskSpaceLow
        expr: |
          (1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100 > 85
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Disk space low on {{ $labels.instance }}"
          description: "Disk usage on {{ $labels.mountpoint }} is above 85%"

      # Target down alert
      - alert: TargetDown
        expr: |
          up == 0
        for: 3m
        labels:
          severity: critical
        annotations:
          summary: "Target {{ $labels.job }} is down"
          description: "{{ $labels.instance }} has been unreachable for more than 3 minutes"

      # HTTP errors alert
      - alert: HighErrorRate
        expr: |
          sum by (job) (rate(http_requests_total{status=~"5.."}[5m])) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate on {{ $labels.job }}"
          description: "HTTP 5xx errors are above 0.1 requests/second"

      # High latency alert
      - alert: HighLatency
        expr: |
          histogram_quantile(0.99, sum by (job, le) (rate(http_request_duration_seconds_bucket[5m]))) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency on {{ $labels.job }}"
          description: "P99 latency is above 2 seconds"

      # Certificate expiry alert
      - alert: TLSCertExpiringSoon
        expr: |
          probe_ssl_earliest_cert_expiry - time() < 86400 * 30
        for: 1h
        labels:
          severity: warning
        annotations:
          summary: "TLS certificate expiring soon"
          description: "Certificate for {{ $labels.instance }} expires in less than 30 days"

      # Pod crash looping
      - alert: PodCrashLooping
        expr: |
          increase(kube_pod_container_status_restarts_total[1h]) > 3
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Pod {{ $labels.namespace }}/{{ $labels.pod }} is crash looping"

      # Node not ready
      - alert: KubeNodeNotReady
        expr: |
          kube_node_status_condition{condition="Ready",status="true"} == 0
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "Node {{ $labels.node }} is not ready"

      # PersistentVolumeClaim pending
      - alert: PVCPending
        expr: |
          kube_persistentvolumeclaim_status_phase{phase="Pending"} == 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "PVC {{ $labels.namespace }}/{{ $labels.persistentvolumeclaim }} is pending"

      # Custom alerting rules
      - alert: ServiceAvailability
        expr: |
          sum by (service) (rate(http_requests_total{status!~"5.."}[5m]))
          /
          sum by (service) (rate(http_request_total[5m])) < 0.99
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Service {{ $labels.service }} availability below 99%"
          description: "Current availability: {{ $value | printf \"%.4f\" }}"
```

### Alertmanager Configuration
```yaml
# /etc/alertmanager/alertmanager.yml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.example.com:587'
  smtp_from: 'alertmanager@example.com'
  smtp_auth_username: 'alertmanager@example.com'
  smtp_auth_password: '${SMTP_PASSWORD}'
  slack_api_url: 'https://hooks.slack.com/services/xxx/xxx/xxx'
  pagerduty_url: 'https://events.pagerduty.com/v2/enqueue'
  opsgenie_api_url: 'https://api.opsgenie.com/v2/alerts'

# Template files
templates:
  - '/etc/alertmanager/template/*.tmpl'

# Route tree
route:
  group_by: ['alertname', 'severity', 'cluster']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'default-receiver'
  routes:
    # Send critical alerts to pager duty
    - match:
        severity: critical
      receiver: 'pagerduty'
      continue: true

    # Send warning alerts to Slack
    - match:
        severity: warning
      receiver: 'slack'

    # Send database alerts to DBA team
    - match:
        team: database
      receiver: 'dba-slack'

    # Send Kubernetes alerts to SRE team
    - match_re:
        namespace: 'kube-.*'
      receiver: 'sre-slack'
      routes:
        - match:
            severity: critical
          receiver: 'pagerduty-sre'

# Inhibition rules
inhibit_rules:
  # Suppress warnings if critical exists for same alert
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'cluster']

  # Suppress alerts for maintenance windows
  - source_match:
      maintenance: 'true'
    target_match_re:
      severity: '.*'
    equal: ['instance']

# Receivers
receivers:
  - name: 'default-receiver'
    email_configs:
      - to: 'alerts@example.com'
        send_resolved: true
        headers:
          Subject: "[{{ .Status }}] {{ .GroupLabels.Summary }}"
    slack_configs:
      - channel: '#alerts'
        send_resolved: true
        icon_emoji: ':bell:'
        title: "[{{ .Status | toUpper }}] {{ .GroupLabels.Summary }}"
        text: |
          {{ range .Alerts }}
          *Alert:* {{ .Annotations.summary }}
          *Description:* {{ .Annotations.description }}
          *Severity:* {{ .Labels.severity }}
          *Start:* {{ .StartsAt.Format "2006-01-02 15:04:05" }}
          {{ end }}
    webhook_configs:
      - url: 'http://webhook-server:5000/receive'
        send_resolved: true

  - name: 'slack'
    slack_configs:
      - channel: '#alerts-critical'
        send_resolved: true

  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: '${PAGERDUTY_SERVICE_KEY}'
        severity: critical
        component: 'prometheus'
        group: 'monitoring'
        class: 'deploy'
        details:
          key1: value1

  - name: 'dba-slack'
    slack_configs:
      - channel: '#dba-alerts'
        send_resolved: true

  - name: 'sre-slack'
    slack_configs:
      - channel: '#sre-alerts'
        send_resolved: true

  - name: 'pagerduty-sre'
    pagerduty_configs:
      - service_key: '${PAGERDUTY_SRE_KEY}'
        severity: critical

  - name: 'opsgenie'
    opsgenie_configs:
      - api_key: '${OPSGENIE_API_KEY}'
        message: "{{ .GroupLabels.Summary }}"
        description: "{{ .Annotations.description }}"
        priority: P1
        tags:
          - cluster={{ .Labels.cluster }}
          - env={{ .Labels.env }}
```

### Alertmanager Templates
```jinja2
# /etc/alertmanager/template/alert.tmpl
{{ define "email.subject" }}{{ if eq .Status "firing" }}[FIRING]{{ else }}[RESOLVED]{{ end }} {{ .GroupLabels.Summary }}{{ end }}

{{ define "email.body" }}
{{ if gt (len .Alerts.Firing) 0 }}
<u><b>Firing Alerts</b></u>
{{ range .Alerts.Firing }}
<b>{{ .Labels.alertname }}</b>
Severity: {{ .Labels.severity }}
Description: {{ .Annotations.description }}
Labels:
{{ range .Labels.SortedPairs }}  {{ .Name }}: {{ .Value }}
{{ end }}
{{ end }}
{{ end }}

{{ if gt (len .Alerts.Resolved) 0 }}
<u><b>Resolved Alerts</b></u>
{{ range .Alerts.Resolved }}
<b>{{ .Labels.alertname }}</b>
Severity: {{ .Labels.severity }}
Labels:
{{ range .Labels.SortedPairs }}  {{ .Name }}: {{ .Value }}
{{ end }}
Started: {{ .StartsAt.Format "2006-01-02 15:04:05" }}
Ended: {{ .EndsAt.Format "2006-01-02 15:04:05" }}
{{ end }}
{{ end }}
{{```

### Alert end }}
manager Commands
```bash
# Start Alertmanager
alertmanager --config.file=/etc/alertmanager/alertmanager.yml

# Check configuration
amtool check-config /etc/alertmanager/alertmanager.yml

# Test routing
amtool config routes test --receiver=slack

# View alerts
amtool alert

# Silence alerts
amtool silence add alertname=HighCPUUsage

# Query silences
amtool silence query

# Expire silence
amtool silence expire <silence-id>
```

---

## 10. Integration with Grafana

### Add Prometheus Data Source
```json
{
  "name": "Prometheus",
  "type": "prometheus",
  "url": "http://localhost:9090",
  "access": "proxy",
  "basicAuth": false,
  "jsonData": {
    "httpMethod": "GET",
    "manageAlerts": true,
    "prometheusVersion": "2.47.0",
    "prometheusType": "Prometheus"
  },
  "secureJsonData": {
    "basicAuthPassword": "password"
  }
}
```

### Example Grafana Dashboards

#### Node Exporter Dashboard
```json
{
  "dashboard": {
    "title": "Node Exporter Full",
    "panels": [
      {
        "title": "CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "100 - (avg by(instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
            "legendFormat": "{{ instance }}"
          }
        ],
        "yaxes": [
          { "format": "percent" }
        ]
      },
      {
        "title": "Memory Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100",
            "legendFormat": "{{ instance }}"
          }
        ]
      },
      {
        "title": "Disk Usage /",
        "type": "gauge",
        "targets": [
          {
            "expr": "(1 - (node_filesystem_avail_bytes{mountpoint=\"/\"} / node_filesystem_size_bytes{mountpoint=\"/\"})) * 100"
          }
        ]
      },
      {
        "title": "Network Traffic",
        "type": "timeseries",
        "targets": [
          {
            "expr": "rate(node_network_receive_bytes_total[5m])",
            "legendFormat": "{{ device }} - {{ instance }}"
          },
          {
            "expr": "rate(node_network_transmit_bytes_total[5m])",
            "legendFormat": "{{ device }} - {{ instance }} - tx"
          }
        ]
      },
      {
        "title": "Disk I/O",
        "type": "timeseries",
        "targets": [
          {
            "expr": "rate(node_disk_read_bytes_total[5m])",
            "legendFormat": "{{ device }} - read"
          },
          {
            "expr": "rate(node_disk_written_bytes_total[5m])",
            "legendFormat": "{{ device }} - written"
          }
        ]
      }
    ]
  }
}
```

### Useful Grafana Functions
```promql
# Rate over time
rate(http_requests_total[5m])

# Percentage calculation
(sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))) * 100

# Histogram quantile
histogram_quantile(0.95, sum by (le, service) (rate(http_request_duration_seconds_bucket[5m])))

# Top 10
topk(10, http_requests_total)

# Moving average
avg_over_time(http_requests_total[1h])

# Count over time
count_over_time(http_requests_total[1h])
```

---

## 11. Performance Tuning

### Prometheus Server Tuning
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Storage configuration
storage:
  tsdb:
    path: /var/lib/prometheus
    retention.time: 15d
    retention.size: 100GB
    max-block-duration: 2h
    min-block-duration: 2h
    no-lockfile: false

# Query configuration
query:
  max-concurrency: 10
  timeout: 2m
  max-samples: 50000000

# Remote write (for HA)
remote_write:
  - url: "https://remote-write:9201/write"
    tls_config:
      ca_file: /etc/prometheus/certs/ca.crt
      cert_file: /etc/prometheus/certs/cert.crt
      key_file: /etc/prometheus/certs/key.key
    queue_config:
      max_samples_per_send: 10000
      batch_send_deadline: 30s
      capacity: 50000
      max_shards: 30
```

### Rule Evaluation Optimization
```yaml
groups:
  - name: performance
    interval: 30s  # Longer interval for complex rules
    rules:
      # Use recording rules to pre-compute expensive queries
      - record: expensive:query:precomputed
        expr: |
          sum by (label) (complex_calculation)
```

### Scrape Configuration Optimization
```yaml
scrape_configs:
  - job_name: 'optimized'
    scrape_interval: 30s  # Longer scrape interval
    scrape_timeout: 20s
    metrics_path: /metrics
   honor_labels: true     # Respect existing labels
    metric_relabel_configs:
      # Drop unnecessary metrics early
      - source_labels: [__name__]
        regex: 'expensive_metric_to_drop'
        action: drop
```

### TSDB Tuning
```bash
# Command line options
--storage.tsdb.path=/var/lib/prometheus
--storage.tsdb.min-block-duration=2h
--storage.tsdb.max-block-duration=2h
--storage.tsdb.wal-compression
--storage.tsdb.allow-overlapping-blocks
--storage.tsdb.lockfile=
--storage.tsdb.max-chunk-bytes=1048576
```

### Query Optimization
```promql
# Use recording rules for expensive queries
expensive:query:precomputed

# Use efficient functions
rate(http_requests_total[5m])  # Good
increase(http_requests_total[1h]) / 12  # Better

# Limit time range
http_requests_total[5m]  # Good
http_requests_total[1h]  # Could be slow

# Use limit in subqueries
topk(10, http_requests_total)

# Avoid wildcards where possible
up{job="prometheus"}  # Better
up  # Could return many series
```

---

## 12. Troubleshooting

### Common Issues

#### Prometheus Won't Start
```bash
# Check configuration
promtool check config /etc/prometheus/prometheus.yml

# Check port availability
netstat -tulpn | grep 9090

# Check logs
tail -f /var/log/prometheus/prometheus.log

# Check permissions
ls -la /var/lib/prometheus/
```

#### No Metrics Scraped
```bash
# Check target status in web UI
# http://localhost:9090/targets

# Test connectivity
curl http://target:9100/metrics

# Check relabeling
# View applied labels in web UI

# Check firewall
iptables -L -n | grep 9100

# Check service discovery
curl http://localhost:9090/api/v1/service discovery
```

#### High Memory Usage
```bash
# Check current memory usage
curl http://localhost:9090/api/v1/status/tsdb

# Reduce retention
--storage.tsdb.retention.time=7d

# Check cardinality
curl http://localhost:9090/api/v1/query?query=count(up)

# Check high cardinality labels
curl http://localhost:9090/api/v1/label/values
```

#### Slow Queries
```bash
# Check query execution time
curl http://localhost:9090/api/v1/query?query=up&timeout=30s

# Use explain in web UI
# Click on query in Graph tab

# Optimize with recording rules
# Create pre-computed metrics
```

#### Alertmanager Issues
```bash
# Test Alertmanager configuration
amtool check-config /etc/alertmanager/alertmanager.yml

# Check Alertmanager logs
tail -f /var/log/alertmanager/alertmanager.log

# Test alert routing
amtool config routes test --receiver=default

# Check alert delivery
curl http://localhost:9093/api/v1/alerts
```

### Debugging Commands
```bash
# Check Prometheus status
curl http://localhost:9090/api/v1/status/config

# View runtime information
curl http://localhost:9090/api/v1/status/runtimeinfo

# View build information
curl http://localhost:9090/api/v1/status/buildinfo

# Query metadata
curl http://localhost:9090/api/v1/metadata

# Check TSDB stats
curl http://localhost:9090/api/v1/tsdb/status

# Query instant vector
curl 'http://localhost:9090/api/v1/query?query=up'

# Query range vector
curl 'http://localhost:9090/api/v1/query_range?query=up&start=2023-01-01T00:00:00Z&end=2023-01-01T01:00:00Z&step=1m'

# Label values
curl http://localhost:9090/api/v1/label/job/values

# Series API
curl 'http://localhost:9090/api/v1/series?match[]=up{job="prometheus"}'

# Delete series
curl -X POST 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]=up{job="test"}'
```

### Log Analysis
```bash
# Enable verbose logging
--log.level=debug

# View recent logs
tail -100 /var/log/prometheus/prometheus.log

# Search for errors
grep "ERROR" /var/log/prometheus/prometheus.log

# Search for warnings
grep "WARN" /var/log/prometheus/prometheus.log

# Search for scraping issues
grep "scrape" /var/log/prometheus/prometheus.log
```

---

## 13. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| Start Prometheus | `prometheus --config.file=prometheus.yml` |
| Check config | `promtool check config /etc/prometheus/prometheus.yml` |
| Check rules | `promtool check rules /etc/prometheus/rules/*.yml` |
| Query instant | `curl 'http://localhost:9090/api/v1/query?query=up'` |
| Query range | `curl 'http://localhost:9090/api/v1/query_range?query=up&start=...&step=...'` |
| View targets | `curl http://localhost:9090/api/v1/targets` |
| View rules | `curl http://localhost:9090/api/v1/rules` |
| View alerts | `curl http://localhost:9090/api/v1/alerts` |

### PromQL Functions Quick Reference
| Function | Description | Example |
|----------|-------------|---------|
| `rate()` | Per-second rate | `rate(http_requests_total[5m])` |
| `increase()` | Total increase | `increase(http_requests_total[1h])` |
| `irate()` | Instant rate | `irate(http_requests_total[5m])` |
| `sum()` | Sum values | `sum(rate(http_requests_total[5m]))` |
| `avg()` | Average | `avg(cpu_usage_seconds_total)` |
| `max()` | Maximum | `max(memory_usage_bytes)` |
| `min()` | Minimum | `min(temperature_celsius)` |
| `topk()` | Top k values | `topk(10, http_requests_total)` |
| `bottomk()` | Bottom k values | `bottomk(5, latency_seconds)` |
| `histogram_quantile()` | Histogram percentile | `histogram_quantile(0.95, ...)` |
| `count()` | Count series | `count(up)` |
| `label_replace()` | Replace label | `label_replace(up, "service", "$1", "job", "(.*)")` |
| `label_join()` | Join labels | `label_join(up, "combined", "-", "job", "instance")` |

### Metric Types
| Type | Description | Use Case |
|------|-------------|----------|
| **Counter** | Cumulative total | Request count, errors |
| **Gauge** | Instant value | Memory usage, temperature |
| **Histogram** | Distribution | Latency, response size |
| **Summary** | Quantiles | Custom quantiles |

### HTTP API Endpoints
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/query` | GET | Instant query |
| `/api/v1/query_range` | GET | Range query |
| `/api/v1/series` | GET | Series selection |
| `/api/v1/label/:name/values` | GET | Label values |
| `/api/v1/targets` | GET | Scrape targets |
| `/api/v1/rules` | GET | Recording/alert rules |
| `/api/v1/alerts` | GET | Active alerts |
| `/api/v1/status/config` | GET | Current config |
| `/api/v1/status/tsdb` | GET | TSDB stats |
| `/api/v1/admin/tsdb/delete_series` | POST | Delete series |

### Exit Codes
| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Usage error |
| 3 | Configuration error |

### Important Labels
| Label | Description |
|-------|-------------|
| `__name__` | Metric name |
| `__address__` | Target address |
| `__metrics_path__` | Metrics path |
| `__scheme__` | HTTP scheme |
| `job` | Job name |
| `instance` | Target instance |
| `env` | Environment |
| `cluster` | Cluster name |

### Common Exporter Ports
| Exporter | Port |
|----------|------|
| Node Exporter | 9100 |
| Prometheus | 9090 |
| Alertmanager | 9093 |
| Blackbox Exporter | 9115 |
| MySQL Exporter | 9104 |
| PostgreSQL Exporter | 9187 |
| Redis Exporter | 9121 |
| SNMP Exporter | 9116 |
| Kafka Exporter | 9308 |
| JMX Exporter | 8080 |
| GPU Exporter (DCGM) | 9400 |
| Cadvisor | 8080 |

---

*Last Updated: January 2026*
*Generated for Prometheus 2.47.x*
