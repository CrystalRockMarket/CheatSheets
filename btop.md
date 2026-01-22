# btop Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Interface Overview](#3-interface-overview)
4. [Navigation](#4-navigation)
5. [Process Management](#5-process-management)
6. [CPU Monitoring](#6-cpu-monitoring)
7. [Memory and Swap](#7-memory-and-swap)
8. [GPU Monitoring](#8-gpu-monitoring)
9. [Disks and Network](#9-disks-and-network)
10. [Customization](#10-customization)
11. [Filters and Search](#11-filters-and-search)
12. [Command-Line Options](#12-command-line-options)
13. [Quick Reference](#13-quick-reference)

---

## 1. Introduction

### What is btop?
btop is a modern, C++ based resource monitor that shows usage and stats for CPU, memory, disks, network, processes, and GPUs with optional charts and a fully featured configuration set.

### Key Features
| Feature | Description |
|---------|-------------|
| **Modern UI** | True-color support with gradients |
| **GPU Support** | NVIDIA and AMD GPU monitoring |
| **Charts** | Optional sparkline charts |
| **Box Selection** | Visual process selection |
| **Flexible Layout** | Movable and resizable boxes |
| **Mouse Support** | Full mouse interaction |
| **Lua Plugins** | Extensible with Lua scripts |
| **Multiple Modes** | Various view options |

### Comparison with Similar Tools
| Feature | top | htop | btop | btm |
|---------|-----|------|------|-----|
| GPU Support | No | No | Yes | Yes |
| True Color | No | Limited | Yes | Yes |
| Mouse Support | Limited | Yes | Yes | Yes |
| Lua Plugins | No | No | Yes | No |
| Box Layout | Fixed | Fixed | Flexible | Fixed |
| Graphs | No | No | Yes | Yes |

---

## 2. Installation

### Debian/Ubuntu
```bash
sudo apt update
sudo apt install btop
```

### RHEL/CentOS/Fedora
```bash
sudo dnf install btop
```

### Arch Linux
```bash
sudo pacman -S btop
```

### macOS
```bash
# Using Homebrew
brew install btop
```

### From Source
```bash
git clone https://github.com/aristocratos/btop.git
cd btop
make
sudo make install

# With custom prefix
make PREFIX=$HOME/.local install
```

### Verify Installation
```bash
btop --version
btop --help
```

---

## 3. Interface Overview

### Main Screen Layout
```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  CPU [||||||||||||||||||||||||||||||||||] 45%  Mem [||||||||||] 62%  Swp [||] 25% │
│  4 cores  Intel(R) Core(TM) i7-10700  @ 2.90GHz                              │
│  1:  45% | 2:  30% | 3:  50% | 4:  55%                                         │
│                                                                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Processes                                                                    │
│  PID    USER    PRI    NI    VIRT    RES    SHR    S    CPU%    MEM%    TIME+  │
│  1234   root     20     0   1.23G   456M   120M   R     45.2     2.8    12:34  │
│  5678   mike     20     0    890M   234M    89M   S     12.1     1.5     3:21  │
│                                                                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Disk I/O                                                                    │
│  nvme0n1  [██████░░░░░░░░░░░░░░░░]  125 MB/s  [██████░░░░░░░░░░░░░]  85 MB/s │
│  sda      [██░░░░░░░░░░░░░░░░░░░░░░]   45 MB/s  [██░░░░░░░░░░░░░░░]  35 MB/s │
│                                                                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Network                                                                     │
│  eth0  [████████████████████████████]  1.2 GB/s  [███████████████]  450 MB/s │
│  wlan0 [██░░░░░░░░░░░░░░░░░░░░░░░░░░]  120 MB/s  [██░░░░░░░░░░░░░░░]  85 MB/s │
│                                                                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│  GPU 0: NVIDIA GeForce RTX 3080                                               │
│  [████████████████████████████████]  78%  Mem: 10/10 GB  Used: 8.5 GB        │
│                                                                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Mem  [||||||||||||||||||||                ]  62%  Total: 16.0 GB            │
│  Swp  [██                                  ]  25%  Total: 8.00 GB            │
│                                                                               │
└─────────────────────────────────────────────────────────────────────────────────┘

  F1 Help  F2 Menu  F3 Search  F4 Filter  F5 Tree  F6 Sort  F7 Nice-  F8 Nice+  F9 Kill  F10 Quit
```

### Box Components

#### CPU Box
```text
- Overall CPU usage bar
- Per-core usage bars
- Frequency and temperature
- Usage history graph
```

#### Memory Box
```text
- Memory usage bar
- Swap usage bar
- Detailed breakdown
- History graph
```

#### Process Box
```text
- Process list with columns
- Sortable and filterable
- Tree view option
- Kill and renice options
```

#### GPU Box
```text
- GPU usage bar
- Memory usage
- Temperature
- Process list
```

#### Disk I/O Box
```text
- Read/Write speeds
- Per-disk bars
- I/O history
```

#### Network Box
```text
- Download/Upload speeds
- Per-interface stats
- Network history
```

---

## 4. Navigation

### Keyboard Navigation
| Key | Action |
|-----|--------|
| `Arrow Up/Down` | Move selection |
| `Arrow Left/Right` | Move between boxes |
| `Page Up/Down` | Page up/down |
| `Home/End` | First/last item |
| `Tab` | Next box |
| `Esc` | Back/Escape |

### Process Navigation
| Key | Action |
|-----|--------|
| `j` | Move down |
| `k` | Move up |
| `h` | Move left |
| `l` | Move right |
| `Ctrl + F` | Page down |
| `Ctrl + B` | Page up |
| `gg` | Go to top |
| `G` | Go to bottom |

### Quick Actions
| Key | Action |
|-----|--------|
| `Enter` | Expand/collapse |
| `Space` | Tag process |
| `u` | Show user processes |
| `U` | Show all users |
| `H` | Toggle threads |
| `K` | Toggle kernel threads |

### Search and Filter
| Key | Action |
|-----|--------|
| `/` | Search |
| `n` | Next search match |
| `N` | Previous search match |
| `\` | Toggle search regex |
| `F4` | Filter menu |
| `Esc` | Clear filter |

### Box Navigation
| Key | Action |
|-----|--------|
| `1` | Focus CPU box |
| `2` | Focus Memory box |
| `3` | Focus Process box |
| `4` | Focus GPU box |
| `5` | Focus Disk box |
| `6` | Focus Network box |
| `0` | Focus all boxes |
| `Tab` | Cycle boxes |
| `Shift + Arrow` | Move between boxes |

---

## 5. Process Management

### Process Actions
| Key | Action |
|-----|--------|
| `F9` | Kill menu |
| `F7` | Decrease nice |
| `F8` | Increase nice |
| `r` | Renice prompt |
| `k` | Kill prompt |
| `Space` | Tag process |
| `t` | Toggle tree |
| `+` | Expand tree |
| `-` | Collapse tree |

### Kill Menu (F9)
```text
┌────────────────────────────────────┐
│            Signal Menu            │
├────────────────────────────────────┤
│  1) SIGHUP     - Hangup           │
│  2) SIGINT     - Interrupt        │
│  3) SIGQUIT    - Quit             │
│  4) SIGKILL    - Kill             │
│  5) SIGTERM    - Terminate        │
│  6) SIGSTOP    - Stop             │
│  7) SIGCONT    - Continue         │
│  8) SIGUSR1    - User 1           │
│  9) SIGUSR2    - User 2           │
│  0) Custom signal                 │
├────────────────────────────────────┤
│  Select signal or press Esc       │
└────────────────────────────────────┘
```

### Renice Process
```bash
# Press 'r' in process list
# Enter new nice value (-20 to 19)

