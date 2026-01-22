# rsync

## Table of Contents
1. [Introduction](#introduction)
2. [Basic Usage](#basic-usage)
3. [Options Reference](#options-reference)
4. [SSH Usage](#ssh-usage)
5. [Filtering and Exclusions](#filtering-and-exclusions)
6. [Incremental Backups](#incremental-backups)
7. [Daemon Mode](#daemon-mode)
8. [Troubleshooting](#troubleshooting)
9. [Quick Reference](#quick-reference)

---

## Introduction

rsync is a fast and versatile file copying tool that synchronizes files and directories between local and remote systems while minimizing data transfer using efficient delta-transfer algorithm.

### How rsync Works

```
Source                         Destination
┌─────────────┐               ┌─────────────┐
│   File A    │               │   File A    │◄── Already exists, check checksum
│   File B    │─── sync ───►  │   File B    │◄── Modified, transfer delta
│   File C    │               │   File C    │◄── New, transfer full
│   File D    │─── delete ──► │             │◄── Deleted from source
└─────────────┘               └─────────────┘

Delta Transfer:
  rsync compares file checksums, not full content
  Only transfers differences (deltas)
```

### Key Features

- **Delta Transfer**: Only sends differences
- **Compression**: Reduces bandwidth
- **Preservation**: Permissions, timestamps, links, etc.
- **SSH Support**: Secure transfers
- **Filtering**: Include/exclude patterns
- **Backup**: Incremental with --link-dest

---

## Basic Usage

### Local Synchronization

```bash
# Sync source to destination
rsync source/ destination/

# Sync with trailing slash behavior
rsync source/ destination/    # Copies contents of source
rsync source destination/     # Creates source dir in destination

# Sync directory recursively
rsync -r source/ destination/

# Sync with archive mode (preserves attributes)
rsync -a source/ destination/

# Verbose output
rsync -av source/ destination/

# Show progress
rsync -av --progress source/ destination/

# Delete files not in source
rsync -av --delete source/ destination/

# Dry run (no changes)
rsync -av --dry-run source/ destination/
```

### File Operations

```bash
# Sync specific files
rsync -av --include='*.txt' source/ destination/

# Sync specific directory
rsync -av /path/to/dir1/ /path/to/dir2/

# Preserve permissions
rsync -ap source/ destination/

# Preserve modification times
rsync -at source/ destination/

# Preserve symbolic links
rsync -al source/ destination/

# Preserve device files
rsync -aD source/ destination/
```

---

## Options Reference

### Essential Options

| Option | Description |
|--------|-------------|
| `-r, --recursive` | Recurse into directories |
| `-a, --archive` | Archive mode (equals -rlptgoD) |
| `-v, --verbose` | Verbose output |
| `-h, --human-readable` | Human-readable numbers |
| `--progress` | Show progress during transfer |
| `-n, --dry-run` | Test without making changes |
| `-z, --compress` | Compress data during transfer |
| `--bwlimit=RATE` | Limit bandwidth (KB/s) |

### Attribute Options

| Option | Description |
|--------|-------------|
| `-p, --perms` | Preserve permissions |
| `-o, --owner` | Preserve owner (root only) |
| `-g, --group` | Preserve group |
| `-t, --times` | Preserve modification times |
| `-l, --links` | Preserve symbolic links |
| `-L, --copy-links` | Copy referent of symlinks |
| `-D` | Preserve devices and specials |
| `--no-OPTION` | Disable an implied option |

### Behavior Options

| Option | Description |
|--------|-------------|
| `--delete` | Delete extraneous files |
| `--delete-before` | Delete before transfer |
| `--delete-after` | Delete after transfer |
| `--delete-excluded` | Delete excluded files |
| `--ignore-errors` | Continue on errors |
| `-e, --rsh=COMMAND` | Use remote shell |
| `--existing` | Skip creating new files |
| `--update` | Skip files newer on dest |
| `--max-size=SIZE` | Skip files larger than SIZE |
| `--min-size=SIZE` | Skip files smaller than SIZE |

### Transfer Options

| Option | Description |
|--------|-------------|
| `--partial` | Keep partial transfers |
| `--partial-dir=DIR` | Dir for partial files |
| `--progress` | Show progress |
| `-P` | Short for --partial --progress |
| `--checksum` | Use checksum (not time/size) |
| `-c, --checksum` | Skip based on checksum |
| `-u, --update` | Skip if newer on dest |
| `-i, --itemize-changes` | Output change summary |

---

## SSH Usage

### Basic SSH Transfer

```bash
# Copy to remote via SSH
rsync -avz source/ user@host:/path/

# Copy from remote via SSH
rsync -avz user@host:/path/source/ destination/

# Specify SSH port
rsync -avz -e "ssh -p 2222" source/ user@host:/path/

# Use specific SSH key
rsync -avz -e "ssh -i ~/.ssh/key" source/ user@host:/path/

# SSH with compression
rsync -avz -e ssh source/ user@host:/path/

# SSH with verbosity
rsync -avz -e "ssh -v" source/ user@host:/path/
```

### SSH Config Integration

```bash
# In ~/.ssh/config
Host backup-server
    HostName backup.example.com
    User backup
    Port 2222
    IdentityFile ~/.ssh/backup_ed25519

# Then use short name
rsync -avz source/ backup-server:/backup/
```

### Common SSH Patterns

```bash
# Sync large file with progress
rsync -avzP -e ssh bigfile.iso user@host:/path/

# Sync with bandwidth limit
rsync -avz --bwlimit=5000 -e ssh source/ user@host:/path/

# Sync over slow connection
rsync -avz --progress -e "ssh -o 'Compression no'" source/ user@host:/path/

# Sync with timeout
rsync -avz -e "ssh -o 'ConnectTimeout 30'" source/ user@host:/path/
```

---

## Filtering and Exclusions

### Include/Exclude Patterns

```bash
# Exclude specific files
rsync -av --exclude='*.tmp' source/ destination/

# Exclude multiple patterns
rsync -av --exclude='*.log' --exclude='*.tmp' source/ destination/

# Exclude directory
rsync -av --exclude='node_modules/' source/ destination/

# Exclude hidden files
rsync -av --exclude='.*' source/ destination/

# Include only specific files
rsync -av --include='*.txt' --include='*.md' --exclude='*' source/ destination/

# Include directory but exclude contents
rsync -av --include='dir/' --exclude='dir/*' source/ destination/
```

### Pattern Rules

```bash
# Use filter file
rsync -av --filter='+ *.txt' --filter='- *.tmp' source/ destination/

# Filter file contents
# filter-rules.txt
+ *.txt
+ *.md
- .git
- node_modules/
- *.log

rsync -av --filter='merge filter-rules.txt' source/ destination/

# Filter rules
+  Include pattern
-  Exclude pattern
-! Exclude if no match
C  CVS exclude mode
S  Split filter rule
```

### Exclude from File

```bash
# Create exclude file
cat > exclude.txt <<EOF
*.tmp
*.log
node_modules/
.git/
__pycache__/
*.pyc
.env
.DS_Store
EOF

# Use exclude file
rsync -av --exclude-from='exclude.txt' source/ destination/
```

### Complex Filtering

```bash
# Include files under specific path
rsync -av --include='/data/**/*.txt' --exclude='/*' source/ destination/

# Exclude all but specific types
rsync -av --include='*/' --include='*.txt' --include='*.md' --exclude='*' source/ destination/

# Size-based filtering
rsync -av --max-size=100M source/ destination/
rsync -av --min-size=1k source/ destination/

# Time-based filtering
rsync -av --exclude='*.tmp' --modify-window=1 source/ destination/
```

---

## Incremental Backups

### Using --link-dest

```bash
#!/bin/bash
# Daily backup script with hard links

SOURCE="/data/"
BACKUP_BASE="/backup/daily/"
DATE=$(date +%Y%m%d)
DEST="$BACKUP_BASE$DATE"
LINK_DEST="$BACKUP_BASE$(ls -td $BACKUP_BASE*/ | head -1)"

# Create new backup with hard links to previous
rsync -av \
    --link-dest="$LINK_DEST" \
    "$SOURCE" \
    "$DEST/"

# Create symlink to latest
rm -f "$BACKUP_BASE/latest"
ln -s "$DEST" "$BACKUP_BASE/latest"

echo "Backup complete: $DEST"
```

### Backup Rotation

```bash
#!/bin/bash
# Keep 7 daily, 4 weekly, 12 monthly backups

BACKUP_DIR="/backup"
DAILY_DIR="$BACKUP_DIR/daily"
WEEKLY_DIR="$BACKUP_DIR/weekly"
MONTHLY_DIR="$BACKUP_DIR/monthly"

# Rotate daily (keep 7)
cd "$DAILY_DIR"
ls -t | tail -n +8 | xargs rm -rf

# Rotate weekly (keep 4)
cd "$WEEKLY_DIR"
ls -t | tail -n +5 | xargs rm -rf

# Rotate monthly (keep 12)
cd "$MONTHLY_DIR"
ls -t | tail -n +13 | xargs rm -rf

# Create symlinks for rotation
ln -sf $(ls -td "$DAILY_DIR"/*/ | head -1) "$BACKUP_DIR/latest"
```

### Incremental Sync Script

```bash
#!/bin/bash
# rsync-incremental.sh

SOURCE="/var/www/html"
BACKUP_DIR="/backup/www"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
DEST="$BACKUP_DIR/incremental_$TIMESTAMP"

# Create backup
rsync -avz \
    --link-dest="$BACKUP_DIR/current" \
    "$SOURCE" \
    "$DEST"

# Update current symlink
rm -f "$BACKUP_DIR/current"
ln -s "$DEST" "$BACKUP_DIR/current"

# Log
echo "$(date): Backup created at $DEST" >> "$BACKUP_DIR/backup.log"
```

---

## Daemon Mode

### Start rsyncd

```bash
# Start daemon
rsync --daemon

# Start with config file
rsync --daemon --config=/etc/rsyncd.conf

# Start on specific address
rsync --daemon --address=192.168.1.100

# Start with logging
rsync --daemon --log-file=/var/log/rsyncd.log
```

### rsyncd.conf

```bash
# /etc/rsyncd.conf
pid file = /var/run/rsyncd.pid
address = 0.0.0.0
port = 873

[backup]
    path = /backup
    comment = Backup Directory
    read only = false
    uid = root
    gid = root
    auth users = backupuser
    secrets file = /etc/rsyncd.secrets
    hosts allow = 192.168.1.0/24
    hosts deny = *

[web]
    path = /var/www
    comment = Web Files
    read only = true
    auth users = wwwuser
    secrets file = /etc/rsyncd.secrets
```

### rsyncd.secrets

```bash
# /etc/rsyncd.secrets (mode 600)
backupuser:secretpassword
wwwuser:webpassword
```

### Daemon Client Usage

```bash
# Connect to rsync daemon
rsync rsync://backupuser@host/backup

# Sync with daemon
rsync -avz rsync://backupuser@host/backup/ /local/backup/

# Push to daemon
rsync -avz /local/data/ rsync://backupuser@host/data/
```

---

## Troubleshooting

### Common Issues

```bash
# Permission denied
# Check SSH keys, permissions on destination

# Connection refused
# Verify rsync daemon is running
# Check firewall rules

# No such file or directory
# Check source path exists
# Verify trailing slashes

# Protocol mismatch
# rsync versions incompatible
# Update rsync on both systems
```

### Debug Commands

```bash
# Verbose output
rsync -av --progress source/ dest/

# Debug SSH
rsync -av -e "ssh -vvv" source/ user@host:/path/

# Show what's being transferred
rsync -avn --itemize-changes source/ dest/

# Check file at destination
rsync -av --checksum source/ dest/

# List files without transferring
rsync -av --list-only source/
```

### Performance Tuning

```bash
# Limit bandwidth
rsync -av --bwlimit=5000 source/ dest/

# Compress for slow networks
rsync -avz source/ dest/

# Skip checksum for fast networks
rsync -av --no-check-sum source/ dest/

# Use newer file heuristic
rsync -av --update source/ dest/

# Parallel transfers
rsync -av --progress -P source/ dest/
```

---

## Quick Reference

### Commands

| Command | Description |
|---------|-------------|
| `rsync -av source/ dest/` | Sync with archive mode |
| `rsync -avz source/ dest/` | Sync with compression |
| `rsync -av --delete source/ dest/` | Delete extra files |
| `rsync -avn source/ dest/` | Dry run |
| `rsync -avP source/ dest/` | Progress and partial |
| `rsync -e ssh source/ user@host:/path/` | Use SSH |

### Common Patterns

```bash
# Sync home directory
rsync -avz --delete ~/ user@host:~/backup/

# Sync with timestamped backup
rsync -avz --backup --backup-dir=/backup/$(date +%Y%m%d) source/ dest/

# Sync website
rsync -avz --delete -e ssh /var/www/ user@host:/var/www/

# Mirror directory
rsync -avz --delete --delete-excluded source/ dest/

# Sync only new files
rsync -avzu source/ dest/
```

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Syntax/usage error |
| 2 | Protocol mismatch |
| 3 | Errors selecting input/output files |
| 23 | Partial transfer due to error |
| 24 | Partial transfer due to vanished source files |

---

## See Also

- [rsync man page](https://linux.die.net/man/1/rsync)
- [rsync documentation](https://rsync.samba.org/documentation.html)
- [Backup with rsync](https://help.ubuntu.com/community/rsync)
