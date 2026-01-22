# UFW Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Basic Commands](#3-basic-commands)
4. [Rule Management](#4-rule-management)
5. [Application Profiles](#5-application-profiles)
6. [IPv6](#6-ipv6)
7. [Logging](#7-logging)
8. [Advanced Configuration](#8-advanced-configuration)
9. [Troubleshooting](#9-troubleshooting)
10. [Quick Reference](#10-quick-reference)

---

## 1. Introduction

UFW (Uncomplicated Firewall) is a user-friendly frontend for iptables, designed to make firewall management simple and intuitive.

### Key Features
| Feature | Description |
|---------|-------------|
| Simple Syntax | Easy-to-understand commands |
| IPv6 Support | Full IPv6 firewall rules |
| Application Profiles | Pre-configured application rules |
| Logging | Configurable logging levels |
| Rate Limiting | Built-in DDoS protection |
| Integration | Works with iptables |

### Architecture
```
┌─────────────────────────────────────────┐
│              UFW (Frontend)              │
│                                         │
│  User-friendly commands                 │
│  ↓                                      │
│  Translation layer                      │
│  ↓                                      │
│           iptables (Backend)            │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│           Kernel Netfilter              │
└─────────────────────────────────────────┘
```

---

## 2. Installation

### Installation
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ufw

# RHEL/CentOS/Fedora
sudo dnf install ufw

# Arch Linux
sudo pacman -S ufw

# Check version
ufw --version
```

### Initial Setup
```bash
# Enable IPv6 support
sudo ufw enable

# Enable at boot
sudo systemctl enable ufw

# Start service
sudo systemctl start ufw

# Check status
sudo ufw status
sudo ufw status verbose
sudo ufw status numbered
```

---

## 3. Basic Commands

### Enabling/Disabling
```bash
# Enable UFW
sudo ufw enable

# Disable UFW
sudo ufw disable

# Reload UFW
sudo ufw reload

# Reset UFW to defaults
sudo ufw reset
```

### Status Commands
```bash
# Check status
sudo ufw status

# Verbose status
sudo ufw status verbose

# Numbered status
sudo ufw status numbered

# Show raw rules
sudo ufw status raw
```

### Default Policies
```bash
# Set incoming policy (default: deny)
sudo ufw default deny incoming

# Set outgoing policy (default: allow)
sudo ufw default allow outgoing

# Set routed policy
sudo ufw default deny routed
```

---

## 4. Rule Management

### Basic Rules
```bash
# Allow SSH
sudo ufw allow ssh
sudo ufw allow 22

# Allow specific port
sudo ufw allow 80

# Allow port range
sudo ufw allow 8000:8100/tcp

# Deny port
sudo ufw deny 8080

# Reject with message
sudo ufw reject 21

# Delete rule
sudo ufw delete allow 80

# Delete by rule number
sudo ufw delete 5
```

### Protocol-Specific Rules
```bash
# Allow TCP only
sudo ufw allow 443/tcp

# Allow UDP only
sudo ufw allow 53/udp

# Allow both TCP and UDP
sudo ufw allow 53
```

### IP-Based Rules
```bash
# Allow from specific IP
sudo ufw allow from 192.168.1.100

# Allow from IP to specific port
sudo ufw allow from 192.168.1.100 to any port 22

# Allow from subnet
sudo ufw allow from 192.168.1.0/24

# Allow from subnet to port
sudo ufw allow from 192.168.1.0/24 to any port 3306

# Deny from IP
sudo ufw deny from 10.0.0.5

# Deny from subnet
sudo ufw deny from 172.16.0.0/12
```

### Interface-Based Rules
```bash
# Allow on specific interface
sudo ufw allow in on eth0 to any port 80

# Allow SSH on eth1
sudo ufw allow in on eth1 to any port 22

# Allow from subnet on interface
sudo ufw allow in on wlan0 from 192.168.1.0/24
```

### Incoming/Outgoing
```bash
# Allow outgoing
sudo ufw allow out 53

# Deny incoming
sudo ufw deny in 8080

# Allow outgoing to specific IP
sudo ufw allow out to 8.8.8.8 port 53

# Deny incoming from specific IP
sudo ufw deny in from 203.0.113.50
```

---

## 5. Application Profiles

### Managing Profiles
```bash
# List application profiles
sudo ufw app list

# Show profile info
sudo ufw app info OpenSSH

# Show profile rules
sudo ufw app info Nginx Full
```

### Common Application Profiles
```bash
# Apache
sudo ufw allow 'Apache'
sudo ufw allow 'Apache Full'
sudo ufw allow 'Apache Secure'

# Nginx
sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS'
sudo ufw allow 'Nginx Full'

# OpenSSH
sudo ufw allow 'OpenSSH'

# CUPS
sudo ufw allow 'CUPS'

# Samba
sudo ufw allow 'Samba'
```

### Create Custom Profile
```bash
# /etc/ufw/applications.d/custom-app
[Custom App]
title=My Custom Application
description=My custom app
ports=9000,9001/tcp

# Apply profile
sudo ufw allow 'Custom App'
```

---

## 6. IPv6

### IPv6 Configuration
```bash
# Enable IPv6
sudo ufw enable

# Check IPv6 status
sudo ufw status verbose

# IPv6 is enabled by default in modern UFW
# Rules automatically apply to IPv6 if IPv6 is enabled in system
```

### IPv6-Specific Rules
```bash
# Allow IPv6
sudo ufw allow in from ::/0 to any port 80

# Allow specific IPv6 address
sudo ufw allow in from 2001:db8::1 to any port 22

# Deny IPv6
sudo ufw deny in from 2001:db8::/32
```

---

## 7. Logging

### Enable Logging
```bash
# Enable logging
sudo ufw logging on

# Set log level
sudo ufw logging low      # All blocked packets
sudo ufw logging medium   # All allowed + blocked
sudo ufw logging high     # All packets + rate limiting
sudo ufw logging off

# Custom log location
sudo ufw logging high /var/log/ufw.log
```

### View Logs
```bash
# Real-time log
sudo tail -f /var/log/ufw.log

# Last 50 entries
sudo ufw logging high
sudo tail -50 /var/log/ufw.log

# Search logs
grep "UFW" /var/log/syslog
grep "BLOCK" /var/log/ufw.log
```

---

## 8. Advanced Configuration

### Rate Limiting
```bash
# Rate limit SSH (6 connections per 30 seconds)
sudo ufw limit 22/tcp

# Rate limit HTTP
sudo ufw limit 80/tcp

# Rate limit with specific IP
sudo ufw limit from 192.168.1.0/24 to any port 22
```

### Connection Tracking
```bash
# Allow established connections
sudo ufw allow out to any port 443
sudo ufw allow in established

# Allow related connections
sudo ufw allow in established related
```

### Precedence Rules
```bash
# Rules are processed in order
# First match wins

# Allow SSH but deny specific IP
sudo ufw allow ssh
sudo ufw deny from 192.168.1.100

# Note: This won't work as expected
# Allow rule must come after deny for specific IP
```

### Using Before/After Rules
```bash
# /etc/ufw/before.rules
# Rules added here are processed before UFW rules

# /etc/ufw/after.rules
# Rules added here are processed after UFW rules

# Example in before.rules
-A ufw-before-input -p icmp --icmp-type echo-request -j DROP
```

### Configuration File
```bash
# Main configuration
/etc/default/ufw

# Rules files
/etc/ufw/before.rules
/etc/ufw/after.rules
/etc/ufw/user.rules
/etc/ufw/user6.rules
```

---

## 9. Troubleshooting

### Check Firewall Status
```bash
# Basic status
sudo ufw status

# Verbose
sudo ufw status verbose

# Numbered
sudo ufw status numbered

# Raw
sudo ufw status raw
```

### Test Connectivity
```bash
# From remote host
nmap -p 80 your-server-ip

# Check if port is open
nc -zv your-server-ip 80

# Test SSH access
ssh -p 22 your-server-ip
```

### View Detailed Rules
```bash
# iptables rules
sudo iptables -L -n -v

# iptables with line numbers
sudo iptables -L -n --line-numbers

# IPv6 rules
sudo ip6tables -L -n -v

# UFW raw rules
sudo ufw status raw
```

### Common Issues
```bash
# UFW not starting
sudo systemctl status ufw
sudo journalctl -u ufw

# Connection refused
# Check if service is running
# Check port is allowed
sudo ufw allow 80/tcp

# Too many connections
# Enable rate limiting
sudo ufw limit 22/tcp

# Cannot connect after enable
# Allow SSH before enabling
sudo ufw allow ssh
sudo ufw enable
```

### Reset and Start Fresh
```bash
# Reset UFW
sudo ufw reset

# Set defaults
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Add essential rules
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

# Enable
sudo ufw enable
```

---

## 10. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| Enable | `sudo ufw enable` |
| Disable | `sudo ufw disable` |
| Status | `sudo ufw status` |
| Verbose | `sudo ufw status verbose` |
| Allow port | `sudo ufw allow 80` |
| Deny port | `sudo ufw deny 80` |
| Allow SSH | `sudo ufw allow ssh` |
| Allow IP | `sudo ufw allow from 192.168.1.100` |
| Delete rule | `sudo ufw delete allow 80` |
| Reset | `sudo ufw reset` |

### Port Numbers Reference
| Service | Port | Command |
|---------|------|---------|
| SSH | 22 | `ufw allow ssh` |
| HTTP | 80 | `ufw allow http` |
| HTTPS | 443 | `ufw allow https` |
| FTP | 21 | `ufw allow 21/tcp` |
| MySQL | 3306 | `ufw allow 3306` |
| PostgreSQL | 5432 | `ufw allow 5432` |
| Redis | 6379 | `ufw allow 6379` |
| MongoDB | 27017 | `ufw allow 27017` |
| Docker | 2375/2376 | `ufw allow 2375/tcp` |

### Log Levels
| Level | Description |
|-------|-------------|
| off | No logging |
| low | Log blocked packets |
| medium | Log allowed + blocked |
| high | All packets + rate limiting |

---

*Last Updated: January 2026*