# Or use F7 (decrease nice / higher priority)
# F8 (increase nice / lower priority)
```

### Process Filtering
```bash
# Press 'u' to filter by user
# Select user from list

# Press 'F4' for filter options
# - Command filter
# - User filter
# - State filter
```

### Process Columns
| Column | Description |
|--------|-------------|
| **PID** | Process ID |
| **PPID** | Parent PID |
| **USER** | Username |
| **PRI** | Priority |
| **NI** | Nice value |
| **VIRT** | Virtual memory |
| **RES** | Resident memory |
| **SHR** | Shared memory |
| **STAT** | Status flags |
| **CPU%** | CPU usage |
| **MEM%** | Memory usage |
| **TIME** | CPU time |
| **RXTX** | Network I/O |
| **COMMAND** | Command |

### Process States
| Flag | State |
|------|-------|
| R | Running |
| S | Sleeping |
| D | Disk sleep |
| Z | Zombie |
| T | Stopped |
| X | Dead |

---

## 6. CPU Monitoring

### CPU Box Information
```text
- Overall CPU usage percentage
- Per-core usage percentages
- CPU frequency (if available)
- CPU temperature (if available)
- Usage history graph
```

### CPU Options
```bash
# In btop: Press F2 -> CPU

Show frequencies=Yes
Show temperatures=Yes
Temp unit=Celsius
Show_cpu_graph=Yes
Graph type=Detailed
Update interval=1000ms
```

### CPU Frequency Scaling
```bash
# Show current frequency
# 4.2 GHz (max: 4.8 GHz)

