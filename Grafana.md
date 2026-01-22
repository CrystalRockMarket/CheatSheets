# Grafana Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Configuration](#3-configuration)
4. [Data Sources](#4-data-sources)
5. [Dashboards](#5-dashboards)
6. [Panels](#6-panels)
7. [PromQL Queries in Grafana](#7-promql-queries-in-grafana)
8. [Alerting](#8-alerting)
9. [Users & Permissions](#9-users--permissions)
10. [Plugins](#10-plugins)
11. [API & Automation](#11-api--automation)
12. [Templating](#12-templating)
13. [Variables](#13-variables)
14. [Transformations](#14-transformations)
15. [Annotations](#15-annotations)
16. [Provisioning](#16-provisioning)
17. [Troubleshooting](#17-troubleshooting)
18. [Quick Reference](#18-quick-reference)

---

## 1. Introduction

### What is Grafana?
Grafana is an open-source platform for monitoring and observability, enabling you to query, visualize, alert on, and understand your metrics no matter where they are stored.

### Key Features
| Feature | Description |
|---------|-------------|
| **Visualizations** | Rich set of panel types (graphs, gauges, heatmaps, etc.) |
| **Dynamic Dashboards** | Create reusable, interactive dashboards |
| **Alerting** | Define alerts with thresholds and notifications |
| **Multi-tenant** | Organizations, teams, and folders |
| **Plugins** | Extend functionality with plugins |
| **Data Sources** | Connect to Prometheus, Elasticsearch, InfluxDB, and more |
| **API** | Full REST API for automation |
| **Auth** | Multiple authentication providers |

### Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                        Grafana Server                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   Frontend  │  │   Backend   │  │    Plugin System    │ │
│  │  (Angular/  │  │   (Go)      │  │                     │ │
│  │   React)    │  │             │  │  - Data Source      │ │
│  └─────────────┘  └─────────────┘  │  - Panel            │ │
│                                     │  - App              │ │
│  ┌─────────────────────────────────│─────────────────────┐ │
│  │              Database (SQLite/MySQL/PostgreSQL)        │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
          │                    │
          │ HTTP               │ Alerting
          ▼                    ▼
┌──────────────┐      ┌────────────────┐
│   Browsers   │      │  Alertmanager  │
│   (UI)       │      │  (External)    │
└──────────────┘      └────────────────┘
                              │
                              ▼
                      ┌──────────────┐
                      │  Notification │
                      │  Channels     │
                      │  (Email,Slack,│
                      │   PagerDuty)  │
                      └──────────────┘
```

### Core Concepts
| Concept | Description |
|---------|-------------|
| **Data Source** | Connection to a metrics/logs database |
| **Dashboard** | Collection of panels organized in rows |
| **Panel** | Individual visualization unit |
| **Row** | Horizontal container for panels |
| **Query** | Request for data from a data source |
| **Variable** | Dynamic value for dashboards |
| **Alert Rule** | Condition that triggers notifications |
| **Organization** | Multi-tenant isolation unit |

---

## 2. Installation

### Binary Installation
```bash
# Download Grafana
wget https://dl.grafana.com/oss/release/grafana-10.2.2.linux-amd64.tar.gz
tar -zxvf grafana-10.2.2.linux-amd64.tar.gz
cd grafana-10.2.2

# Run Grafana
./bin/grafana-server

# Install as service
sudo cp bin/grafana-server /usr/local/bin/
sudo useradd --system --no-create-home --shell /bin/false grafana
sudo mkdir -p /var/lib/grafana /var/log/grafana /etc/grafana
sudo chown -R grafana:grafana /var/lib/grafana /var/log/grafana
```

### Debian/Ubuntu Installation
```bash
# Add Grafana repository
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list

# Install Grafana
sudo apt-get update
sudo apt-get install grafana

# Start Grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

### RHEL/CentOS/Fedora Installation
```bash
# Add repository
cat <<EOF | sudo tee /etc/yum.repos.d/grafana.repo
[grafana]
name=Grafana
baseurl=https://packages.grafana.com/oss/rpm
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
EOF

# Install Grafana
sudo dnf install grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

### Docker Installation
```bash
# Run Grafana container
docker run -d \
  --name=grafana \
  -p 3000:3000 \
  -v grafana_data:/var/lib/grafana \
  grafana/grafana

# With custom configuration
docker run -d \
  --name=grafana \
  -p 3000:3000 \
  -v /path/to/grafana.ini:/etc/grafana/grafana.ini \
  -v /path/to/provisioning:/etc/grafana/provisioning \
  -v grafana_data:/var/lib/grafana \
  grafana/grafana

# Docker Compose
cat <<EOF > docker-compose.yml
version: '3'
services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=password
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana.ini:/etc/grafana/grafana.ini
      - ./provisioning:/etc/grafana/provisioning
    restart: unless-stopped

volumes:
  grafana_data:
EOF
```

### Kubernetes Installation
```bash
# Using Helm
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana \
  --set admin.password='admin' \
  --set persistence.enabled=true \
  --set persistence.size=10Gi

# Using manifest
kubectl apply -f https://raw.githubusercontent.com/grafana-operator/grafana-operator/main/deploy.yaml
```

### Verify Installation
```bash
# Check Grafana version
grafana-server -v

# Check service status
systemctl status grafana-server

# Access web interface
# http://localhost:3000
# Default credentials: admin / admin

# Check health
curl http://localhost:3000/api/health

# Check API info
curl http://localhost:3000/api/health
```

---

## 3. Configuration

### Main Configuration File
```ini
; /etc/grafana/grafana.ini

[paths]
data = /var/lib/grafana
logs = /var/log/grafana
plugins = /var/lib/grafana/plugins
provisioning = /etc/grafana/provisioning

[server]
http_port = 3000
domain = grafana.example.com
root_url = %(protocol)s://%(domain)s/
serve_from_sub_path = false

[database]
type = postgres
host = localhost:5432
name = grafana
user = grafana
password = password
ssl_mode = require

[security]
admin_user = admin
admin_password = admin
secret_key = your_secret_key
disable_gravatar = false
data_source_proxy_whitelist =

[users]
allow_sign_up = false
allow_org_create = false
auto_assign_org = true
auto_assign_org_role = Viewer

[auth]
login_cookie_name = grafana_session
login_maximum_inactive_lifetime = 7d
login_maximum_lifetime = 30d
token_rotation_interval_minutes = 10

[auth.basic]
enabled = true

[auth.ldap]
enabled = false
config_file = /etc/grafana/ldap.toml
allow_sign_up = true

[auth.github]
enabled = false
client_id = github_client_id
client_secret = github_client_secret
scopes = user:email
auth_url = https://github.com/login/oauth/authorize
token_url = https://github.com/login/oauth/access_token
api_url = https://api.github.com/user
allowed_domains = example.com

[auth.google]
enabled = false
client_id = google_client_id
client_secret = google_client_secret
scopes = https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email
auth_url = https://accounts.google.com/o/oauth2/v2/auth
token_url = https://oauth2.googleapis.com/token
api_url = https://www.googleapis.com/oauth2/v2/userinfo
allowed_domains = example.com

[smtp]
enabled = true
host = smtp.example.com:587
user = smtp_user
password = smtp_password
from_address = grafana@example.com
from_name = Grafana

[alerting]
enabled = true
execute_alerts = true
evaluation_timeout = 30s
notification_timeout = 30s
max_attempts = 3
min_interval = 10s

[log]
mode = console file
level = info
filters = category:sqlx:debug

[log.console]
level = info
format = console

[log.file]
level = info
log_format = text
max_days = 7

[metrics]
enabled = true
basic_auth_username = metrics_user
basic_auth_password = metrics_password

[metrics.graphite]
address = localhost:2003
prefix = prod.grafana.

[snapshots]
external_enabled = true
external_snapshot_url = https://snapshots.raintank.io
external_snapshot_name = Publish to snapshot.raintank.io

[tracing.jaeger]
enabled = false
address = localhost:6831
always_included_tag = server=grafana

[external_image_storage]
provider = s3

[external_image_storage.s3]
bucket = grafana-snapshots
region = us-west-2
path =
endpoint =
sigv4_auth = false
```

### Environment Variables
```bash
# Override config with environment variables
GF_SERVER_HTTP_PORT=3000
GF_SERVER_ROOT_URL=https://grafana.example.com
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=password
GF_USERS_ALLOW_SIGN_UP=false
GF_DATABASE_TYPE=postgres
GF_DATABASE_HOST=localhost:5432
GF_DATABASE_NAME=grafana
GF_DATABASE_USER=grafana
GF_DATABASE_PASSWORD=password
GF_LOG_MODE=console file
GF_LOG_LEVEL=info
GF_METRICS_ENABLED=true
GF_SMTP_ENABLED=true
GF_SMTP_HOST=smtp.example.com:587
```

---

## 4. Data Sources

### Prometheus
```json
{
  "name": "Prometheus",
  "type": "prometheus",
  "url": "http://localhost:9090",
  "access": "proxy",
  "basicAuth": false,
  "isDefault": true,
  "jsonData": {
    "httpMethod": "GET",
    "manageAlerts": true,
    "prometheusVersion": "2.47.0",
    "prometheusType": "Prometheus"
  },
  "secureJsonData": {
    "basicAuthPassword": ""
  }
}
```

### Elasticsearch
```json
{
  "name": "Elasticsearch",
  "type": "elasticsearch",
  "url": "http://localhost:9200",
  "access": "proxy",
  "jsonData": {
    "timeField": "@timestamp",
    "interval": "Daily",
    "logMessageField": "message",
    "logLevelField": "level",
    "maxConcurrentShardRequests": 5
  },
  "secureJsonData": {
    "basicAuthPassword": ""
  }
}
```

### InfluxDB
```json
{
  "name": "InfluxDB",
  "type": "influxdb",
  "url": "http://localhost:8086",
  "access": "proxy",
  "jsonData": {
    "organization": "my-org",
    "bucket": "my-bucket",
    "token": "my-token",
    "version": "Flux"
  },
  "secureJsonData": {
    "token": ""
  }
}
### MySQL
```json
{
  "name": "MySQL",
  "type": "mysql",
  "url": "localhost:3306",
  "access": "proxy",
  "jsonData": {
    "database": "metrics",
    "table": "metric",
    "timeColumn": "time_sec",
    "timeColumnType": "timestamp",
    "metricColumn": "name",
    "valueColumn": "value",
    "group": [],
    "filters": [],
    "orderByTime": "ASC"
  },
  "secureJsonData": {
    "password": ""
  }
}
```

### PostgreSQL
```json
{
  "name": "PostgreSQL",
  "type": "postgres",
  "url": "localhost:5432",
  "access": "proxy",
  "jsonData": {
    "database": "metrics",
    "schema": "public",
    "timeColumn": "time",
    "timeColumnType": "timestamp",
    "metricColumn": "name",
    "valueColumn": "value",
    "group": [],
    "filters": [],
    "orderByTime": "ASC"
  },
  "secureJsonData": {
    "password": ""
  }
}
```

### Loki (Logs)
```json
{
  "name": "Loki",
  "type": "loki",
  "url": "http://localhost:3100",
  "access": "proxy",
  "jsonData": {
    "maxLines": 1000,
    "derivedFields": [
      {
        "datasourceName": "Prometheus",
        "matchRegex": "trace_id=(\\w+)",
        "url": "http://localhost:9090/explore?left=%7B%22datasource%22%3A%22Prometheus%22%2C%22queries%22%3A%5B%7B%22expr%22%3A%22trace_id%3D%5C%22$1%5C%22%22%7D%5D%7D"
      }
    ]
  }
}
```

### Tempo (Traces)
```json
{
  "name": "Tempo",
  "type": "tempo",
  "url": "http://localhost:16686",
  "access": "proxy",
  "jsonData": {
    "tracesToLogs": {
      "datasourceName": "Loki",
      "tags": ["cluster", "hostname"],
      "mapTagNamesEnabled": false,
      "filterByTraceID": false,
      "filterBySpanID": false
    }
  }
}
```

### Graphite
```json
{
  "name": "Graphite",
  "type": "graphite",
  "url": "http://localhost:8080",
  "access": "proxy",
  "jsonData": {
    "graphiteVersion": "1.1",
    "proxySettings": {}
  }
}
```

### Azure Monitor
```json
{
  "name": "Azure Monitor",
  "type": "grafana-azure-monitor-datasource",
  "jsonData": {
    "cloudName": "Azure",
    "subscriptionId": "subscription-id",
    "tenantId": "tenant-id",
    "clientId": "client-id",
    "azureLogAnalytics": {
      "subscriptionId": "subscription-id",
      "clientId": "client-id",
      "tenantId": "tenant-id"
    },
    "azureMonitor": {
      "subscriptionId": "subscription-id",
      "clientId": "client-id",
      "tenantId": "tenant-id"
    }
  },
  "secureJsonData": {
    "clientSecret": ""
  }
}
```

### CloudWatch
```json
{
  "name": "CloudWatch",
  "type": "cloudwatch",
  "jsonData": {
    "authType": "Keys",
    "defaultRegion": "us-east-1",
    "customMetricsNamespaces": "CustomNamespace",
    "datasources": [],
    "logGroupNames": ["/aws/lambda/my-function"]
  },
  "secureJsonData": {
    "accessKey": "",
    "secretKey": ""
  }
}
```

### Data Source API
```bash
# List data sources
curl -u admin:password http://localhost:3000/api/datasources

# Get data source by ID
curl -u admin:password http://localhost:3000/api/datasources/1

# Get data source by name
curl -u admin:password http://localhost:3000/api/datasources/name/Prometheus

# Create data source
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{"name":"Prometheus","type":"prometheus","url":"http://localhost:9090","access":"proxy","isDefault":true}' \
  http://localhost:3000/api/datasources

# Update data source
curl -u admin:password -X PUT \
  -H "Content-Type: application/json" \
  -d '{"name":"Prometheus","type":"prometheus","url":"http://prometheus:9090"}' \
  http://localhost:3000/api/datasources/1

# Delete data source
curl -u admin:password -X DELETE http://localhost:3000/api/datasources/1

# Test data source
curl -u admin:password -X POST http://localhost:3000/api/datasources/proxy/1/api/v1/query?query=up
```

---

## 5. Dashboards

### Dashboard Structure
```json
{
  "annotations": {
    "list": [
      {
        "builtIn": 1,
        "datasource": {
          "type": "grafana",
          "uid": "-- Grafana --"
        },
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "fiscalYearStartMonth": 0,
  "graphTooltip": 0,
  "id": null,
  "links": [],
  "liveNow": false,
  "panels": [],
  "refresh": "5s",
  "schemaVersion": 38,
  "style": "dark",
  "tags": ["production", "kubernetes"],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-6h",
    "to": "now"
  },
  "timepicker": {},
  "timezone": "browser",
  "title": "Kubernetes Cluster Overview",
  "uid": "cluster-overview",
  "version": 1,
  "weekStart": ""
}
```

### Panel Structure
```json
{
  "id": 1,
  "title": "CPU Usage",
  "type": "timeseries",
  "datasource": {
    "type": "prometheus",
    "uid": "prometheus"
  },
  "gridPos": {
    "x": 0,
    "y": 0,
    "w": 12,
    "h": 8
  },
  "targets": [
    {
      "expr": "100 - (avg by(instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
      "legendFormat": "{{ instance }}"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "color": {
        "mode": "palette-classic"
      },
      "custom": {
        "axisCenteredZero": false,
        "axisColorMode": "text",
        "axisLabel": "",
        "axisPlacement": "auto",
        "barAlignment": 0,
        "drawStyle": "line",
        "fillOpacity": 10,
        "gradientMode": "none",
        "hideFrom": {
          "legend": false,
          "tooltip": false,
          "viz": false
        },
        "lineInterpolation": "linear",
        "lineWidth": 1,
        "pointSize": 5,
        "scaleDistribution": {
          "type": "linear"
        },
        "showPoints": "auto",
        "spanNulls": false,
        "stacking": {
          "group": "A",
          "mode": "none"
        },
        "thresholdsStyle": {
          "mode": "off"
        }
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": null
          }
        ]
      },
      "unit": "percent"
    },
    "overrides": []
  },
  "options": {
    "legend": {
      "calcs": ["mean", "max"],
      "displayMode": "table",
      "placement": "bottom",
      "showLegend": true
    },
    "tooltip": {
      "mode": "single",
      "sort": "none"
    }
  }
}
```

### Dashboard Import/Export
```bash
# Export dashboard via API
curl -u admin:password http://localhost:3000/api/dashboards/uid/cluster-overview > dashboard.json

# Import dashboard
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{"dashboard": <dashboard_json>, "overwrite": true, "folderId": 0}' \
  http://localhost:3000/api/dashboards/db

# Import from file
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d @dashboard.json \
  http://localhost:3000/api/dashboards/db

# Export all dashboards
for uid in $(curl -s -u admin:password http://localhost:3000/api/search?type=dash-db | jq -r '.[].uid'); do
  curl -s -u admin:password "http://localhost:3000/api/dashboards/uid/$uid" | jq '.dashboard' > "$uid.json"
done
```

### Dashboard Permissions
```bash
# Get dashboard permissions
curl -u admin:password http://localhost:3000/api/dashboards/uid/cluster-overview/permissions

# Update dashboard permissions
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '[
    {"userId": 1, "permission": 4},
    {"teamId": 1, "permission": 2},
    {"role": "Editor", "permission": 2}
  ]' \
  http://localhost:3000/api/dashboards/uid/cluster-overview/permissions
```

---

## 6. Panels

### Panel Types
| Type | Description | Best For |
|------|-------------|----------|
| **Time series** | Line/area charts | Time-based metrics |
| **Stat** | Single value display | KPIs, counters |
| **Gauge** | Radial gauge | Percentage metrics |
| **Bar gauge** | Horizontal bars | Progress metrics |
| **Table** | Tabular data | Log data, metadata |
| **Heatmap** | Color-coded matrix | Density, histograms |
| **Logs** | Log viewer | Log exploration |
| **Pie chart** | Circular chart | Proportions |
| **Graph** | Classic line graph | Legacy time series |
| **Dashboard list** | Dashboard links | Navigation |
| **Alert list** | Active alerts | Monitoring alerts |
| **Text** | Markdown/HTML | Documentation |
| **News** | RSS feed | External updates |
| **Traces** | Distributed traces | Trace visualization |
| **Flame graph** | CPU profiling | Performance analysis |

### Time Series Panel Options
```json
{
  "options": {
    "legend": {
      "calcs": ["mean", "min", "max", "sum"],
      "displayMode": "table",
      "placement": "right",
      "showLegend": true,
      "sortBy": "Max",
      "sortDesc": true
    },
    "tooltip": {
      "mode": "multi",
      "sort": "desc"
    },
    "thresholdsStyle": {
      "mode": "line"
    }
  },
  "fieldConfig": {
    "defaults": {
      "unit": "percent",
      "min": 0,
      "max": 100,
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "smooth",
        "fillOpacity": 20,
        "gradientMode": "opacity",
        "showPoints": "never"
      }
    }
  }
}
```

### Stat Panel Options
```json
{
  "options": {
    "colorMode": "value",
    "graphMode": "area",
    "justifyMode": "auto",
    "orientation": "horizontal",
    "reduceOptions": {
      "calcs": ["lastNotNull"],
      "fields": "",
      "values": false
    },
    "textMode": "auto",
    "wideLayout": true
  },
  "fieldConfig": {
    "defaults": {
      "color": {
        "mode": "thresholds"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {"color": "green", "value": null},
          {"color": "yellow", "value": 70},
          {"color": "red", "value": 90}
        ]
      },
      "unit": "percent"
    }
  }
}
```

### Gauge Panel Options
```json
{
  "options": {
    "min": 0,
    "max": 100,
    "showThresholdLabels": true,
    "showThresholdMarkers": true,
    "orientation": "auto"
  },
  "fieldConfig": {
    "defaults": {
      "color": {
        "mode": "thresholds"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {"color": "green", "value": null},
          {"color": "yellow", "value": 70},
          {"color": "red", "value": 90}
        ]
      }
    }
  }
}
```

### Table Panel Options
```json
{
  "options": {
    "showHeader": true,
    "frameIndex": 0,
    "enablePagination": false,
    "cellHeight": "sm",
    "footer": {
      "show": true,
      "reducer": ["sum"],
      "countRows": false,
      "fields": ""
    }
  },
  "fieldConfig": {
    "defaults": {
      "custom": {
        "align": "auto",
        "cellOptions": {
          "type": "auto"
        },
        "inspect": false
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {"color": "green", "value": null}
        ]
      }
    },
    "overrides": [
      {
        "matcher": {
          "id": "byName",
          "options": "Status"
        },
        "properties": [
          {
            "id": "custom.cellOptions",
            "value": {"type": "color-text"}
          },
          {
            "id": "mappings",
            "value": [
              {"type": "value", "options": {"0": {"color": "red", "text": "Down"}}},
              {"type": "value", "options": {"1": {"color": "green", "text": "Up"}}}
            ]
          }
        ]
      }
    ]
  }
}
```

### Heatmap Panel Options
```json
{
  "options": {
    "calculate": false,
    "calculation": {
      "xBuckets": {
        "value": ""
      },
      "yBuckets": {
        "value": ""
      }
    },
    "cellGap": 2,
    "cellRadius": null,
    "cellValues": {},
    "colors": ["rgba(33, 38, 255, 0.18)", "rgba(0, 221, 255, 0.55)", "rgba(255, 241, 0, 0.73)", "rgba(255, 0, 221, 0.8)"],
    "exemplars": {
      "color": "rgba(255,0,255,0.7)",
      "show": true
    },
    "filterValues": {
      "lane": 0,
      "show": true
    },
    "legend": {
      "show": true
    },
    "min": 0,
    "max": 100,
    "onZeroCell": false,
    "showColorScale": true,
    "xBucketNumber": 10,
    "xBucketSize": null,
    "yBucketNumber": null,
    "yBucketSize": null
  }
}
```

---

## 7. PromQL Queries in Grafana

### Basic Queries
```promql
# Simple metric
up

# With labels
up{job="prometheus"}

# Rate calculation
rate(http_requests_total[5m])

# Increase calculation
increase(http_requests_total[1h])

# With time range
rate(http_requests_total[5m])

# Histogram quantile
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Sum by labels
sum by (job) (rate(http_requests_total[5m]))

# Count series
count(up)
```

### Query Patterns

#### CPU Metrics
```promql
# CPU usage percentage
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Per-core CPU usage
100 - (rate(node_cpu_seconds_total{mode="idle"}[5m]) * 100)

# CPU by mode
sum by (mode) (rate(node_cpu_seconds_total[5m])) * 100
```

#### Memory Metrics
```promql
# Memory usage percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Memory by type
node_memory_MemTotal_bytes - node_memory_MemFree_bytes

# Container memory
container_memory_usage_bytes{image!=""}
```

#### Disk Metrics
```promql
# Disk usage percentage
(1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100

# Disk I/O read
rate(node_disk_read_bytes_total[5m])

# Disk I/O write
rate(node_disk_written_bytes_total[5m])

# Disk space
node_filesystem_avail_bytes{mountpoint="/"} / 1024 / 1024 / 1024
```

#### Network Metrics
```promql
# Network receive
rate(node_network_receive_bytes_total[5m])

# Network transmit
rate(node_network_transmit_bytes_total[5m])

# Network errors
rate(node_network_receive_errs_total[5m])
rate(node_network_transmit_errs_total[5m])
```

#### HTTP Metrics
```promql
# Request rate
sum by (status) (rate(http_requests_total[5m]))

# Error rate
sum by (job) (rate(http_requests_total{status=~"5.."}[5m]))

# Latency percentiles
histogram_quantile(0.50, sum by (job, le) (rate(http_request_duration_seconds_bucket[5m])))
histogram_quantile(0.95, sum by (job, le) (rate(http_request_duration_seconds_bucket[5m])))
histogram_quantile(0.99, sum by (job, le) (rate(http_request_duration_seconds_bucket[5m])))
```

#### Kubernetes Metrics
```promql
# Pod CPU usage
rate(container_cpu_usage_seconds_total{pod!=""}[5m])

# Pod memory
container_memory_working_set_bytes{pod!=""}

# Pod restarts
increase(kube_pod_container_status_restarts_total[1h])

# Node conditions
kube_node_status_condition{condition="Ready",status="true"}
```

### Transformation Queries
```promql
# Top 10 by value
topk(10, http_requests_total)

# Bottom 5
bottomk(5, http_requests_total)

# Group by
sum by (job) (rate(http_requests_total[5m]))

# Without
sum without (instance) (rate(http_requests_total[5m]))

# Aggregation
avg(rate(http_requests_total[5m]))
max(rate(http_requests_total[5m]))
min(rate(http_requests_total[5m]))
stddev(rate(http_requests_total[5m]))
```

---

## 8. Alerting

### Alert Rule Configuration
```json
{
  "id": 1,
  "uid": "alert-1",
  "orgId": 1,
  "title": "High CPU Usage",
  "condition": "C",
  "data": [
    {
      "refId": "A",
      "queryType": "",
      "relativeTimeRange": {
        "from": 300,
        "to": 0
      },
      "datasourceUid": "prometheus",
      "model": {
        "expr": "100 - (avg by(instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
        "refId": "A"
      }
    },
    {
      "refId": "B",
      "datasourceUid": "__expr__",
      "model": {
        "expr": "$A > 80",
        "type": "reduce",
        "reducer": "mean",
        "refId": "B"
      }
    },
    {
      "refId": "C",
      "datasourceUid": "__expr__",
      "model": {
        "expr": "$B > 0",
        "type": "threshold",
        "refId": "C"
      }
    }
  ],
  "folderUid": "folder-1",
  "for": "5m",
  "annotations": {
    "summary": "High CPU usage on {{ $labels.instance }}",
    "description": "CPU usage is above 80% (current value: {{ $value | printf \"%.2f\" }}%)"
  },
  "labels": {
    "severity": "warning"
  },
  "noDataState": "OK",
  "execErrState": "Error",
  "notification_settings": {
    "receiver": "default",
    "group_by": ["alertname", "instance"]
  }
}
```

### Notification Channels
```json
{
  "uid": "channel-1",
  "name": "Slack Alerts",
  "type": "slack",
  "orgId": 1,
  "isDefault": false,
  "sendReminder": true,
  "disableResolveMessage": false,
  "frequency": "5m",
  "settings": {
    "recipient": "#alerts",
    "username": "Grafana Alerts",
    "icon_emoji": ":bell:",
    "icon_url": "",
    "mentionUsers": "",
    "mentionGroups": "",
    "bot_username": "",
    "uploadImage": true,
    "httpHeader": ""
  },
  "secureFields": {
    "token": true
  }
}
```

### Alert Rules API
```bash
# List alert rules
curl -u admin:password http://localhost:3000/api/ruler/grafana/api/v1/rules

# Get rules for specific folder
curl -u admin:password http://localhost:3000/api/ruler/grafana/api/v1/rules/default

# Create alert rule
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "name": "High CPU Alert",
    "rules": [
      {
        "alert": "HighCPU",
        "expr": "100 - (avg by(instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100) > 80",
        "for": "5m",
        "labels": {"severity": "warning"},
        "annotations": {"summary": "High CPU usage"}
      }
    ]
  }' \
  http://localhost:3000/api/ruler/grafana/api/v1/rules/default

# Update alert rule
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d @alert-rule.json \
  http://localhost:3000/api/ruler/grafana/api/v1/rules/default

# Delete alert rule
curl -u admin:password -X DELETE \
  http://localhost:3000/api/ruler/grafana/api/v1/rules/default/HighCPU
```

### Notification Channel API
```bash
# List notification channels
curl -u admin:password http://localhost:3000/api/alert-notifications

# Create notification channel
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Slack Alerts",
    "type": "slack",
    "isDefault": false,
    "settings": {
      "recipient": "#alerts",
      "token": "xoxb-token"
    }
  }' \
  http://localhost:3000/api/alert-notifications

# Test notification channel
curl -u admin:password -X POST \
  http://localhost:3000/api/alert-notifications/test

# Delete notification channel
curl -u admin:password -X DELETE \
  http://localhost:3000/api/alert-notifications/1
```

---

## 9. Users & Permissions

### Organization Management
```bash
# Get current organization
curl -u admin:password http://localhost:3000/api/org

# Switch organization
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{"orgId": 2}' \
  http://localhost:3000/api/user/using/2

# Create organization
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{"name": "Production"}' \
  http://localhost:3000/api/orgs

# List organizations
curl -u admin:password http://localhost:3000/api/orgs
```

### User Management
```bash
# List users
curl -u admin:password http://localhost:3000/api/users

# Get user by ID
curl -u admin:password http://localhost:3000/api/users/1

# Get user by username
curl -u admin:password http://localhost:3000/api/users/lookup?loginOrEmail=admin

# Create user
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "login": "john",
    "password": "password123"
  }' \
  http://localhost:3000/api/admin/users

# Update user
curl -u admin:password -X PUT \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "name": "John Doe"
  }' \
  http://localhost:3000/api/users/2

# Change password
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{"userId": 2, "newPassword": "newpassword"}' \
  http://localhost:3000/api/admin/users/2/password

# Delete user
curl -u admin:password -X DELETE http://localhost:3000/api/admin/users/2
```

### Team Management
```bash
# List teams
curl -u admin:password http://localhost:3000/api/teams

# Create team
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{"name": "SRE Team", "email": "sre@example.com"}' \
  http://localhost:3000/api/teams

# Get team by ID
curl -u admin:password http://localhost:3000/api/teams/1

# Update team
curl -u admin:password -X PUT \
  -H "Content-Type: application/json" \
  -d '{"name": "Platform Team"}' \
  http://localhost:3000/api/teams/1

# Delete team
curl -u admin:password -X DELETE http://localhost:3000/api/teams/1

# List team members
curl -u admin:password http://localhost:3000/api/teams/1/members

# Add team member
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{"userId": 2}' \
  http://localhost:3000/api/teams/1/members

# Remove team member
curl -u admin:password -X DELETE http://localhost:3000/api/teams/1/members/2
```

### Permissions
```bash
# Get user permissions
curl -u admin:password http://localhost:3000/api/user/permissions

# Get folder permissions
curl -u admin:password http://localhost:3000/api/folders/folder-uid/permissions

# Update folder permissions
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '[
    {"userId": 1, "permission": 4},
    {"teamId": 1, "permission": 2}
  ]' \
  http://localhost:3000/api/folders/folder-uid/permissions
```

### Role-Based Access Control
```bash
# Assign role to user
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{"role": "Editor"}' \
  http://localhost:3000/api/access-control/users/2/roles

# List available roles
curl -u admin:password http://localhost:3000/api/access-control/roles

# Assign role to team
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{"role": "Viewer"}' \
  http://localhost:3000/api/access-control/teams/1/roles
```

---

## 10. Plugins

### Plugin Management
```bash
# List installed plugins
curl -u admin:password http://localhost:3000/api/health

# Install plugin
grafana-cli plugins install <plugin-id>

# Update plugin
grafana-cli plugins update <plugin-id>

# Remove plugin
grafana-cli plugins remove <plugin-id>

# List available plugins
grafana-cli plugins list-remote

# Install specific version
grafana-cli plugins install <plugin-id>@<version>
```

### Common Plugins
| Plugin ID | Name | Description |
|-----------|------|-------------|
| `grafana-clock-panel` | Clock | Display current time |
| `grafana-polystat-panel` | Polystat | Status panel |
| `grafana-worldmap-panel` | World Map | Geospatial data |
| `grafana-piechart-panel` | Pie Chart | Circular charts |
| `briangann-datatable-panel` | Data Table | Advanced tables |
| `aidanmountford-html-panel` | HTML | Custom HTML |
| `marcuscalidus-svg-panel` | SVG | SVG graphics |
| `vonage-status-panel` | Status | Status indicators |
| `yesoreyeram-infinity-datasource` | Infinity | JSON/SOAP API |
| `grafana-simple-json-datasource` | Simple JSON | Custom backends |
| `lukas65-annotation-helper` | Annotation Helper | Annotations management |

---

## 11. API & Automation

### Authentication
```bash
# Basic auth
curl -u admin:password http://localhost:3000/api/dashboards

# API token
curl -H "Authorization: Bearer eyJrIjoi..." http://localhost:3000/api/dashboards

# Generate API token
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name": "API Token", "role": "Admin"}' \
  http://localhost:3000/api/auth/keys
```

### Common API Endpoints

#### Dashboards
```bash
# Search dashboards
curl -u admin:password "http://localhost:3000/api/search?type=dash-db&folderIds=0"

# Get dashboard by UID
curl -u admin:password http://localhost:3000/api/dashboards/uid/cluster-overview

# Get dashboard by slug
curl -u admin:password http://localhost:3000/api/dashboards/db/cluster-overview

# Create/Update dashboard
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{"dashboard": {<dashboard_json>}, "overwrite": true}' \
  http://localhost:3000/api/dashboards/db

# Delete dashboard
curl -u admin:password -X DELETE http://localhost:3000/api/dashboards/uid/cluster-overview

# Get home dashboard
curl -u admin:password http://localhost:3000/api/dashboards/home
```

#### Folders
```bash
# List folders
curl -u admin:password http://localhost:3000/api/folders

# Create folder
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{"title": "Production", "uid": "prod-folder"}' \
  http://localhost:3000/api/folders

# Update folder
curl -u admin:password -X PUT \
  -H "Content-Type: application/json" \
  -d '{"title": "Production Dashboards"}' \
  http://localhost:3000/api/folders/prod-folder

# Delete folder
curl -u admin:password -X DELETE http://localhost:3000/api/folders/prod-folder
```

#### Explore
```bash
# Query data source
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "datasource": {"uid": "prometheus"},
    "queries": [{"expr": "up", "refId": "A"}],
    "range": {"from": "now-1h", "to": "now"}
  }' \
  http://localhost:3000/api/datasources/proxy/1/api/v1/query_range
```

### Automation Examples

#### Provision Dashboards with curl
```bash
#!/bin/bash
# deploy-dashboards.sh

GRAFANA_URL="http://localhost:3000"
USER="admin"
PASSWORD="password"
FOLDER="production"

# Create folder if not exists
FOLDER_ID=$(curl -s -u $USER:$PASSWORD \
  "$GRAFANA_URL/api/folders" | jq -r ".[] | select(.title==\"$FOLDER\").id")

if [ -z "$FOLDER_ID" ]; then
  FOLDER_ID=$(curl -s -u $USER:$PASSWORD \
    -X POST \
    -H "Content-Type: application/json" \
    -d "{\"title\": \"$FOLDER\"}" \
    "$GRAFANA_URL/api/folders" | jq -r ".id")
fi

# Deploy dashboards
for dashboard in dashboards/*.json; do
  if [ -f "$dashboard" ]; then
    echo "Deploying $dashboard..."
    curl -s -u $USER:$PASSWORD \
      -X POST \
      -H "Content-Type: application/json" \
      -d "{\"dashboard\": $(cat $dashboard), \"folderId\": $FOLDER_ID, \"overwrite\": true}" \
      "$GRAFANA_URL/api/dashboards/db"
  fi
done
```

#### Python Grafana Client
```python
import requests
from requests.auth import HTTPBasicAuth

class GrafanaClient:
    def __init__(self, base_url, username, password):
        self.base_url = base_url
        self.auth = HTTPBasicAuth(username, password)
        self.headers = {"Content-Type": "application/json"}
    
    def get_dashboards(self):
        response = requests.get(
            f"{self.base_url}/api/search?type=dash-db",
            auth=self.auth
        )
        return response.json()
    
    def get_dashboard(self, uid):
        response = requests.get(
            f"{self.base_url}/api/dashboards/uid/{uid}",
            auth=self.auth
        )
        return response.json()
    
    def create_dashboard(self, dashboard, folder_id=0):
        payload = {
            "dashboard": dashboard,
            "folderId": folder_id,
            "overwrite": True
        }
        response = requests.post(
            f"{self.base_url}/api/dashboards/db",
            auth=self.auth,
            json=payload
        )
        return response.json()
    
    def delete_dashboard(self, uid):
        response = requests.delete(
            f"{self.base_url}/api/dashboards/uid/{uid}",
            auth=self.auth
        )
        return response.status_code == 200
    
    def get_datasource(self, name):
        response = requests.get(
            f"{self.base_url}/api/datasources/name/{name}",
            auth=self.auth
        )
        return response.json()
    
    def create_datasource(self, datasource):
        response = requests.post(
            f"{self.base_url}/api/datasources",
            auth=self.auth,
            json=datasource
        )
        return response.json()
    
    def query_prometheus(self, datasource_name, query):
        ds = self.get_datasource(datasource_name)
        response = requests.get(
            f"{self.base_url}/api/datasources/proxy/{ds['id']}/api/v1/query",
            params={"query": query},
            auth=self.auth
        )
        return response.json()
    
    def create_alert_rule(self, rule, folder_uid):
        response = requests.post(
            f"{self.base_url}/api/ruler/grafana/api/v1/rules/{folder_uid}",
            auth=self.auth,
            json=rule
        )
        return response.json()
    
    def get_annotations(self, params=None):
        response = requests.get(
            f"{self.base_url}/api/annotations",
            params=params,
            auth=self.auth
        )
        return response.json()
    
    def create_annotation(self, dashboard_id, text, time=None):
        payload = {
            "dashboardId": dashboard_id,
            "text": text,
            "time": time
        }
        response = requests.post(
            f"{self.base_url}/api/annotations",
            auth=self.auth,
            json=payload
        )
        return response.json()

# Usage
client = GrafanaClient(
    base_url="http://localhost:3000",
    username="admin",
    password="password"
)

# List dashboards
dashboards = client.get_dashboards()
for d in dashboards:
    print(f"{d['title']} ({d['uid']})")
```

---

## 12. Templating

### Variable Types
| Type | Description | Example |
|------|-------------|---------|
| **Query** | Data source query | `label_values(up, job)` |
| **Interval** | Time interval values | `5m,10m,30m,1h` |
| **Custom** | Manual values | `prod,stage,dev` |
| **Constant** | Hidden constant | `100` |
| **Data source** | Data source selector | Prometheus |
| **Ad hoc filters** | Auto-generated filters | `env=$env` |

### Query Variables
```json
{
  "name": "job",
  "type": "query",
  "datasource": {"uid": "prometheus"},
  "refresh": 2,
  "regex": "",
  "sort": 1,
  "definition": "label_values(up, job)",
  "multi": false,
  "includeAll": false,
  "query": "label_values(up, job)"
}
```

### Interval Variables
```json
{
  "name": "interval",
  "type": "interval",
  "options": [
    {"text": "5m", "value": "5m"},
    {"text": "10m", "value": "10m"},
    {"text": "30m", "value": "30m"},
    {"text": "1h", "value": "1h"},
    {"text": "6h", "value": "6h"},
    {"text": "12h", "value": "12h"},
    {"text": "1d", "value": "1d"}
  ],
  "auto": true,
  "auto_min": "1s",
  "auto_count": 200
}
```

### Custom Variables
```json
{
  "name": "env",
  "type": "custom",
  "options": [
    {"text": "production", "value": "production"},
    {"text": "staging", "value": "staging"},
    {"text": "development", "value": "development"}
  ],
  "multi": true,
  "includeAll": false
}
```

### Using Variables in Queries
```promql
# Use variable in metric
up{job="$job"}

# Use variable in rate query
rate(http_requests_total{env="$env"}[$interval])

# Use multi-value variable
sum by (job) (rate(http_requests_total{env=~"$env"}[5m]))

# Use all option
sum by (job) (rate(http_requests_total{env=~"$env|all"}[5m]))

# Use variable in label
label_replace(up{job="$job"}, "instance", "$1:9100", "job", "(.*)")

# Chained variables
# Parent: $cluster
# Child: $namespace
label_values(kube_pod_info{cluster="$cluster"}, namespace)

# Nested variable
label_values(kube_pod_info{cluster="$cluster", namespace="$namespace"}, pod)
```

### Dependent Variables
```json
{
  "name": "namespace",
  "type": "query",
  "datasource": {"uid": "prometheus"},
  "refresh": 2,
  "definition": "label_values(kube_pod_info{cluster=\"$cluster\"}, namespace)",
  "query": "label_values(kube_pod_info{cluster=\"$cluster\"}, namespace)",
  "dependsOn": ["cluster"]
}
```

### Global Variables
| Variable | Description | Example |
|----------|-------------|---------|
| `$__dashboard` | Current dashboard name | `My Dashboard` |
| `$__panel` | Current panel title | `CPU Usage` |
| `$__from` | Time range from | `1672531200000` |
| `$__to` | Time range to | `1672617600000` |
| `$__timeFilter` | Time filter clause | `time >= 1672531200000 and time <= 1672617600000` |
| `$__interval` | Auto interval | `1m` |
| `$__interval_ms` | Interval in ms | `60000` |
| `$__user.id` | Current user ID | `1` |

---

## 13. Variables

### Variable Syntax
```promql
# Single value
$variable_name

# Multiple values (comma-separated)
${variable_name}

# Regex matching
${variable_name:regex}

# Escaped
\$$

# All values with wildcard
$variable_name:*
```

### Advanced Variable Usage
```promql
# Variable in legend
rate(http_requests_total{env=~"$env"}[5m]) by (env)

# With legend format
rate(http_requests_total{env=~"$env"}[5m]) by (env)

# Using template variable in template
{{ $labels.$variable_name }}

# Nested variable expansion
${var1}-${var2}

# Conditional variable
${var1:-default_value}

# Raw variable
[[variable_name]]
```

### Variable Filters
```json
{
  "name": "region",
  "type": "query",
  "datasource": {"uid": "prometheus"},
  "query": "label_values(node_uname_info, region)",
  "refresh": 2,
  "regex": "^us-.*"
}
```

### Global Date/Time Variables
```json
{
  "name": "time_range",
  "type": "interval",
  "options": [
    {"text": "Last 15 minutes", "value": "15m"},
    {"text": "Last 1 hour", "value": "1h"},
    {"text": "Last 6 hours", "value": "6h"},
    {"text": "Last 24 hours", "value": "24h"}
  ]
}
```

---

## 14. Transformations

### Transformation Types
| Transformation | Description |
|---------------|-------------|
| **Organize Fields** | Rename, hide, reorder fields |
| **Merge** | Merge multiple queries |
| **Labels to Fields** | Convert labels to fields |
| **Time series to wide** | Transform to wide format |
| **Labels as fields** | Keep labels as fields |
| **Reduce** | Aggregate values |
| **Group by** | Group and aggregate |
| **Sort by** | Sort by field |
| **Filter by name** | Filter fields |
| **Filter by value** | Filter by field value |
| **Calculate field** | Create new field |
| **Config from query** | Extract config from query |
| **Series to rows** | Transform to rows |
| **Pivot** | Pivot rows to columns |
| **Join by field** | Join by field value |
| **Outer join** | Outer join series |
| **Add field from calculation** | Calculate new field |
| **Histogram** | Create histogram |
| **Convert field type** | Convert field type |
| **Sort by** | Sort by field |
```

### Transformation Configuration
```json
{
  "id": "organizeFields",
  "options": {
    "fields": {
      "Time": {
        "index": 0,
        "show": true
      },
      "Value #A": {
        "index": 1,
        "show": true
      },
      "instance": {
        "index": 2,
        "show": false
      },
      "job": {
        "index": 3,
        "show": true
      }
    }
  }
}
```

```json
{
  "id": "groupBy",
  "options": {
    "fields": {
      "Value": {
        "aggregations": ["mean", "max", "min"]
      }
    },
    "operation": "groupby",
    "groupByFields": ["instance"]
  }
}
```

```json
{
  "id": "calculateField",
  "options": {
    "mode": "binary",
    "binary": {
      "operator": "*",
      "right": "100"
    },
    "alias": "percentage",
    "replaceFields": false
  }
}
```

```json
{
  "id": "filterByValue",
  "options": {
    "match": "gte",
    "field": "Value",
    "value": "80"
  }
}
```

```json
{
  "id": "sortBy",
  "options": {
    "sort": [
      {"field": "Value", "desc": true}
    ]
  }
}
```

---

## 15. Annotations

### Annotation Types
| Type | Description |
|------|-------------|
| **Dashboard** | Annotation tied to dashboard |
| **API** | Manually created via API |
| **Alert** | Alert state changes |
| **Elasticsearch** | From Elasticsearch queries |
| **Legacy** | Deprecated annotation format |

### Create Annotation via API
```bash
# Create annotation
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "dashboardId": 1,
    "panelId": 1,
    "time": 1672531200000,
    "timeEnd": 1672534800000,
    "text": "Deployment started",
    "tags": ["deployment", "production"]
  }' \
  http://localhost:3000/api/annotations

# Create graphite annotation
curl -u admin:password -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "what": "Event name",
    "when": 1672531200000,
    "data": "Event description",
    "tags": ["event"]
  }' \
  http://localhost:3000/api/annotations/graphite

# List annotations
curl -u admin:password "http://localhost:3000/api/annotations?dashboardId=1&from=1672531200000&to=1672617600000"

# Update annotation
curl -u admin:password -X PUT \
  -H "Content-Type: application/json" \
  -d '{"text": "Updated annotation"}' \
  http://localhost:3000/api/annotations/1

# Delete annotation
curl -u admin:password -X DELETE http://localhost:3000/api/annotations/1

# Delete annotations by dashboard
curl -u admin:password -X DELETE "http://localhost:3000/api/annotations?dashboardId=1"
```

### Annotation Query in Dashboard
```json
{
  "name": "Annotations",
  "enable": true,
  "datasource": {
    "type": "elasticsearch",
    "uid": "elasticsearch"
  },
  "expr": "",
  "refreshOnTimeRangeUnchanged": true,
  "drillDown": false,
  "bucketAggs": [
    {
      "id": "2",
      "type": "date_histogram",
      "field": "@timestamp",
      "settings": {
        "interval": "auto"
      }
    }
  ],
  "metrics": [
    {
      "id": "1",
      "type": "count",
      "settings": {}
    }
  ],
  "query": "annotations.event"
}
```

---

## 16. Provisioning

### File Structure
```
/etc/grafana/
├── provisioning/
│   ├── dashboards/
│   │   ├── all.yml              # Dashboard provisioner config
│   │   ├── default/
│   │   │   └── dashboard.yaml   # Dashboard files
│   │   └── custom/
│   │       └── dashboard.yaml
│   ├── datasources/
│   │   └── all.yml              # Datasource provisioner config
│   │       └── datasources.yaml # Datasource files
│   └── alerting/
│       └── all.yml              # Alerting provisioner config
│           └── notification_channels.yaml
└── grafana.ini                  # Main configuration
```

### Dashboard Provisioning
```yaml
# /etc/grafana/provisioning/dashboards/all.yml
apiVersion: 1

providers:
  - name: 'Default'
    orgId: 1
    folder: ''
    folderUid: ''
    type: file
    options:
      path: /var/lib/grafana/dashboards
      foldersFromFilesStructure: true

  - name: 'Custom'
    orgId: 1
    folder: 'Production'
    folderUid: prod-folder
    type: file
    disableDeletion: false
    editable: true
    options:
      path: /etc/grafana/provisioning/dashboards/custom
      widgetsFromFilesStructure: true
```

### Datasource Provisioning
```yaml
# /etc/grafana/provisioning/datasources/all.yml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    uid: prometheus
    access: proxy
    url: http://localhost:9090
    isDefault: true
    jsonData:
      httpMethod: GET
      manageAlerts: true
      prometheusVersion: 2.47.0
      prometheusType: Prometheus

  - name: Loki
    type: loki
    uid: loki
    access: proxy
    url: http://localhost:3100
    jsonData:
      maxLines: 1000

  - name: Elasticsearch
    type: elasticsearch
    uid: elasticsearch
    access: proxy
    url: http://localhost:9200
    jsonData:
      timeField: "@timestamp"
      interval: Daily
    secureJsonData:
      password: password

  - name: CloudWatch
    type: cloudwatch
    jsonData:
      authType: default
      defaultRegion: us-east-1
    secureJsonData:
      accessKey: access_key
      secretKey: secret_key

deleteDatasources:
  - name: Old Datasource
    orgId: 1
```

### Alerting Provisioning
```yaml
# /etc/grafana/provisioning/alerting/all.yml
apiVersion: 1

notifiers:
  - name: Slack Alerts
    type: slack
    uid: slack-alerts
    orgId: 1
    isDefault: false
    sendReminder: true
    frequency: 5m
    disableResolveMessage: false
    settings:
      recipient: "#alerts"
      username: Grafana Alerts
      uploadImage: true
    secureSettings:
      token: xoxb-token

  - name: PagerDuty
    type: pagerduty
    uid: pagerduty-alerts
    orgId: 1
    isDefault: false
    settings:
      integrationKey: integration_key
      severity: critical

rules:
  - orgId: 1
    folder: Production
    name: High CPU Alert
    rules:
      - uid: high_cpu_alert
        title: High CPU Usage
        condition: C
        data:
          - refId: A
            relativeTimeRange:
              from: 300
              to: 0
            datasourceUid: prometheus
            model:
              expr: '100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)'
              refId: A
          - refId: C
            datasourceUid: __expr__
            model:
              expr: '$A > 80'
              type: threshold
        for: 5m
        annotations:
          summary: High CPU usage on {{ $labels.instance }}
        labels:
          severity: warning
```

### Reload Configuration
```bash
# Trigger dashboard reload
curl -u admin:password -X POST http://localhost:3000/api/dashboards/reload

# Trigger datasource reload
curl -u admin:password -X POST http://localhost:3000/api/datasources/reload

# Trigger alerting reload
curl -u admin:password -X POST http://localhost:3000/api/admin/provisioning/alerting/reload
```

---

## 17. Troubleshooting

### Common Issues

#### Dashboard Not Loading
```bash
# Check browser console for errors
# F12 -> Console

# Check Grafana server logs
tail -f /var/log/grafana/grafana.log | grep ERROR

# Verify datasource is configured
curl -u admin:password http://localhost:3000/api/datasources

# Test datasource connection
curl -u admin:password -X POST \
  http://localhost:3000/api/datasources/proxy/1/api/v1/query?query=up

# Check browser network tab
# Look for failed requests to /api/dashboards/uid/...
```

#### Slow Performance
```bash
# Increase logging level temporarily
curl -u admin:password -X PUT \
  -H "Content-Type: application/json" \
  -d '{"loglevel": "debug"}' \
  http://localhost:3000/api/admin/settings

# Check query performance
# Use Grafana's Query Inspector (i icon on panel)

# Optimize PromQL queries
# Use recording rules for expensive queries

# Reduce dashboard refresh rate
# Change "refresh" from "5s" to "30s" or "1m"

# Check Grafana server resources
top
htop
```

#### Authentication Issues
```bash
# Reset admin password
grafana-cli admin reset-admin-password <new_password>

# Check LDAP configuration
cat /etc/grafana/ldap.toml

# Test LDAP connection
curl -u admin:password \
  http://localhost:3000/api/admin/ldap/test

# Check auth settings
curl -u admin:password \
  http://localhost:3000/api/health | jq '.auth'
```

#### Data Source Errors
```bash
# Check data source configuration
curl -u admin:password \
  http://localhost:3000/api/datasources/name/Prometheus

# Test connection
curl -u admin:password -X POST \
  http://localhost:3000/api/datasources/proxy/1/api/v1/query?query=up

# Check TLS/SSL settings
# Verify certificates are valid

# Check network connectivity
curl -v http://localhost:9090/metrics

# View data source plugin logs
tail -f /var/log/grafana/grafana.log | grep datasource
```

#### Alerting Not Working
```bash
# Check alert rules
curl -u admin:password \
  http://localhost:3000/api/ruler/grafana/api/v1/rules

# Check notification channels
curl -u admin:password \
  http://localhost:3000/api/alert-notifications

# Test notification channel
curl -u admin:password -X POST \
  http://localhost:3000/api/alert-notifications/test

# Check Alertmanager configuration
# Verify Alertmanager is reachable

# Check alert evaluation logs
tail -f /var/log/grafana/grafana.log | grep -i alert
```

### Debugging Commands
```bash
# Check Grafana version
curl http://localhost:3000/api/health | jq '.version'

# Check database connectivity
curl -u admin:password \
  http://localhost:3000/api/health | jq '.database'

# View all settings
curl -u admin:password \
  http://localhost:3000/api/admin/settings

# Check license status
curl -u admin:password \
  http://localhost:3000/api/licensing/check

# View built-in plugins
curl -u admin:password \
  http://localhost:3000/api/health | jq '.plugins'

# Check live streaming
curl -u admin:password \
  http://localhost:3000/api/live/ping
```

### Log Analysis
```bash
# Enable debug logging
export GF_LOG_LEVEL=debug

# View logs with filtering
tail -f /var/log/grafana/grafana.log | grep -E "(ERROR|WARN)"

# Search for specific query
tail -f /var/log/grafana/grafana.log | grep "query"

# Search for data source errors
tail -f /var/log/grafana/grafana.log | grep -i datasource

# Search for rendering issues
tail -f /var/log/grafana/grafana.log | grep -i render
```

---

## 18. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| Start Grafana | `systemctl start grafana-server` |
| Check version | `grafana-server -v` |
| Reset admin password | `grafana-cli admin reset-admin-password` |
| Install plugin | `grafana-cli plugins install <plugin>` |
| Update plugins | `grafana-cli plugins update-all` |
| Reload dashboards | `curl -X POST http://localhost:3000/api/dashboards/reload` |
| Check health | `curl http://localhost:3000/api/health` |

### API Endpoints Quick Reference
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/health` | GET | Health check |
| `/api/dashboards` | GET/POST | List/Create dashboards |
| `/api/dashboards/uid/:uid` | GET/PUT/DELETE | Dashboard by UID |
| `/api/datasources` | GET/POST | List/Create data sources |
| `/api/search` | GET | Search dashboards |
| `/api/folders` | GET/POST | List/Create folders |
| `/api/ruler/grafana/api/v1/rules` | GET/POST | Alert rules |
| `/api/alert-notifications` | GET/POST | Notification channels |
| `/api/user` | GET | Current user |
| `/api/admin/users` | GET/POST | User management |
| `/api/admin/settings` | GET | All settings |

### Grafana URLs
| URL | Description |
|-----|-------------|
| `http://localhost:3000` | Main UI |
| `http://localhost:3000/explore` | Explore mode |
| `http://localhost:3000/alerting` | Alerting |
| `http://localhost:3000/dashboards` | Dashboards list |
| `http://localhost:3000/alerting/list` | Alert list |
| `http://localhost:3000/admin` | Admin settings |

### Variable Syntax
| Syntax | Description |
|--------|-------------|
| `$var` | Single value |
| `${var}` | Single value |
| `$var:regex` | Regex filtered |
| `[[var]]` | Alternative syntax |
| `$__dashboard` | Dashboard name |
| `$__from` | Time from (Unix ms) |
| `$__to` | Time to (Unix ms) |
| `$__interval` | Auto interval |
| `$__timeFilter` | Time filter |

### Panel Types Quick Reference
| Type | Key | Best For |
|------|-----|----------|
| Time Series | `timeseries` | Time-based metrics |
| Stat | `stat` | Single values |
| Gauge | `gauge` | Percentage display |
| Table | `table` | Tabular data |
| Logs | `logs` | Log exploration |
| Heatmap | `heatmap` | Density visualization |
| Bar Gauge | `bargauge` | Progress display |
| Pie Chart | `piechart` | Proportions |
| Dashboard List | `dashlist` | Navigation |
| Alert List | `alertlist` | Alert monitoring |
| Text | `text` | Markdown/HTML |

### Color Schemes
| Scheme | Description |
|--------|-------------|
| `palette-classic` | Default multi-color |
| `palette-classic-by-name` | By value name |
| `continuous-GrYlRd` | Green-Yellow-Red |
| `continuous-BlPu` | Blue-Purple |
| `continuous-YlBl` | Yellow-Blue |
| `continuous-Blues` | Blue scale |
`continuous-Reds` | Red scale |
| `fixed` | Single color |
| `shades` | Gray shades |

### Alert States
| State | Description |
|-------|-------------|
| **OK** | Alert condition not met |
| **Alerting** | Alert condition met |
| **Pending** | Alert condition met, awaiting duration |
| **No Data** | No data available |
| **Error** | Query error |

---

*Last Updated: January 2026*
*Generated for Grafana 10.2.x*
