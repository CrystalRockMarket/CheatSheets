# systemctl Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Service Management](#2-service-management)
3. [Unit Management](#3-unit-management)
4. [Target Management](#4-target-management)
5. [Timer Units](#5-timer-units)
6. [Socket Management](#6-socket-management)
7. [Automount and Mount](#7-automount-and-mount)
8. [Scope Management](#8-scope-management)
9. [Environment and Resource Control](#9-environment-and-resource-control)
10. [Troubleshooting](#10-troubleshooting)
11. [Quick Reference](#11-quick-reference)

---

## 1. Introduction

systemctl is the central management tool for controlling the systemd init system and services.

### Key Concepts
| Concept | Description |
|---------|-------------|
| Units | Configuration files describing resources |
| Services | Daemon processes |
| Sockets | IPC sockets for activation |
| Timers | Scheduled tasks |
| Mounts | Filesystem mounts |
| Targets | Groups of units |

### Unit Types
| Type | Extension | Description |
|------|-----------|-------------|
| Service | .service | Daemon processes |
| Socket | .socket | Network/local sockets |
| Timer | .timer | Scheduled tasks |
| Path | .path | Filesystem paths |
| Mount | .mount | Mount points |
| Automount | .automount | Auto-mount points |
| Swap | .swap | Swap configuration |
| Target | .target | Groups of units |
| Device | .device | Device nodes |
| Scope | .scope | External processes |

---

## 2. Service Management

### Basic Commands
```bash
# Start service
sudo systemctl start nginx

# Stop service
sudo systemctl stop nginx

# Restart service
sudo systemctl restart nginx

# Reload configuration
sudo systemctl reload nginx

# Restart or reload
sudo systemctl reload-or-restart nginx

# Check status
systemctl status nginx

# Check if active
systemctl is-active nginx

# Check if enabled
systemctl is-enabled nginx

# Check if failed
systemctl is-failed nginx
```

### Enable/Disable
```bash
# Enable service (start at boot)
sudo systemctl enable nginx

# Disable service (don't start at boot)
sudo systemctl disable nginx

# Enable and start
sudo systemctl enable --now nginx

# Disable and stop
sudo systemctl disable --now nginx

# Mask service (prevent all activation)
sudo systemctl mask nginx
sudo systemctl unmask nginx

# Re-enable (reset symlinks)
sudo systemctl reenable nginx
```

### Service Status
```bash
# Full status
systemctl status nginx

# Active status only
systemctl is-active nginx
systemctl is-failed nginx
systemctl is-enabled nginx

# Active and enabled combined
systemctl status nginx --no-pager

# View service logs
journalctl -u nginx

# View service description
systemctl cat nginx

# View service dependencies
systemctl list-dependencies nginx
```

### Service Files
```bash
# Location of service files
# /etc/systemd/system/ (user-created)
# /usr/lib/systemd/system/ (distro-provided)

# Edit service file
sudo systemctl edit nginx

# Edit full service file
sudo systemctl edit --full nginx

# Show service file
systemctl cat nginx

# Show unit properties
systemctl show nginx
```

---

## 3. Unit Management

### List Units
```bash
# List all active units
systemctl list-units

# List all units (including inactive)
systemctl list-units --all

# List by type
systemctl list-units --type=service
systemctl list-units --type=socket
systemctl list-units --type=timer

# List by state
systemctl list-units --state=running
systemctl list-units --state=failed
systemctl list-units --state=active

# List with dependencies
systemctl list-units --type=service --all --no-pager
```

### Unit Files
```bash
# List unit files
systemctl list-unit-files

# List by state
systemctl list-unit-files --state=enabled
systemctl list-unit-files --state=disabled
systemctl list-unit-files --state=static
systemctl list-unit-files --state=masked

# Show unit file
systemctl cat sshd.service

# Edit unit file
sudo systemctl edit sshd.service

# Edit full unit file
sudo systemctl edit --full httpd.service

# Reload daemon (after changes)
sudo systemctl daemon-reload
```

### Unit Properties
```bash
# Show unit properties
systemctl show nginx

# Show specific property
systemctl show nginx -p MainPID
systemctl show nginx -p ExecStart

# Get unit property
systemctl property --value MainPID nginx
```

### Unit Dependencies
```bash
# List dependencies
systemctl list-dependencies nginx

# List reverse dependencies
systemctl list-dependencies --reverse nginx

# Show unit requires
systemctl show nginx -p Requires

# Show unit wants
systemctl show nginx -p Wants
```

---

## 4. Target Management

### Common Targets
```bash
# graphical.target - Graphical interface
# multi-user.target - Multi-user text mode
# rescue.target - Single user mode
# emergency.target - Emergency mode
# reboot.target - Reboot system
# poweroff.target - Power off
# halt.target - Halt system
```

### Target Commands
```bash
# Get default target
systemctl get-default

# Set default target
sudo systemctl set-default multi-user.target

# Get current target
systemctl get-default

# List targets
systemctl list-targets --all

# Isolate target (start all units in target)
sudo systemctl isolate rescue.target
```

### Change Target
```bash
# Change to rescue mode
sudo systemctl rescue

# Change to emergency mode
sudo systemctl emergency

# Reboot system
sudo systemctl reboot

# Power off
sudo systemctl poweroff

# Halt system
sudo systemctl halt

# Suspend system
sudo systemctl suspend

# Hibernate system
sudo systemctl hibernate

# Hybrid sleep
sudo systemctl hybrid-sleep
```

---

## 5. Timer Units

### Timer Commands
```bash
# List timers
systemctl list-timers

# List all timers
systemctl list-timers --all

# Start timer
sudo systemctl start backup.timer

# Stop timer
sudo systemctl stop backup.timer

# Enable timer
sudo systemctl enable backup.timer

# Disable timer
sudo systemctl disable backup.timer
```

### Timer Configuration
```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true
RandomizedDelaySec=30min
Unit=backup.service

[Install]
WantedBy=timers.target
```

### OnCalendar Syntax
```bash
# Every day at 2 AM
OnCalendar=*-*-* 02:00:00

# Every Monday at 9 AM
OnCalendar=Mon *-*-* 09:00:00

# Every 15 minutes
OnCalendar=*:0/15

# Every hour
OnCalendar=hourly

# Every day at noon
OnCalendar=*-*-* 12:00:00

# Weekly
OnCalendar=weekly

# Monthly
OnCalendar=monthly

# Yearly
OnCalendar=yearly

# Specific date
OnCalendar=2025-01-01 00:00:00

# Weekdays only
OnCalendar=Mon..Fri 09:00:00
```

---

## 6. Socket Management

### Socket Commands
```bash
# List sockets
systemctl list-sockets

# List all sockets
systemctl list-sockets --all

# Start socket
sudo systemctl start ssh.socket

# Stop socket
sudo systemctl stop ssh.socket

# Enable socket
sudo systemctl enable ssh.socket

# Disable socket
sudo systemctl disable ssh.socket
```

### Socket Configuration
```ini
# /etc/systemd/system/ssh.socket
[Unit]
Description=SSH Socket for systemd

[Socket]
ListenStream=22
Accept=yes

[Install]
WantedBy=sockets.target
```

---

## 7. Automount and Mount

### Mount Commands
```bash
# List mounts
systemctl list-mounts

# List automounts
systemctl list-automounts --all

# Start mount
sudo systemctl start home.mount

# Stop mount
sudo systemctl stop home.mount

# Enable mount
sudo systemctl enable home.mount
```

### Mount Configuration
```ini
# /etc/systemd/system/data.mount
[Unit]
Description=Data Disk Mount

[Mount]
What=/dev/sdb1
Where=/data
Type=ext4
Options=defaults

[Install]
WantedBy=multi-user.target
```

### Automount Configuration
```ini
# /etc/systemd/system/data.automount
[Unit]
Description=Data Disk Automount

[Automount]
Where=/data
DirectoryMode=0755

[Install]
WantedBy=multi-user.target
```

---

## 8. Scope Management

### Scope Commands
```bash
# List scopes
systemctl list-scope-units

# List all scopes
systemctl list-units --type=scope

# Show scope properties
systemctl show *.scope
```

### Scope Configuration
```bash
# Create transient scope
sudo systemd-run --scope -p MemoryMax=500M sleep 60

# Run in scope
sudo systemd-run --scope --unit=my-app ./my-app
```

---

## 9. Environment and Resource Control

### Environment Variables
```bash
# Show environment
systemctl show-environment

# Set environment variable
sudo systemctl set-environment VAR=value

# Unset environment variable
sudo systemctl unset-environment VAR
```

### Resource Control
```bash
# Show unit properties
systemctl show nginx -p MemoryMax
systemctl show nginx -p CPUQuota

# Set memory limit (in bytes)
sudo systemctl set-property nginx MemoryMax=500M

# Set CPU limit (percentage)
sudo systemctl set-property nginx CPUQuota=50%

# Reset properties
sudo systemctl revert nginx
```

### Resource Limits
```ini
# In service file
[Service]
MemoryMax=512M
MemoryHigh=400M
MemorySwapMax=0
CPUQuota=50%
IOReadBandwidthMax=/dev/sda 10M
IOWriteBandwidthMax=/dev/sda 10M
TasksMax=100
```

---

## 10. Troubleshooting

### Check Service Status
```bash
# Full status
systemctl status nginx

# Check active state
systemctl is-active nginx

# Check failed state
systemctl is-failed nginx

# Check enabled state
systemctl is-enabled nginx

# View recent logs
journalctl -u nginx -n 50

# Follow logs
journalctl -u nginx -f

# Check for errors
journalctl -u nginx --since today | grep ERROR
```

### Common Issues
```bash
# Service won't start
systemctl status nginx
journalctl -u nginx

# Check configuration
systemctl cat nginx

# Reload after changes
sudo systemctl daemon-reload

# Reset failed state
sudo systemctl reset-failed

# Check dependencies
systemctl list-dependencies nginx
```

### Emergency Mode
```bash
# Boot to rescue mode
# Add systemd.unit=rescue.target to kernel cmdline

# In rescue mode
# Root filesystem read-only
mount -o remount,rw /

# Enable networking
systemctl start NetworkManager

# Exit rescue mode
exit
```

### View Logs
```bash
# All logs
journalctl

# Kernel messages
journalctl -k

# Service logs
journalctl -u nginx

# Last 100 entries
journalctl -n 100

# Follow
journalctl -f

# Since specific time
journalctl --since "2025-01-01 00:00:00"

# Until specific time
journalctl --until "2025-01-01 12:00:00"

# By priority
journalctl -p err
journalctl -p warning
```

---

## 11. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| Start service | `systemctl start nginx` |
| Stop service | `systemctl stop nginx` |
| Restart | `systemctl restart nginx` |
| Reload | `systemctl reload nginx` |
| Status | `systemctl status nginx` |
| Enable | `systemctl enable nginx` |
| Disable | `systemctl disable nginx` |
| Enable now | `systemctl enable --now nginx` |
| Is active | `systemctl is-active nginx` |
| Is enabled | `systemctl is-enabled nginx` |

### Service File Locations
| Location | Description |
|----------|-------------|
| /etc/systemd/system/ | User-created |
| /usr/lib/systemd/system/ | Distro-provided |
| /run/systemd/system/ | Runtime |

### Common Unit Options
| Option | Description |
|--------|-------------|
| ExecStart | Command to start |
| ExecStop | Command to stop |
| ExecReload | Command to reload |
| Restart | Auto-restart policy |
| User | Run as user |
| Group | Run as group |
| Environment | Environment variables |
| WorkingDirectory | Working directory |

---

*Last Updated: January 2026*