# CPU governor information
# Available governors: performance, powersave, ondemand, conservative
```

### Per-Core Monitoring
```text
Core 1: [███████████████░░░░░░░]  75%  3.5 GHz  65C
Core 2: [█████████░░░░░░░░░░░░░]  55%  3.2 GHz  62C
Core 3: [███████████████████████]  95%  4.0 GHz  70C
Core 4: [█████████████████░░░░░░]  80%  3.8 GHz  68C
```

---

## 7. Memory and Swap

### Memory Information
```text
Memory:
- Total: 16.0 GB
- Used: 9.92 GB (62%)
- Available: 6.08 GB
- Buffers: 1.2 GB
- Cached: 4.5 GB

Breakdown:
- Applications: 5.8 GB
- Page Tables: 256 MB
- Slab: 512 MB
```

### Swap Information
```text
Swap:
- Total: 8.00 GB
- Used: 2.00 GB (25%)
- Free: 6.00 GB

Swap Activity:
- Swap in: 125 MB/s
- Swap out: 85 MB/s
```

### Memory Options
```bash
# In btop: Press F2 -> Mem

Show_memory_graph=Yes
Graph_type=Detailed
Update_interval=1000ms
```

### Memory Troubleshooting
```bash
# High memory usage
# Check for memory leaks

# Low available memory
# Consider increasing RAM
# Check for memory leaks
# Review cached memory

# High swap usage
# Indicates RAM shortage
# Consider adding RAM
# Optimize applications
```

---

## 8. GPU Monitoring

### GPU Box Information
```text
GPU 0: NVIDIA GeForce RTX 3080
  Usage: [███████████████████████]  78%
  Memory: [██████████████░░░░░░░░░]  85%  8.5/10 GB
  Temp: 72C
  Power: 320W / 350W (91%)
  Fans: 55%
```

### GPU Metrics
| Metric | Description |
|--------|-------------|
| **Usage** | GPU utilization percentage |
| **Memory** | GPU memory usage |
| **Temperature** | GPU temperature |
| **Power** | Power draw |
| **Fans** | Fan speed percentage |
| **Encoder/Decoder** | NVDEC/NVENC usage |

### GPU Options
```bash
# In btop: Press F2 -> GPU

Show_gpu=Yes
Update_interval=1000ms
```

### NVIDIA GPU Monitoring
```bash
# Requires nvidia-smi
nvidia-smi

# Check GPU info
nvidia-smi -q

# Monitor GPU
nvidia-smi dmon
```

### AMD GPU Monitoring
```bash
# Requires rocm-smi (for AMD)
rocm-smi

# Or use sensors
sensors | grep -i amd
```

---

## 9. Disks and Network

### Disk I/O Box
```text
Disk I/O:
nvme0n1
  Read:  [██████████░░░░░░░░░░░░]  125 MB/s
  Write: [███████░░░░░░░░░░░░░░░]  85 MB/s
  Iops:  12,500 ops/s

sda
  Read:  [████░░░░░░░░░░░░░░░░░░]  45 MB/s
  Write: [███░░░░░░░░░░░░░░░░░░░]  35 MB/s
  Iops:  4,200 ops/s
