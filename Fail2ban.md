# Fail2ban

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Architecture](#architecture)
4. [Configuration Files](#configuration-files)
5. [Jails](#jails)
6. [Actions](#actions)
7. [Filters](#filters)
8. [Commands](#commands)
9. [Custom Rules](#custom-rules)
10. [Troubleshooting](#troubleshooting)
11. [Quick Reference](#quick-reference)

---

## Introduction

Fail2ban is an intrusion prevention software framework that protects servers from brute-force attacks by monitoring log files and banning IP addresses that show malicious patterns.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Fail2ban Architecture                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                     fail2ban-server                        │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌───────────────────┐  │  │
│  │  │  Poller     │  │  Action     │  │   Ban/Unban       │  │  │
│  │  │  (Log Read) │──│  Executor   │──│   Manager         │  │  │
│  │  └─────────────┘  └─────────────┘  └───────────────────┘  │  │
│  │         │                │                   │             │  │
│  └─────────┼────────────────┼───────────────────┼─────────────┘  │
│            │                │                   │                 │
│            ▼                ▼                   ▼                 │
│  ┌─────────────┐  ┌─────────────────┐  ┌───────────────────┐   │
│  │ Log Files   │  │ Action Commands │  │   iptables/nft    │   │
│  │ /var/log/   │  │ (sendmail, etc) │  │   (banning)       │   │
│  └─────────────┘  └─────────────────┘  └───────────────────┘   │
│                                                                 │
│  Client <--> Server (IPC) <--> Filters + Actions               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Features

- Real-time log monitoring
- Configurable ban actions (iptables, firewalld, nftables, etc.)
- Multiple authentication failure detection
- Time-based unbanning (configurable)
- Notification system (email, Slack, etc.)
- Whitelist support
- Multi-threaded operation

---

## Installation

### Linux Installation

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install fail2ban

# RHEL/CentOS/Fedora
sudo yum install epel-release
sudo yum install fail2ban-firewalld
sudo systemctl enable --now fail2ban

# Arch Linux
sudo pacman -S fail2ban

# openSUSE
sudo zypper install fail2ban

# From source
git clone https://github.com/fail2ban/fail2ban.git
cd fail2ban
sudo python setup.py install
```

### Verify Installation

```bash
# Check version
fail2ban-client version

# Check status
sudo fail2ban-client status

# Check service status
sudo systemctl status fail2ban

# Check configuration
sudo fail2ban-client -c /etc/fail2ban version
```

### Initial Configuration

```bash
# Create required directories
sudo mkdir -p /var/run/fail2ban
sudo mkdir -p /var/lib/fail2ban

# Ensure proper permissions
sudo chown -R root:root /etc/fail2ban
sudo chmod 755 /etc/fail2ban

# Start service
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

---

## Architecture

### Components

| Component | Description |
|-----------|-------------|
| **fail2ban-server** | Core daemon that monitors and bans |
| **fail2ban-client** | Command-line interface |
| **Filters** | Regex patterns to identify failures |
| **Actions** | Commands to execute when banning |
| **Jails** | Combinations of filters and actions |

### Log Processing Flow

```
Log Entry → Filter Match → Failure Count → Ban Trigger → Action Execute
              │               │               │              │
              ▼               ▼               ▼              ▼
         Regex Pattern   maxretry=3     Bantime=600    iptables -A
         findfailure     findtime=600   (10 minutes)  -j DROP
```

### Default Paths

| Path | Description |
|------|-------------|
| `/etc/fail2ban/` | Configuration directory |
| `/etc/fail2ban/jail.conf` | Main configuration |
| `/etc/fail2ban/jail.local` | Local overrides |
| `/etc/fail2ban/filter.d/` | Filter patterns |
| `/etc/fail2ban/action.d/` | Action scripts |
| `/var/log/fail2ban.log` | Fail2ban log file |
| `/var/lib/fail2ban/` | Database and state |

---

## Configuration Files

### Configuration Hierarchy

```
/etc/fail2ban/
├── jail.conf           # Default configuration (DO NOT EDIT)
├── jail.local          # Local overrides (CREATE THIS)
├── fail2ban.conf       # Fail2ban daemon settings
├── filter.d/           # Filter patterns
│   ├── sshd.conf
│   ├── apache-auth.conf
│   └── ...
├── action.d/           # Action scripts
│   ├── iptables.conf
│   ├── sendmail.conf
│   └── ...
└── paths-debian.conf   # OS-specific paths
```

### Main Configuration (jail.local)

```ini
[DEFAULT]
# Global ignore IPs (whitelist)
ignoreip = 127.0.0.1/8 ::1 192.168.1.0/24 10.0.0.0/8

# Default ban time (seconds) - -1 = permanent
bantime = 600

# Time window for failure counting (seconds)
findtime = 600

# Maximum failures before ban
maxretry = 5

# Default action
action = %(action_)s

# Ban mechanism
banaction = iptables-multiport

# Backend for log monitoring
backend = pyinotify

# Log level
loglevel = INFO

# Log target
logtarget = /var/log/fail2ban.log

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
findtime = 600
bantime = 3600
action = %(action_mwl)s

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
port = 80,443
logpath = /var/log/nginx/error.log
maxretry = 3
bantime = 3600
```

### Server Configuration (fail2ban.conf)

```ini
[Definition]
# Log level options: CRITICAL, ERROR, WARNING, NOTICE, INFO, DEBUG
loglevel = INFO

# Log target: STDOUT, SYSLOG, FILE
logtarget = /var/log/fail2ban.log

# Socket file location
socket = /var/run/fail2ban/fail2ban.sock

# PID file location
pidfile = /var/run/fail2ban/fail2ban.pid

# Database file
dbfile = /var/lib/fail2ban/fail2ban.sqlite3

# Force socket access mode
socketmode = 0660
```

### Action Configuration

```ini
# /etc/fail2ban/action.d/iptables.conf
[Definition]
# Actions to take when banning
actionstart = iptables -N fail2ban-<name>
              iptables -A fail2ban-<name> -j RETURN
              iptables -I INPUT -p <protocol> -m multiport --dports <port> -j fail2ban-<name>

actionstop = iptables -D INPUT -p <protocol> -m multiport --dports <port> -j fail2ban-<name>
             iptables -F fail2ban-<name>
             iptables -X fail2ban-<name>

actioncheck = iptables -n -L INPUT | grep -q fail2ban-<name>

actionban = iptables -I fail2ban-<name> 1 -s <ip> -j DROP

actionunban = iptables -D fail2ban-<name> -s <ip> -j DROP

[Init]
# Default port, protocol
name = default
protocol = tcp
port = ssh
```

---

## Jails

### Jail Structure

```ini
[jail-name]
enabled = true
port = port[,port...]
filter = filter-name
logpath = /path/to/log/file
maxretry = 5           # failures before ban
findtime = 600         # time window (seconds)
bantime = 600          # ban duration (seconds)
action = action-name   # action to execute
```

### Common Jails

```ini
# SSH Jail
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
findtime = 600
bantime = 3600
action = %(action_mwl)s

# SSH with IPv6
[sshd-ddos]
enabled = true
port = ssh
filter = sshd-ddos
logpath = /var/log/auth.log
maxretry = 6
findtime = 600
bantime = 3600

# Apache Authentication
[apache-auth]
enabled = true
port = http,https
filter = apache-auth
logpath = /var/log/apache2/error.log
maxretry = 3
bantime = 3600

# Nginx Authentication
[nginx-http-auth]
enabled = true
port = 80,443
filter = nginx-http-auth
logpath = /var/log/nginx/error.log
maxretry = 3
bantime = 3600

# Apache DoS (Slowloris protection)
[apache-noscript]
enabled = true
port = http,https
filter = apache-noscript
logpath = /var/log/apache2/access.log
maxretry = 6
findtime = 60
bantime = 3600

# Nginx DoS
[nginx-dos]
enabled = true
port = 80,443
filter = nginx-dos
logpath = /var/log/nginx/access.log
maxretry = 100
findtime = 60
bantime = 3600

# ProFTPD
[proftpd]
enabled = true
port = ftp,ftp-data,ftps,ftps-data
filter = proftpd
logpath = /var/log/proftpd/proftpd.log
maxretry = 3
bantime = 3600

# Dovecot (IMAP/POP3)
[dovecot]
enabled = true
port = imap,imaps,pop3,pop3s
filter = dovecot
logpath = /var/log/dovecot.log
maxretry = 3
bantime = 3600

# Postfix
[postfix]
enabled = true
port = smtp,submission,smtps
filter = postfix
logpath = /var/log/mail.log
maxretry = 3
bantime = 3600

# MySQL/MariaDB
[mysqld]
enabled = true
port = 3306
filter = mysqld-auth
logpath = /var/log/mysql/error.log
maxretry = 3
bantime = 3600

# Recidive (ban repeat offenders longer)
[recidive]
enabled = true
filter = recidive
logpath = /var/log/fail2ban.log
action = iptables-allports[name=recidive]
         sendmail-whois-lines[name=recidive]
maxretry = 3
findtime = 86400    # 1 day
bantime = 604800    # 1 week
```

### Custom Jail Example

```ini
# /etc/fail2ban/jail.local
[custom-ssh]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
findtime = 300
bantime = 1800
action = iptables[name=SSH, port=2222, protocol=tcp]
         sendmail-whois[name=SSH, dest=admin@example.com]

# Web application login
[webapp-login]
enabled = true
port = http,https
filter = webapp-login
logpath = /var/log/webapp/access.log
maxretry = 5
findtime = 300
bantime = 1800
action = iptables-multiport[name=webapp, port="80,443"]

# API rate limiting
[api-limit]
enabled = true
port = 8000
filter = api-limit
logpath = /var/log/api/access.log
maxretry = 100
findtime = 60
bantime = 300
action = iptables[name=API, port=8000, protocol=tcp]
```

---

## Actions

### Built-in Actions

```ini
# iptables (default)
action = iptables[name=%(__name__)s, port=%(port)s, protocol=tcp]

# iptables with multiport
action = iptables-multiport[name=%(__name__)s, port="%(port)s", protocol=tcp]

# iptables with whois notification
action = iptables-whois[name=%(__name__)s, dest="admin@example.com"]

# iptables with sendmail
action = iptables-whois[name=%(__name__)s, sender=fail2ban@example.com]

# firewalld
action = firewalld[name=%(__name__)s, port="%(port)s", protocol=tcp]

# nftables
action = nftables[name=%(__name__)s, table=filter, chain=input, port="%(port)s"]

# CSF (ConfigServer Firewall)
action = csf[name=%(__name__)s]

# CloudFlare
action = cloudflare[cfuser=your@email.com, cftoken=yourapikey]
```

### Email Notification Actions

```ini
# Sendmail with whois
action_mwl = %(action_)s
             sendmail-whois-lines[name=%(name)s, dest="%(destemail)s", sender="%(sender)s", logpath=%(logpath)s]

# Multiple actions
action = %(action_mw)s
         slack[channel="#security", webhook_url="https://hooks.slack.com/..."]
```

### Custom Action Example

```ini
# /etc/fail2ban/action.d/slack.conf
[Definition]
actionstart = curl -s -X POST -d 'payload={"channel": "#security", "username": "Fail2Ban", "icon_emoji": ":rotating_light:", "text": "Fail2Ban started"}' <webhook_url>

actionstop = curl -s -X POST -d 'payload={"channel": "#security", "username": "Fail2Ban", "icon_emoji": ":white_check_mark:", "text": "Fail2Ban stopped"}' <webhook_url>

actionban = curl -s -X POST -d 'payload={"channel": "#security", "username": "Fail2Ban", "icon_emoji": ":no_entry_sign:", "text": "Banned <ip> for %(bantime)s seconds after %(failures)s failures on %(name)s"}' <webhook_url>

actionunban = curl -s -X POST -d 'payload={"channel": "#security", "username": "Fail2Ban", "icon_emoji": ":leftwards_arrow_with_hook:", "text": "Unbanned <ip> on %(name)s"}' <webhook_url>

[Init]
webhook_url = https://hooks.slack.com/services/YOUR/WEBHOOK/URL
```

---

## Filters

### Filter Structure

```ini
[Definition]
# Failregex: Pattern to match failed attempts
failregex = Failed password for .* from <HOST>

# Ignore regex: Pattern to ignore
ignoreregex =

# Notes added to logs
notes = Invalid user
```

### SSH Filter

```ini
# /etc/fail2ban/filter.d/sshd.conf
[INCLUDES]
before = common.conf

[Definition]
# Match various SSH failure patterns
failregex = ^%(__prefix_line)sFailed (?:publickey|password) for (?:invalid user )?<HOST> from <IP>(?: port \d+)?(?: ssh\d+)?$
            ^%(__prefix_line)sFailed (?:publickey|password) for <HOST> from <IP>(?: port \d+)?(?: ssh\d+)?$
            ^%(__prefix_line)sROOT LOGIN REFUSED.*from <HOST>\s*$
            ^%(__prefix_line)s[iI]nvalid user .* from <HOST>\s*$
            ^%(__prefix_line)sDisconnected from (?:invalid user )?<HOST> port \d+(?: ssh\d+)?$
            ^%(__prefix_line)sReceived disconnect: 11: .+ from <HOST>(?: port \d+)?$

ignoreregex =
```

### Apache Filter

```ini
# /etc/fail2ban/filter.d/apache-auth.conf
[Definition]
failregex = ^%(_apache_error_client)s (AH01617| AH01618): .*client <HOST>(?: port \d+)?\s*$
            ^%(_apache_error_client)s (AH01797| AH01775| AH01774): .*client <HOST>(?: port \d+)?\s*$
            ^%(_apache_error_client)s (AH01416| AH01335): .*client <HOST>(?: port \d+)?\s*$

ignoreregex =

[Init]
apache_error_client = \[\] \S+ \S+ \S+ \[client <HOST>:\d+\]
```

### Custom Filter Example

```ini
# /etc/fail2ban/filter.d/webapp-login.conf
[Definition]
# Match web application login failures
failregex = ^\s*<HOST> - - \[.*\] "POST /login.* 401
            ^\s*<HOST> - - \[.*\] "POST /api/auth.* 401
            ^\s*<HOST> - - \[.*\] "POST /wp-login.php.* 200

ignoreregex =

# Pattern for log format: 127.0.0.1 - - [10/Jan/2024:12:00:00 +0000] "POST /login HTTP/1.1" 401
```

### Filter Testing

```bash
# Test filter against log file
fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf

# Test with ignore regex
fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf --ignoreregex /etc/fail2ban/filter.d/common.conf

# Test custom filter
fail2ban-regex /var/log/webapp/access.log /etc/fail2ban/filter.d/webapp-login.conf

# Show matched lines
fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf -v

# Count matches
fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf 2>&1 | grep "Matched"
```

---

## Commands

### fail2ban-client Commands

```bash
# Status
sudo fail2ban-client status                    # Overall status
sudo fail2ban-client status sshd              # Specific jail status
sudo fail2ban-client status --ips             # Show banned IPs

# Start/Stop
sudo fail2ban-client start                    # Start server
sudo fail2ban-client stop                     # Stop server
sudo fail2ban-client reload                   # Reload configuration
sudo fail2ban-client reload jail-name         # Reload specific jail

# Ban/Unban
sudo fail2ban-client set sshd banip 192.168.1.100   # Ban IP
sudo fail2ban-client set sshd unbanip 192.168.1.100 # Unban IP
sudo fail2ban-client set sshd banip 192.168.1.0/24  # Ban subnet

# Add to whitelist
sudo fail2ban-client set sshd addignoreip 10.0.0.5

# Remove from whitelist
sudo fail2ban-client set sshd delignoreip 10.0.0.5

# View whitelist
sudo fail2ban-client get sshd ignoreip

# Set parameters
sudo fail2ban-client set sshd bantime 3600    # Set ban time
sudo fail2ban-client set sshd maxretry 3      # Set max retries
sudo fail2ban-client set sshd findtime 600    # Set find time

# Get parameters
sudo fail2ban-client get sshd bantime
sudo fail2ban-client get sshd maxretry
sudo fail2ban-client get sshd findtime

# Check if IP is banned
sudo fail2ban-client sshd getbanip | grep 192.168.1.100

# View all banned IPs for jail
sudo fail2ban-client get sshd banip

# View banned IPs with extra info
sudo fail2ban-client get sshd banip --with-time
```

### fail2ban-server Commands

```bash
# Check server status
sudo fail2ban-server status

# Ping server
sudo fail2ban-client ping

# Version check
sudo fail2ban-client version

# Get log
sudo fail2ban-client get logtarget
sudo fail2ban-client set logtarget /var/log/fail2ban.log

# Get log level
sudo fail2ban-client get loglevel
sudo fail2ban-client set loglevel DEBUG
```

### systemctl Commands

```bash
# Service management
sudo systemctl start fail2ban
sudo systemctl stop fail2ban
sudo systemctl restart fail2ban
sudo systemctl reload fail2ban

# Enable on boot
sudo systemctl enable fail2ban
sudo systemctl disable fail2ban

# Check status
sudo systemctl status fail2ban
sudo systemctl is-enabled fail2ban
```

### Other Commands

```bash
# Check configuration syntax
sudo fail2ban-client -c /etc/fail2ban check

# Validate jail configuration
sudo fail2ban-client check jail-name

# View logs
tail -f /var/log/fail2ban.log

# Test filter
fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf

# Generate recidive report
sudo fail2ban-client recidive status
```

---

## Custom Rules

### Creating Custom Jails

```bash
# Edit jail.local
sudo nano /etc/fail2ban/jail.local

# Add custom jail at the end
[wordpress]
enabled = true
port = http,https
filter = wordpress
logpath = /var/log/nginx/access.log
maxretry = 5
findtime = 600
bantime = 3600
action = iptables-multiport[name=wordpress, port="80,443"]
         sendmail-whois[name=wordpress, dest=admin@example.com]
```

### Creating Custom Filters

```bash
# Create filter file
sudo nano /etc/fail2ban/filter.d/wordpress.conf

# Add filter pattern
[Definition]
failregex = ^<HOST> - - \[.*\] "POST /wp-login.php
            ^<HOST> - - \[.*\] "POST /xmlrpc.php
            ^<HOST> - - \[.*\] "GET /wp-admin/admin-ajax.php.* 200

ignoreregex =
```

### Creating Custom Actions

```bash
# Create action file
sudo nano /etc/fail2ban/action.d/custom.conf

# Add action definition
[Definition]
actionstart = echo "Fail2Ban custom action started" | tee -a /var/log/fail2ban-custom.log
actionstop = echo "Fail2Ban custom action stopped" | tee -a /var/log/fail2ban-custom.log
actionban = echo "Banned <ip> at $(date)" | tee -a /var/log/fail2ban-custom.log
actionunban = echo "Unbanned <ip> at $(date)" | tee -a /var/log/fail2ban-custom.log
```

### Whitelist Management

```ini
# In jail.local
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 10.0.0.0/8 192.168.0.0/16 172.16.0.0/12
           # Office IP
           203.0.113.0/24
           # Admin IPs
           198.51.100.50

# Per-jail whitelist
[sshd]
enabled = true
ignoreip = 127.0.0.1/8 10.0.0.5 192.168.1.100
```

### Time-based Unbanning

```bash
# Temporary unban
sudo fail2ban-client set sshd unbanip 192.168.1.100

# Check when IP will be unbanned
sudo fail2ban-client get sshd banip --with-time | grep 192.168.1.100
```

---

## Troubleshooting

### Common Issues

**Fail2ban Won't Start**

```bash
# Check for configuration errors
sudo fail2ban-client -c /etc/fail2ban check

# Check log file
tail -50 /var/log/fail2ban.log

# Check port conflicts
sudo netstat -tlnp | grep fail2ban

# Check socket permissions
ls -la /var/run/fail2ban/

# Fix socket permissions
sudo chown root:fail2ban /var/run/fail2ban
sudo chmod 660 /var/run/fail2ban/fail2ban.sock
```

**No Banning Occurring**

```bash
# Check jail status
sudo fail2ban-client status

# Check filter is working
fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf

# Check jail configuration
sudo fail2ban-client get sshd logpath
sudo fail2ban-client get sshd filter

# Verify log file exists and is readable
ls -la /var/log/auth.log
sudo cat /var/log/auth.log | tail -5

# Check ignore IP settings
sudo fail2ban-client get sshd ignoreip

# Check maxretry and findtime
sudo fail2ban-client get sshd maxretry
sudo fail2ban-client get sshd findtime
```

**Incorrect Ban Times**

```bash
# Check current bantime
sudo fail2ban-client get sshd bantime

# Check for persistent bans in database
sudo fail2ban-client get sshd banip --with-time

# Check database for permanent bans
sqlite3 /var/lib/fail2ban/fail2ban.sqlite3 \
  "SELECT * FROM bans WHERE jail='sshd';"
```

**Email Not Sending**

```bash
# Test sendmail
echo "Test" | sudo sendmail -v admin@example.com

# Check mail logs
sudo tail -20 /var/log/mail.log

# Check fail2ban email settings
sudo fail2ban-client get sshd action
```

### Monitoring and Debugging

```bash
# Increase log level
sudo fail2ban-client set loglevel DEBUG

# Watch fail2ban logs
tail -f /var/log/fail2ban.log

# Monitor real-time bans
sudo fail2ban-client status | grep "Banned"

# Check iptables rules
sudo iptables -L fail2ban-sshd -n -v

# List all fail2ban chains
sudo iptables -L | grep fail2ban

# Check firewall backend
sudo fail2ban-client get iptables action

# Test if IP is actually blocked
sudo fail2ban-client set sshd banip 127.0.0.1
# Try SSH to localhost
# Should fail or timeout
sudo fail2ban-client set sshd unbanip 127.0.0.1
```

### Reset Everything

```bash
# Unban all IPs
sudo fail2ban-client unban --all

# Restart fail2ban
sudo systemctl restart fail2ban

# Clear database
sudo rm /var/lib/fail2ban/fail2ban.sqlite3
sudo systemctl restart fail2ban
```

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| `fail2ban-client status` | Overall status |
| `fail2ban-client status <jail>` | Specific jail status |
| `fail2ban-client set <jail> banip <ip>` | Ban IP |
| `fail2ban-client set <jail> unbanip <ip>` | Unban IP |
| `fail2ban-client reload` | Reload config |
| `fail2ban-regex <log> <filter>` | Test filter |
| `fail2ban-client ping` | Ping server |

### Configuration Files

| File | Purpose |
|------|---------|
| `/etc/fail2ban/jail.conf` | Default config |
| `/etc/fail2ban/jail.local` | Local overrides |
| `/etc/fail2ban/fail2ban.conf` | Server config |
| `/etc/fail2ban/filter.d/` | Filter patterns |
| `/etc/fail2ban/action.d/` | Action scripts |

### Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `bantime` | Ban duration (seconds) | 600 |
| `findtime` | Time window for failures | 600 |
| `maxretry` | Failures before ban | 5 |
| `ignoreip` | Whitelisted IPs | 127.0.0.1 |
| `port` | Service port(s) | varies |
| `filter` | Filter pattern name | varies |
| `logpath` | Log file to monitor | varies |

### Ban Actions

| Action | Description |
|--------|-------------|
| `iptables` | Basic iptables blocking |
| `iptables-multiport` | Block multiple ports |
| `iptables-whois` | Ban + whois email |
| `firewalld` | Firewalld backend |
| `nftables` | nftables backend |
| `cloudflare` | CloudFlare API ban |

### Common Ports

| Service | Port(s) |
|---------|---------|
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |
| SMTP | 25, 587 |
| IMAP | 143, 993 |
| POP3 | 110, 995 |
| FTP | 21 |
| MySQL | 3306 |
| PostgreSQL | 5432 |

---

## See Also

- [Fail2ban Official Documentation](https://www.fail2ban.org/)
- [Fail2ban Wiki](https://github.com/fail2ban/fail2ban/wiki)
- [Available Filters](https://www.fail2ban.org/wiki/index.php/Category:Filters)
- [Available Actions](https://www.fail2ban.org/wiki/index.php/Category:Actions)
