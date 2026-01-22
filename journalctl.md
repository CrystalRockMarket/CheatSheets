# journalctl Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Basic Commands](#2-basic-commands)
3. [Filtering](#3-filtering)
4. [Viewing Options](#4-viewing-options)
5. [Format Options](#5-format-options)
6. [Log Management](#6-log-management)
7. [Priority Filtering](#7-priority-filtering)
8. [Advanced Queries](#8-advanced-queries)
9. [Troubleshooting](#9-troubleshooting)
10. [Quick Reference](#10-quick-reference)

---

## 1. Introduction

journalctl is the command-line utility for querying and displaying logs from systemd's journal service (journald).

### Key Features
| Feature | Description |
|---------|-------------|
| Binary Logs | Structured, indexed logs |
| Time-based | Precise timestamps |
| Priority Filtering | Filter by severity |
| Multiple Fields | Rich metadata |
| Rotation | Automatic log management |
| Persistence | Persistent or volatile |

### Architecture
```
┌─────────────────────────────────────┐
│         systemd-journald            │
│  - Receives logs                    │
│  - Indexes logs                     │
│  - Stores logs                      │
└─────────────────────────────────────┘
         │                  │
         ▼                  ▼
┌──────────────┐    ┌──────────────┐
│   /run/      │    │   /var/lib/  │
│   journal    │    │   journal    │
│  (volatile)  │    │ (persistent) │
└──────────────┘    └──────────────┘
```

---

## 2. Basic Commands

### Viewing Logs
```bash
# View all logs
journalctl

# View kernel messages
journalctl -k
journalctl --dmesg

# View current boot logs
journalctl -b

# View previous boot (-1)
journalctl -b -1

# View boot before last (-2)
journalctl -b -2

# List available boots
journalctl --list-boots
```

### Service Logs
```bash
# View service logs
journalctl -u nginx

# View service and follow
journalctl -u nginx -f

# View service with lines
journalctl -u nginx -n 100

# View since boot
journalctl -u nginx --since boot

# View until specific time
journalctl -u nginx --until "2025-01-01 12:00:00"
```

### Unit Files
```bash
# View all units
journalctl --system

# View user units
journalctl --user

# View specific unit
journalctl -u nginx.service

# View multiple units
journalctl -u nginx -u php-fpm -u mysql
```

---

## 3. Filtering

### Time-Based
```bash
# Since specific time
journalctl --since "2025-01-01 00:00:00"

# Until specific time
journalctl --until "2025-01-01 12:00:00"

# Since 1 hour ago
journalctl --since "1 hour ago"

# Since yesterday
journalctl --since "yesterday"

# Since last Friday
journalctl --since "last Friday"

# Today only
journalctl --since "today"

# This week
journalctl --since "this week"

# Custom format
journalctl --since "2025-01-01T00:00:00"
```

### Field-Based
```bash
# Filter by PID
journalctl _PID=1234

# Filter by UID
journalctl _UID=1000

# Filter by GID
journalctl _GID=1000

# Filter by executable
journalctl _EXE=/usr/sbin/nginx

# Filter by command line
journalctl _COMM=nginx

# Filter by systemd unit
journalctl _SYSTEMD_UNIT=nginx.service

# Filter by systemd invocation ID
journalctl _SYSTEMD_INVOCATION_ID=abc123

# Filter by journal namespace
journalctl --namespace=myapp

# Multiple fields
journalctl _PID=1234 _UID=1000
```

### Process and User
```bash
# Filter by process name
journalctl /usr/sbin/nginx

# Filter by user
journalctl _UID=1000

# Filter by group
journalctl _GID=1000

# Filter by effective user
journalctl _EUID=1000
```

### Host and Machine
```bash
# Filter by host
journalctl _HOSTNAME=web-server-1

# Filter by machine ID
journalctl _MACHINE_ID=abc123

# Filter by container
journalctl _MACHINE=container-name

# Remote host
journalctl --host=remote-server
```

---

## 4. Viewing Options

### Line Count
```bash
# Show last N lines
journalctl -n 50

# Show all lines (verbose)
journalctl -a

# Show last N lines, follow
journalctl -n 100 -f
```

### Follow Mode
```bash
# Follow new entries
journalctl -f

# Follow specific service
journalctl -u nginx -f

# Follow with timestamp
journalctl -f --show-cursor

# Follow until match
journalctl -f --grep="error"
```

### Pagination
```bash
# Use pager (default)
journalctl

# No pager
journalctl --no-pager

# Use less with options
journalctl --pager-end

# Page with cursor
journalctl --cursor-file=/tmp/cursor
```

---

## 5. Format Options

### Output Formats
```bash
# Short format (default)
journalctl -o short

# Short with timestamps
journalctl -o short-iso

# Short with monotonic timestamps
journalctl -o short-monotonic

# Verbose format
journalctl -o verbose

# Export to JSON
journalctl -o json

# Pretty JSON
journalctl -o json-pretty

# Export to CBOR
journalctl -o cbor

# Export to export format
journalctl -o export

# Cat (raw message only)
journalctl -o cat

# With syslog facility
journalctl -o syslog
```

### Custom Formatting
```bash
# Show specific fields
journalctl -o json-pretty | jq '.MESSAGE,._PID'

# Export specific fields
journalctl -o export --fields=MESSAGE

# Custom template
journalctl -o template --template='{{.MESSAGE}}\n'
```

---

## 6. Log Management

### Disk Usage
```bash
# Show disk usage
journalctl --disk-usage

# Show verbose usage
journalctl --disk-usage -v

# Vacuum logs (keep last 100MB)
journalctl --vacuum-size=100M

# Vacuum logs (keep last 7 days)
journalctl --vacuum-time=7d

# Vacuum logs (keep last 100 entries)
journalctl --vacuum-size=100

# Remove old archives
journalctl --vacuum-files=5
```

### Rotation
```bash
# Rotate logs
journalctl --rotate

# Check archive files
ls -la /var/log/journal/

# Manual rotation
sudo kill -SIGUSR1 systemd-journald
```

### Persistent Storage
```bash
# Configure persistent storage
# /etc/systemd/journald.conf

[Journal]
Storage=persistent
SystemMaxUse=500M
RuntimeMaxUse=100M
MaxRetentionSec=30d
MaxFileSec=1month
```

### Clear Logs
```bash
# Clear all logs
sudo journalctl --rotate
sudo journalctl --vacuum-size=0

# Clear by time
sudo journalctl --vacuum-time=1d

# Clear by size
sudo journalctl --vacuum-size=10M
```

---

## 7. Priority Filtering

### Priority Levels
| Level | Value | Description |
|-------|-------|-------------|
| emerg | 0 | System unusable |
| alert | 1 | Action required |
| crit | 2 | Critical conditions |
| err | 3 | Error conditions |
| warning | 4 | Warning conditions |
| notice | 5 | Normal but significant |
| info | 6 | Informational |
| debug | 7 | Debug-level messages |

### Filter by Priority
```bash
# Show only errors and worse
journalctl -p err
journalctl -p error
journalctl -p 3

# Show warnings and above
journalctl -p warning
journalctl -p 4

# Show info and above
journalctl -p info
journalctl -p 6

# Show debug and above
journalctl -p debug
journalctl -p 7

# Multiple priorities
journalctl -p err,warning
journalctl -p 3..4

# Range of priorities
journalctl -p 0..4
```

### Priority in Output
```bash
# Color-coded output
journalctl -p err --color=always

# Highlight priorities
journalctl --level=warning
```

---

## 8. Advanced Queries

### Boolean Operators
```bash
# AND (both conditions)
journalctl _PID=1234 PRIORITY=3

# OR (multiple commands)
journalctl PRIORITY=3 | grep -E "pattern"

# NOT (grep -v)
journalctl | grep -v "INFO"

# Complex query
journalctl _UID=1000 PRIORITY=0..4
```

### Regex Matching
```bash
# Grep for pattern
journalctl | grep "error"

# Grep case insensitive
journalctl | grep -i "ERROR"

# Extended regex
journalctl | grep -E "error|warning"

# Find by message content
journalctl MESSAGE="Failed to start"

# Find in MESSAGE field
journalctl _MESSAGE="Failed to start"
```

### Cursor-Based
```bash
# Get cursor
journalctl --show-cursor

# Output cursor
journalctl -n 1 -o export | grep _CURSOR

# Seek to cursor
journalctl --cursor=CURSOR_VALUE

# Resume from cursor
journalctl --after-cursor=CURSOR_VALUE
```

### Batch Processing
```bash
# Count entries
journalctl --count | grep total

# Unique values
journalctl -o json | jq -r '._SYSTEMD_UNIT' | sort | uniq -c

# Statistics
journalctl -o json | jq '.'

# Export to file
journalctl > /tmp/logs.txt

# Export specific fields
journalctl -o json | jq -c '. | {unit:._SYSTEMD_UNIT, msg:.MESSAGE}' > logs.json
```

---

## 9. Troubleshooting

### Common Issues
```bash
# No logs showing
journalctl --system --no-pager

# Permission denied
sudo journalctl

# Logs not persisting
# Check /etc/systemd/journald.conf
# Storage=persistent

# Logs too large
journalctl --disk-usage
journalctl --vacuum-size=200M

# Service not logging
sudo systemctl status systemd-journald
sudo systemctl restart systemd-journald
```

### Debug Commands
```bash
# Check journald status
systemctl status systemd-journald

# View journald configuration
journalctl --system --grep=Journal

# Check kernel ring buffer
dmesg | tail -100

# Force flush
sudo kill -SIGUSR1 systemd-journald

# Test logging
logger "Test message from user"
journalctl --since "1 minute ago" | grep "Test"
```

### Network Diagnostics
```bash
# View network logs
journalctl -u NetworkManager

# View firewall logs
journalctl -u firewalld

# View DNS queries
journalctl -u systemd-resolved

# View connection issues
journalctl | grep -E "connection|failed|error"
```

### Performance
```bash
# Enable real-time indexing
systemctl set-property systemd-journald RuntimeMaxUse=100M

# Limit rate
# In journald.conf:
RateLimitInterval=30s
RateLimitBurst=1000

# Check indexing
systemctl status systemd-journald
```

---

## 10. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| View all | `journalctl` |
| Follow | `journalctl -f` |
| Service logs | `journalctl -u nginx` |
| Kernel | `journalctl -k` |
| Current boot | `journalctl -b` |
| Lines | `journalctl -n 50` |
| JSON | `journalctl -o json` |
| Disk usage | `journalctl --disk-usage` |
| Vacuum | `journalctl --vacuum-size=100M` |

### Priority Shortcuts
| Command | Priority |
|---------|----------|
| `journalctl -p 0` | emerg |
| `journalctl -p 1` | alert |
| `journalctl -p 2` | crit |
| `journalctl -p 3` | err |
| `journalctl -p 4` | warning |
| `journalctl -p 5` | notice |
| `journalctl -p 6` | info |
| `journalctl -p 7` | debug |

### Format Options
| Option | Description |
|--------|-------------|
| `-o short` | Default format |
| `-o verbose` | Full details |
| `-o json` | JSON format |
| `-o json-pretty` | Formatted JSON |
| `-o cat` | Message only |
| `-o export` | Export format |
| `-o short-iso` | ISO timestamps |

---

*Last Updated: January 2026*
