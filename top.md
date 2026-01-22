# top Complete Cheat Sheet

## Table of Contents
1. [Overview](#1-overview)
2. [Basic Usage](#2-basic-usage)
3. [Interactive Commands](#3-interactive-commands)
4. [Display Fields](#4-display-fields)
5. [Configuration](#5-configuration)
6. [Color and Layout](#6-color-and-layout)
7. [Process Management](#7-process-management)
8. [Batch Mode](#8-batch-mode)
9. [Quick Reference](#9-quick-reference)

---

## 1. Overview

`top` is a Unix/Linux system monitoring tool that displays system summary information and a list of processes currently managed by the Linux kernel.

### Key Features
| Feature | Description |
|---------|-------------|
| Real-time updates | Default 3-second refresh interval |
| Process sorting | Sort by CPU, memory, time, etc. |
| Interactive control | Keyboard shortcuts for filtering and management |
| Multi-core support | Shows per-CPU statistics |
| Memory display | Shows physical and swap memory |

### What top Shows
- System uptime
- Number of users
- Load average (1, 5, 15 minutes)
- Task summary (running, sleeping, stopped, zombie)
- CPU states (user, system, nice, idle, iowait, irq, softirq, steal, guest, guest_nice)
- Memory and swap usage
- Process list with various metrics

---

## 2. Basic Usage

### Starting top
```bash
top                      # Start with default settings
top -d 5                 # Start with 5-second refresh
top -p 1234              # Monitor specific PID only
top -u username          # Show processes for specific user
top -b                   # Batch mode (non-interactive)
top -n 10                # Exit after 10 iterations
top -H                   # Show threads
```

### Default Output
```
top - 10:15:30 up 5 days, 2:16, 2 users, load average: 0.52, 0.58, 0.59
Tasks: 245 total, 1 running, 244 sleeping, 0 stopped, 0 zombie
%Cpu(s): 2.3 us, 1.0 sy, 0.0 ni, 96.5 id, 0.2 wa, 0.0 hi, 0.0 si, 0.0 st
MiB Mem :  16384 total,   4125 free,   8976 used,   3283 buff/cache
MiB Swap:   8192 total,   8192 free,      0 used.   6125 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 1234 root      20   0  234567  12345   6789 R  12.5  0.8   0:12.34 process
```

### Field Descriptions
| Field | Description |
|-------|-------------|
| PID | Process ID |
| USER | Username |
| PR | Priority |
| NI | Nice value |
| VIRT | Virtual memory |
| RES | Resident memory |
| SHR | Shared memory |
| S | Status (R/S/D/Z/T) |
| %CPU | CPU percentage |
| %MEM | Memory percentage |
| TIME+ | CPU time |
| COMMAND | Command name |

---

## 3. Interactive Commands

### Navigation and Scrolling
| Key | Function |
|-----|----------|
| `Enter` or `Space` | Refresh display |
| `Arrow keys` | Scroll process list |
| `Home` | Jump to top |
| `End` | Jump to bottom |
| `Page Up/Down` | Scroll page |

### Sorting
| Key | Function | Default Key |
|-----|----------|-------------|
| `P` | Sort by CPU % | `M` (shift+p) |
| `M` | Sort by memory % | `P` (shift+m) |
| `T` | Sort by time+ | `t` |
| `N` | Sort by PID | `n` |
| `A` | Toggle sort field | - |

### Process Management
| Key | Function |
|-----|----------|
| `k` | Kill process (enter PID) |
| `r` | Renice process |
| `f` | Field management |
| `o` | Filter processes |
| `=` | Remove filters |

### Display Toggles
| Key | Function |
|-----|----------|
| `1` | Toggle single/all CPU |
| `i` | Toggle idle processes |
| `c` | Toggle command line |
| `H` | Toggle threads |
| `j` | Toggle decimal alignment |
| `x` | Highlight sort column |
| `b` | Bold/Reverse highlighting |

### Views and Windows
| Key | Function |
|-----|----------|
| `A` | Alternate display mode |
| `G` | Change window name |
| `W` | Write configuration |
| `?` | Help |

---

## 4. Display Fields

### Available Fields
| Field | Description |
|-------|-------------|
| PID | Process ID |
| USER | User name |
| PR | Priority |
| NI | Nice value |
| VIRT | Virtual Image (kb) |
| RES | Resident size (kb) |
| SHR | Shared Memory (kb) |
| S | Status |
| %CPU | CPU usage |
| %MEM | Memory usage (RES) |
| TIME+ | CPU Time, hundredths |
| COMMAND | Command name |
| PPID | Parent Process PID |
| RUSER | Real user name |
| UID | User ID |
| GROUP | Group name |
| TTY | Controlling TTY |
| P | Last used CPU (SMP) |
| SWAP | Virtual memory (kb) |
| CODE | Code size (kb) |
| DATA | Data+Stack size (kb) |
| nFLT | Page fault count |
| nDRT | Dirty pages count |
| WCHAN | Waiting channel |
| Flags | Task flags |

### Adding/Removing Fields
```bash
# Press 'f' or 'F' to enter field management
# Navigate with arrow keys
# Press space to toggle field
# Press 'q' to exit
```

### Field Order Management
```bash
# Press 'f' to enter field management
# Navigate to field
# Press 'Right Arrow' to move to "Current Order" field
# Press Up/Down to reorder
# Press 'q' to exit
```

---

## 5. Configuration

### Configuration File
```bash
~/.toprc          # User configuration file
```

### Saving Configuration
```bash
# Make changes in top
W                 # Write configuration to ~/.toprc
```

### Sample ~/.toprc
```
rc::1:0:0:1000:0.50:0.20:0.00:0:0:1:0
idle:1
bold_selected:0
col1_fields:0:1:2:3:4:5:6:7:8:9:10:11:12:13:14:15:16:17:18:19:20:21:22:23:24:25:26:27:28:29:30:31:32:33:34:35:36:37:38:39:40:41:42:43:44:45:46:47:48:49:50:51:52:53:54:55:56:57:58:59:60:61:62:63:64:65:66:67:68:69:70:71:72:73:74:75:76:77:78:79:80:81:82:83:84:85:86:87:88:89:90:91:92:93:94:95:96:97:98:99
col2_fields:0:1:2:3:4:5:6:7
```

### Command Line Options
| Option | Description |
|--------|-------------|
| `-b` | Batch mode |
| `-c` | Show command line |
| `-d <seconds>` | Delay between updates |
| `-H` | Show threads |
| `-i` | Ignore idle processes |
| `-n <iterations>` | Number of iterations |
| `-p <pids>` | Monitor specific PIDs |
| `-u <user>` | Monitor specific user |
| `-U <user>` | Monitor specific user (any UID) |
| `-s <secure>` | Secure mode (disable some commands) |

---

## 6. Color and Layout

### Color Mode
```bash
# Press 'Z' to access color settings
# Choose color scheme:
#   B - Black/White
#   C - Classic (monochrome)
#   d - Dark background (default)
#   g - Gray
#   t - Terminal
#   u - User defined
```

### Highlight Toggle
```bash
x           # Toggle column highlight
y           # Toggle row highlight
b           # Toggle bold/reverse
```

### Summary Area Toggle
```bash
# Press 'l' to toggle load average line
# Press 't' to toggle task/CPU states
# Press 'm' to toggle memory/swap lines
# Press '1' to toggle CPU graph
```

---

## 7. Process Management

### Killing Processes
```bash
# In top interactive mode:
k           # Press 'k', enter PID, then signal number
# Common signals:
#   9 - SIGKILL (force kill)
#  15 - SIGTERM (graceful)
#   1 - SIGHUP (hangup)
```

### Renicing Processes
```bash
# In top interactive mode:
r           # Press 'r', enter PID, then nice value (-20 to 19)
# Lower nice = higher priority
# Higher nice = lower priority
```

### Filtering Processes
```bash
# Press 'o' or 'O' to add filter
# Format: field=value
# Examples:
#   COMMAND=nginx
#   USER=root
#   %CPU>5.0
#   %MEM>10.0
#   PID>1000
```

### Search
```bash
# Press 'L' to search
# Enter search string
# Press '&' to find next
```

---

## 8. Batch Mode

### Basic Batch Mode
```bash
# Run for 5 iterations with 2-second delay
top -b -d 2 -n 5 > top_output.txt

# Batch mode with specific columns
top -b -n 1 | head -20
```

### Parsing Batch Output
```bash
# Get only process list
top -b -n 1 | tail -n +8

# Get CPU summary
top -b -n 1 | head -3

# Get memory info
top -b -n 1 | grep Mem

# Get specific process
top -b -n 1 | grep nginx
```

### Continuous Batch Output
```bash
# Run continuously to log file
top -b -d 60 >> /var/log/top.log &

# Monitor specific process
watch -n 1 'top -b -n 1 | grep myprocess'
```

### Scripts with top
```bash
#!/bin/bash
# Monitor CPU usage
while true; do
    top -b -n 1 | awk '/^%Cpu/{print "CPU: " 100-$8 "%"}'
    top -b -n 1 | awk '/^Mem:/{print "Mem: " $4 "/" $2}'
    sleep 5
done
```

---

## 9. Quick Reference

### Common Tasks
| Task | Command |
|------|---------|
| Start top | `top` |
| Refresh every 1 second | `top -d 1` |
| Show only user processes | `top -u $USER` |
| Show threads | `top -H` |
| Batch mode output | `top -b -n 5` |
| Sort by memory | Press `M` |
| Sort by CPU | Press `P` |
| Kill process | Press `k` |
| Save config | Press `W` |
| Help | Press `h` or `?` |

### Process States
| State | Meaning |
|-------|---------|
| R | Running |
| S | Sleeping |
| D | Uninterruptible sleep |
| Z | Zombie |
| T | Stopped by job control signal |
| t | Tracing stop |

### Signal Numbers
| Signal | Number | Description |
|--------|--------|-------------|
| SIGHUP | 1 | Hangup |
| SIGINT | 2 | Interrupt |
| SIGQUIT | 3 | Quit |
| SIGKILL | 9 | Kill (cannot be caught) |
| SIGTERM | 15 | Termination |
| SIGSTOP | 19 | Stop (cannot be caught) |

### Load Average Interpretation
| Load Average | System State |
|--------------|--------------|
| < CPU count | Under-utilized |
| = CPU count | Fully utilized |
| > CPU count | Overloaded |

### Troubleshooting
| Problem | Solution |
|---------|----------|
| Top not updating | Press `Space` or `Enter` |
| Can't kill process | Use `sudo` or check permissions |
| Too many processes | Filter with `-u` or `o` |
| Unresponsive | Use `kill -9 PID` from terminal |
| Memory shows negative | Normal due to kernel buffers |

---

## Comparison with Alternatives

| Feature | top | htop | btop |
|---------|-----|------|------|
| Interactive | Yes | Yes | Yes |
| Mouse support | No | Yes | Yes |
| Color support | Limited | Yes | Yes |
| Process tree | No | Yes | Yes |
| Built-in | Most systems | Requires install | Requires install |
| Resource usage | Very low | Low | Medium |

---

*Last Updated: January 2026*
*Compatible with procps-ng top*
