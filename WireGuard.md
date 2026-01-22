# WireGuard

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Key Concepts](#key-concepts)
4. [Configuration](#configuration)
5. [Peer Setup](#peer-setup)
6. [Routing](#routing)
7. [Firewall Rules](#firewall-rules)
8. [Tools and Utilities](#tools-and-utilities)
9. [Troubleshooting](#troubleshooting)
10. [Quick Reference](#quick-reference)

---

## Introduction

WireGuard is a modern, high-performance VPN protocol designed to be simple yet secure using state-of-the-art cryptography.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      WireGuard Architecture                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      Peer A (Client)                       │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              wg0 Interface                           │  │  │
│  │  │  - Private Key (keep secret)                        │  │  │
│  │  │  - ListenPort (optional)                            │  │  │
│  │  │  - PostUp/PostDown scripts                          │  │  │
│  │  └─────────────────────┬───────────────────────────────┘  │  │
│  │                        │                                    │  │
│  │                        ▼                                    │  │
│  │              ┌─────────────────┐                            │  │
│  │              │   Kernel Module │ (fast, in-kernel)         │  │
│  │              └─────────────────┘                            │  │
│  └─────────────────────────────┬───────────────────────────────┘  │
│                                │                                    │
│                                │  UDP (port 51820)                 │
│                                ▼                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      Internet                              │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                │                                    │
│                                │  UDP (port 51820)                 │
│                                ▼                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    Peer B (Server)                         │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              wg0 Interface                           │  │  │
│  │  │  - Private Key                                       │  │  │
│  │  │  - ListenPort = 51820                               │  │  │
│  │  │  - PostUp/PostDown (iptables NAT)                   │  │  │
│  │  │  - AllowedIPs = 10.0.0.0/24                         │  │  │
│  │  └─────────────────────┬───────────────────────────────┘  │  │
│  │                        │                                    │  │
│  │                        ▼                                    │  │
│  │              ┌─────────────────┐                            │  │
│  │              │   Local Network │                            │  │
│  │              └─────────────────┘                            │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Cryptography:                                                   │
│  - Curve25519 (key exchange)                                     │
│  - ChaCha20-Poly1305 (encryption)                                │
│  - BLAKE2s (hash)                                                │
│  - HKDF (key derivation)                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Features

- **Simple**: ~4000 lines of code (vs OpenVPN's 400,000+)
- **Fast**: In-kernel implementation, minimal latency
- **Modern Crypto**: Latest primitives (Curve25519, ChaCha20-Poly1305)
- **Mobile-friendly**: Handshake every ~60 seconds
- **Roaming**: Seamless IP changes
- **No config complexity**: Simple INI-style config

---

## Installation

### Linux

```bash
# Ubuntu/Debian (18.04+)
sudo apt update
sudo apt install wireguard

# RHEL/CentOS/Fedora (EPEL)
sudo yum install epel-release
sudo yum install wireguard-tools

# Arch Linux
sudo pacman -S wireguard-tools

# Check version
wg --version

# Load kernel module
sudo modprobe wireguard
lsmod | grep wireguard

# Load at boot
echo "wireguard" | sudo tee /etc/modules-load.d/wireguard.conf
```

### macOS

```bash
# Install WireGuard App
# Download from https://www.wireguard.com/install/

# Or via Homebrew
brew install wireguard-tools
```

### Windows

```bash
# Download installer
# https://www.wireguard.com/install/

# Install and use WireGuard GUI or CLI
```

### Android/iOS

```bash
# Download from app stores
# WireGuard (official app)
```

---

## Key Concepts

### Key Generation

```bash
# Generate private key
wg genkey > privatekey

# Generate public key from private
wg pubkey < privatekey > publickey

# Generate preshared key (for extra security)
wg genpsk > presharedkey

# Generate key in single command
wg genkey | tee privatekey | wg pubkey > publickey

# View keys
cat privatekey
cat publickey
```

### Key Types

| Key | Purpose | File |
|-----|---------|------|
| Private Key | Never shared, used locally | Keep secret |
| Public Key | Shared with peers | Can be public |
| Preshared Key | Optional, extra security | Keep secret |

### Protocol Stack

```
┌─────────────────────────────────────────┐
│           WireGuard Protocol             │
├─────────────────────────────────────────┤
│  Transport:  UDP (port 51820)            │
│  Key Exchange: Curve25519 ECDH           │
│  Encryption: ChaCha20-Poly1305 AEAD      │
│  Authentication: Curve25519 public key   │
│  Hash: BLAKE2s                           │
│  Key Derivation: HKDF                    │
└─────────────────────────────────────────┘
```

---

## Configuration

### Main Interface Config

```bash
# Create interface configuration
sudo nano /etc/wireguard/wg0.conf

# Configuration file format
[Interface]
PrivateKey = <server-private-key>
Address = 10.0.0.1/24
ListenPort = 51820
DNS = 1.1.1.1
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT

[Peer]
# Peer configuration below...
```

### Interface Options

```ini
[Interface]
PrivateKey = SERVER_PRIVATE_KEY_HERE
Address = 10.0.0.1/24
ListenPort = 51820

# Optional settings
DNS = 1.1.1.1              # DNS servers for clients
DNS = 10.0.0.1             # Use server as DNS

MTU = 1420                 # Default MTU

# Firewall rules on up/down
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT

# NAT for internet access
PostUp = iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE

# Table for routing
Table = off                # Don't auto-add routes
FwMark = 0x1234           # Firewall mark

# Persistent keepalive
PersistentKeepalive = 25
```

### Complete Server Config

```ini
[Interface]
PrivateKey = <server-private-key>
Address = 10.0.0.1/24
ListenPort = 51820
DNS = 1.1.1.1
MTU = 1420

# NAT for internet access
PostUp = iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE

# Forward traffic
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT

[Peer]
# Alice's laptop
PublicKey = <alice-public-key>
PresharedKey = <preshared-key>
AllowedIPs = 10.0.0.2/32

[Peer]
# Bob's phone
PublicKey = <bob-public-key>
AllowedIPs = 10.0.0.3/32

[Peer]
# Charlie's desktop
PublicKey = <charlie-public-key>
AllowedIPs = 10.0.0.4/32
```

### Complete Client Config

```ini
[Interface]
PrivateKey = <client-private-key>
Address = 10.0.0.2/24
DNS = 1.1.1.1
MTU = 1420

[Peer]
# Server
PublicKey = <server-public-key>
Endpoint = vpn.example.com:51820
PresharedKey = <preshared-key>
AllowedIPs = 0.0.0.0/0  # Route all traffic through VPN
PersistentKeepalive = 25
```

---

## Peer Setup

### Adding a New Peer

```bash
# On server: Generate keys for new peer
wg genkey | tee /tmp/client_privatekey | wg pubkey > /tmp/client_publickey

# View public key
cat /tmp/client_publickey

# Add peer to server config
sudo wg set wg0 peer <client-public-key> \
    allowed-ips 10.0.0.5/32 \
    persistent-keepalive 25

# Or edit config file
sudo nano /etc/wireguard/wg0.conf

# Reload config
sudo wg-quick down wg0
sudo wg-quick up wg0
```

### Quick Peer Addition Script

```bash
#!/bin/bash
# add-peer.sh
PEER_NAME=$1
WG_INTERFACE=${2:-wg0}

if [ -z "$PEER_NAME" ]; then
    echo "Usage: $0 <peer-name> [interface]"
    exit 1
fi

# Generate keys
PRIVATE_KEY=$(wg genkey)
PUBLIC_KEY=$(echo "$PRIVATE_KEY" | wg pubkey)
PSK=$(wg genpsk)

# Assign IP
IP_NUM=$(wg show $WG_INTERFACE peers | wc -l)
IP="10.0.0.$((IP_NUM + 2))/32"

# Add peer
wg set $WG_INTERFACE peer "$PUBLIC_KEY" \
    allowed-ips $IP \
    persistent-keepalive 25

# Output config for peer
echo "
[Interface]
PrivateKey = $PRIVATE_KEY
Address = $IP
DNS = 1.1.1.1

[Peer]
PublicKey = $(wg show $WG_INTERFACE public-key)
Endpoint = $(cat /etc/wireguard/endpoint.txt):51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
" > /tmp/$PEER_NAME.conf

echo "Config saved to /tmp/$PEER_NAME.conf"
echo "Public Key: $PUBLIC_KEY"
echo "IP Address: $IP"
```

### Client Setup Examples

**Linux Client**

```bash
# Install wireguard
sudo apt install wireguard-tools

# Create client config
sudo nano /etc/wireguard/client.conf

# Connect
sudo wg-quick up client

# Auto-connect on boot
sudo systemctl enable wg-quick@client
```

**macOS Client**

```bash
# Using WireGuard app or CLI
brew install wireguard-tools

# Create config
sudo mkdir -p /etc/wireguard
sudo nano /etc/wireguard/home.conf

# Connect
sudo wg-quick up home
```

**Android/iOS**

```bash
# 1. Install WireGuard app from store
# 2. Generate QR code on server
qrencode -t ansiutf8 < /etc/wireguard/client.conf

# 3. Scan QR code with app
```

---

## Routing

### Full Tunnel (Default Route)

```ini
[Peer]
PublicKey = <server-public-key>
Endpoint = vpn.example.com:51820
AllowedIPs = 0.0.0.0/0  # Route all IPv4 traffic
AllowedIPs = ::/0       # Route all IPv6 traffic
PersistentKeepalive = 25
```

### Split Tunnel (Specific Routes)

```ini
[Peer]
PublicKey = <server-public-key>
Endpoint = vpn.example.com:51820
AllowedIPs = 10.0.0.0/24      # VPN subnet
AllowedIPs = 192.168.1.0/24   # Home network
AllowedIPs = 172.16.0.0/12    # Corporate network
PersistentKeepalive = 25
```

### Routing Table

```bash
# Show routing table
ip route show table all
ip route show dev wg0

# Show WireGuard routes
wg show wg0 allowed-ips

# Add route manually
ip route add 10.0.0.0/24 dev wg0

# Delete route
ip route del 10.0.0.0/24 dev wg0
```

### NAT for Internet Access

```bash
# Enable IP forwarding
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# NAT rule (add to PostUp)
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE

# Remove NAT rule (add to PostDown)
iptables -t nat -D POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE

# Allow forwarding
iptables -A FORWARD -i wg0 -j ACCEPT
iptables -A FORWARD -o wg0 -j ACCEPT
```

---

## Firewall Rules

### iptables Rules

```bash
# Allow WireGuard traffic
iptables -A INPUT -p udp --dport 51820 -j ACCEPT
iptables -A OUTPUT -p udp --sport 51820 -j ACCEPT

# Allow forwarding from VPN
iptables -A FORWARD -i wg0 -j ACCEPT
iptables -A FORWARD -o wg0 -j ACCEPT

# NAT for internet
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE

# Block unwanted traffic
iptables -A INPUT -i wg0 -j DROP
iptables -A FORWARD -i wg0 -j DROP
```

### Persistent Rules

```bash
# Using iptables-persistent (Debian/Ubuntu)
sudo apt install iptables-persistent
sudo netfilter-persistent save
sudo netfilter-persistent reload

# Using firewall-cmd (RHEL)
sudo firewall-cmd --permanent --add-port=51820/udp
sudo firewall-cmd --permanent --add-interface=wg0
sudo firewall-cmd --permanent --add-masquerade
sudo firewall-cmd --reload

# Using ufw
sudo ufw allow 51820/udp
sudo ufw allow in on wg0
sudo ufw status
```

### nftables Rules

```nft
# /etc/nftables.conf
table ip filter {
    chain input {
        udp dport 51820 accept
    }
    chain forward {
        iif wg0 accept
        oif wg0 accept
    }
}

table ip nat {
    chain postrouting {
        ip saddr 10.0.0.0/24 oif eth0 masquerade
    }
}
```

---

## Tools and Utilities

### wg Command

```bash
# Show interface status
wg

# Show specific interface
wg show wg0

# Show with limited output
wg show wg0 brief
wg show wg0 dump

# Show allowed IPs only
wg show wg0 allowed-ips

# Show peers only
wg show wg0 peers

# Add/remove interface
ip link add wg0 type wireguard
ip link delete wg0

# Set interface config
wg setconf wg0 /etc/wireguard/wg0.conf
```

### wg-quick Command

```bash
# Bring up interface
sudo wg-quick up wg0

# Bring down interface
sudo wg-quick down wg0

# Show interface status
wg-quick show wg0

# Generate config from running interface
wg-quick strip wg0

# Debug mode
sudo wg-quick up wg0 --debug
```

### Configuration Generation

```bash
# Generate random key
wg genkey | tee privatekey | wg pubkey > publickey

# Generate preshared key
wg genpsk > presharedkey

# Generate all keys
wg genkey | tee privatekey | wg pubkey > publickey && wg genpsk > presharedkey

# Generate QR code
qrencode -t ansiutf8 < client.conf
qrencode -t png -o client.png < client.conf

# Export config as QR
cat client.conf | qrencode -t ansiutf8
```

### Monitoring

```bash
# Real-time traffic
sudo wg show wg0 dump

# Watch handshake
sudo wg show wg0 handshake

# Check transfer statistics
sudo wg show wg0 transfer

# Monitor with watch
watch -n 5 'wg show wg0'

# Check logs
journalctl -u wg-quick@wg0 -f
dmesg | grep wireguard
```

---

## Troubleshooting

### Connection Issues

```bash
# Check if WireGuard is loaded
lsmod | grep wireguard
sudo modprobe wireguard

# Check interface status
ip link show wg0
wg show wg0

# Verify config syntax
wg-quick check wg0

# Test UDP connectivity
nc -uvz vpn.example.com 51820

# Check firewall
sudo iptables -L -n | grep 51820
sudo ufw status | grep 51820
```

### Handshake Issues

```bash
# Check handshake times
wg show wg0 handshake

# If no handshake in last 2 minutes:
# 1. Check endpoint IP/port
# 2. Verify public keys match
# 3. Check firewall allows UDP 51820
# 4. Verify NAT rules

# Test key exchange manually
# Compare keys on both sides
wg pubkey < client_privatekey
# Should match PublicKey in server config
```

### Routing Issues

```bash
# Check routing table
ip route show
ip route show dev wg0

# Test connectivity through VPN
ping 10.0.0.1
ping 8.8.8.8

# Check DNS
dig google.com
nslookup google.com

# Trace path
traceroute 8.8.8.8
mtr 8.8.8.8

# Check MTU
ping -M do -s 1400 10.0.0.1
```

### Performance Issues

```bash
# Check transfer speeds
iperf3 -c 10.0.0.1

# Check latency
ping -c 10 10.0.0.1

# Check CPU usage
top
htop

# Verify encryption overhead
# Default MTU 1420 works for most
# Try lowering MTU if issues:
MTU = 1400
```

### Common Error Messages

```bash
# "Protocol not supported"
# - WireGuard not installed
# - Kernel module not loaded

# "No such device"
# - Interface not created
# - Run: wg-quick up wg0

# "Key does not match"
# - Public keys don't match between peers
# - Regenerate and redistribute keys

# "Handshake did not complete"
# - Network connectivity issue
# - Wrong endpoint address/port
# - Firewall blocking UDP
```

### Debug Commands

```bash
# Enable debug logging
sudo sysctl -w net.ipv4.conf.all.accept_local=1
echo module wireguard +p > /sys/kernel/debug/dynamic_debug/control

# Check kernel debug output
sudo dmesg | grep wireguard
sudo cat /proc/kmsg | grep wireguard

# Use tcpdump to capture WireGuard traffic
sudo tcpdump -i any -nn -v udp port 51820
```

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| `wg` | Main WireGuard command |
| `wg show` | Show interface info |
| `wg set` | Configure interface |
| `wg genkey` | Generate private key |
| `wg pubkey` | Generate public key |
| `wg genpsk` | Generate preshared key |
| `wg-quick up` | Bring up interface |
| `wg-quick down` | Bring down interface |
| `wg-quick strip` | Export config |

### Interface Options

| Option | Description |
|--------|-------------|
| `PrivateKey` | Private key |
| `Address` | IP addresses (CIDR) |
| `ListenPort` | UDP port |
| `DNS` | DNS servers |
| `MTU` | Maximum transmission unit |
| `PostUp` | Command after up |
| `PostDown` | Command after down |
| `Table` | Routing table |

### Peer Options

| Option | Description |
|--------|-------------|
| `PublicKey` | Peer public key |
| `PresharedKey` | Preshared key |
| `AllowedIPs` | Allowed IP ranges |
| `Endpoint` | Peer address:port |
| `PersistentKeepalive` | Keepalive interval |

### File Locations

| Path | Purpose |
|------|---------|
| `/etc/wireguard/` | Config directory |
| `/etc/wireguard/wg0.conf` | Interface config |
| `/usr/bin/wg` | Main binary |
| `/usr/bin/wg-quick` | Quick scripts |
| `/var/run/wireguard/` | Runtime data |

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |

---

## See Also

- [WireGuard Official Site](https://www.wireguard.com/)
- [WireGuard Documentation](https://www.wireguard.com/quickstart/)
- [WireGuard GitHub](https://github.com/WireGuard/wireguard-tools)
- [WireGuard Installation](https://www.wireguard.com/install/)
