# Glances Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Usage](#3-usage)
4. [Views and Layout](#4-views-and-layout)
5. [Process Management](#5-process-management)
6. [Plugins](#6-plugins)
7. [Alert System](#7-alert-system)
8. [Web Mode](#8-web-mode)
9. [API](#9-api)
10. [Customization](#10-customization)
11. [Troubleshooting](#11-troubleshooting)
12. [Quick Reference](#12-quick-reference)

---

## 1. Introduction

### What is Glances?
Glances is a cross-platform system monitoring tool written in Python that displays important information about your system in a terminal or web interface.

### Key Features
| Feature | Description |
|---------|-------------|
| **Cross-platform** | Linux, macOS, Windows, BSD |
| **Multiple views** | Terminal, web, CSV, JSON |
| **Plugin architecture** | Extensible monitoring |
| **Alert system** | Threshold-based alerts |
| **Client-server mode** | Remote monitoring |
| **REST API** | Built-in API |
| **Lightweight** | Python-based, low resource usage |

---

## 2. Installation

### pip (All Platforms)
```bash
pip install glances

# With all optional dependencies
pip install glances[all]

# With Docker support
pip install glances[docker]

# With GPU support
pip install glances[gpu]
```

### Debian/Ubuntu
```bash
sudo apt install glances
```

### RHEL/CentOS/Fedora
```bash
sudo dnf install glances
```

### macOS
```bash
brew install glances
```

### Docker
```bash
# Standalone
docker run --rm -v /proc:/host/proc:ro -v /sys:/host/sys:ro -v /dev:/host/dev:ro --network=host -it nicola/glances

# Full features
docker run -d --name glances \
  --restart=always \
  -p 61208:61208 \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /dev:/host/dev:ro \
  -e GLANCES_OPT="-w" \
  nicolargo/glances:latest
```

---

## 3. Usage

### Basic Commands
```bash
# Run glances
glances

# With specific theme
glances --theme dark

# Light theme
glances --theme light

# Full screen mode
glances -t 1

# Export to CSV
glances --export csv --export-file /tmp/glances.csv

# Export to JSON
glances --export json --export-file /tmp/glances.json

# Quiet mode
glances -q

# Help
glances --help
```

### Command-Line Options
| Option | Description |
|--------|-------------|
| `-t SECONDS` | Refresh interval |
| `-d DISK` | Enable disk I/O |
| `-n NETWORK` | Enable network I/O |
| `-s` | Server mode |
| `-c IP` | Client mode |
| `-w` | Web mode |
| `--export CSV` | Export mode |
| `--theme THEME` | Color theme |
| `-q` | Quiet mode |
| `-v` | Version |
| `-h` | Help |

---

## 4. Views and Layout

### Terminal Interface
```
CPU    [|||||||||||||||     ] 45%  MEM   [|||||||||       ] 62%  SWAP  [||       ] 25%
User: 45%  system: 30%  idle: 25%       Total: 16.0 GB  Used: 9.92 GB     Total: 8.0 GB  Used: 2.0 GB

LOAD   1-core  [|||||            ]  25%  DISK I/O  [██░░░░░░░░░░░] 125 MB/s  NETWORK  [██░░░░░░░] 1.2 GB/s
1 min: 1.25  5 min: 1.18  15 min: 1.05   sda: 45 MB/s  nvme: 80 MB/s       eth0: 1.2 GB/s  wlan0: 120 MB/s

PROCESSES
  PID  USER    CPU%  MEM%  COMMAND         2h
 1234  root    45.2   2.8  python3         ^
 5678  mike    12.1   1.5  chrome          |
 9012  www     8.3    0.8  nginx           |
```

### Quick Keys
| Key | Action |
|-----|--------|
| `a` | Auto sort by CPU |
| `c` | Sort by CPU% |
| `m` | Sort by MEM% |
| `p` | Sort by name |
| `i` | Sort by IOPs |
| `n` | Sort by network |
| `d` | Show/hide disk I/O |
| `f` | Show/hide filesystem |
| `w` | Delete warning logs |
| `x` | Delete warning and critical logs |
| `h` | Help |
| `q` | Quit |
| `l` | Show logs |
| `Enter` | Expand process |

### Views
```bash
# CPU view
glances --enable-cpu

# Memory view
glances --enable-mem

# Network view
glances --enable-network

# Disk view
glances --enable-diskio

# GPU view (with nvidia-smi)
glances --enable-gpu
```

---

## 5. Process Management

### Process Columns
| Column | Description |
|--------|-------------|
| **PID** | Process ID |
| **USER** | Process owner |
| **CPU%** | CPU usage |
| **MEM%** | Memory usage |
| **VIRT** | Virtual memory |
| **RES** | Resident memory |
| **IOR/s** | Read I/O |
| **IOW/s** | Write I/O |
| **COMMAND** | Command |
| **ARGS** | Arguments |

### Process Actions
| Key | Action |
|-----|--------|
| `k` | Kill process |
| `s` | Send signal menu |
| `l` | Show/hide logs |
| `Enter` | Expand process info |

### Kill Process
```bash
# Press 'k' in Glances
# Enter PID or select process
# Confirm kill

# Or use command line
glances --kill -p PID
```

### Process Filters
```bash
# Filter by user
glances --filter username

# Filter by command name
glances --process-name nginx

# Hide kernel threads
glances --hide-kernel-threads
```

---

## 6. Plugins

### Available Plugins
| Plugin | Description | Status |
|--------|-------------|--------|
| **cpu** | CPU monitoring | Built-in |
| **mem** | Memory monitoring | Built-in |
| **load** | Load average | Built-in |
| **network** | Network usage | Built-in |
| **diskio** | Disk I/O | Built-in |
| **process** | Process list | Built-in |
| **quicklook** | Quick summary | Built-in |
| **amps** | App monitoring | Built-in |
| **docker** | Docker containers | Optional |
| **gpu** | GPU monitoring | Optional |
| **cloud** | Cloud stats | Optional |
| **folders** | Folder monitoring | Optional |

### Docker Plugin
```bash
# Enable Docker plugin
glances --enable-docker

# Show Docker containers
# - Container name
# - Status
# - CPU %
# - Memory %
# - Network I/O
# - Block I/O

# Docker stats
docker stats

# Via Glances
glances --docker
```

### GPU Plugin
```bash
# Enable GPU plugin (requires nvidia-ml-py3)
pip install nvidia-ml-py3
glances --enable-gpu

# Shows:
# - GPU utilization
# - Memory usage
# - Temperature
# - Fan speed
# - Power draw
```

### Custom Plugin
```python
# /path/to/glances_plugins/myplugin.py

from glances.plugins.plugin.model import GlancesPluginModel

class PluginModel(GlancesPluginModel):
    """Glances plugin for custom metrics."""

    def __init__(self):
        super(PluginModel, self).__init__()
        self.reset()
        self.set_event_message('OK')

    @GlancesPluginModel._log_result_decorator
    def update(self):
        """Update custom metrics."""
        self.update_stats()
        return self.stats

    def update_stats(self):
        """Fetch custom metrics."""
        self.stats = {
            'custom_metric': get_custom_value(),
        }

    def get_alert(self):
        """Check thresholds."""
        if self.stats['custom_metric'] > 100:
            return 'CRITICAL'
        return 'OK'
```

---

## 7. Alert System

### Alert Configuration
```bash
# Location
~/.config/glances/glances.conf

[cpu]
cpu_careful=50
cpu_warning=70
cpu_critical=90

[mem]
mem_careful=50
mem_warning=70
mem_critical=90

[load]
load_careful=0.7
load_warning=1.0
load_critical=1.5

[network]
network_careful=50
network_warning=70
network_critical=90
```

### Alert Thresholds
| Status | Color | Default |
|--------|-------|---------|
| **CAREFUL** | Blue | 50% |
| **WARNING** | Yellow | 70% |
| **CRITICAL** | Red | 90% |

### Per-Process Alerts
```bash
# Enable process alerts
glances --enable-process-thresholds

# Configure in glances.conf
[process]
process_careful=50
process_warning=70
process_critical=90
```

---

## 8. Web Mode

### Start Web Server
```bash
# Start web interface
glances -w

# With custom port
glances -w -p 8080

# With password protection
glances -w --password

# Access web interface
# http://localhost:61208
```

### Web Interface Features
```text
- Real-time monitoring
- Multiple pages
- Dark/Light theme
- Mobile responsive
- Process management
```

### REST API
```bash
# Get all stats
curl http://localhost:61208/api/3/all

# Get specific plugin
curl http://localhost:61208/api/3/cpu
curl http://localhost:61208/api/3/mem
curl http://localhost:61208/api/3/network

# Get process list
curl http://localhost:61208/api/3/processlist

# Get plugins list
curl http://localhost:61208/api/3/pluginslist
```

---

## 9. API

### REST API Endpoints
| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/3/all` | GET | All stats |
| `/api/3/cpu` | GET | CPU stats |
| `/api/3/mem` | GET | Memory stats |
| `/api/3/load` | GET | Load stats |
| `/api/3/network` | GET | Network stats |
| `/api/3/diskio` | GET | Disk I/O stats |
| `/api/3/processlist` | GET | Process list |
| `/api/3/disk` | GET | Disk usage |
| `/api/3/fs` | GET | Filesystem |
| `/api/3/sensors` | GET | Sensors |
| `/api/3/docker` | GET | Docker stats |
| `/api/3/ports` | GET | Port checks |
| `/api/3/pluginslist` | GET | Available plugins |

### Client Library
```python
from glances import Glances

# Create client
client = Glances()

# Get all stats
stats = client.get_all()

# Get specific
cpu = client.get_cpu()
mem = client.get_mem()
load = client.get_load()
network = client.get_network()
diskio = client.get_diskio()
processlist = client.get_processlist()

# Get process info
process = client.get_process(pid)
```

### WebSocket
```javascript
// Connect to WebSocket
const ws = new WebSocket('ws://localhost:61208/ws');

ws.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log(data);
};
```

---

## 10. Customization

### Configuration File
```bash
# Location
~/.config/glances/glances.conf

# System-wide
/etc/glances/glances.conf
```

### Sample Configuration
```ini
[global]
# Refresh rate
refresh=3

# History size
history_size=100

# Theme
theme=system

[cpu]
cpu_careful=50
cpu_warning=70
cpu_critical=90

[mem]
mem_careful=50
mem_warning=70
mem_critical=90

[load]
load_careful=0.7
load_warning=1.0
load_critical=1.5

[network]
network_careful=50
network_warning=70
network_critical=90

[diskio]
diskio_careful=20
diskio_warning=40
diskio_critical=60

[process]
process_careful=50
process_warning=70
process_critical=90

[docker]
# Docker socket
docker_socket=unix:///var/run/docker.sock

[gpu]
# GPU monitoring
gpu_warning=70
gpu_critical=90

[ports]
# Network ports to check
ports=[22,80,443,3306,5432,8080,27017]

[folders]
# Folders to monitor
folders=/,/home,/var,/opt

[amp]
# Application monitoring profiles
[amp_nginx]
enable=true
regex=nginx
refresh=60

[amp_mysql]
enable=true
regex=mysql
refresh=60
```

### Environment Variables
```bash
# Config file
GLANCES_CONFIG=/path/to/glances.conf

# Disable plugins
GLANCES_DISABLE_PLUGINS=cpu,mem

# Enable plugins
GLANCES_ENABLE_PLUGINS=docker,gpu

# Export file
GLANCES_EXPORT_CSV=/tmp/glances.csv

# Web server port
GLANCES_PORT=61208
```

---

## 11. Troubleshooting

### Common Issues

#### Permission Errors
```bash
# Add user to groups
sudo usermod -aG video $USER
sudo usermod -aG docker $USER

# Check permissions
ls -la /dev/ipmi*
ls -la /var/run/docker.sock
```

#### Missing Metrics
```bash
# Check plugin status
glances --plugin docker

# Debug mode
glances -d

# Check logs
glances -l DEBUG

# Reinstall with all dependencies
pip install glances[all]
```

#### High Resource Usage
```bash
# Increase refresh interval
glances -t 5

# Disable plugins
glances --disable-plugin docker

# Check current plugins
glances --plugin list
```

### Debug Commands
```bash
# Verbose output
glances -v

# Debug mode
glances -d

# Check Python version
glances --version

# List all options
glances --help | grep -E "^\s+--"
```

### Log Analysis
```bash
# Enable logging
glances --log file

# View logs
cat ~/.config/glances/glances.log
```

---

## 12. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| Install | `pip install glances` |
| Run | `glances` |
| Web mode | `glances -w` |
| Server mode | `glances -s` |
| Client mode | `glances -c IP` |
| Export CSV | `glances --export csv` |
| Export JSON | `glances --export json` |
| Help | `glances --help` |

### Quick Keys
| Key | Action |
|-----|--------|
| `a` | Auto sort |
| `c` | Sort by CPU |
| `m` | Sort by MEM |
| `d` | Show/hide disk I/O |
| `f` | Show/hide filesystem |
| `h` | Help |
| `k` | Kill process |
| `l` | Show logs |
| `q` | Quit |
| `Enter` | Expand |

### Alert Colors
| Status | Color | Threshold |
|--------|-------|-----------|
| CAREFUL | Blue | 50% |
| WARNING | Yellow | 70% |
| CRITICAL | Red | 90% |

### Important Ports
| Port | Service |
|------|---------|
| 61208 | Web/API |

### File Locations
| File | Path |
|------|------|
| Config | `~/.config/glances/glances.conf` |
| Logs | `~/.config/glances/glances.log` |
| Plugins | `~/.local/lib/python*/site-packages/glances_plugins/` |

---

*Last Updated: January 2026*
*Generated for Glances 4.0.x*
