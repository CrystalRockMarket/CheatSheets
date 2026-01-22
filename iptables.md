# iptables

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Concepts and Architecture](#concepts-and-architecture)
4. [Basic Commands](#basic-commands)
5. [Chain Management](#chain-management)
6. [Rule Management](#rule-management)
7. [NAT Configuration](#nat-configuration)
8. [Connection Tracking](#connection-tracking)
9. [Logging](#logging)
10. [IPv6 with ip6tables](#ipv6-with-ip6tables)
11. [Persistence](#persistence)
12. [Troubleshooting](#troubleshooting)
13. [Quick Reference](#quick-reference)

---

## Introduction

iptables is a user-space utility program that allows a system administrator to configure the IP packet filter rules of the Linux kernel firewall, implemented as different Netfilter modules.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Network Traffic                           │
│                                                                 │
│    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐       │
│    │   Incoming  │───▶│  PREROUTING │───▶│  Routing    │       │
│    │   Packet    │    │  (NAT)      │    │  Decision   │       │
│    └─────────────┘    └─────────────┘    └──────┬──────┘       │
│                                                  │              │
│                    ┌─────────────────────────────┼──────┐       │
│                    ▼                             │      ▼       │
│             ┌───────────┐                 ┌──────────┐          │
│             │  FORWARD  │                 │ INPUT    │          │
│             │  Chain    │                 │ Chain    │          │
│             └─────┬─────┘                 └────┬─────┘          │
│                   │                           │                 │
│                   ▼                           ▼                 │
│            ┌───────────┐               ┌───────────┐           │
│            │ Postrouting│              │ Local     │           │
│            │ (NAT)     │               │ Process   │           │
│            └─────┬─────┘               └───────────┘           │
│                  │                                             │
│                  ▼                                             │
│           ┌───────────┐                                       │
│           │ Outgoing  │                                       │
│           │ Packet    │                                       │
│           └───────────┘                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Netfilter Hooks

| Hook | Description | Tables |
|------|-------------|--------|
| PREROUTING | After receiving packet, before routing | raw, mangle, nat |
| INPUT | For packets destined for local | mangle, filter |
| FORWARD | For packets being forwarded | mangle, filter |
| OUTPUT | For locally-generated packets | raw, mangle, nat, filter |
| POSTROUTING | Before sending packet out | mangle, nat |

### Key Features

- Packet filtering (stateless and stateful)
- Network address translation (NAT)
- Port forwarding
- Connection tracking
- Rate limiting
- Traffic shaping
- Logging

---

## Installation

### Check if Installed

```bash
# Check iptables version
iptables --version

# Check if service is running
systemctl status iptables

# List current rules
iptables -L -n -v
```

### Installation on Different Distros

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install iptables-persistent

# RHEL/CentOS/Fedora
sudo yum install iptables-services
sudo systemctl enable --now iptables

# Arch Linux
sudo pacman -S iptables

# openSUSE
sudo zypper install iptables
```

### Enable Service

```bash
# Start service
sudo systemctl start iptables

# Enable at boot
sudo systemctl enable iptables

# Stop service
sudo systemctl stop iptables
```

---

## Concepts and Architecture

### Tables

| Table | Purpose | Chains |
|-------|---------|--------|
| **filter** | Default table, packet filtering | INPUT, FORWARD, OUTPUT |
| **nat** | Network address translation | PREROUTING, OUTPUT, POSTROUTING |
| **mangle** | Packet modification | All chains |
| **raw** | Exclude from connection tracking | PREROUTING, OUTPUT |
| **security** | SELinux context | INPUT, FORWARD, OUTPUT |

### Chains

| Chain | Description | Typical Use |
|-------|-------------|-------------|
| **INPUT** | Incoming packets to local | Block/allow access to server |
| **FORWARD** | Packets routed through server | Router/firewall rules |
| **OUTPUT** | Outgoing packets from local | Control outbound access |
| **PREROUTING** | Pre-routing modifications | Port forwarding, DNAT |
| **POSTROUTING** | Post-routing modifications | SNAT, MASQUERADE |

### Targets/Actions

| Target | Description |
|--------|-------------|
| **ACCEPT** | Allow packet through |
| **DROP** | Silently discard packet |
| **REJECT** | Reject with error message |
| **LOG** | Log packet to syslog |
| **SNAT** | Source NAT (change source IP) |
| **DNAT** | Destination NAT (change dest IP) |
| **MASQUERADE** | Dynamic SNAT for interfaces |
| **REDIRECT** | Redirect to local port |
| **QUEUE** | Pass to userspace |

### Rule Matching

| Match | Description | Example |
|-------|-------------|---------|
| `-p, --protocol` | Protocol | `-p tcp` |
| `-s, --source` | Source IP/CIDR | `-s 192.168.1.0/24` |
| `-d, --destination` | Destination IP/CIDR | `-d 10.0.0.0/8` |
| `-i, --in-interface` | Input interface | `-i eth0` |
| `-o, --out-interface` | Output interface | `-o eth1` |
| `--sport` | Source port | `--sport 80` |
| `--dport` | Destination port | `--dport 443` |
| `-m, --match` | Extended match | `-m state --state ESTABLISHED` |

---

## Basic Commands

### Listing Rules

```bash
# List all rules in filter table
iptables -L

# List with line numbers
iptables -L --line-numbers

# List verbose
iptables -L -v

# List with numeric output
iptables -L -n

# List all chains and tables
iptables -S

# List specific chain
iptables -L INPUT

# List chain with numbers
iptables -L INPUT --line-numbers

# List NAT table
iptables -t nat -L

# List raw table
iptables -t raw -L
```

### Flushing Rules

```bash
# Flush all rules
iptables -F

# Flush specific chain
iptables -F INPUT

# Flush all chains in NAT table
iptables -t nat -F

# Flush all rules and delete custom chains
iptables -X

# Set default policies to ACCEPT
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT

# Reset counters
iptables -Z
```

### Default Policies

```bash
# Set default policy for INPUT chain
iptables -P INPUT DROP

# Set default policy for FORWARD chain
iptables -P FORWARD DROP

# Set default policy for OUTPUT chain
iptables -P OUTPUT ACCEPT

# Save policies
iptables-save > /etc/iptables/rules.v4

# Restore policies
iptables-restore < /etc/iptables/rules.v4
```

---

## Chain Management

### Creating Custom Chains

```bash
# Create custom chain
iptables -N CUSTOM_CHAIN

# Create custom chain in nat table
iptables -t nat -N CUSTOM_NAT_CHAIN

# List custom chains
iptables -L | grep Chain

# Delete custom chain
iptables -X CUSTOM_CHAIN

# Flush custom chain
iptables -F CUSTOM_CHAIN

# Add rule to custom chain from built-in chain
iptables -A INPUT -j CUSTOM_CHAIN

# Jump to custom chain based on condition
iptables -A INPUT -p tcp -dport 22 -j CUSTOM_CHAIN
```

### Redirecting to Custom Chains

```bash
# Create rate-limit chain
iptables -N RATELIMIT

# Add rate limiting rules
iptables -A RATELIMIT -m recent --set
iptables -A RATELIMIT -m recent --update --seconds 60 --hitcount 10 -j DROP
iptables -A RATELIMIT -j ACCEPT

# Use in INPUT chain
iptables -A INPUT -p tcp --dport 22 -j RATELIMIT

# Create logging chain
iptables -N LOGGING

iptables -A LOGGING -m limit --limit 5/min --limit-burst 10 -j LOG --log-prefix "IPT-DROP: " --log-level 4
iptables -A LOGGING -j DROP

# Use in INPUT chain
iptables -A INPUT -j LOGGING
```

---

## Rule Management

### Adding Rules

```bash
# Append rule to end of chain
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Insert rule at position 1
iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT

# Insert rule at position 5
iptables -I INPUT 5 -p tcp --dport 80 -j ACCEPT

# Add rule with comment
iptables -A INPUT -p tcp --dport 22 -m comment --comment "Allow SSH" -j ACCEPT
```

### Deleting Rules

```bash
# Delete rule by specification
iptables -D INPUT -p tcp --dport 22 -j ACCEPT

# Delete rule by line number
iptables -D INPUT 3

# Delete all rules matching criteria
iptables -D INPUT -p tcp --dport 80
```

### Replacing Rules

```bash
# Replace rule at position 3
iptables -R INPUT 3 -p tcp --dport 443 -j ACCEPT

# Replace rule in nat table
iptables -t nat -R PREROUTING 1 -p tcp --dport 80 -j REDIRECT --to-port 8080
```

### Common Rule Examples

```bash
# Allow loopback interface
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow ping
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Allow DNS
iptables -A INPUT -p udp --dport 53 -j ACCEPT
iptables -A INPUT -p tcp --dport 53 -j ACCEPT

# Block specific IP
iptables -A INPUT -s 192.168.1.100 -j DROP

# Block IP range
iptables -A INPUT -s 192.168.1.0/24 -j DROP

# Block specific port
iptables -A INPUT -p tcp --dport 23 -j DROP

# Allow specific source port
iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT

# Match multiple ports
iptables -A INPUT -p tcp -m multiport --dports 80,443 -j ACCEPT

# Match IP range
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/8 -j ACCEPT

# Limit connections
iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT

# Drop invalid packets
iptables -A INPUT -m state --state INVALID -j DROP
```

---

## NAT Configuration

### Source NAT (SNAT)

```bash
# SNAT for outgoing traffic
iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 203.0.113.10

# SNAT for specific subnet
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o eth0 -j SNAT --to-source 203.0.113.10
```

### Masquerade

```bash
# Dynamic IP (DHCP)
iptables -t nat -A POSTROUTING -o ppp0 -j MASQUERADE

# For specific source
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o ppp0 -j MASQUERADE
```

### Destination NAT (DNAT)

```bash
# Port forwarding (HTTP)
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:80

# Port forwarding (HTTPS)
iptables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to-destination 192.168.1.100:443

# Port range forwarding
iptables -t nat -A PREROUTING -p tcp --dport 8000:9000 -j DNAT --to-destination 192.168.1.100:8000-9000

# Forward to different port
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:8080

# Local redirect
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

### Local Redirect

```bash
# Redirect incoming connections to local port
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080

# Redirect with iptables-redirect
iptables -t nat -A OUTPUT -p tcp --dport 80 -j REDIRECT --to-port 8080
```

### NAT for Docker/Kubernetes

```bash
# Allow Docker bridge network
iptables -A FORWARD -i docker0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i eth0 -o docker0 -j ACCEPT

# Allow Kubernetes CNI
iptables -A FORWARD -s 10.244.0.0/16 -j ACCEPT
iptables -A FORWARD -d 10.244.0.0/16 -j ACCEPT
```

---

## Connection Tracking

### State Matching

| State | Description |
|-------|-------------|
| **NEW** | New connection |
| **ESTABLISHED** | Part of existing connection |
| **RELATED** | Related to existing connection |
| **INVALID** | Unknown connection state |

### Connection Tracking Commands

```bash
# List tracked connections
conntrack -L

# List by state
conntrack -L -p tcp --state ESTABLISHED

# Count connections
conntrack -C

# Show connection tracking statistics
conntrack -S

# Clear connection tracking
conntrack -F
```

### Using Connection Tracking

```bash
# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow outgoing connections
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow NEW SSH connections
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -j ACCEPT

# Drop invalid packets
iptables -A INPUT -m state --state INVALID -j DROP

# Limit NEW connections per source
iptables -A INPUT -p tcp --dport 80 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 80 -m state --state NEW -m recent --update --seconds 60 --hitcount 10 -j DROP
```

### Connection Tracking Modules

```bash
# Enable connection tracking
modprobe nf_conntrack

# Enable NAT helpers
modprobe nf_conntrack_ftp
modprobe nf_conntrack_irc
modprobe nf_conntrack_sip
modprobe nf_conntrack_h323

# Check loaded modules
lsmod | grep conntrack
```

### nf_conntrack Parameters

```bash
# View current settings
sysctl net.netfilter.nf_conntrack_count
sysctl net.netfilter.nf_conntrack_max

# Increase connection tracking max
sysctl -w net.netfilter.nf_conntrack_max=524288

# Set hash size
sysctl -w net.netfilter.nf_conntrack_buckets=65536

# TCP timeout settings
sysctl -w net.netfilter.nf_conntrack_tcp_timeout_established=432000
sysctl -w net.netfilter.nf_conntrack_tcp_timeout_close_wait=3600
```

---

## Logging

### Basic Logging

```bash
# Log dropped packets
iptables -A INPUT -j LOG --log-prefix "IPT-DROP: " --log-level 4

# Log with rate limiting
iptables -A INPUT -m limit --limit 5/min --limit-burst 10 -j LOG --log-prefix "IPT-LOG: " --log-level 4

# Log to separate file via rsyslog
# Add to /etc/rsyslog.d/iptables.conf
:msg,contains,"IPT-" /var/log/iptables.log
& stop

# Reload rsyslog
systemctl restart rsyslog
```

### Advanced Logging

```bash
# Log with all details
iptables -A INPUT -j LOG --log-prefix "IPT-INPUT: " --log-level 6 \
  --log-tcp-options --log-ip-options

# Log outgoing connections
iptables -A OUTPUT -m state --state NEW -j LOG --log-prefix "IPT-OUTPUT: " --log-level 4

# Log rejected packets only
iptables -A INPUT -p tcp --dport 22 -j REJECT --reject-with tcp-reset \
  -m limit --limit 1/s --limit-burst 3 -j LOG --log-prefix "IPT-SSH-BLOCK: "
```

### Log Levels

| Level | Name | Description |
|-------|------|-------------|
| 0 | emerg | Emergency |
| 1 | alert | Alert |
| 2 | crit | Critical |
| 3 | err | Error |
| 4 | warning | Warning |
| 5 | notice | Notice |
| 6 | info | Informational |
| 7 | debug | Debug |

---

## IPv6 with ip6tables

```bash
# List IPv6 rules
ip6tables -L -n -v

# List with line numbers
ip6tables -L --line-numbers

# Flush IPv6 rules
ip6tables -F

# Set default policies
ip6tables -P INPUT DROP
ip6tables -P FORWARD DROP
ip6tables -P OUTPUT ACCEPT

# Allow established connections
ip6tables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
ip6tables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow loopback
ip6tables -A INPUT -i lo -j ACCEPT
ip6tables -A OUTPUT -o lo -j ACCEPT

# Save IPv6 rules
ip6tables-save > /etc/iptables/rules.v6

# Restore IPv6 rules
ip6tables-restore < /etc/iptables/rules.v6
```

---

## Persistence

### Saving Rules (Debian/Ubuntu)

```bash
# Install persistence package
apt install iptables-persistent

# Save current rules
netfilter-persistent save

# Or manually
iptables-save > /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6

# Restore on boot
systemctl enable netfilter-persistent
```

### Saving Rules (RHEL/CentOS)

```bash
# Save rules
service iptables save

# Rules saved to /etc/sysconfig/iptables

# Restore manually
iptables-restore < /etc/sysconfig/iptables

# Enable service
systemctl enable iptables
```

### Script-based Approach

```bash
#!/bin/bash
# /etc/iptables/rules.sh

# Flush existing rules
iptables -F
iptables -t nat -F
iptables -X

# Set default policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Allow established
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Save rules
iptables-save > /etc/iptables/rules.v4
```

### Make Script Executable

```bash
chmod +x /etc/iptables/rules.sh

# Add to rc.local or create systemd service
```

---

## Troubleshooting

### Basic Diagnostics

```bash
# List all rules with numbers
iptables -L -n --line-numbers

# Check specific chain
iptables -L INPUT -n -v

# Count packets/bytes per rule
iptables -L OUTPUT -n -v

# Check NAT rules
iptables -t nat -L -n -v

# Check raw table
iptables -t raw -L -n -v

# List counters only
iptables -L -v | head -20

# Reset counters
iptables -Z
```

### Testing Rules

```bash
# Test from remote host
nmap -p 80,443 <server-ip>

# Check if port is open
nc -zv <server-ip> 80

# Test specific rule
iptables -C INPUT -p tcp --dport 22 -j ACCEPT

# Check rule exists
iptables -S INPUT | grep "22"

# Trace packet path
iptables -t raw -A PREROUTING -p tcp --dport 80 -j TRACE
```

### Common Issues

**Rules Not Applied**

```bash
# Check if iptables is running
iptables -L

# List all chains
iptables -S

# Check service status
systemctl status iptables

# Restart service
systemctl restart iptables
```

**Connection Refused**

```bash
# Check if port is listening
ss -tlnp | grep 80

# Check firewall rules
iptables -L INPUT -n | grep 80

# Check NAT rules
iptables -t nat -L PREROUTING -n | grep 80
```

**Cannot Connect Outbound**

```bash
# Check OUTPUT chain
iptables -L OUTPUT -n -v

# Check if OUTPUT is DROP
iptables -S OUTPUT | grep "DROP"

# Allow outbound
iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -j ACCEPT
```

**Port Forwarding Not Working**

```bash
# Check PREROUTING rules
iptables -t nat -L PREROUTING -n -v

# Verify IP forwarding enabled
sysctl net.ipv4.ip_forward

# Enable if disabled
sysctl -w net.ipv4.ip_forward=1

# Make permanent in /etc/sysctl.conf
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

### Monitoring

```bash
# Watch rule changes
watch -n 1 'iptables -L -v -n'

# Monitor connections
conntrack -L -p tcp | grep ESTABLISHED

# Check connection limits
cat /proc/sys/net/netfilter/nf_conntrack_count
cat /proc/sys/net/netfilter/nf_conntrack_max

# View kernel log
dmesg | grep -i iptables
journalctl -k | grep -i iptables
```

---

## Quick Reference

### Command Summary

| Command | Description |
|---------|-------------|
| `iptables -L` | List rules |
| `iptables -A` | Append rule |
| `iptables -I` | Insert rule |
| `iptables -D` | Delete rule |
| `iptables -R` | Replace rule |
| `iptables -F` | Flush chain/table |
| `iptables -X` | Delete custom chain |
| `iptables -P` | Set policy |
| `iptables -N` | Create chain |
| `iptables -S` | Show rules (script format) |
| `iptables -Z` | Zero counters |

### Common Flags

| Flag | Description |
|------|-------------|
| `-t, --table` | Specify table |
| `-L, --list` | List rules |
| `-A, --append` | Add rule |
| `-I, --insert` | Insert rule |
| `-D, --delete` | Delete rule |
| `-R, --replace` | Replace rule |
| `-F, --flush` | Flush rules |
| `-N, --new-chain` | Create chain |
| `-X, --delete-chain` | Delete chain |
| `-P, --policy` | Set chain policy |
| `-j, --jump` | Target action |
| `-s, --source` | Source IP/CIDR |
| `-d, --destination` | Destination IP/CIDR |
| `-p, --protocol` | Protocol |
| `-i, --in-interface` | Input interface |
| `-o, --out-interface` | Output interface |
| `--dport` | Destination port |
| `--sport` | Source port |
| `-m, --match` | Extended match |
| `-v, --verbose` | Verbose output |
| `-n, --numeric` | Numeric output |
| `--line-numbers` | Show line numbers |

### Extended Matches

| Match | Description |
|-------|-------------|
| `-m state --state` | Connection state |
| `-m limit` | Rate limiting |
| `-m recent` | Recent connections |
| `-m multiport` | Multiple ports |
| `-m iprange` | IP range |
| `-m mac` | MAC address |
| `-m owner` | Packet owner |
| `-m conntrack` | Connection tracking |
| `-m length` | Packet length |
| `-m ttl` | TTL value |

### Actions/Targets

| Action | Description |
|--------|-------------|
| `ACCEPT` | Allow packet |
| `DROP` | Discard packet |
| `REJECT` | Reject with ICMP |
| `LOG` | Log to syslog |
| `SNAT` | Source NAT |
| `DNAT` | Destination NAT |
| `MASQUERADE` | Dynamic SNAT |
| `REDIRECT` | Redirect locally |
| `QUEUE` | Userspace queue |
| `RETURN` | Return from chain |

### File Locations

| File | Description |
|------|-------------|
| `/etc/iptables/rules.v4` | IPv4 rules |
| `/etc/iptables/rules.v6` | IPv6 rules |
| `/etc/sysconfig/iptables` | RHEL rules |
| `/etc/iptables.up.rules` | Arch rules |

### Useful Files

```bash
# Check if packet was dropped
cat /var/log/kern.log | grep "IPT-DROP"

# Check connection tracking
cat /proc/net/nf_conntrack

# Check module parameters
ls /sys/module/nf_conntrack/parameters/

# Check IP forwarding
cat /proc/sys/net/ipv4/ip_forward
```

---

## See Also

- [iptables man page](https://linux.die.net/man/8/iptables)
- [netfilter documentation](https://www.netfilter.org/)
- [Arch Wiki iptables](https://wiki.archlinux.org/title/iptables)
- [DigitalOcean iptables guide](https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands)