```

### Disk Information
```bash
# Mount points
# - / (root): 45% used
# - /home: 32% used
# - /data: 67% used

# SMART status
# - nvme0n1: Healthy
# - sda: Healthy
```

### Network Box
```text
Network:
eth0
  Download: [████████████████████]  1.2 GB/s
  Upload:   [███████████████░░░░░]  450 MB/s
  Packets:  125,000/45,000 pps
  Errors:   0/0

wlan0
  Download: [████░░░░░░░░░░░░░░░░░]  120 MB/s
  Upload:   [██░░░░░░░░░░░░░░░░░░░]  85 MB/s
  Signal:   -45 dBm (Good)
```

### Network Information
```bash
# Interface details
eth0: 192.168.1.100
wlan0: 192.168.1.101

# Connection counts
- Established: 1,234
- Timewait: 45
- Synrecv: 12
```

---

## 10. Customization

### Configuration File
```bash
# Location
~/.config/btop/btop.conf

# Default location
$HOME/.config/btop/
```

### Sample Configuration
```ini
# ~/.config/btop/btop.conf

[settings]
# Theme
theme="default"

# Colors
color_theme="default"

# Update interval
update_ms=1000

# Check for updates
check_updates=true

[visual]
# Show graphs
cpu_graph=true
mem_graph=true
net_graph=true
disk_graph=true
gpu_graph=true

# Graph style
graph_symbol="│▌█"

# Box size
proc_box_size="percentage=50"
cpu_box_size="percentage=25"
mem_box_size="percentage=15"
net_box_size="percentage=15"

[proc]
# Process sorting
sort_column="CPU"
sort_order="desc"

# Process columns
columns="PID,USER,PRI,NI,VIRT,RES,SHR,STATE,CPU,MEM,TIME,COMMAND"

# Show threads
show_threads=false

# Show kernel threads
show_kernel_threads=false

[cpu]
# Show frequencies
show_freq=true

# Show temperatures
show_temp=true

# Temp unit
temp_unit="C"

# Core view
core_view="all"

[mem]
# Show memory graph
show_graph=true

# Unit
mem_unit="GiB"

[gpu]
# Show GPU
show_gpu=true

# GPU index
gpu_index=0

[net]
# Show network graph
show_graph=true

# Unit
net_unit="MiB"
```

### Themes
```bash
# In btop: Press F2 -> Theme

# Available themes:
default       - Default dark theme
gruvbox       - Gruvbox colors
nord          - Nord colors
onedark       - One Dark colors
catppuccin    - Catppuccin colors
dracula       - Dracula colors
monokai       - Monokai colors
solarized     - Solarized colors
```

### Custom Theme Configuration
```ini
[colors]
# Main background
main_bg="#1e1e2e"

# Main foreground
main_fg="#cdd6f4"

# Title background
title_bg="#1e1e2e"
title_fg="#89b4fa"

# Process box
proc_bg="#1e1e2e"
proc_fg="#cdd6f4"
proc_select="#89b4fa"
proc_title="#89b4fa"

# CPU box
cpu_bg="#1e1e2e"
cpu_fg="#cdd6f4"
cpu_graph="#a6e3a1"

# Memory box
mem_bg="#1e1e2e"
mem_fg="#cdd6f4"
mem_graph="#f9e2af"

# Graph colors
graph_cpu="#89b4fa"
graph_mem="#f9e2af"
graph_net="#f38ba8"
graph_disk="#fab387"
graph_gpu="#cba6f7"
```

### Box Layout Customization
```bash
# In btop: Press F2 -> Layout

# Move boxes
# - Press F2 -> Layout
# - Select box to move
# - Use arrow keys
# - Press Enter to confirm

# Resize boxes
# - Press F2 -> Layout
# - Select box
# - Adjust size
# - Press Enter to confirm
```

---

## 11. Filters and Search

### Search
```bash
# Press '/' to search
# Type search term
# Press Enter

# Navigation
n - Next match
N - Previous match
Esc - Cancel
```

### Filter
```bash
# Press 'F4' for filter menu

# Filter types:
1) Command filter
2) User filter
3) State filter
4) Arguments filter

