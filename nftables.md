# nftables

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Architecture](#architecture)
4. [Basic Commands](#basic-commands)
5. [Tables and Chains](#tables-and-chains)
6. [Rules](#rules)
7. [NAT Configuration](#nat-configuration)
8. [Sets and Maps](#sets-and-maps)
9. [Flow Tables](#flow-tables)
10. [Persistence](#persistence)
11. [Migration from iptables](#migration-from-iptables)
12. [Troubleshooting](#troubleshooting)
13. [Quick Reference](#quick-reference)

---

## Introduction

nftables is the modern Linux firewall subsystem that replaces iptables, ip6tables, ebtables, and arptables. It provides a more efficient and flexible framework for packet filtering and network address translation.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         nftables Framework                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌─────────────────────────────────────────────────────────┐  │
│    │                      nft Command                         │  │
│    │              (Userspace Configuration)                   │  │
│    └────────────────────────┬────────────────────────────────┘  │
│                             │                                    │
│    ┌────────────────────────▼────────────────────────────────┐  │
│    │                  libnftnl (Library)                      │  │
│    └────────────────────────┬────────────────────────────────┘  │
│                             │                                    │
│    ┌────────────────────────▼────────────────────────────────┐  │
│    │                   Netlink Socket                         │  │
│    └────────────────────────┬────────────────────────────────┘  │
│                             │                                    │
│    ┌────────────────────────▼────────────────────────────────┐  │
│    │                  nf_tables (Kernel)                      │  │
│    │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │  │
│    │  │  Tables  │  │  Chains  │  │  Rules   │  │  Sets    │ │  │
│    │  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │  │
│    └────────────────────────┬────────────────────────────────┘  │
│                             │                                    │
│    ┌────────────────────────▼────────────────────────────────┐  │
│    │                    Network Stack                          │  │
│    │              (Netfilter Hooks)                            │  │
│    └─────────────────────────────────────────────────────────┘  │
│                                                                 │
│    Packet Flow:                                                 │
│    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    │
│    │ PREROUT │───▶│ ROUTING │───▶│ FORWARD │───▶│ POSTROUT │   │
│    │ (raw)   │    │         │    │ (filter)│    │ (nat)   │    │
│    └─────────┘    └─────────┘    └─────────┘    └─────────┘    │
│         │               │               │               │       │
│         ▼               ▼               ▼               ▼       │
│    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    │
│    │  INPUT  │    │  OUTPUT │    │  FORWARD │    │  INPUT  │    │
│    │(filter) │    │ (filter)│    │         │    │(filter) │    │
│    └─────────┘    └─────────┘    └─────────┘    └─────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Key Advantages over iptables

| Feature | iptables | nftables |
|---------|----------|----------|
| Configuration | Multiple tools | Single tool (nft) |
| Tables | Fixed 5 tables | User-defined tables |
| Chains | Fixed chains | User-defined chains |
| Performance | Single rule evaluation | Batch processing |
| Memory | Stateful tracking | Flow tables |
| Updates | Kernel modules | Single nf_tables module |
| Syntax | Inconsistent | Consistent syntax |
| Sets | Limited | Native support |

---

## Installation

### Check Installation

```bash
# Check version
nft --version

# Check if nftables is running
nft list ruleset

# Check service status
systemctl status nftables
```

### Installation on Different Distros

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install nftables

# RHEL/CentOS/Fedora
sudo yum install nftables
sudo dnf install nftables

# Arch Linux
sudo pacman -S nftables

# openSUSE
sudo zypper install nftables
```

### Enable Service

```bash
# Start service
sudo systemctl start nftables

# Enable at boot
sudo systemctl enable nftables

# Check status
sudo systemctl status nftables

# View current ruleset
sudo nft -s list ruleset
```

---

## Architecture

### Components

| Component | Description |
|-----------|-------------|
| **Tables** | Containers for chains, organize by family/proto |
| **Chains** | Containers for rules, attach to netfilter hooks |
| **Rules** | Individual filtering/actions statements |
| **Sets** | Collections of IP addresses, ports, etc. |
| **Maps** | Key-value pairs for lookups |
| **Flow Tables** | Fast path for known flows |

### Address Families

| Family | Description | Replaces |
|--------|-------------|----------|
| `ip` | IPv4 | iptables |
| `ip6` | IPv6 | ip6tables |
| `inet` | Dual-stack | Both |
| `arp` | ARP | arptables |
| `bridge` | Bridge | ebtables |
| `netdev` | Netdev | eBPF/XDP |

### Hooks

| Hook | Family | Description |
|------|--------|-------------|
| `prerouting` | All | Before routing decision |
| `ingress` | netdev, inet, ip, ip6 | On packet ingress |
| `forward` | ip, ip6, inet | Packets being forwarded |
| `input` | All | Packets to local |
| `output` | All | Locally-generated packets |
| `egress` | netdev, inet | On packet egress |
| `postrouting` | ip, ip6, inet | After routing decision |

### Priorities

| Priority | Name | Purpose |
|----------|------|---------|
| -300 | raw | Connection tracking bypass |
| 0 | filter | Main filtering |
| 100 | nat | NAT (prerouting/output) |
| 200 | dstnat | Destination NAT |
| 300 | srcnat | Source NAT |

---

## Basic Commands

### Listing Rules

```bash
# List all rules
nft list ruleset

# List in terse format
nft list ruleset -t

# List with handle numbers
nft list ruleset -a

# List specific table
nft list table inet filter

# List specific chain
nft list chain ip filter INPUT

# List all IPv4 tables
nft list tables ip

# List all inet tables
nft list tables inet

# List with JSON output
nft list ruleset json

# List in XML format
nft list ruleset xml
```

### Flushing Rules

```bash
# Flush entire ruleset
nft flush ruleset

# Flush specific table
nft flush table ip filter

# Flush specific chain
nft flush chain ip filter INPUT

# Flush all rules (keep tables)
nft -f /dev/null
```

### Saving and Restoring

```bash
# Save current ruleset
nft list ruleset > /etc/nftables.conf

# Save in canonical format
nft -c list ruleset >> /etc/nftables.conf

# Restore ruleset
nft -f /etc/nftables.conf

# Atomic replace
nft -f - <<< "$(nft list ruleset)"
```

### Interactive Mode

```bash
# Enter interactive mode
nft -i

# In interactive mode:
# nft> flush ruleset
# nft> list tables
# nft> quit
```

---

## Tables and Chains

### Creating Tables

```bash
# Create IPv4 filter table
nft add table ip filter

# Create IPv6 filter table
nft add table ip6 filter

# Create dual-stack table
nft add table inet filter

# Create NAT table
nft add table ip nat

# Create ARP table
nft add table arp filter

# Create bridge table
nft add table bridge filter

# Create netdev table (ingress)
nft add table netdev filter
```

### Creating Chains

```bash
# Create base chain (with hook and priority)
nft add chain ip filter INPUT { type filter hook input priority 0; }

# Create base chain with policy
nft add chain ip filter INPUT { type filter hook input priority 0; policy drop; }

# Create base chain for forwarding
nft add chain ip filter FORWARD { type filter hook forward priority 0; }

# Create base chain for output
nft add chain ip filter OUTPUT { type filter hook output priority 0; }

# Create regular chain (no hook)
nft add chain ip filter my_custom_chain

# Create chain with comment
nft add chain ip filter INPUT { type filter hook input priority 0; comment "Main input chain"; }

# Create NAT chain for prerouting
nft add chain ip nat PREROUTING { type nat hook prerouting priority 100; }

# Create NAT chain for postrouting
nft add chain ip nat POSTROUTING { type nat hook postrouting priority 100; }

# Create inet (dual-stack) chain
nft add chain inet filter INPUT { type filter hook input priority 0; policy accept; }
```

### Chain Management

```bash
# Rename chain
nft rename chain ip filter INPUT input_filter

# Delete chain
nft delete chain ip filter my_custom_chain

# Delete base chain
nft delete chain ip filter INPUT

# Flush chain
nft flush chain ip filter INPUT

# List chain rules with handles
nft list chain ip filter INPUT -a

# Add rule to chain
nft add rule ip filter INPUT tcp dport 22 accept

# Insert rule at position
nft insert rule ip filter INPUT position 3 tcp dport 80 accept
```

### Complete Table Setup Example

```bash
#!/usr/sbin/nft -f

# Flush existing ruleset
flush ruleset

# Create table
table inet filter {
    # INPUT chain
    chain INPUT {
        type filter hook input priority 0; policy drop;

        # Allow loopback
        iif lo accept;

        # Allow established/related
        ct state established,related accept;

        # Allow SSH
        tcp dport 22 accept;

        # Allow HTTP/HTTPS
        tcp dport {80, 443} accept;

        # Allow ping
        icmp type echo-request accept;

        # Drop invalid
        ct state invalid drop;
    }

    # FORWARD chain
    chain FORWARD {
        type filter hook forward priority 0; policy drop;

        ct state established,related accept;
    }

    # OUTPUT chain
    chain OUTPUT {
        type filter hook output priority 0; policy accept;

        ct state established,related accept;
    }
}
```

---

## Rules

### Rule Syntax

```bash
nft add rule <family> <table> <chain> <match> <action>
nft insert rule <family> <table> <chain> <position> <match> <action>
nft replace rule <family> <table> <chain> <handle> <match> <action>
nft delete rule <family> <table> <chain> <handle>
```

### Basic Matching

```bash
# Match protocol
nft add rule inet filter INPUT ip protocol tcp accept

# Match source IP
nft add rule inet filter INPUT ip saddr 192.168.1.0/24 accept

# Match destination IP
nft add rule inet filter INPUT ip daddr 10.0.0.5 accept

# Match source port (TCP/UDP)
nft add rule inet filter OUTPUT ip sport 80 accept

# Match destination port (TCP/UDP)
nft add rule inet filter INPUT tcp dport 443 accept

# Match TCP flags
nft add rule inet filter INPUT tcp flags syn accept

# Match interface
nft add rule inet filter INPUT iif eth0 accept
nft add rule inet filter OUTPUT oif eth0 accept

# Match ICMP
nft add rule inet filter INPUT icmp type echo-request accept
nft add rule inet filter INPUT icmp type echo-reply accept
```

### Extended Matching

```bash
# Connection tracking state
nft add rule inet filter INPUT ct state established,related accept
nft add rule inet filter INPUT ct state invalid drop

# TCP flags (syn, ack, fin, rst, psh, urg)
nft add rule inet filter INPUT tcp flags & (fin|syn|rst|ack) == 0 drop
nft add rule inet filter INPUT tcp flags & (syn|ack) == syn accept

# Limit rate
nft add rule inet filter INPUT tcp dport 22 limit rate 5/minute accept
nft add rule inet filter INPUT tcp dport 22 limit rate over 10/minute drop

# Recent matching
nft add rule inet filter INPUT tcp dport 22 ct state new \
    update @blacklist{10m} drop
nft add rule inet filter INPUT tcp dport 22 \
    ip saddr @whitelist accept

# Packet length
nft add rule inet filter INPUT ip length lt 64 drop
nft add rule inet filter INPUT ip length gt 1500 drop

# TOS/DSCP
nft add rule inet filter OUTPUT ip dscp af11 accept
```

### Complex Matching

```bash
# Multiple ports
nft add rule inet filter INPUT tcp dport { 80, 443 } accept

# IP set
nft add rule inet filter INPUT ip saddr @trusted_ips accept

# Verdict maps
nft add rule inet filter INPUT tcp dport 22 accept
nft add rule inet filter INPUT tcp dport 80 accept
nft add rule inet filter INPUT drop

# Not operator
nft add rule inet filter INPUT ip saddr != 192.168.1.0/24 accept

# Range matching
nft add rule inet filter INPUT ip saddr 192.168.1.0-192.168.1.255 accept

# Prefix matching
nft add rule inet filter INPUT ip saddr & 255.255.255.0 == 192.168.1.0 accept
```

### Actions/Verdicts

```bash
# Accept
nft add rule inet filter INPUT accept

# Drop
nft add rule inet filter INPUT drop

# Reject
nft add rule inet filter INPUT reject
nft add rule inet filter INPUT reject with icmpx type port-unreachable

# Jump to chain
nft add rule inet filter INPUT jump my_custom_chain

# Go to chain (doesn't return)
nft add rule inet filter INPUT goto my_custom_chain

# Return from chain
nft add rule inet filter my_custom_chain accept

# Log
nft add rule inet filter INPUT log prefix "nft-drop: "

# Queue (userspace)
nft add rule inet filter INPUT queue

# Continue (default)
nft add rule inet filter INPUT accept
```

### Metainformation

```bash
# Time-based rules
nft add rule inet filter INPUT \
    ip daddr 192.168.1.100 tcp dport 22 \
    @hour >= 9 @hour < 17 accept

# Day of week (mon=1, tue=2, etc.)
nft add rule inet filter INPUT \
    ip daddr 192.168.1.100 tcp dport 22 \
    @day 1-5 accept

# UID/GID
nft add rule inet filter OUTPUT ip protocol tcp \
    meta skuid 1000 accept

# Interface name
nft add rule inet filter INPUT iifname "eth0" accept
nft add rule inet filter INPUT iifname != "docker0" accept
```

---

## NAT Configuration

### NAT Table Setup

```bash
# Create NAT table
nft add table ip nat

# Create PREROUTING chain
nft add chain ip nat PREROUTING { type nat hook prerouting priority 100; }

# Create POSTROUTING chain
nft add chain ip nat POSTROUTING { type nat hook postrouting priority 100; }

# Create OUTPUT chain
nft add chain ip nat OUTPUT { type nat hook output priority 100; }
```

### Source NAT (SNAT)

```bash
# Static SNAT
nft add rule ip nat POSTROUTING oif eth0 snat to 203.0.113.10

# SNAT with range
nft add rule ip nat POSTROUTING oif eth0 snat to 203.0.113.10-203.0.113.20

# SNAT with specific source port
nft add rule ip nat POSTROUTING oif eth0 snat to 203.0.113.10:1024-65535
```

### Masquerade

```bash
# Dynamic SNAT (masquerade)
nft add rule ip nat POSTROUTING oif ppp0 masquerade

# Masquerade specific subnet
nft add rule ip nat POSTROUTING ip saddr 192.168.0.0/24 oif ppp0 masquerade
```

### Destination NAT (DNAT)

```bash
# Port forwarding (HTTP)
nft add rule ip nat PREROUTING tcp dport 80 dnat to 192.168.1.100:80

# Port forwarding (HTTPS)
nft add rule ip nat PREROUTING tcp dport 443 dnat to 192.168.1.100:443

# Port range forwarding
nft add rule ip nat PREROUTING tcp dport 8000-9000 dnat to 192.168.1.100:8000-9000

# Redirect to different port
nft add rule ip nat PREROUTING tcp dport 80 redirect to :8080

# Local redirect
nft add rule ip nat OUTPUT tcp dport 80 redirect to :8080

# DNAT to different IP
nft add rule ip nat PREROUTING tcp dport 22 dnat to 10.0.0.5:22

# Random load balancing
nft add rule ip nat PREROUTING tcp dport 80 dnat to 10.0.0.10-10.0.0.20
```

### Complete NAT Example

```bash
#!/usr/sbin/nft -f

flush ruleset

table ip nat {
    chain PREROUTING {
        type nat hook prerouting priority 100; policy accept;

        # Port forwarding
        tcp dport 80 dnat to 192.168.1.100:80
        tcp dport 443 dnat to 192.168.1.100:443
        tcp dport 2222 dnat to 192.168.1.100:22

        # Local redirect
        tcp dport 8080 redirect to :80
    }

    chain POSTROUTING {
        type nat hook postrouting priority 100; policy accept;

        # SNAT for internal network
        ip saddr 192.168.1.0/24 oif eth0 snat to 203.0.113.10

        # Masquerade for dynamic IPs
        oif ppp0 masquerade
    }

    chain OUTPUT {
        type nat hook output priority 100; policy accept;
    }
}
```

---

## Sets and Maps

### Sets

```bash
# Create anonymous set (inline)
nft add rule inet filter INPUT tcp dport { 80, 443, 22 } accept

# Create named set (type: IP addresses)
nft add set inet filter trusted_ips { type ipv4_addr; size 64; }

# Create named set (type: IP prefixes)
nft add set inet filter trusted_nets { type ipv4_addr; flags interval; }

# Create named set (type: ports)
nft add set inet filter allowed_ports { type inet_service; size 64; }

# Add elements to set
nft add element inet filter trusted_ips { 192.168.1.10, 192.168.1.20 }

# Add range to set
nft add element inet filter trusted_nets { 10.0.0.0/8, 192.168.0.0/16 }

# Add port range
nft add element inet filter allowed_ports { 8000-9000 }

# Delete element from set
nft delete element inet filter trusted_ips { 192.168.1.10 }

# List set contents
nft list set inet filter trusted_ips

# Flush set
nft flush set inet filter trusted_ips

# Delete set
nft delete set inet filter trusted_ips

# Create set with timeout
nft add set inet filter temp_ips { type ipv4_addr; flags timeout; }

# Add element with timeout (30 seconds)
nft add element inet filter temp_ips { 192.168.1.100 timeout 30s }

# Create set with dynamic flag
nft add set inet filter blacklist { type ipv4_addr; flags dynamic; size 64; }
```

### Maps

```bash
# Create map (key: port, value: IP)
nft add map inet filter port_forward { type inet_service:ipv4_addr; }

# Add mapping
nft add element inet filter port_forward { 80 : 192.168.1.100, 443 : 192.168.1.101 }

# Use map in rule
nft add rule inet filter PREROUTING tcp dport vmap @port_forward

# Create verdict map
nft add map inet filter access_rules { type inet_service: verdict; }

# Add verdict mapping
nft add element inet filter access_rules { 22 : drop, 80 : accept, 443 : accept }

# Use verdict map
nft add rule inet filter INPUT tcp dport vmap @access_rules

# Create NAT destination map
nft add map inet nat dnat_map { type inet_service: ipv4_addr; }

# Add DNAT entries
nft add element inet nat dnat_map { 80 : 10.0.0.10, 443 : 10.0.0.11 }

# Use in NAT rule
nft add rule inet nat PREROUTING tcp dport vmap @dnat_map
```

### Concatenated Sets/Maps

```bash
# Create concatenated set
nft add set inet filter conn_set { type ipv4_addr . inet_service; }

# Add concatenated elements
nft add element inet filter conn_set { 192.168.1.10 . 80, 192.168.1.20 . 443 }

# Use in rule
nft add rule inet filter INPUT ip saddr . tcp dport @conn_set accept

# Concatenated map
nft add map inet filter nat_map { type ipv4_addr . inet_service : ipv4_addr; }

# Add mapping
nft add element inet filter nat_map { 203.0.113.10 . 80 : 192.168.1.100 }

# Use in NAT
nft add rule inet nat PREROUTING ip daddr . tcp dport vmap @nat_map dnat to
```

### Complete Sets Example

```bash
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    # Define sets
    set trusted_ips {
        type ipv4_addr
        elements = { 192.168.1.10, 192.168.1.20, 10.0.0.0/8 }
    }

    set blocked_ips {
        type ipv4_addr
        elements = { 203.0.113.100, 203.0.113.200 }
    }

    set allowed_ports {
        type inet_service
        elements = { 22, 80, 443, 8080-8090 }
    }

    set ssh_ports {
        type inet_service
        elements = { 22, 2222 }
    }

    chain INPUT {
        type filter hook input priority 0; policy drop;

        iif lo accept;
        ct state established,related accept;
        ct state invalid drop;

        # Block listed IPs
        ip saddr @blocked_ips drop;

        # Allow trusted IPs
        ip saddr @trusted_ips accept;

        # Allow specific ports
        tcp dport @allowed_ports accept;
        udp dport @allowed_ports accept;

        # SSH rate limiting with set
        tcp dport @ssh_ports \
            ip saddr . tcp dport @ssh_conn_set \
            limit rate over 10/minute burst 5 packets drop;
        tcp dport @ssh_ports \
            ip saddr . tcp dport add @ssh_conn_set {1m} accept;
    }

    set ssh_conn_set {
        type ipv4_addr . inet_service
        flags timeout
        size 65536
    }
}
```

---

## Flow Tables

```bash
# Create flow table
nft add table inet filter
nft add flowtable inet filter fastpath { hook ingress priority 0; devices = { eth0, eth1 }; }

# Use flowtable in rule
nft add rule inet filter FORWARD \
    ip protocol tcp flow table @fastpath accept

# Flow table for acceleration
table inet xdp {
    flowtable f {
        hook ingress priority 0
        devices = { eth0, eth1 }
        size 65536
    }

    chain forward {
        type filter hook forward priority 0; policy accept;
        ip protocol tcp flow table @f accept;
    }
}
```

---

## Persistence

### Save Rules

```bash
# Save to file
nft list ruleset > /etc/nftables.conf

# Save in canonical format
nft -c list ruleset >> /etc/nftables.conf

# Save with comments
nft list ruleset -s > /etc/nftables.conf
```

### Load on Boot

```bash
# Debian/Ubuntu
systemctl enable nftables

# Create systemd service
cat > /etc/systemd/system/nftables.service <<EOF
[Unit]
Description=nftables
Documentation=man:nftables(8)
Wants=network-pre.target
After=network-pre.target

[Service]
Type=oneshot
ExecStart=/etc/nftables.conf
ExecReload=/etc/nftables.conf
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable nftables
```

### Atomic Replace

```bash
# Atomic ruleset replacement
nft -f - <<< "$(nft list ruleset)"

# Or with script
#!/bin/bash
NEW_RULESET=$(mktemp)
nft -c list ruleset > "$NEW_RULESET"
if [ $? -eq 0 ]; then
    nft -f "$NEW_RULESET"
fi
rm "$NEW_RULESET"
```

---

## Migration from iptables

### Using iptables-translate

```bash
# Install iptables-translate
apt install iptables-nft

# Translate single rule
iptables-translate -A INPUT -p tcp --dport 22 -j ACCEPT
# Output: nft add rule ip filter INPUT tcp dport 22 accept

# Translate entire ruleset
iptables-save | iptables-translate

# Translate iptables-restore file
iptables-restore-translate -f /etc/iptables/rules.v4 > /etc/nftables.conf
```

### Translation Table

| iptables | nftables |
|----------|----------|
| `iptables -L` | `nft list ruleset` |
| `iptables -A INPUT -j DROP` | `nft add rule ip filter INPUT drop` |
| `iptables -I INPUT 1` | `nft insert rule ip filter INPUT position 1` |
| `iptables -D INPUT -p tcp --dport 22` | `nft delete rule ip filter INPUT tcp dport 22` |
| `iptables -F` | `nft flush ruleset` |
| `iptables -X` | `nft flush ruleset` (delete user chains) |
| `iptables -P INPUT DROP` | `nft add rule ip filter INPUT policy drop` |
| `iptables -t nat -A PREROUTING` | `nft add rule ip nat PREROUTING` |
| `iptables -A INPUT -m state --state ESTABLISHED` | `nft add rule ip filter INPUT ct state established` |
| `iptables -A INPUT -p tcp -m multiport --dports 80,443` | `nft add rule ip filter INPUT tcp dport { 80, 443 }` |

### Hybrid Mode

```bash
# Keep iptables-nft for backward compatibility
apt install iptables-nft

# Use nftables-native in kernel
modprobe nf_tables
modprobe nft_compat
```

---

## Troubleshooting

### Basic Diagnostics

```bash
# List all rules with handles
nft list ruleset -a

# List specific table with handles
nft list table inet filter -a

# List chain with rules and handles
nft list chain ip filter INPUT -a

# Check ruleset syntax
nft -c -f /etc/nftables.conf

# Verbose output
nft list ruleset -v

# Debug mode
nft --debug all list ruleset
```

### Monitoring

```bash
# Watch rule changes
watch -n 1 'nft list ruleset | head -50'

# Check counters
nft list ruleset -c

# Reset counters
nft reset ruleset counters

# List counters per rule
nft list ruleset | grep -E "counter|count"

# Monitor packets
nft list ruleset -a | while read line; do echo "$line"; done
```

### Common Issues

**Rules Not Applied**

```bash
# Check syntax
nft -c -f /etc/nftables.conf

# Check if nftables is running
nft list ruleset

# Check service status
systemctl status nftables

# Reload rules
nft -f /etc/nftables.conf

# Check kernel messages
dmesg | grep nft
journalctl -k | grep nft
```

**Connection Issues**

```bash
# Check if rule exists
nft list ruleset | grep "dport 22"

# Check chain policy
nft list chain ip filter INPUT | head -5

# Verify NAT rules
nft list table ip nat

# Check counters
nft list ruleset -c | grep -A2 "22"
```

**Performance Issues**

```bash
# Check rule count
nft list ruleset | wc -l

# List with timings
nft list ruleset -t

# Profile ruleset
nft -t list ruleset

# Check for duplicate rules
nft list ruleset | sort | uniq -d
```

### Useful Commands

```bash
# Find rule by port
nft list ruleset | grep "dport 80"

# Find rule by IP
nft list ruleset | grep "192.168.1.100"

# Count rules per chain
nft list ruleset | grep -c "accept\|drop\|reject"

# Export to JSON
nft list ruleset json

# Import from JSON
nft -f ruleset.json

# Backup ruleset
nft list ruleset > backup_$(date +%Y%m%d).nft
```

---

## Quick Reference

### Command Summary

| Command | Description |
|---------|-------------|
| `nft list ruleset` | List all rules |
| `nft list table <family> <table>` | List table |
| `nft list chain <family> <table> <chain>` | List chain |
| `nft add table <family> <table>` | Create table |
| `nft add chain <family> <table> <chain>` | Create chain |
| `nft add rule <family> <table> <chain> <match> <action>` | Add rule |
| `nft insert rule <family> <table> <chain> <pos> <match>` | Insert rule |
| `nft delete rule <family> <table> <chain> <handle>` | Delete rule |
| `nft delete table <family> <table>` | Delete table |
| `nft flush ruleset` | Flush all rules |
| `nft -f <file>` | Load ruleset from file |

### Common Flags

| Flag | Description |
|------|-------------|
| `-a, --handles` | Show rule handles |
| `-c, --check` | Check syntax only |
| `-f, --file` | Read from file |
| `-i, --interactive` | Interactive mode |
| `-n, --numeric` | Numeric output |
| `-s, --stateless` | Stateless listing |
| `-t, --terse` | Terse output |
| `-v, --verbose` | Verbose output |
| `-j, --json` | JSON output |

### Matching Operators

| Operator | Description |
|----------|-------------|
| `==` | Equal |
| `!=` | Not equal |
| `<` | Less than |
| `>` | Greater than |
| `<=` | Less or equal |
| `>=` | Greater or equal |
| `&` | Bitwise AND |
| `|` | Bitwise OR |
| `^` | Bitwise XOR |
| `<<` | Left shift |
| `>>` | Right shift |
| `~` | Bitwise NOT (inverted) |

### Special Values

| Value | Description |
|-------|-------------|
| `any` | Match any |
| `drop` | Drop packet |
| `accept` | Accept packet |
| `reject` | Reject packet |
| `continue` | Continue to next rule |
| `return` | Return from chain |

### File Locations

| File | Description |
|------|-------------|
| `/etc/nftables.conf` | Main configuration file |
| `/etc/nftables/` | Directory for include files |
| `/var/lib/nftables/` | State directory |

### Useful Debug Commands

```bash
# Check loaded modules
lsmod | grep nf_

# Check kernel parameters
sysctl net.netfilter.nf_conntrack_count
sysctl net.netfilter.nf_conntrack_max

# View netlink messages
nft -v -d list ruleset

# Monitor nft events
nft monitor

# Monitor with rules
nft monitor trace
```

---

## See Also

- [nftables wiki](https://wiki.nftables.org/)
- [nftables man page](https://linux.die.net/man/8/nft)
- [Arch Wiki nftables](https://wiki.archlinux.org/title/nftables)
- [nftables quick reference](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_sect Configuring_firewalls_using_nftables)
- [iptables to nftables migration](https://wiki.nftables.org/wiki-nftables/index.php/Migration_tool)
