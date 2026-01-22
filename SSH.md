# SSH

## Table of Contents
1. [Introduction](#introduction)
2. [Installation and Setup](#installation-and-setup)
3. [Basic Connection](#basic-connection)
4. [Key-Based Authentication](#key-based-authentication)
5. [Configuration Files](#configuration-files)
6. [Tunnels and Port Forwarding](#tunnels-and-port-forwarding)
7. [SCP and SFTP](#scp-and-sftp)
8. [X11 Forwarding](#x11-forwarding)
9. [SSH Agent](#ssh-agent)
10. [Jump Hosts and Proxies](#jump-hosts-and-proxies)
11. [Multiplexing](#multiplexing)
12. [Troubleshooting](#troubleshooting)
13. [Quick Reference](#quick-reference)

---

## Introduction

SSH (Secure Shell) is a cryptographic network protocol for secure remote access to systems over an unsecured network.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         SSH Protocol Stack                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Application Layer                     │    │
│  │  (shell, sftp, scp, git, rsync, etc.)                   │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│  ┌────────────────────────▼────────────────────────────────┐    │
│  │                  SSH Protocol Layer                      │    │
│  │  ┌──────────────────────────────────────────────────┐  │    │
│  │  │              SSH Connection Protocol              │  │    │
│  │  │        (Channels, Shell, Subsystem)              │  │    │
│  │  └────────────────────────┬─────────────────────────┘  │    │
│  ┌──────────────────────────▼─────────────────────────────┐  │  │
│  │                 SSH Authentication                      │  │  │
│  │  (publickey, password, keyboard-interactive, GSSAPI)   │  │  │
│  └────────────────────────┬───────────────────────────────┘  │  │
│                           │                                      │
│  ┌────────────────────────▼───────────────────────────────┐  │  │
│  │               SSH Transport Layer                       │  │  │
│  │         (encryption, compression, integrity)            │  │  │
│  └────────────────────────┬───────────────────────────────┘  │  │
│                           │                                      │
│  ┌────────────────────────▼───────────────────────────────┐  │  │
│  │           SSH Key Exchange (KEX)                        │  │  │
│  │  (diffie-hellman-group-exchange, ECDH)                  │  │  │
│  └────────────────────────┬───────────────────────────────┘  │  │
│                           │                                      │
│  ┌────────────────────────▼───────────────────────────────┐  │  │
│  │                TCP/IP Layer                             │  │  │
│  │                  (port 22)                              │  │  │
│  └────────────────────────────────────────────────────────┘  │  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Features

- Encrypted communication
- Strong authentication (keys, passwords)
- Secure file transfer (SCP, SFTP)
- Port forwarding/tunneling
- X11 forwarding
- Agent forwarding
- Jump hosts
- Connection multiplexing

---

## Installation and Setup

### Installation

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install openssh-server openssh-client

# RHEL/CentOS/Fedora
sudo yum install openssh-server openssh-clients

# Arch Linux
sudo pacman -S openssh

# macOS
# Built-in, install via Homebrew if needed
brew install openssh

# Start service
sudo systemctl enable --now sshd

# Verify installation
ssh -V
```

### Server Configuration

```bash
# Main SSH daemon configuration
sudo nano /etc/ssh/sshd_config

# Key files
/etc/ssh/sshd_config          # Main server config
/etc/ssh/ssh_host_rsa_key     # RSA host key
/etc/ssh/ssh_host_ecdsa_key   # ECDSA host key
/etc/ssh/ssh_host_ed25519_key # Ed25519 host key
```

### Generate Host Keys

```bash
# Generate new host keys (if needed)
sudo ssh-keygen -A

# View existing host keys
ls -la /etc/ssh/ssh_host_*

# Regenerate specific key type
sudo ssh-keygen -t ed25519 -f /etc/ssh/ssh_host_ed25519_key
```

### Basic sshd_config

```bash
Port 22
AddressFamily any
ListenAddress 0.0.0.0
ListenAddress ::

HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

# Logging
SyslogFacility AUTH
LogLevel INFO

# Authentication
PermitRootLogin no
MaxAuthTries 3
MaxSessions 10
PubkeyAuthentication yes
PasswordAuthentication yes
PermitEmptyPasswords no
ChallengeResponseAuthentication no

# PAM
UsePAM yes

# X11 Forwarding
X11Forwarding yes

# Print MOTD
PrintMotd no

# Chroot directory
ChrootDirectory /sftp

# Subsystems
Subsystem sftp /usr/lib/openssh/sftp-server

# Override default of no subsystems
Subsystem sftp internal-sftp

# Allow client to pass environment variables
AcceptEnv LANG LC_*

# override default of no delayed services
TCPKeepAlive yes

# Disable tunneling, DHCP, DNS, and multimedia
PermitTunnel no
AllowTcpForwarding yes
GatewayPorts no

ClientAliveInterval 300
ClientAliveCountMax 2

# Banner
Banner /etc/ssh/banner

# Max startups (prevent DoS)
MaxStartups 10:30:100

# Allow users
AllowUsers user1 user2
AllowGroups ssh-users
DenyUsers baduser
DenyGroups badgroup

# Match block for specific users
Match User admin
    AllowTcpForwarding yes
    X11Forwarding no
    PermitRootLogin no
```

### Test Configuration

```bash
# Test configuration syntax
sudo sshd -t

# Test with verbose output
sudo sshd -T

# Check running process
ps aux | grep sshd

# Check listening port
ss -tlnp | grep 22
netstat -tlnp | grep 22
```

---

## Basic Connection

### Connecting

```bash
# Basic connection
ssh user@hostname

# Connect with specific port
ssh -p 2222 user@hostname

# Connect with specific identity file
ssh -i ~/.ssh/id_ed25519 user@hostname

# Connect with verbose debugging
ssh -vvv user@hostname

# Connect with specific cipher
ssh -c aes256-ctr user@hostname

# Connect with compression
ssh -C user@hostname

# Connect and execute command
ssh user@hostname "ls -la"

# Connect and execute multiple commands
ssh user@hostname "cd /var/log && ls -la"
```

### Connection Options

| Option | Description |
|--------|-------------|
| `-p` | Port |
| `-i` | Identity file |
| `-l` | Login name |
| `-A` | Agent forwarding |
| `-X` | X11 forwarding |
| `-C` | Compression |
| `-v` | Verbose (1-3 for levels) |
| `-f` | Background after auth |
| `-N` | No command (tunnel) |
| `-T` | Disable pseudo-terminal |
| `-o` | SSH config option |

### SSH Config Host

```bash
# Add to ~/.ssh/config
Host myserver
    HostName server.example.com
    User admin
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    Compression yes
    ServerAliveInterval 60
    ServerAliveCountMax 3

Host *.example.com
    User deploy
    ForwardAgent no

Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
    AddKeysToAgent yes
```

### Using SSH Config

```bash
# Connect using config alias
ssh myserver

# SFTP using config
sftp myserver

# SCP using config
scp file.txt myserver:/tmp/
```

---

## Key-Based Authentication

### Generate SSH Keys

```bash
# Generate Ed25519 key (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Generate RSA key (4096 bits)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Generate key with comment
ssh-keygen -t ed25519 -C "work-key-2024"

# Generate with custom location
ssh-keygen -t ed25519 -f ~/.ssh/work_key -C "work-key"
```

### Key Types Comparison

| Type | Bits | Security | Speed | Use Case |
|------|------|----------|-------|----------|
| **Ed25519** | 256 | Excellent | Fast | Modern systems |
| **ECDSA** | 256/384/521 | Good | Fast | Modern systems |
| **RSA** | 2048/4096 | Good | Slow | Legacy systems |

### Managing Keys

```bash
# Change key passphrase
ssh-keygen -p -f ~/.ssh/id_ed25519

# Change comment
ssh-keygen -c -f ~/.ssh/id_ed25519

# View public key
cat ~/.ssh/id_ed25519.pub

# View key fingerprint
ssh-keygen -lf ~/.ssh/id_ed25519

# View key fingerprint in MD5
ssh-keygen -lf -E md5 ~/.ssh/id_ed25519

# Convert key format
ssh-keygen -p -m PEM -f ~/.ssh/id_ed25519

# Export public key
ssh-keygen -y -f ~/.ssh/id_ed25519 > public_key.pub
```

### Deploying Public Keys

```bash
# Method 1: ssh-copy-id (easiest)
ssh-copy-id user@hostname
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@hostname

# Method 2: Manual deployment
# Copy public key to server
scp ~/.ssh/id_ed25519.pub user@hostname:/tmp/

# SSH to server and add to authorized_keys
ssh user@hostname
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cat /tmp/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
rm /tmp/id_ed25519.pub

# Method 3: One-liner
cat ~/.ssh/id_ed25519.pub | ssh user@hostname "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### Server-Side Key Configuration

```bash
# In /etc/ssh/sshd_config
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2

# Directory for user keys
AuthorizedKeysFile /etc/ssh/authorized_keys/%u

# Require specific CA
TrustedUserCAKeys /etc/ssh/ca_keys.pub
AuthorizedPrincipalsFile /etc/ssh/auth_principals/%u

# Revoked keys
RevokedKeys /etc/ssh/revoked_keys
```

### SSH Certificates

```bash
# CA key setup
ssh-keygen -t ed25519 -f ~/.ssh/ca_key -C "CA"

# Sign user key
ssh-keygen -s ~/.ssh/ca_key -I "user@hostname" -n username -V +52w id_ed25519.pub

# Sign host key
ssh-keygen -s ~/.ssh/ca_key -h -I "host.example.com" -h -n host.example.com /etc/ssh/ssh_host_ed25519_key.pub

# Verify certificate
ssh-keygen -L -f id_ed25519-cert.pub
```

---

## Configuration Files

### Client Configuration Files

| File | Scope | Purpose |
|------|-------|---------|
| `~/.ssh/config` | User | Personal SSH settings |
| `/etc/ssh/ssh_config` | System | Default system settings |
| `~/.ssh/known_hosts` | User | Trusted host keys |
| `/etc/ssh/ssh_known_hosts` | System | System-wide known hosts |

### ssh_config Examples

```bash
# Global defaults
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes
    TCPKeepAlive yes

# Specific host
Host prod-server
    HostName 203.0.113.10
    User admin
    Port 2222
    IdentityFile ~/.ssh/prod_ed25519
    PasswordAuthentication no
    ForwardAgent no
    LogLevel ERROR

# Development environment
Host dev-*
    HostName 10.0.0.%h
    User developer
    ForwardAgent yes
    AddKeysToAgent yes

# GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
    IdentitiesOnly yes
    AddKeysToAgent yes

# AWS EC2
Host *.compute.amazonaws.com
    User ec2-user
    IdentityFile ~/.ssh/aws_ed25519.pem
    StrictHostKeyChecking accept-new
    UserKnownHostsFile ~/.ssh/known_hosts.aws
```

### Server Configuration Files

```bash
# /etc/ssh/sshd_config

# Connection settings
Port 22
ListenAddress 0.0.0.0
ListenAddress ::

# Host keys
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

# Security
Protocol 2
PermitRootLogin no
MaxAuthTries 3
MaxSessions 10
MaxStartups 10:30:100

# Authentication
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentication yes
PermitEmptyPasswords no
ChallengeResponseAuthentication no

# GSSAPI (Kerberos)
GSSAPIAuthentication no

# PAM
UsePAM yes

# Logging
SyslogFacility AUTH
LogLevel INFO

# X11
X11Forwarding yes
X11DisplayOffset 10

# Environment
AcceptEnv LANG LC_*

# Subsystems
Subsystem sftp /usr/lib/openssh/sftp-server

# Tunneling
PermitTunnel no
AllowTcpForwarding yes
GatewayPorts no

# Keep alive
ClientAliveInterval 300
ClientAliveCountMax 2

# Banner
Banner /etc/ssh/banner

# Match block
Match Address 192.168.1.0/24
    PermitRootLogin yes
    X11Forwarding no
```

### Managing known_hosts

```bash
# View known_hosts
cat ~/.ssh/known_hosts

# Hash host in known_hosts
ssh-keygen -H -f ~/.ssh/known_hosts

# Remove specific host
ssh-keygen -R hostname
ssh-keygen -R 203.0.113.10
ssh-keygen -R hostname.example.com

# Remove all keys for a domain
ssh-keygen -R github.com

# Check host key fingerprint
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub

# Verify host key on connection
ssh -o FingerprintHash=md5 user@hostname
```

---

## Tunnels and Port Forwarding

### Local Port Forwarding

```bash
# Forward local port to remote service
ssh -L 8080:localhost:80 user@remote-server

# Forward local port to different remote host
ssh -L 8080:db.example.com:5432 user@remote-server

# Forward with non-standard local port
ssh -L 5432:localhost:5432 user@remote-server

# Multiple local forwards
ssh -L 8080:localhost:80 -L 5432:localhost:5432 user@remote-server

# Bind to specific interface
ssh -L 127.0.0.1:8080:localhost:80 user@remote-server

# Gateway mode (bind to all interfaces)
ssh -g -L 8080:localhost:80 user@remote-server
```

### Remote Port Forwarding

```bash
# Forward remote port to local service
ssh -R 8080:localhost:80 user@remote-server

# Forward remote port to different local host
ssh -R 8080:internal-db:5432 user@remote-server

# Remote forward on all interfaces
ssh -R 0.0.0.0:8080:localhost:80 user@remote-server

# Remote forward on specific port
ssh -R 2222:localhost:22 user@remote-server

# Set up reverse tunnel for home server
ssh -R 22:localhost:22 user@vpn-server.example.com -N
```

### Dynamic Port Forwarding (SOCKS)

```bash
# Create SOCKS proxy
ssh -D 1080 user@remote-server

# SOCKS with specific local port
ssh -D 127.0.0.1:1080 user@remote-server

# SOCKS with username
ssh -D 1080 -l username user@remote-server

# Use with curl
curl --socks5 127.0.0.1:1080 https://api.example.com

# Configure browser to use SOCKS proxy
# SOCKS5 proxy: 127.0.0.1:1080
```

### SSH Tunnel Scripts

```bash
#!/bin/bash
# ~/bin/ssh-tunnel

LOCAL_PORT=$1
REMOTE_HOST=$2
REMOTE_PORT=$3
SSH_HOST=$4

if [ -z "$LOCAL_PORT" ] || [ -z "$REMOTE_HOST" ] || [ -z "$REMOTE_PORT" ] || [ -z "$SSH_HOST" ]; then
    echo "Usage: $0 <local_port> <remote_host> <remote_port> <ssh_host>"
    exit 1
fi

ssh -L ${LOCAL_PORT}:${REMOTE_HOST}:${REMOTE_PORT} -N $SSH_HOST
```

### Keep-Alive Tunnels

```bash
# Auto-reconnect tunnel
while true; do
    ssh -L 8080:localhost:80 -N user@server.example.com
    sleep 5
done

# Using autossh (auto-reconnect)
sudo apt install autossh
autossh -M 0 -L 8080:localhost:80 -N user@server.example.com

# autossh with monitoring
autossh -M 20000 -L 8080:localhost:80 -N user@server.example.com

# systemd service for tunnel
sudo nano /etc/systemd/system/ssh-tunnel.service
```

---

## SCP and SFTP

### SCP Commands

```bash
# Copy local to remote
scp file.txt user@hostname:/path/
scp file.txt user@hostname:/path/newfilename.txt

# Copy remote to local
scp user@hostname:/path/file.txt ./
scp user@hostname:/path/file.txt ./newfilename.txt

# Copy directory recursively
scp -r directory/ user@hostname:/path/

# Copy with specific port
scp -P 2222 file.txt user@hostname:/path/

# Copy with compression
scp -C file.txt user@hostname:/path/

# Copy with specific cipher
scp -c aes256-ctr file.txt user@hostname:/path/

# Copy preserve attributes
scp -p file.txt user@hostname:/path/

# Copy between two remotes
scp user1@host1:/file1.txt user2@host2:/path/

# Copy multiple files
scp file1.txt file2.txt user@hostname:/path/

# Quiet mode
scp -q file.txt user@hostname:/path/
```

### SFTP Commands

```bash
# Connect to server
sftp user@hostname
sftp -i ~/.ssh/key user@hostname
sftp -oPort=2222 user@hostname

# SFTP commands (interactive)
sftp> help
sftp> ls -la
sftp> cd /path
sftp> pwd
sftp> get file.txt
sftp> get file.txt /local/path/
sftp> put file.txt
sftp> put file.txt /remote/path/
sftp> mget *.log
sftp> mput *.txt
sftp> mkdir files
sftp> rmdir empty_dir
sftp> rm file.txt
sftp> rename old.txt new.txt
sftp> lcd /local/path
sftp> lpwd
sftp> lls -la
sftp> chmod 644 file.txt
sftp> chown user:group file.txt
sftp> !command
sftp> exit

# Non-interactive SFTP
sftp user@hostname <<EOF
cd /remote/path
get file.txt
bye
EOF
```

### rsync over SSH

```bash
# Sync local to remote
rsync -avz -e ssh ./dir/ user@hostname:/path/

# Sync remote to local
rsync -avz -e ssh user@hostname:/path/ ./local/

# Sync with deletion
rsync -avz --delete -e ssh ./dir/ user@hostname:/path/

# Sync with progress
rsync -avzP -e ssh ./dir/ user@hostname:/path/

# Sync specific files
rsync -avz --include='*.txt' --include='*/' --exclude='*' -e ssh ./ user@hostname:/path/

# Sync with bandwidth limit
rsync -avz --bwlimit=1000 -e ssh ./ user@hostname:/path/
```

---

## X11 Forwarding

### Enable X11 Forwarding

```bash
# Client side
ssh -X user@hostname

# Forward X11 with trusted authentication
ssh -Y user@hostname

# X11 with compression
ssh -XC user@hostname

# In SSH config
Host remote
    HostName remote.example.com
    ForwardX11 yes
    ForwardX11Trusted yes
```

### Using X11 Forwarding

```bash
# After connecting
ssh -X user@hostname

# Run GUI application
firefox
nautilus
gedit
xclock
xeyes

# Check X11 forwarding status
echo $DISPLAY

# List X11 cookies
xauth list

# Test X11 forwarding
xeyes
```

### X11 Configuration

```bash
# Server configuration (sshd_config)
X11Forwarding yes
X11DisplayOffset 10
X11UseLocalhost no

# X11 forwarding with specific protocol
X11Forwarding yes
X11UseLocalhost yes

# Trusted X11 forwarding (less secure)
ForwardX11Trusted yes
```

### Troubleshooting X11

```bash
# Check DISPLAY variable
echo $DISPLAY

# Set DISPLAY manually
export DISPLAY=:0

# Check X11 forwarding
ssh -v -X user@hostname 2>&1 | grep X11

# Test with simple X11 app
xauth generate
xclock

# Check xauth
which xauth
xauth list
```

---

## SSH Agent

### Starting Agent

```bash
# Start ssh-agent
eval $(ssh-agent)

# Start with specific shell
ssh-agent bash
ssh-agent zsh

# Add to ~/.bashrc
echo 'eval $(ssh-agent)' >> ~/.bashrc
echo 'ssh-add ~/.ssh/id_ed25519' >> ~/.bashrc
```

### Managing Keys

```bash
# Add default key
ssh-add ~/.ssh/id_ed25519

# Add specific key
ssh-add ~/.ssh/work_key
ssh-add ~/.ssh/github_ed25519

# Add all keys
ssh-add ~/.ssh/*

# List added keys
ssh-add -l

# List keys with fingerprints
ssh-add -L

# List keys with SHA256 fingerprints
ssh-add -l -E sha256

# Delete specific key
ssh-add -d ~/.ssh/id_ed25519

# Delete all keys
ssh-add -D

# Set lifetime for key
ssh-add -t 3600 ~/.ssh/id_ed25519

# Lock/unlock agent
ssh-add -x
ssh-add -X
```

### Agent Forwarding

```bash
# Enable agent forwarding
ssh -A user@hostname

# Agent forwarding in config
Host remote
    HostName remote.example.com
    ForwardAgent yes

# Test agent forwarding
ssh user@remote "ssh user@another-host echo 'Agent forwarding works'"
```

### SSH Agent Configuration

```bash
# Add to ~/.ssh/config
AddKeysToAgent yes
IdentityFile ~/.ssh/id_ed25519
IdentityFile ~/.ssh/work_ed25519
IdentityFile ~/.ssh/github_ed25519

# GnuPG Agent with SSH support
# Add to ~/.gnupg/gpg-agent.conf
enable-ssh-support

# Add to ~/.bashrc or ~/.zshrc
export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)
```

---

## Jump Hosts and Proxies

### Jump Host (ProxyJump)

```bash
# Single jump host
ssh -J user1@jumphost.example.com user2@target.example.com

# Multiple jump hosts
ssh -J user1@jumphost1.example.com,user2@jumphost2.example.com target.example.com

# Jump host with specific port
ssh -J user@jumphost:2222 user@target:22

# In SSH config
Host target
    HostName target.example.com
    User deploy
    ProxyJump user@jumphost.example.com

# Gateway (old syntax)
ssh -o "ProxyCommand ssh -W %h:%p user@gateway" user@target
```

### SSH ProxyCommand

```bash
# Direct connection via proxy
ssh -o "ProxyCommand nc -X 5 -x proxy:1080 %h %p" user@hostname

# Netcat proxy
ssh -o "ProxyCommand nc -q 0 %h %p" user@hostname

# Using socat
ssh -o "ProxyCommand socat - TCP4:proxy:1080" user@hostname

# ProxyCommand in config
Host external
    HostName external.example.com
    ProxyCommand ssh user@gateway -W %h:%p

Host via-bastion
    HostName internal.example.com
    ProxyCommand ssh user@bastion.example.com -W %h:%p
```

### SSH Config for Jump Hosts

```bash
# Bastion host configuration
Host bastion
    HostName bastion.example.com
    User admin
    IdentityFile ~/.ssh/bastion_ed25519

# Internal hosts via bastion
Host app-*
    HostName 10.0.1.*
    User deploy
    ProxyJump bastion

Host db-*
    HostName 10.0.2.*
    User dba
    ProxyJump bastion
```

### AWS SSM Session Manager

```bash
# SSH via SSM
ssh -i key.pem ec2-user@instance-id

# Add to config
Host i-*
    HostName %h
    User ec2-user
    ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'PortNumber=%p'"

# Session Manager plugin
# https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html
```

---

## Multiplexing

### Enable Multiplexing

```bash
# ControlMaster in SSH config
Host *
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h-%p
    ControlPersist 600

# Create socket directory
mkdir -p ~/.ssh/sockets
chmod 700 ~/.ssh/sockets
```

### Manual Multiplexing

```bash
# Start master connection
ssh -M -S ~/.ssh/socket-%r@%h:%p user@hostname

# Check master status
ssh -S ~/.ssh/socket-%r@%h:%p user@hostname -O check

# List master connections
ssh -S ~/.ssh/socket-%r@%h:%p user@hostname -O show

# Stop master connection
ssh -S ~/.ssh/socket-%r@%h:%p user@hostname -O exit

# Use multiplexed connection
ssh -S ~/.ssh/socket-%r@%h:%p user@hostname

# Copy via multiplexed connection
scp -o ControlPath=~/.ssh/socket-%r@%h:%p file.txt user@hostname:/path/
```

### Multiplexing Timeout

```bash
# Configuration
Host *
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h-%p
    ControlPersist 10m    # Keep connection for 10 minutes after disconnect
    ControlPersist 600    # Keep for 10 minutes idle
    ControlPersist yes    # Keep until manually killed
```

### Verify Multiplexing

```bash
# Check active connections
ls -la ~/.ssh/sockets/

# Test multiplexing
ssh user@hostname
# New terminal
ssh user@hostname "uptime"
# Should be faster

# Check connection
netstat -a | grep socket
```

---

## Troubleshooting

### Connection Issues

```bash
# Verbose connection debugging
ssh -vvv user@hostname

# Check DNS resolution
ssh -o "VerifyHostKeyDNS ask" user@hostname

# Check host key
ssh-keygen -R hostname
ssh user@hostname

# Check authentication method
ssh -v user@hostname 2>&1 | grep "Authentications"

# Check TCP connection
nc -zv hostname 22
telnet hostname 22

# Check firewall
sudo iptables -L -n | grep 22
sudo ufw status
```

### Authentication Issues

```bash
# Check key permissions
ls -la ~/.ssh/
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_*
chmod 644 ~/.ssh/id_*.pub

# Check authorized_keys
ls -la ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Test key authentication
ssh -v user@hostname 2>&1 | grep "Offering"

# Check SELinux context (RHEL)
sudo restorecon -Rv ~/.ssh

# Check PAM for SSH
grep sshd /etc/pam.d/sshd
```

### Performance Issues

```bash
# Enable compression
ssh -C user@hostname

# Check connection speed
time ssh user@hostname "exit"

# Disable password authentication for faster login
ssh -o "PreferredAuthentications publickey" user@hostname

# Use faster cipher
ssh -c chacha20-poly1305@openssh.com user@hostname
ssh -c aes256-gcm@openssh.com user@hostname

# Check available ciphers
ssh -Q cipher
```

### Common Error Messages

```bash
# "Connection refused"
# - SSH daemon not running
# - Wrong port
# - Firewall blocking

# "Connection timeout"
# - Network issue
# - Wrong IP/hostname
# - Firewall blocking

# "Permission denied"
# - Wrong username
# - Key not in authorized_keys
# - Wrong key file

# "Host key verification failed"
# - Host key changed
# - Run: ssh-keygen -R hostname

# "Connection closed by remote host"
# - Too many connections
# - MaxStartups limit
# - Idle timeout
```

### Debugging Commands

```bash
# Check SSH daemon status
sudo systemctl status sshd

# Check listening ports
ss -tlnp | grep 22
netstat -tlnp | grep 22

# Check active sessions
who
w

# Check failed logins
last
lastlog | grep user

# Check authentication logs
sudo tail -f /var/log/auth.log | grep ssh
sudo journalctl -u sshd -f
```

---

## Quick Reference

### SSH Options

| Option | Short | Description |
|--------|-------|-------------|
| `-p` | Port | Connect to port |
| `-i` | Identity file | Private key file |
| `-l` | Login name | Username |
| `-A` | Agent forwarding | Enable SSH agent |
| `-X` | X11 forward | X11 forwarding |
| `-C` | Compression | Enable compression |
| `-v` | Verbose | Debug output |
| `-N` | No command | Tunnel only |
| `-T` | No TTY | No pseudo-terminal |
| `-f` | Background | Run in background |
| `-o` | Option | SSH config option |

### SCP Options

| Option | Description |
|--------|-------------|
| `-r` | Recursive |
| `-p` | Preserve attributes |
| `-P` | Port |
| `-C` | Compression |
| `-c` | Cipher |
| `-i` | Identity file |
| `-q` | Quiet |
| `-l` | Bandwidth limit |

### SSH Config Options

| Option | Description |
|--------|-------------|
| `Host` | Host pattern |
| `HostName` | Actual hostname |
| `User` | Username |
| `Port` | SSH port |
| `IdentityFile` | Private key |
| `ForwardAgent` | Agent forwarding |
| `ForwardX11` | X11 forwarding |
| `ProxyJump` | Jump host |
| `ProxyCommand` | Proxy command |
| `ControlMaster` | Connection sharing |
| `ControlPath` | Socket path |
| `ServerAliveInterval` | Keep-alive |
| `ConnectionAttempts` | Retry count |

### Default Files

| File | Purpose |
|------|---------|
| `~/.ssh/id_{type}` | Private key |
| `~/.ssh/id_{type}.pub` | Public key |
| `~/.ssh/config` | User config |
| `~/.ssh/known_hosts` | Trusted hosts |
| `/etc/ssh/ssh_config` | System config |
| `/etc/ssh/sshd_config` | Server config |

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 255 | SSH error |

---

## See Also

- [OpenSSH Manual](https://www.openssh.com/manual.html)
- [ssh_config man page](https://man.openbsd.org/ssh_config)
- [sshd_config man page](https://man.openbsd.org/sshd_config)
- [SSH Essentials](https://www.ssh.com/academy/ssh)
