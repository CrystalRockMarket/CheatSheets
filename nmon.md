# nmon Complete Cheat Sheet

## Table of Contents
1. [Overview](#1-overview)
2. [Installation](#2-installation)
3. [Basic Usage](#3-basic-usage)
4. [Interactive Modes](#4-interactive-modes)
5. [Data Collection](#5-data-collection)
6. [Data Analysis](#6-data-analysis)
7. [Command Line Options](#7-command-line-options)
8. [Quick Reference](#8-quick-reference)

---

## 1. Overview

nmon (Nigel's Monitor) is a system administrator tool for monitoring and analyzing Linux system performance. It was originally developed for AIX but is now available for Linux.

### Key Features
| Feature | Description |
|---------|-------------|
| Real-time monitoring | Live CPU, memory, disk, and network stats |
| Low overhead | Minimal impact on system performance |
| Multiple output modes | Interactive, data collection, and report generation |
| Cross-platform | Works on AIX, Linux, and other Unix systems |

### What nmon Monitors
- CPU utilization
- Memory usage
- Disk I/O and throughput
- Network statistics
- Disk space
- Top processes
- Kernel statistics
- Power management (on some systems)

---

## 2. Installation

### Debian/Ubuntu
```bash
sudo apt update
sudo apt install nmon
```

### RHEL/CentOS/Fedora
```bash
sudo dnf install nmon
# or
sudo yum install nmon
```

### Arch Linux
```bash
sudo pacman -S nmon
```

### From Source
```bash
wget https://sourceforge.net/projects/nmon/files/lmon版本号.tar.gz
tar -xzf lmon版本号.tar.gz
cd lmon版本号
make
sudo make install
```

---

## 3. Basic Usage

### Starting nmon
```bash
nmon                    # Start with interactive mode
nmon -f                 # Start data collection in background
nmon -s -c 10           # Sample every second, 10 samples
nmon -F /path/to/file.nmon  # Specify output file
```

### Interactive Mode Controls
| Key | Function |
|-----|----------|
| `c` | CPU utilization |
| `m` | Memory |
| `d` | Disk |
| `n` | Network |
| `t` | Top processes |
| `k` | Kernel stats |
| `j` | File system |
| `l` | Long-term statistics |
| `h` | Help |
| `q` | Quit |
| `0` | Reset peak values |

### Color Coding in nmon
| Color | Meaning |
|-------|---------|
| Green | Good/Healthy |
| Blue | Normal |
| Yellow | Warning |
| Red | Critical |

---

## 4. Interactive Modes

### CPU Mode (c)
```bash
nmon        # Start nmon
c           # Press 'c' for CPU view
```
Shows:
- User vs System vs Wait CPU percentages
- CPU utilization over time
- Per-core statistics (on multi-core systems)

### Memory Mode (m)
```bash
nmon
m           # Press 'm' for memory view
```
Shows:
- Total memory
- Used memory
- Free memory
- Cached memory
- Swap usage

### Disk Mode (d)
```bash
nmon
d           # Press 'd' for disk view
```
Shows:
- Disk I/O (read/write)
- Disk throughput (KB/s)
- Disk utilization
- Read/write ratios

### Network Mode (n)
```bash
nmon
n           # Press 'n' for network view
```
Shows:
- Network interface statistics
- Packet counts
- Errors
- Throughput

### Top Processes Mode (t)
```bash
nmon
t           # Press 't' for top processes
```
Shows:
- Top CPU consumers
- Top memory consumers
- Process details

---

## 5. Data Collection

### Starting Data Collection
```bash
# Collect data for 1 hour, sampling every 30 seconds
nmon -f -s 30 -c 120 -F /var/log/nmon_data.nmon

# Collect for 10 minutes, sampling every 10 seconds
nmon -f -s 10 -c 60 -F ~/nmon_$(date +%Y%m%d_%H%M).nmon

# Collect with all statistics
nmon -f -s 5 -c 720 -T -F /tmp/full_stats.nmon
```

### Data Collection Options
| Option | Description |
|--------|-------------|
| `-f` | Foreground mode (no fork) |
| `-F <filename>` | Output filename |
| `-s <seconds>` | Sample interval |
| `-c <count>` | Number of samples |
| `-T` | Include top processes |
| `-m <directory>` | Directory for output |
| `-r <runname>` | Run name (appears in reports) |

### Background Collection
```bash
# Start in background
nmon -f -s 30 -c 288 -F /var/log/nmon_$(hostname)_$(date +%Y%m%d).nmon &

# Using systemd timer
# Create /etc/systemd/system/nmon.service
# Create /etc/systemd/system/nmon.timer

# Using cron
# 0 * * * * /usr/bin/nmon -f -s 60 -c 1440 -F /var/log/nmon/nmon_$(date +\%Y\%m\%d_\%H\%M).nmon
```

---

## 6. Data Analysis

### Using nmonchart
```bash
# Install nmonchart
wget https://sourceforge.net/projects/nmon/files/nmonchart版本号.tar.gz
tar -xzf nmonchart版本号.tar.gz
sudo cp nmonchart /usr/local/bin/

# Generate interactive HTML chart
nmonchart /var/log/data.nmon /var/log/data.html
```

### Using nmonexcel (Deprecated)
```bash
# For older systems
nmonexcel data.nmon
```

### Analyzing with AWK
```bash
# Extract CPU stats
awk -F, '/CPU_ALL/ {print $1, $3, $4, $5}' data.nmon

# Extract memory stats
awk -F, '/MEM/ {print $1, $3, $4, $5}' data.nmon

# Extract disk I/O
awk -F, '/DISK/ {print $1, $3, $4, $5}' data.nmon

# Extract network stats
awk -F, '/NET/ {print $1, $3, $4, $5}' data.nmon
```

### Using Python for Analysis
```python
import pandas as pd

# Read nmon data
df = pd.read_csv('data.nmon', skiprows=[0,1,2,3])
df.columns = df.columns.str.strip()

# Filter CPU data
cpu_data = df[df['label'] == 'CPU_ALL']

# Calculate averages
avg_cpu = cpu_data[['User%', 'Sys%', 'Wait%', 'Idle%']].mean()
```

### Using R for Analysis
```r
# Read nmon data
data <- read.csv("data.nmon", skip=5)

# Filter CPU data
cpu_data <- subset(data, label == "CPU_ALL")

# Summary statistics
summary(cpu_data)
```

---

## 7. Command Line Options

### Complete Option List
| Option | Description |
|--------|-------------|
| `-h` | Display help |
| `-s <seconds>` | Seconds between samples |
| `-c <count>` | Number of samples |
| `-f` | Foreground mode |
| `-F <file>` | Output filename |
| `-T` | Include top processes |
| `-d <disks>` | Disk statistics |
| `-k <disks>` | Disk subsystem |
| `-m <dir>` | Directory for output |
| `-r <name>` | Run name |
| `-p <file>` | PID file |
| `-x` | Excel spreadsheet format |
| `-z` | Same as -x, no fork |
| `-l <lines>` | Lines per page for output |
| `-e` | Only ESS (disk subsystem) stats |
| `-E` | Only extended disk stats |

### Examples
```bash
# Basic data collection
nmon -f -s 10 -c 100 -F test.nmon

# With top processes
nmon -f -s 30 -c 120 -T -F detailed.nmon

# For AIX disk statistics
nmon -f -s 5 -c 200 -d -k -F aix_disk.nmon

# Excel format
nmon -f -s 60 -c 60 -x -F spreadsheet.xls

# Help and version
nmon -h
nmon -V
```

---

## 8. Quick Reference

### Common Tasks
| Task | Command |
|------|---------|
| Start interactive | `nmon` |
| Collect 1 hour data | `nmon -f -s 30 -c 120 -F data.nmon` |
| Collect with top processes | `nmon -f -s 10 -c 360 -T data.nmon` |
| Start on boot | Add to /etc/rc.local or cron |
| Analyze with chart | `nmonchart data.nmon data.html` |
| Extract CPU stats | `awk -F, '/CPU_ALL/' data.nmon` |

### Interactive Keys Summary
| Key | Function |
|-----|----------|
| `c` | CPU |
| `m` | Memory |
| `d` | Disk |
| `n` | Network |
| `t` | Top processes |
| `k` | Kernel |
| `j` | Filesystem |
| `l` | Long-term |
| `h` | Help |
| `q` | Quit |

### Troubleshooting
| Problem | Solution |
|---------|----------|
| nmon not starting | Check if installed: `which nmon` |
| No data in file | Ensure `-f` flag is used |
| Permission denied | Run with sudo or check file permissions |
| Slow performance | Increase sample interval with `-s` |
| Missing charts | Install nmonchart package |

---

## Files and Locations

| File/Directory | Purpose |
|----------------|---------|
| `/usr/bin/nmon` | Main executable |
| `/var/log/nmon/` | Default log directory |
| `*.nmon` | Data files |
| `*.html` | Generated charts |
| `nmonchart` | Chart generation tool |

---

## Alternatives to nmon

| Tool | Description |
|------|-------------|
| `htop` | Interactive process viewer |
| `glances` | Cross-platform monitoring |
| `iotop` | I/O monitoring |
| `iftop` | Network monitoring |
| `bpytop` | Modern resource monitor |

---

*Last Updated: January 2026*
*Compatible with nmon version 16+*
