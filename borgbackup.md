# BorgBackup

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Repository Setup](#repository-setup)
4. [Basic Operations](#basic-operations)
5. [Retention Policies](#retention-policies)
6. [Remote Backups](#remote-backups)
7. [Encryption](#encryption)
8. [Compression and Deduplication](#compression-and-deduplication)
9. [Mount and Extract](#mount-and-extract)
10. [Maintenance](#maintenance)
11. [Troubleshooting](#troubleshooting)
12. [Quick Reference](#quick-reference)

---

## Introduction

BorgBackup is a deduplicating backup program that supports compression and authenticated encryption.

### How BorgBackup Works

```
┌─────────────────────────────────────────────────────────────────┐
│                    BorgBackup Architecture                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                     Source Files                           │  │
│  │  file1.txt (10MB)                                         │  │
│  │  file2.txt (5MB)                                          │  │
│  │  file3.txt (3MB)                                          │  │
│  └────────────────────────┬────────────────────────────────┘   │
│                           │                                      │
│                           ▼                                      │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                  Borg Backup Process                       │  │
│  │                                                              │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌───────────────────┐  │  │
│  │  │   Chunking  │──│ Deduplication│──│   Compression    │  │  │
│  │  │   (Rabin)   │  │   (SHA256)   │  │   (lz4/zstd)     │  │  │
│  │  └─────────────┘  └─────────────┘  └───────────────────┘  │  │
│  │                          │                                  │  │
│  │                          ▼                                  │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │              Encryption (AES-256-CTR)                │  │  │
│  │  │                 + HMAC-SHA256                        │  │  │
│  │  └────────────────────────┬───────────────────────────┘  │  │
│  └───────────────────────────┼──────────────────────────────┘  │
│                              │                                    │
│                              ▼                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                  Backup Repository                         │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │  data/        Chunks of data (deduplicated)         │  │  │
│  │  │  config       Repository configuration               │  │  │
│  │  │  index        Chunk index                            │  │  │
│  │  │  integrity    Integrity check data                   │  │  │
│  │  │  hints/       Metadata hints                         │  │  │
│  │  │  archive/     Named backup archives                  │  │  │
│  │  │    2024-01-15  Backup from Jan 15, 2024              │  │  │
│  │  │    2024-01-16  Backup from Jan 16, 2024              │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Features

- **Deduplication**: Only stores unique data chunks
- **Compression**: lz4, zstd, lzma, zlib
- **Encryption**: AES-256-CTR + HMAC-SHA256
- **Compression**: Multiple algorithms supported
- **Integrity**: Built-in integrity checking
- **Efficiency**: Remote backup support
- **Mount**: Mount archives as filesystems

---

## Installation

### Package Installation

```bash
# Ubuntu/Debian
sudo apt install borgbackup

# RHEL/CentOS/Fedora
sudo yum install borgbackup
sudo dnf install borgbackup

# Arch Linux
sudo pacman -S borg

# macOS
brew install borgbackup

# From pip
pip install borgbackup

# Verify installation
borg --version
```

### Dependencies

```bash
# For encryption support
sudo apt install libssl-dev

# For FUSE support (for mounting)
sudo apt install libfuse-dev

# For progress bar
sudo apt install python3-llfuse
```

---

## Repository Setup

### Create Repository

```bash
# Create local repository
borg init --encryption=repokey /path/to/backup

# Create with key stored in repo (recommended)
borg init --encryption=repokey /path/to/backup

# Create with key stored in environment
borg init --encryption=keyfile /path/to/backup

# Create repository with specific compression
borg init --encryption=repokey --compression lz4 /path/to/backup

# List available encryption modes
borg init --help
```

### Repository Types

| Mode | Description | Key Storage |
|------|-------------|-------------|
| `none` | No encryption | - |
| `repokey` | Key stored in repo | Repo root |
| `keyfile` | Key stored locally | ~/.config/borg/ |
| `authenticated` | Auth only, no encryption | - |

### Initialize Remote Repository

```bash
# SSH-based remote
borg init user@host:/path/to/backup

# With specific compression
borg init --compression zstd user@host:/path/to/backup

# SFTP-based remote
borg init sftp://user@host/path/to/backup

# With custom SSH port
borg init user@host:backup --remote-path /usr/bin/borg
```

---

## Basic Operations

### Create Backup

```bash
# Basic backup
borg create /path/to/backup::Monday ~/Documents

# Backup with description
borg create /path/to/backup::backup-2024-01-15 ~/Documents ~/Pictures

# Backup with exclude patterns
borg create /path/to/backup::backup-2024-01-15 \
    ~/Documents \
    --exclude '*.tmp' \
    --exclude '*.log' \
    --exclude-cache

# Backup with exclude file
borg create /path/to/backup::backup-2024-01-15 ~/Documents \
    --exclude-from ~/excludes.txt

# Backup with progress
borg create /path/to/backup::backup-2024-01-15 ~/Documents --progress

# Backup with stats
borg create /path/to/backup::backup-2024-01-15 ~/Documents --stats

# Backup with checkpoint
borg create /path/to/backup::backup-2024-01-15 ~/Documents \
    --checkpoint-interval 3600 \
    --chunker-params 19,23,21,4095

# Dry run
borg create --dry-run /path/to/backup::backup-2024-01-15 ~/Documents
```

### List Archives

```bash
# List all archives
borg list /path/to/backup

# List archives with details
borg list /path/to/backup --info

# List specific archive contents
borg list /path/to/backup::backup-2024-01-15

# List with JSON output
borg list /path/to/backup --json

# List with specific format
borg list /path/to/backup --format '{name} {time}{NL}'
```

### List Files

```bash
# List files in archive
borg list /path/to/backup::backup-2024-01-15

# List files matching pattern
borg list /path/to/backup::backup-2024-01-15 --pattern 'p:*/*.txt'

# List with long format
borg list /path/to/backup::backup-2024-01-15 -l

# List specific directory in archive
borg list /path/to/backup::backup-2024-01-15 path/to/directory
```

### Check Integrity

```bash
# Check repository integrity
borg check /path/to/backup

# Check specific archive
borg check /path/to/backup::backup-2024-01-15

# Check with repair
borg check --repair /path/to/backup

# Check only metadata
borg check --metadata-only /path/to/backup

# Verify consistency
borg check --verify-data /path/to/backup
```

### Delete Archives

```bash
# Delete specific archive
borg delete /path/to/backup::backup-2024-01-15

# Delete oldest archives matching pattern
borg delete /path/to/backup --glob-archives '*-2023-*'

# Delete everything (keep repo)
borg delete /path/to/backup

# Delete with confirmation
borg delete --dry-run /path/to/backup::backup-2024-01-15
```

---

## Retention Policies

### Prune Command

```bash
# Keep last 7 daily, 4 weekly, 12 monthly
borg prune /path/to/backup \
    --keep-daily 7 \
    --keep-weekly 4 \
    --keep-monthly 12

# Keep last 10 backups regardless of time
borg prune /path/to/backup --keep-last 10

# Keep backups from last 30 days
borg prune /path/to/backup --keep-within 30d

# Prune with prefix
borg prune /path/to/backup --prefix 'backup-' \
    --keep-daily 7 --keep-weekly 4 --keep-monthly 12

# Prune and compact
borg prune --compact /path/to/backup \
    --keep-daily 7 --keep-weekly 4 --keep-monthly 12

# Prune dry run
borg prune --dry-run /path/to/backup \
    --keep-daily 7 --keep-weekly 4 --keep-monthly 12
```

### Retention Options

| Option | Description |
|--------|-------------|
| `--keep-last N` | Keep last N archives |
| `--keep-hourly N` | Keep last N hourly archives |
| `--keep-daily N` | Keep last N daily archives |
| `--keep-weekly N` | Keep last N weekly archives |
| `--keep-monthly N` | Keep last N monthly archives |
| `--keep-yearly N` | Keep last N yearly archives |
| `--keep-within INTERVAL` | Keep archives within time |
| `--prefix PREFIX` | Archive name prefix filter |

### Backup Script with Retention

```bash
#!/bin/bash
# /usr/local/bin/backup-home.sh
set -e

export BORG_REPO="/backup/borg"
export BORG_PASSPHRASE="your-passphrase"

# Create backup with date archive name
borg create \
    ::home-{now:%Y-%m-%d} \
    /home/user \
    --compression lz4 \
    --exclude '~/.cache/*' \
    --exclude '*.tmp' \
    --exclude-caches \
    --info \
    --stats

# Prune old backups
borg prune \
    --keep-daily 7 \
    --keep-weekly 4 \
    --keep-monthly 12 \
    --info

# Compact repository
borg compact

echo "Backup complete"
```

---

## Remote Backups

### SSH Configuration

```bash
# Create backup on remote server
borg create user@backup-server:/path/to/backup::backup-{now:%Y-%m-%d} /path/to/data

# List remote archives
borg list user@backup-server:/path/to/backup

# Check remote integrity
borg check user@backup-server:/path/to/backup

# Prune remote
borg prune user@backup-server:/path/to/backup --keep-daily 7
```

### SSH Shortcuts

```bash
# Add to ~/.ssh/config
Host backup-server
    HostName backup.example.com
    User backup
    Port 2222
    IdentityFile ~/.ssh/borg_ed25519
    BatchMode yes
    ConnectTimeout 30

# Then use short name
borg create backup-server:/backup::backup-$(date +%Y-%m-%d) /data
```

### Environment Setup

```bash
# Set repository
export BORG_REPO="user@host:/path/to/backup"

# Set passphrase
export BORG_PASSPHRASE="your-secure-passphrase"

# Or use key file
export BORG_PASSCOMMAND="cat /path/to/passfile"

# Create systemd service
```

### systemd Service

```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Backup with BorgBackup
After=network-online.target

[Service]
Type=oneshot
Environment="BORG_REPO=user@host:/backup"
Environment="BORG_PASSPHRASE=your-passphrase"
ExecStart=/usr/bin/borg create \
    ::home-{now:%Y-%m-%d} /home/user \
    --compression lz4 \
    --exclude-caches
ExecStart=/usr/bin/borg prune \
    --keep-daily 7 \
    --keep-weekly 4 \
    --keep-monthly 12
ExecStart=/usr/bin/borg compact
User=backup
Nice=10
IOSchedulingClass=best-effort
IOSchedulingPriority=7

[Install]
WantedBy=multi-user.target
```

### systemd Timer

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily backup with BorgBackup

[Timer]
OnCalendar=*-*-02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

---

## Encryption

### Key Management

```bash
# Create key (during init)
borg init --encryption=repokey /path/to/backup

# Change passphrase
borg key change-passphrase /path/to/backup

# Export key
borg key export /path/to/backup

# Export key to file
borg key export /path/to/backup /path/to/key-backup.txt

# Import key
borg key import /path/to/backup /path/to/key-backup.txt

# Export public key (for keyfile mode)
borg key export --paper /path/to/backup

# Check key location
borg config /path/to/backup encryption
```

### Repokey Mode

```bash
# Initialize with repokey
borg init --encryption=repokey /path/to/backup

# Key stored in: /path/to/backup/keyfile
# Must backup this key separately

# Export for safekeeping
borg key export /path/to/backup > ~/borg-key-backup.txt
```

### Keyfile Mode

```bash
# Initialize with keyfile
borg init --encryption=keyfile /path/to/backup

# Key stored in: ~/.config/borg/keys/
# Automatic discovery by Borg

# Transfer key to another machine
scp ~/.config/borg/keys/* user@other-machine:~/.config/borg/keys/
```

---

## Compression and Deduplication

### Compression Options

```bash
# Create with lz4 (fast, low compression)
borg create --compression lz4 /path/to/backup::backup ~/data

# Create with zstd (balanced)
borg create --compression zstd /path/to/backup::backup ~/data

# Create with lzma (high compression, slow)
borg create --compression lzma /path/to/backup::backup ~/data

# Create with zlib (medium compression)
borg create --compression zlib /path/to/backup::backup ~/data

# Create with auto (detect per file)
borg create --compression auto /path/to/backup::backup ~/data

# List compression algorithms
borg create --help | grep -A 10 "compression"
```

### Compression Comparison

| Algorithm | Speed | Compression | Use Case |
|-----------|-------|-------------|----------|
| `lz4` | Very fast | Low | Fast backups, low compression |
| `lz4hc` | Fast | Medium | Good balance |
| `zstd` | Fast | High | Best balance |
| `zstd:19` | Slow | Very high | Maximum compression |
| `zlib` | Medium | Medium | Compatible |
| `lzma` | Slow | Very high | Maximum compression |

### Deduplication

```bash
# Deduplication happens automatically
# Based on content-defined chunking (Rabin)

# View deduplication stats
borg create --stats /path/to/backup::backup ~/data

# Example output:
# ------------------------------------------------------------------------------
# Archive name: backup-2024-01-15
# Time: 2024-01-15 02:00:00
# Duration: 00:05:23.45
# Number of files: 15000
# This archive:                    5.23 GB
# All archives:                   50.23 GB
# Deduplication: 85.2% saved (30 GB)
# ------------------------------------------------------------------------------

# Chunk size affects deduplication
borg create --chunker-params 19,23,21,4096 /path/to/backup::backup ~/data
```

### Chunking Parameters

```bash
# Default: 19,23,21,4096
# Smaller chunks: more deduplication, more metadata
borg create --chunker-params 10,13,12,1024 /path/to/backup::backup ~/data

# Larger chunks: less deduplication, less metadata
borg create --chunker-params 22,23,21,65536 /path/to/backup::backup ~/data
```

---

## Mount and Extract

### Mount Archive

```bash
# Mount archive as filesystem
borg mount /path/to/backup::backup-2024-01-15 /mnt/backup

# Browse files
ls /mnt/backup
cd /mnt/backup/path/to/files
cat filename

# Unmount
borg umount /mnt/backup

# Mount specific path from archive
borg mount /path/to/backup::backup-2024-01-15 /mnt/backup \
    --path /home/user/Documents

# Mount with FUSE options
borg mount -o allow_other /path/to/backup::backup /mnt/backup
```

### Extract Files

```bash
# Extract entire archive
borg extract /path/to/backup::backup-2024-01-15

# Extract to specific location
borg extract /path/to/backup::backup-2024-01-15 --destination /tmp/restore

# Extract specific files
borg extract /path/to/backup::backup-2024-01-15 home/user/Document.txt

# Extract with pattern
borg extract /path/to/backup::backup-2024-01-15 \
    --pattern '**/Documents/*.txt'

# Extract matching patterns
borg extract /path/to/backup::backup-2024-01-15 \
    --exclude '*.tmp' \
    path/to/directory
```

### Export and Recovery

```bash
# Export archive as tar
borg export-tar /path/to/backup::backup-2024-01-15 backup.tar

# Export with compression
borg export-tar /path/to/backup::backup-2024-01-15 - | gzip > backup.tar.gz

# Export specific files
borg export-tar /path/to/backup::backup-2024-01-15 - -- \
    path/to/file1 path/to/file2 | tar -xf -

# List archive contents before extract
borg list /path/to/backup::backup-2024-01-15
```

---

## Maintenance

### Compact Repository

```bash
# Compact repository ( reclaim space)
borg compact /path/to/backup

# Compact with threshold
borg compact --threshold 10M /path/to/backup

# Compact after pruning
borg prune --compact /path/to/backup --keep-daily 7
```

### Check Repository

```bash
# Basic check
borg check /path/to/backup

# Check with data verification
borg check --verify-data /path/to/backup

# Check specific archive
borg check /path/to/backup::backup-2024-01-15

# Check with repair
borg check --repair /path/to/backup

# Show repository info
borg info /path/to/backup
borg info /path/to/backup::backup-2024-01-15
```

### Repository Status

```bash
# List archives with sizes
borg list --sort-by size /path/to/backup

# Show repository config
borg config /path/to/backup

# Show archive info
borg info /path/to/backup::backup-2024-01-15

# Show archive contents with sizes
borg list --verbose /path/to/backup::backup-2024-01-15

# Check for orphaned chunks
borg check --repair /path/to/backup
```

### Upgrade Repository

```bash
# Upgrade repository format
borg upgrade /path/to/backup

# Upgrade to newer format
borg upgrade --progress /path/to/backup

# Note: Always backup before upgrading
```

---

## Troubleshooting

### Common Issues

**Permission Denied**

```bash
# Check repository permissions
ls -la /path/to/backup

# Fix permissions
sudo chown -R user:user /path/to/backup
chmod 700 /path/to/backup

# Check SSH permissions
ssh -i ~/.ssh/key user@host
```

**Repository Locked**

```bash
# Check for running processes
ps aux | grep borg

# Remove stale lock
borg break-lock /path/to/backup

# Force break lock
borg break-lock --force /path/to/backup
```

**Wrong Passphrase**

```bash
# Verify passphrase
export BORG_PASSPHRASE="correct-passphrase"
borg list /path/to/backup

# Test key
borg key export /path/to/backup | gpg -d
```

**Insufficient Space**

```bash
# Check disk space
df -h /path/to/backup

# Use smaller chunks
borg create --chunker-params 19,23,16,2048 /path/to/backup::backup ~/data

# Compact first
borg compact /path/to/backup
```

### Debug Commands

```bash
# Verbose output
borg create --info --progress /path/to/backup::backup ~/data

# Debug mode
borg create --debug /path/to/backup::backup ~/data 2>&1 | tee debug.log

# List repository config
borg config /path/to/backup

# Show chunks
borg list-chunks /path/to/backup

# Check archive consistency
borg check /path/to/backup::archive --verify-data
```

### Recovery Scenarios

```bash
# Lost key - cannot recover encrypted data
# Keep key backups secure!

# Corrupted repository
borg check --repair /path/to/backup

# Partial backup recovery
borg extract /path/to/backup::backup-2024-01-15 --dry-run
borg extract /path/to/backup::backup-2024-01-15 path/to/file

# Remote recovery
borg extract user@host:/backup::backup-2024-01-15 path/to/file
```

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| `borg init` | Initialize repository |
| `borg create` | Create backup |
| `borg list` | List archives/files |
| `borg extract` | Extract archive |
| `borg mount` | Mount archive |
| `borg umount` | Unmount archive |
| `borg delete` | Delete archive |
| `borg prune` | Remove old archives |
| `borg check` | Check integrity |
| `borg info` | Show info |
| `borg compact` | Compact repository |

### Common Options

| Option | Description |
|--------|-------------|
| `--compression` | Set compression algorithm |
| `--exclude` | Exclude patterns |
| `--exclude-from` | Exclude from file |
| `--exclude-caches` | Exclude cache dirs |
| `--progress` | Show progress |
| `--stats` | Show statistics |
| `--info` | Info-level logging |
| `--debug` | Debug output |
| `--dry-run` | Test without changes |
| `--chunker-params` | Set chunking parameters |

### Environment Variables

| Variable | Description |
|----------|-------------|
| `BORG_REPO` | Repository path |
| `BORG_PASSPHRASE` | Encryption passphrase |
| `BORG_PASSCOMMAND` | Passphrase command |
| `BORG_RSH` | SSH command |
| `BORG_LOGGING_CONF` | Logging config |
| `BORG_FILES_CACHE_TTL` | Cache TTL |

### File Locations

| Path | Purpose |
|------|---------|
| `~/.config/borg/` | Config and keys |
| `/path/to/backup/` | Repository |
| `/var/log/borg/` | Logs |

---

## See Also

- [BorgBackup Documentation](https://borgbackup.readthedocs.io/)
- [Quick Start](https://borgbackup.readthedocs.io/en/stable/quickstart.html)
- [FAQ](https://borgbackup.readthedocs.io/en/stable/faq.html)
- [Installation](https://borgbackup.readthedocs.io/en/stable/installation.html)
