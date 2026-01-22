# netdata Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Configuration](#3-configuration)
4. [Dashboard](#4-dashboard)
5. [Collectors](#5-collectors)
6. [Alerts](#6-alerts)
7. [Health Configuration](#7-health-configuration)
8. [Streaming](#8-streaming)
9. [Plugins](#9-plugins)
10. [API](#10-api)
11. [Troubleshooting](#11-troubleshooting)
12. [Quick Reference](#12-quick-reference)

---

## 1. Introduction

### What is netdata?
netdata is a distributed, real-time health and performance monitoring system for Linux, FreeBSD, and macOS systems, known for its beautiful, low-latency dashboard and comprehensive metrics.

### Key Features
| Feature | Description |
|---------|-------------|
| **Real-time Dashboard** | Millisecond latency, beautiful visualizations |
| **Lightweight** | 1-3% CPU, minimal memory |
| **No Dependencies** | Self-contained binary |
| **Plugins** | Extensible plugin architecture |
| **Alerting** | Built-in health monitoring |
| **Streaming** | Centralized metrics collection |
| **Long-term Storage** | Optional database tier |
| **API** | RESTful API for integrations |

### Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                        netdata Agent                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │  Collectors │  │   Health    │  │   Web Server           │ │
│  │  (C, Python)│  │  Engine     │  │   (Embedded Go server) │ │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘ │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │              Internal Database (RRD)                     │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
          │                    │
          │ Stream              │ HTTP
          ▼                    ▼
┌──────────────┐      ┌────────────────────┐
│  netdata     │      │   Web Dashboard    │
│  Parents     │      │   (Browser)        │
└──────────────┘      └────────────────────┘
```

---

## 2. Installation

### One-Line Installation
```bash
# Automatic installation
bash <(curl -Ss https://my-netdata.io/kickstart.sh)

# With options
bash <(curl -Ss https://my-netdata.io/kickstart.sh) --no-updates --stable-channel
```

### Docker
```bash
# Run netdata container
docker run -d \
  --name=netdata \
  -p 19999:19999 \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  --cap-add SYS_PTRACE \
  netdata/netdata:latest

# With persistent storage
docker run -d \
  --name=netdata \
  -p 19999:19999 \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v netdata-lib:/var/lib/netdata \
  -v netdata-cache:/var/cache/netdata \
  netdata/netdata:latest
```

### Kubernetes
```bash
# Using helm
helm repo add netdata https://netdata.github.io/helm-chart
helm install netdata netdata/netdata \
  --set parent.claimDiskSize=10Gi \
  --set child.claimDiskSize=5Gi
```

### Manual Installation (Source)
```bash
# Clone repository
git clone https://github.com/netdata/netdata.git
cd netdata

# Install dependencies
sudo ./netdata-installer.sh

# Or build from source
sudo ./build-install.sh
```

### Post-Installation
```bash
# Enable at startup
sudo systemctl enable netdata

# Start service
sudo systemctl start netdata

# Check status
sudo systemctl status netdata

# Access dashboard
# http://localhost:19999
```

---

## 3. Configuration

### Configuration Files
```bash
# Main configuration
/etc/netdata/netdata.conf

# Health configuration
/etc/netdata/health.d/*.conf

# Collector configuration
/etc/netdata/python.d/*.conf
/etc/netdata/charts.d/*.conf
/etc/netdata/go.d/*.conf

# Streaming configuration
/etc/netdata/stream.conf

# Alarm configuration
/etc/netdata/health_alarm_notify.conf
```

### Main Configuration
```ini
[global]
    hostname = my-server
    bind to = 0.0.0.0:19999
    port = 19999
    default port = 19999
    bind socket to = /var/run/netdata/netdata.sock
    disconnect idle files after seconds = 3600
    enable metric correlations = yes
    run as user = netdata
    web files mode = 0755

[web]
    enable gzip compression = yes
    gzip compression level = 3
    allow connections from = localhost
    allow dashboard from = localhost
    allow badges from = *
    allow streaming from = *
    allow netdata.conf from = *

[health]
    enabled = yes
    in memory max health log entries = 1000
    script to execute on alarm = /usr/libexec/netdata/plugins.d/alarm-notify.sh
    execute alarm recipients = root

[statsd]
    enabled = yes
    histograms = yes
    max unique charts = 200
    max unique dimensions = 200
    buffer size bytes = 1000000
```

### Collector Configuration
```bash
# Disable collector
# /etc/netdata/python.d.conf
# Disable nginx: no

# Configure collector
# /etc/netdata/python.d/nginx.conf
update_every: 10
priority: 90000

jobs:
  local:
    url: 'http://localhost/status/nginx_status'
```

### Environment Variables
```bash
# Netdata configuration via environment
NETDATA_CONFIGURE_OPTS=--enable-plugin-nfacct
NETDATA_ADDITIONAL_CONFIGURE_OPTS=--with-math
```

---

## 4. Dashboard

### Accessing Dashboard
```bash
# Local access
http://localhost:19999

# Remote access
http://your-server-ip:19999

# Cloud access (netdata.cloud)
# Sign up and claim your node
```

### Dashboard Sections
```text
# Default view includes:
- CPU usage
- Memory usage
- Disk I/O
- Network interfaces
- Processes
- System load
- Interrupts
- Softirqs
- Forks
- Active processes
```

### Dashboard Controls
| Control | Description |
|---------|-------------|
| **Play/Pause** | Pause/Resume data collection |
| **Time Range** | Select time range (1h-30d) |
| **Refresh Rate** | Auto-refresh interval |
| **Export** | Export chart data |
| **Fullscreen** | Expand chart |
| **Save Configuration** | Save dashboard layout |

### Charts Types
| Type | Description |
|------|-------------|
| **Line** | Time series line chart |
| **Area** | Filled area chart |
| **Stacked** | Stacked area chart |
| **Bar** | Horizontal bar chart |
| **Gauge** | Circular gauge |
| **Number** | Single value display |

### URL Parameters
```bash
# Time range
?chart=system.cpu&after=-300&before=0

# Specific chart
?chart=system.load

# Dashboard view
?view=custom_dashboard

# Theme
?theme=dark

# Disable refresh
?refresh=0
```

---

## 5. Collectors

### Built-in Collectors

#### System Collectors
| Collector | Description | Key Charts |
|-----------|-------------|------------|
| **cpu** | CPU utilization | cpu, percpu |
| **mem** | Memory usage | ram, swap, available |
| **disk** | Disk I/O | disk_util, disk_ops |
| **net** | Network interfaces | net, net_packets |
| **load** | System load | load, load_cpu |
| **proc** | Process statistics | processes, meminfo |
| **ip** | IP statistics | ipv4, ipv6 |
| **softirq** | Softirqs | softirqs |
| **interrupts** | Interrupts | interrupts |

#### Application Collectors
| Collector | Package | Charts |
|-----------|---------|--------|
| **nginx** | python.d | nginx.requests |
| **apache** | python.d | apache.requests |
| **mysql** | python.d | mysql.queries |
| **postgres** | python.d | postgres.connections |
| **redis** | python.d | redis.clients |
| **mongodb** | python.d | mongodb.connections |
| **elasticsearch** | python.d | elasticsearch.searches |
| **php-fpm** | python.d | phpfpm.requests |
| **haproxy** | python.d | haproxy.backend |

#### Hardware Collectors
| Collector | Description | Charts |
|-----------|-------------|--------|
| **sensors** | Hardware sensors | sensors.temp, sensors.volt |
| **ipmi** | IPMI sensors | ipmi.temp, ipmi.volt |
| **nvidia-smi** | NVIDIA GPU | nvidia.gpu_usage |
| **lmsensors** | Linux sensors | sensors.fan, sensors.temp |

### Enabling/Disabling Collectors
```bash
# Disable collector
# Edit /etc/netdata/python.d.conf
# Set: nginx: no

# Or use edit-config
sudo /etc/netdata/edit-config python.d.conf

# Enable collector
# Remove 'no' from collector entry
nginx: yes
```

### Plugin Configuration
```bash
# Python plugins
/etc/netdata/python.d/

# Go plugins
/etc/netdata/go.d/

# Bash plugins
/etc/netdata/charts.d/

# C plugins
/usr/libexec/netdata/plugins.d/
```

### Custom Collector Example
```bash
#!/usr/bin/env python3
# /etc/netdata/python.d/custom.conf

update_every: 10
priority: 90000

job_name:
  name: 'custom'
  url: 'http://localhost:8080/metrics'

# Or create custom plugin
# /usr/libexec/netdata/plugins.d/custom.py
```

---

## 6. Alerts

### Alert Structure
```yaml
alarm: 'cpu_100'
on: 'system.cpu'
lookup: 'average -60s over 10s'
every: '10s'
warn: '$this > 80'
crit: '$this > 95'
to: 'sysadmin'
exec: '/usr/libexec/netdata/plugins.d/alarm-notify.sh'
```

### Alert Configuration
```bash
# Location
/etc/netdata/health.d/*.conf

# Enable health monitoring
sudo /etc/netdata/edit-config health.d/system-alarms.conf
```

### Common Alert Templates
```yaml
# CPU alerts
alarm: cpu_nginx_warning
on: nginx.requests
every: 10s
warn: $this > 1000
crit: $this > 5000
info: "nginx requests per second"

# Memory alerts
alarm: low_ram_warning
on: system.ram
lookup: average -60s
every: 10s
warn: $this < 10%
crit: $this < 5%
exec: /usr/libexec/netdata/plugins.d/alarm-notify.sh

# Disk alerts
alarm: disk_full_warning
on: disk.util
lookup: average -5m over 1m
every: 30s
warn: $this > 80
crit: $this > 95
```

### Alert Variables
| Variable | Description |
|----------|-------------|
| `$this` | Current value |
| `$host` | Hostname |
| `$chart` | Chart name |
| `$family` | Chart family |
| `$status` | Alert status |
| `$old_status` | Previous status |
| `$now` | Current time |
| `$unique_id` | Alert ID |

### Alert Statuses
| Status | Description |
|--------|-------------|
| **REMOVED** | Alert was removed |
| **UNDEFINED** | Alert not initialized |
| **CLEAR** | Alert recovered |
| **WARNING** | Warning threshold |
| **CRITICAL** | Critical threshold |

---

## 7. Health Configuration

### Health Configuration File
```ini
[health]
    enabled = yes
    script to execute on alarm = /usr/libexec/netdata/plugins.d/alarm-notify.sh
    execute alarm recipients = root
    enable automatic health archives = yes
    health log history = 43200

[alarm-telegram]
    enabled = yes
    type: telegram
    chath_id: 123456789
    bot_api_token: 'your-bot-token'

[alarm-email]
    enabled = yes
    type: email
    smtp_host: smtp.example.com
    smtp_port: 587
    from: netdata@example.com
    to: admin@example.com
```

### Notification Configuration
```bash
# /etc/netdata/health_alarm_notify.conf

# Enable Telegram
SEND_TELEGRAM="YES"
TELEGRAM_BOT_API_TOKEN="your-token"
TELEGRAM_CHAT_ID="chat-id"

# Enable Email
SEND_EMAIL="YES"
MAIL_TO="admin@example.com"
SMTP_HOST="smtp.example.com"
SMTP_PORT="587"
SMTP_USER="user"
SMTP_PASSWORD="password"

# Enable Slack
SEND_SLACK="YES"
SLACK_WEBHOOK_URL="https://hooks.slack.com/services/xxx"
SLACK_CHANNEL="#alerts"
```

### Custom Notification Script
```bash
#!/bin/bash
# /usr/libexec/netdata/plugins.d/custom-notify.sh

ALARM="$1"
VALUE="$2"
STATUS="$3"
HOST="$4"
CHART="$5"
FAMILY="$6"
WHEN="$7"
DURATION="$8"

# Custom notification logic
if [ "$STATUS" = "CRITICAL" ]; then
    # Send to pager
    curl -X POST "https://alerting.example.com" \
        -d "message=$ALARM on $HOST: $VALUE"
fi
```

---

## 8. Streaming

### Master Configuration
```ini
# /etc/netdata/netdata.conf

[stream]
    enabled = yes
    destination = netdata-master:19999
    api key = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Child Configuration
```ini
# /etc/netdata/stream.conf

[stream]
    enabled = yes
    destination = 192.168.1.100:19999
    api key = xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    timeout = 60
    buffer size = 256
```

### API Key Management
```bash
# Generate API key
uuidgen

# Add to master configuration
# /etc/netdata/netdata.conf

[stream]
    api key = generated-uuid
```

### Cluster Monitoring
```bash
# Access all nodes
# http://master:19999/hosts

# View all hosts
# Click on host selector
```

### Replication and Archive
```yaml
# Streaming with replication
[stream]
    enabled = yes
    destination = parent1:19999,parent2:19999
    api key = primary-key
    replication = [dimension1, dimension2]
```

---

## 9. Plugins

### Plugin Architecture
```bash
# Plugin types
/usr/libexec/netdata/plugins.d/

# C plugins (native)
# - freebsd.plugin
# - proc.plugin
# - cgroups.plugin

# Python plugins
# - python.d.plugin
# - nginx.plugin
# - mysql.plugin

# Go plugins
# - go.d.plugin
# - prometheus.plugin
# - k8sstate.plugin
```

### Python Plugin Development
```python
#!/usr/bin/env python3
# my_collector.py

from collections import namedtuple
from os import environ

try:
    from systemd import journal
except ImportError:
    pass

import netdata_pandas as npd

ORDER = ['my_metric']
CHARTS = {
    'my_metric': {
        'options': [None, 'My Metric', 'units', 'my_group', 'my_metric', 'line'],
        'lines': [
            ['metric1', 'Metric 1', 'absolute'],
            ['metric2', 'Metric 2', 'absolute'],
        ]
    }
}

def get_data():
    # Fetch metrics
    data = {'metric1': 42, 'metric2': 100}
    return data

def main():
    npd.main(ORDER, CHARTS, get_data)

if __name__ == '__main__':
    main()
```

### Enable Custom Plugin
```bash
# Create symlink
sudo ln -s /path/to/my_collector.py /usr/libexec/netdata/plugins.d/my_collector.py

# Make executable
sudo chmod +x /path/to/my_collector.py

# Restart netdata
sudo systemctl restart netdata
```

### StatsD Collector
```bash
# Send metrics to netdata StatsD
# Default port: 8125

# Using netcat
echo "myapp.counter:1|c" | nc -w 1 -u localhost 8125

# Using Python
import socket
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.sendto(b"myapp.counter:1|c", ("localhost", 8125))

# StatsD charts automatically created
# Access at: http://localhost:19999/statsd
```

---

## 10. API

### REST API Endpoints
```bash
# All metrics as JSON
curl http://localhost:19999/api/v1/allmetrics?format=json

# Specific chart data
curl "http://localhost:19999/api/v1/data?chart=system.cpu&after=-300&before=0"

# Chart info
curl http://localhost:19999/api/v1/chart?chart=system.cpu

# List all charts
curl http://localhost:19999/api/v1/charts

# Registry info
curl http://localhost:19999/api/v1/info

# Health alarms
curl http://localhost:19999/api/v1/alarms?active

# Netdata info
curl http://localhost:19999/api/v1/info
```

### Data Query API
```bash
# Get data for specific chart
curl "http://localhost:19999/api/v1/data?chart=system.cpu&after=-300&before=0&points=100&format=json"

# Format options
# format=json (default)
# format=csv
# format=tsv
# format=shell
# format=prometheus

# Prometheus format
curl http://localhost:19999/api/v1/charts | grep -v "^#" | head -20

# Prometheus metrics endpoint
curl http://localhost:19999/api/v1/allmetrics?format=prometheus
```

### API Authentication
```bash
# Disable auth (not recommended)
# In netdata.conf:
[web]
    allow connection from = 0.0.0.0
    basic login = 0

# Enable SSL/TLS
[web]
    ssl = yes
    ssl key = /etc/netdata/ssl/key.pem
    ssl certificate = /etc/netdata/ssl/cert.pem
```

### Cloud API
```bash
# Claim node to cloud
sudo netdata-claim.sh -token=YOUR_TOKEN -rooms=ROOM_ID

# Cloud API endpoints
# https://app.netdata.io/api/v1/nodes
# https://app.netdata.io/api/v1/alerts
```

---

## 11. Troubleshooting

### Common Issues

#### Service Not Starting
```bash
# Check service status
sudo systemctl status netdata

# Check logs
sudo journalctl -u netdata -n 100

# Check for port conflicts
sudo lsof -i :19999

# Check configuration
sudo netdata -d -c /etc/netdata/netdata.conf
```

#### Missing Metrics
```bash
# Check collector status
# http://localhost:19999/api/v1/charts

# Enable collector
sudo /etc/netdata/edit-config python.d.conf

# Restart netdata
sudo systemctl restart netdata

# Check collector logs
sudo journalctl | grep netdata
```

#### High Resource Usage
```bash
# Check netdata CPU
top -p $(pgrep netdata)

# Reduce update frequency
# Edit netdata.conf
[global]
    update every = 5

# Disable unused collectors
# Edit /etc/netdata/python.d.conf

# Limit database size
[global]
    history = 86400
```

### Debug Commands
```bash
# Test configuration
sudo netdata -c /etc/netdata/netdata.conf --test-config

# Debug mode
sudo netdata -D

# Check plugin output
sudo /usr/libexec/netdata/plugins.d/python.d.plugin debug 1

# Check health
sudo /usr/libexec/netdata/plugins.d/health-engine debug
```

### Log Analysis
```bash
# View all logs
sudo journalctl -u netdata -f

# Search for errors
sudo journalctl -u netdata | grep ERROR

# Search for specific chart
sudo journalctl -u netdata | grep nginx
```

---

## 12. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| Install | `bash <(curl -Ss https://my-netdata.io/kickstart.sh)` |
| Start | `sudo systemctl start netdata` |
| Stop | `sudo systemctl stop netdata` |
| Restart | `sudo systemctl restart netdata` |
| Status | `sudo systemctl status netdata` |
| Config | `sudo /etc/netdata/edit-config netdata.conf` |
| Test config | `sudo netdata --test-config` |
| View logs | `sudo journalctl -u netdata -f` |

### Key URLs
| URL | Description |
|-----|-------------|
| `http://localhost:19999` | Main dashboard |
| `http://localhost:19999/api/v1/charts` | All charts |
| `http://localhost:19999/api/v1/allmetrics` | All metrics |
| `http://localhost:19999/api/v1/alarms` | Active alarms |
| `http://localhost:19999/netdata.conf` | Agent config |
| `http://localhost:19999/statsd` | StatsD metrics |

### Collector Paths
| Component | Path |
|-----------|------|
| Main config | `/etc/netdata/netdata.conf` |
| Python collectors | `/etc/netdata/python.d/` |
| Go collectors | `/etc/netdata/go.d/` |
| Health config | `/etc/netdata/health.d/` |
| Stream config | `/etc/netdata/stream.conf` |
| Plugins | `/usr/libexec/netdata/plugins.d/` |

### Important Ports
| Port | Service | Description |
|------|---------|-------------|
| 19999 | HTTP | Dashboard/API |
| 8125 | UDP | StatsD |
| 19998 | TCP | Streaming (parent) |
| 19997 | TCP | Streaming (child) |

### Alert Status Colors
| Status | Color | Icon |
|--------|-------|------|
| CLEAR | Green | ✓ |
| WARNING | Yellow | ⚠ |
| CRITICAL | Red | ✗ |
| UNDEFINED | Gray | ? |
| REMOVED | Gray | - |

---

*Last Updated: January 2026*
*Generated for netdata 1.45.x*