# Example filters:
nginx          - Command contains nginx
^python        - Command starts with python
root           - User is root
R              - Running processes
```

### Box Selection
```bash
# Use mouse or arrow keys
# Click to select
# Press Enter to confirm
```

### Process Tagging
```bash
# Space - Tag/untag process
# t - Tag all children
# T - Untag all
# U - Untag all

# Actions on tagged:
k - Kill tagged
r - Renice tagged
```

---

## 12. Command-Line Options

### Basic Options
| Option | Description |
|--------|-------------|
| `-h, --help` | Show help |
| `-v, --version` | Show version |
| `-t, --tty` | TTY mode |
| `-p, --pid` | Monitor PID |
| `-d, --delay` | Update delay |
| `--utf` | Force UTF-8 |
| `--debug` | Debug mode |

### Advanced Options
| Option | Description |
|--------|-------------|
| `-c, --config` | Config file |
| `-r, --refresh` | Refresh rate |
| `-m, --maximum` | Maximum Y value |
| `--hide-kernel` | Hide kernel threads |
| `--hide-userland` | Hide userland threads |
| `--tree` | Tree view |
| `--sort` | Sort column |
| `--order` | Sort order |

### Examples
```bash
# Basic run
btop

# Custom delay (2 seconds)
btop -d 2000

# Monitor specific PID
btop -p 1234

# Tree view
btop --tree

# Sort by memory
btop --sort MEM

# Custom config
btop --config /path/to/config

# Hide kernel threads
btop --hide-kernel

# Batch mode (for scripting)
btop -d 5000 -t -c
```

### Environment Variables
```bash
# Config directory
BTOP_DIR=~/.config/btop

# Theme
BTOP_THEME=dracula

# Color scheme
BTOP_COLOR_THEME=default

# Font
BTOP_FONT=JetBrains Mono
```

---

## 13. Quick Reference

### Essential Keyboard Shortcuts
| Key | Action |
|-----|--------|
| `F1` | Help |
| `F2` | Menu/Setup |
| `F3` | Search |
| `F4` | Filter |
| `F5` | Tree view |
| `F6` | Sort menu |
| `F7` | Nice - |
| `F8` | Nice + |
| `F9` | Kill menu |
| `F10` | Quit |
| `Esc` | Back/Escape |
| `q` | Quit |
| `Ctrl + C` | Quit |
| `/` | Search |
| `Space` | Tag process |
| `Enter` | Expand/collapse |
| `u` | User filter |
| `H` | Toggle threads |
| `K` | Toggle kernel threads |
| `t` | Toggle tree |
| `1-6` | Focus box |

### Box Keys
| Key | Box |
|-----|-----|
| `1` | CPU |
| `2` | Memory |
| `3` | Process |
| `4` | GPU |
| `5` | Disk |
| `6` | Network |
| `0` | All boxes |

### Process Columns Quick Reference
| Column | Short | Description |
|--------|-------|-------------|
| PID | PID | Process ID |
| USER | USR | Username |
| PRI | PRI | Priority |
| NI | NI | Nice value |
| VIRT | VIR | Virtual memory |
| RES | RES | Resident memory |
| SHR | SHR | Shared memory |
| STATE | STA | Status flags |
| CPU% | CPU | CPU percentage |
| MEM% | MEM | Memory percentage |
| TIME | TIM | CPU time |
| RXTX | RXTX | Network I/O |
| COMMAND | CMD | Command |

### Command Examples
| Task | Command |
|------|---------|
| Install | `apt install btop` |
| Run | `btop` |
| Monitor PID | `btop -p 1234` |
| Tree view | `btop --tree` |
| Sort by CPU | `btop --sort CPU` |
| Custom delay | `btop -d 2000` |
| Custom config | `btop -c config.conf` |
| Batch mode | `btop -d 5000 -t -c` |

### Color Legend
| Color | Usage |
|-------|-------|
| Green | Normal processes |
| Blue | Selected process |
| Red | High CPU/Memory |
| Yellow | Medium usage |
| Purple | GPU processes |
| Orange | I/O operations |

---

*Last Updated: January 2026*
*Generated for btop 1.3.x*
