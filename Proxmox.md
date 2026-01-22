# Proxmox VE Complete Cheat Sheet

## Table of Contents
1. [System Basics](#1-system-basics)
2. [VM Management](#2-vm-management)
3. [Container Management (LXC)](#3-container-management-lxc)
4. [Cluster Management](#4-cluster-management)
5. [Storage Management](#5-storage-management)
6. [Network Configuration](#6-network-configuration)
7. [Backup & Restore](#7-backup--restore)
8. [User & Permissions](#8-user--permissions)
9. [Monitoring & Logs](#9-monitoring--logs)
10. [Updates & Subscription](#10-updates--subscription)
11. [High Availability](#11-high-availability)
12. [Useful Utilities](#12-useful-utilities)

---

## 1. System Basics

### Proxmox VE Access
| Command | Description |
|---------|-------------|
| `https://your-host:8006` | Web UI access |
| `pvesh get /version` | API version check |
| `pvesh get /cluster/resources` | Cluster resources |

### System Information
```bash
pvesh get /nodes/localhost/status          # Node status
uname -a                                   # Kernel info
hostnamectl                                # Hostname
cat /etc/proxmox-release                   # Proxmox version
cat /etc/os-release                        # OS info
pveversion                                 # Proxmox version
```

### Power Management
```bash
systemctl reboot                           # Reboot node
systemctl poweroff                         # Shutdown node
shutdown -h now                            # Shutdown
shutdown -r +10                            # Reboot in 10 min
```

### Service Management
```bash
systemctl status pve-cluster               # Cluster service
systemctl status pveproxy                  # Web UI service
systemctl status pvedaemon                 # PVE daemon
systemctl status rpcbind                   # RPC binding
systemctl restart pve-cluster              # Restart cluster
systemctl restart pveproxy                 # Restart web UI
systemctl stop pve-cluster                 # Stop cluster
systemctl start pve-cluster                # Start cluster
```

### Proxmox Services
| Service | Description | Port |
|---------|-------------|------|
| `pveproxy` | Web UI | 8006 |
| `pvedaemon` | PVE daemon | |
| `pve-cluster` | Cluster manager | |
| `spiceproxy` | SPICE proxy | |
| `pve-firewall` | Firewall | |

---

## 2. VM Management

### VM Creation
```bash
# Create VM with defaults
qm create 100                              # Create VM ID 100

# Create VM with specifications
qm create 100 \
  --name myvm \
  --memory 4096 \
  --cores 2 \
  --cpu host \
  --net0 virtio,bridge=vmbr0 \
  --scsi0 local-lvm:32 \
  --bootdisk scsi0 \
  --ide2 local:cloudinit \
  --serial0 socket \
  --vga serial0

# Create VM from ISO
qm create 101 \
  --name ubuntu-vm \
  --memory 2048 \
  --cores 2 \
  --cdrom local:iso/ubuntu-22.04.iso \
  --scsi0 local-lvm:20 \
  --bootdisk scsi0

# Create VM with cloud-init
qm create 102 \
  --name cloud-vm \
  --memory 4096 \
  --cores 2 \
  --scsi0 local-lvm:30 \
  --ide2 local:cloudinit \
  --net0 virtio,bridge=vmbr0 \
  --autostart 1
```

### VM Lifecycle
```bash
qm start 100                               # Start VM
qm stop 100                                # Stop VM (graceful)
qm shutdown 100                            # Shutdown VM
qm reset 100                               # Reset VM
qm suspend 100                             # Suspend VM
qm resume 100                              # Resume VM
qm migrate 100 node2 --online              # Live migrate
qm clone 100 200 --name clone-vm           # Clone VM
qm template 100                            # Convert to template
qm destroy 100                             # Delete VM
qm destroy 100 --purge                     # Delete with storage
```

### VM Configuration
```bash
qm showcmd 100                             # Show creation command
qm config 100                              # Show VM config
qm set 100 --memory 8192                   # Update memory
qm set 100 --cores 4                       # Update cores
qm set 100 --cpu host,flags=+pcid          # Update CPU
qm set 100 --net0 virtio,bridge=vmbr0      # Update network
qm set 100 --scsi1 local-lvm:50            # Add disk
qm set 100 --cdrom local:iso/test.iso      # Change ISO
qm set 100 --delete scsi1                  # Remove disk
qm set 100 --host 1                        # Enable host CPU
qm set 100 --balloon 2048                  # Set balloon
qm set 100 --autostart 1                   # Enable autostart
qm set 100 --onboot 1                      # Start on boot
```

### VM Information
```bash
qm list                                    # List all VMs
qm list --vmid 100                         # Specific VM
qm status 100                              # VM status
qm guest cmd 100 status                    # Guest agent status
qm guest cmd 100 exec -- /bin/uptime       # Execute in guest
qm vncproxy 100                            # VNC proxy
qm terminal 100                            # Terminal console
qm monitor 100                             # Monitor mode
```

### VM Disk Management
```bash
qm disk resize 100 scsi0 +20G              # Resize disk
qm disk move 100 scsi0 --target-storagedir # Move disk
qm disk import 100 /path/to/disk.qcow2 local-lvm
qm disk detach 100 scsi1                   # Detach disk
qm disk remove 100 scsi1                   # Remove disk
```

### VM Templates
```bash
qm template 100                            # Create template
qm clone 100 200 --name new-vm             # Clone template
qm clone 100 200 --full --name new-vm      # Full clone
qm clone 100 200 --linked 1 --name new-vm  # Linked clone
```

### SPICE Remote Display
```bash
qm set 100 --vga qxl                       # Enable SPICE
qm set 100 --serial0 socket
spice-html5                               # Web SPICE
```

### VM Resource Limits
```bash
qm set 100 --cpu 1 --cpulimit 1            # CPU limit
qm set 100 --cpu 2 --cpuunits 1024         # CPU shares
qm set 100 --memory 4096 --swap 2048       # Memory + swap
qm set 100 --iobw 100 --iops rd=50,wr=30   # I/O limits
```

### VM Snapshots
```bash
qm snapshot 100 snap1                      # Create snapshot
qm snapshot-list 100                       # List snapshots
qm rollback 100 snap1                      # Rollback snapshot
qm delsnapshot 100 snap1                   # Delete snapshot
```

---

## 3. Container Management (LXC)

### Container Creation
```bash
# Create container with defaults
pct create 200 local-lvm:latest            # Create CT ID 200

# Create container with specifications
pct create 200 \
  --hostname mycontainer \
  --memory 1024 \
  --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --rootfs local-lvm:10 \
  --ostemplate local:vztmpl/ubuntu-22.04.tar.gz

# Create from CT template
pct create 201 \
  --hostname web-server \
  --memory 2048 \
  --cores 4 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.1.50/24 \
  --rootfs local-lvm:20 \
  --ostemplate local:vztmpl/ubuntu-22.04.tar.gz \
  --password secret
```

### Container Lifecycle
```bash
pct start 200                              # Start container
pct stop 200                               # Stop container
pct shutdown 200                           # Shutdown gracefully
pct reboot 200                             # Reboot container
pct suspend 200                            # Suspend container
pct resume 200                             # Resume container
pct migrate 200 node2 --online             # Live migrate
pct clone 200 201 --hostname clone-ct      # Clone container
pct destroy 200                            # Delete container
pct destroy 200 --purge                    # Delete with storage
```

### Container Configuration
```bash
pct list                                   # List all containers
pct status 200                             # Container status
pct config 200                             # Show config
pct fstab 200                              # Show fstab
pct set 200 --memory 2048                  # Update memory
pct set 200 --cores 4                      # Update cores
pct set 200 --hostname new-hostname        # Change hostname
pct set 200 --net0 ip=192.168.1.60/24      # Set static IP
pct set 200 --net0 ip=dhcp                 # Set DHCP
pct set 200 --rootfs local-lvm:30          # Resize root
pct set 200 --password newpass             # Change password
pct set 200 --unprivileged 1               # Enable unprivileged
pct set 200 --features keyctl=1            # Enable keyctl
pct set 200 --autostart 1                  # Enable autostart
```

### Container Operations
```bash
pct enter 200                              # Enter container
pct exec 200 -- /bin/bash                  # Execute command
pct exec 200 -- apt update                 # Run command
pct console 200                            # Console access
pct mount 200                              # Mount for chroot
pct umount 200                             # Unmount
```

### Container Templates
```bash
pct template 200                           # Create template
pct clone 200 201 --hostname new-ct        # Clone template
vztmpl-dl ubuntu-22.04                     # Download template
pveam update                               # Update appliance list
pveam available                            # List available templates
pveam download local ubuntu-22.04          # Download to local
```

### Container Resource Limits
```bash
pct set 200 --cpu 1 --cpulimit 1           # CPU limit
pct set 200 --cpu 2 --cpuunits 2048        # CPU shares
pct set 200 --memory 1024 --swap 512       # Memory + swap
pct set 200 --disk 10                      # Disk quota
pct set 200 --shares 100                   # Share percentage
```

### Container Snapshots
```bash
pct snapshot 200 snap1                     # Create snapshot
pct snapshot-list 200                      # List snapshots
pct rollback 200 snap1                     # Rollback snapshot
pct delsnapshot 200 snap1                  # Delete snapshot
```

### Container Backup
```bash
vzdump 200                                 # Backup container
vzdump 200 --storage local                 # Backup to storage
vzdump 200 --mode snapshot                 # Snapshot mode
vzdump 200 --compress gzip                 # Compress backup
vzdump 200 --exclude-path /var/cache       # Exclude paths
vzdump 200-210                             # Backup multiple
vzdump --all                               # Backup all containers
vzdump --all --storage local --max-workers 2
```

### Container Restore
```bash
pct restore 200 /var/lib/vz/dump/vzdump-200.tar.gz
pct restore 201 /var/lib/vz/dump/vzdump-200.tar.gz \
  --hostname new-ct \
  --restore
pct restore 200 /var/lib/vz/dump/vzdump-200.tar.gz \
  --rootfs local-lvm:20
```

---

## 4. Cluster Management

### Cluster Operations
```bash
pvecm status                               # Cluster status
pvecm nodes                                # List nodes
pvecm add node2                            # Add node to cluster
pvecm delnode node1                        # Remove node
pvecm expected 1                           # Set expected votes
pvecm poll all                             # Poll all nodes
pvecm仲裁                                  # Cluster quorum info
```

### Cluster Creation
```bash
# On first node (already has cluster)
pvecm create mycluster                     # Create cluster

# On additional nodes
pvecm add node1-pve.local                  # Join cluster
pvecm add 192.168.1.10                     # Join by IP
pvecm add node1-pve.local --use-ipswan     # Use IP WAN
```

### Cluster Membership
```bash
pvecm status                               # Show cluster status
pvecm nodes -details                       # Detailed node list
corosync-cfgtool -s                        # View cluster status
corosync-cmapctl | grep members            # View members
systemctl status corosync                  # Corosync status
journalctl -u corosync                     # Corosync logs
```

### Cluster Networking
```bash
pvecm addnode node2 --link0 192.168.1.11 --link1 192.168.2.11
pvecm addnode node2 --link0 192.168.1.11
cluster.conf                                # Cluster config location
```

### Quorum Management
```bash
pvecm expected 3                           # Set expected votes
pvecm qdisk -l                             # List quorum disks
pvecm qdisk -c /dev/sda1                   # Create quorum disk
pvecm qdisk -d /dev/sda1                   # Delete quorum disk
```

### Cluster Resource Management
```bash
pcs status                                 # Pacemaker status
pcs cluster status                         # Cluster status
pcs resource show                          # Resources
pcs resource start ResourceName            # Start resource
pcs resource stop ResourceName             # Stop resource
```

### Migration
```bash
qm migrate 100 node2 --online              # Live VM migration
qm migrate 100 node2 --offline             # Offline migration
pct migrate 200 node2 --online             # Live CT migration
pct migrate 200 node2 --offline            # Offline CT migration
```

### Two-Factor Authentication
```bash
pvecm addnode node2 --totp                 # TOTP for join
```

---

## 5. Storage Management

### Storage Configuration
```bash
# Add storage
pvesm add lvmthin pve --thin-pool data --vgname pve
pvesm add directory local -path /var/lib/vz
pvesm add lvm local --vgname local --content rootdir,images
pvesm add nfs nfs-share --server 192.168.1.10 --export /data
pvesm add cifs cifs-share --server 192.168.1.10 --share data
pvesm add ceph-rbd ceph --name admin --monhost 192.168.1.10
pvesm add zfspool zpool --pool tank --target /tank

# Update storage
pvesm set local --content images,rootdir
pvesm set nfs-share --max 10000           # Set max size (GB)
pvesm set nfs-share --nodes node1,node2   # Restrict to nodes
```

### Storage Operations
```bash
pvesm list local                          # List storage contents
pvesm status                              # Show storage status
pvesm scan nfs 192.168.1.10               # Scan NFS shares
pvesm scan lspci                          # Scan for hardware
pvesm free local                          # Show free space
pvesm usage                               # Storage usage
```

### Storage Management
```bash
# LVM
lvdisplay                                 # Show LVM volumes
lvcreate -L 100G -n data pve              # Create LV
lvextend -L +50G /dev/pve/data            # Extend LV
lvremove /dev/pve/old                     # Remove LV

# ZFS
zfs list                                  # ZFS datasets
zpool status                              # ZFS pools
zfs create tank/data                      # Create dataset
zfs snapshot tank/data@snap1              # Create snapshot
zfs rollback tank/data@snap1              # Rollback snapshot
zfs destroy tank/data@snap1               # Destroy snapshot
zpool add tank mirror /dev/sda /dev/sdb   # Add mirror
```

### Disk Management
```bash
lsblk                                     # List block devices
fdisk -l /dev/sda                         # Show partitions
parted -l                                 # List partitions
pvesm disk list                            # List Proxmox disks
pvesm disk initdb                          # Initialize disk DB
smartctl -a /dev/sda                      # SMART info
hdparm -I /dev/sda                        # Disk info
```

### ISO/Template Storage
```bash
# Upload ISO
pveceph upload ISO/ubuntu-22.04.iso /path/to/file.iso
# Or via web UI: Datacenter -> Storage -> ISO Images -> Upload

# Download template
pveam update                              # Update template list
pveam available                           # List available
pveam download local ubuntu-22.04         # Download
```

### Backup Storage
```bash
vzdump --storage local --compress gzip 100
vzdump --storage nfs-backup --all
# Restore
pct restore 200 /var/lib/vz/dump/vzdump-200.tar.gz
qmrestore /var/lib/vz/dump/vzdump-100.tar.gz 101
```

---

## 6. Network Configuration

### Network Basics
```bash
# Bridge configuration
networkctl status                         # Show network status
ip addr                                   # Show IP addresses
ip link                                   # Show interfaces
ip route                                  # Show routing table
bridge fdb show                           # Bridge FDB
brctl show                                # Show bridges
cat /etc/network/interfaces               # Network config
systemctl restart networking              # Restart network
```

### Proxmox Bridges
```bash
# Create bridge (in /etc/network/interfaces)
auto vmbr0
iface vmbr0 inet static
address 192.168.1.100
netmask 255.255.255.0
gateway 192.168.1.1
bridge-ports enp0s31f6
bridge-stp off
bridge-fd 0

# NAT bridge
auto vmbr1
iface vmbr1 inet static
address 10.0.0.1
netmask 255.255.255.0
post-up iptables -t nat -A POSTROUTING -s '10.0.0.0/24' -o eth0 -j MASQUERADE
post-down iptables -t nat -D POSTROUTING -s '10.0.0.0/24' -o eth0 -j MASQUERADE
```

### VLAN Configuration
```bash
# Tagged VLAN
auto vmbr0.100
iface vmbr0.100 inet static
address 192.168.100.1
netmask 255.255.255.0
vlan-raw-device vmbr0

# VM with VLAN
qm set 100 --net0 virtio,bridge=vmbr0,tag=100
pct set 200 --net0 name=eth0,bridge=vmbr0,tag=100
```

### Bond Configuration
```bash
# LACP bond
auto bond0
iface bond0 inet manual
slaves enp0s31f6 enp1s0
bond-mode 802.3ad
bond-miimon 100
bond-lacp-rate 1
bond-min-links 1

auto vmbr0
iface vmbr0 inet static
address 192.168.1.100
netmask 255.255.255.0
gateway 192.168.1.1
bridge-ports bond0
bridge-stp off
bridge-fd 0
```

### Firewall
```bash
# Proxmox VE Firewall
iptables -L                               # List rules
iptables -A INPUT -p tcp --dport 8006 -j ACCEPT
iptables -t nat -L                        # NAT rules
iptables-save > /etc/iptables.rules       # Save rules
systemctl status pve-firewall             # Firewall status
pve-firewall stop                         # Stop firewall
pve-firewall start                        # Start firewall

# Guest firewall
qm set 100 --firewall 1                   # Enable firewall
pct set 200 --features firewall=1         # Enable CT firewall
```

### Network Troubleshooting
```bash
ping google.com                           # Test connectivity
traceroute google.com                     # Trace route
nslookup google.com                       # DNS lookup
dig google.com                            # DNS query
netstat -tulpn                            # Open ports
ss -tulpn                                 # Open ports
tcpdump -i eth0 port 8006                 # Capture traffic
```

---

## 7. Backup & Restore

### VZDump (Container/VM Backup)
```bash
# Single backup
vzdump 100                                # Backup VM 100
vzdump 200                                # Backup CT 200

# Backup with options
vzdump 100 --storage local --compress gzip
vzdump 100 --mode snapshot                # Snapshot mode
vzdump 100 --mail  admin@example.com      # Email notification
vzdump 100 --max 20000                    # Max size in MB
vzdump 100 --exclude /var/cache           # Exclude paths

# Bulk backup
vzdump --all                              # Backup all
vzdump 100-110                            # Backup range
vzdump --storage nfs --all                # Backup to NFS

# Scheduled backup
# Edit /etc/vzdump.conf
```

### VZDump Configuration
```bash
# /etc/vzdump.conf
dumpdir: /var/lib/vz/dump
storage: local
compress: gzip
mode: snapshot
mailto: admin@example.com
maxjobs: 2
exclude: /var/cache/*,/var/tmp/*
```

### VM Restore
```bash
# Restore VM
qmrestore /var/lib/vz/dump/vzdump-100.tar.gz 101
qmrestore /var/lib/vz/dump/vzdump-100.tar.gz 101 --storage local-lvm

# Restore with new ID
qmrestore /var/lib/vz/dump/vzdump-100.tar.gz 200
```

### Container Restore
```bash
pct restore 201 /var/lib/vz/dump/vzdump-200.tar.gz
pct restore 201 /var/lib/vz/dump/vzdump-200.tar.gz --hostname new-ct
pct restore 201 /var/lib/vz/dump/vzdump-200.tar.gz --rootfs local-lvm:30
```

### PBS (Proxmox Backup Server)
```bash
# Configure PBS datastore
pvebackup add --server backup.example.com --datastore main

# Backup to PBS
vzdump 100 --storage proxmox-backup

# Restore from PBS
pbs-destore --server backup.example.com --datastore main 100
pbs-destore --server backup.example.com --datastore main 100 --target /var/lib/vz/dump

# List PBS backups
pvesm list proxmox-backup
```

### Replication
```bash
# Configure replication
pvesr add --id local:replication/vm-100 --target remote --schedule "*:00"

# Manual replication
pvesr schedule --id local:replication/vm-100 --enable
pvesr sync --id local:replication/vm-100

# Replication status
pvesr list
```

---

## 8. User & Permissions

### User Management
```bash
# Add user
pveuser add admin@pve --password secret

# Update user
pveuser update admin@pve --password newpass
pveuser update admin@pve --email admin@example.com

# Delete user
pveuser delete olduser@pve

# List users
pveuser list
cat /etc/pve/user.cfg                     # User config file
```

### Group Management
```bash
# Add group
pvegroup add admins

# Add user to group
pveuser add admin@pve --group admins

# Remove from group
pveuser update admin@pve --delete-group admins

# List groups
pvegroup list
```

### Permission Management
```bash
# Add permission
pveperm set /pool/admins --users admin@pve --role Administrator

# Remove permission
pveperm delete /pool/admins --users admin@pve

# View permissions
pveperm list
pveuser get admin@pve                     # User permissions
```

### Role Management
```bash
# Create custom role
pveuser add-role MyRole

# Modify role permissions
pveuser modify-role MyRole --privileges VM.Audit,VM.Clone

# View roles
pveuser list-roles
cat /etc/pve/roles.cfg                    # Roles config
```

### API Token
```bash
# Create API token
pveum token add mytoken --privsep 0

# API token with specific permissions
pveum token add readonly-token --privsep 0 --expire 30d

# Revoke token
pveum token delete mytoken
```

### Authentication Realms
```bash
# Enable/disable realm
pveum realm disable ldap
pveum realm enable ldap

# Configure realm
pveum realm add ldap --server ldap.example.com --base-dn "dc=example,dc=com"

# List realms
pveum realm list
```

### Two-Factor Authentication
```bash
# Enable TOTP for user
pveum user totp admin@pve

# Disable TOTP
pveum user disable-totp admin@pve
```

---

## 9. Monitoring & Logs

### System Logs
```bash
# Proxmox logs
tail -f /var/log/pve/cluster.log          # Cluster log
tail -f /var/log/pve/local.log            # Local log
tail -f /var/log/pve/tasks.log            # Tasks log
tail -f /var/log/pve/rollback.log         # Rollback log
tail -f /var/log/pve-firewall.log         # Firewall log

# System logs
journalctl -u pve-cluster                 # Cluster service
journalctl -u pveproxy                    # Web UI service
journalctl -u pvedaemon                   # Daemon service
journalctl -u corosync                    # Corosync service
journalctl -f                             # Follow all logs
```

### Task Management
```bash
# View tasks
pvesh get /cluster/tasks                  # List tasks
pvesh get /cluster/tasks --vmid 100       # VM-specific tasks

# Task status
pvesh get /cluster/tasks/UPID:node1:0001:1234

# Stop task
pvesh delete /cluster/tasks/UPID:node1:0001:1234

# Task log
pvesh get /cluster/tasks/UPID:node1:0001:1234/log
```

### Resource Monitoring
```bash
# Cluster resources
pvesh get /cluster/resources              # All resources
pvesh get /cluster/resources --type vm    # VMs only
pvesh get /cluster/resources --node node1 # Node resources

# Node resources
pvesh get /nodes/localhost/status         # Node status
pvesh get /nodes/localhost/rrd            # RRD data
pvesh get /nodes/localhost/rrd --timeframe hour
```

### Statistics
```bash
# VM statistics
pvesh get /nodes/localhost/vzdump/100 --type

# Storage statistics
pvesm status                              # Storage status
pvesm usage                               # Storage usage

# Network statistics
ip -s link show vmbr0                     # Interface stats
```

### Health Checks
```bash
# Node health
pvesh get /nodes/localhost/status
cat /proc/meminfo                         # Memory info
cat /proc/loadavg                         # Load average
df -h                                     # Disk usage
free -m                                   # Memory usage

# Ceph health (if installed)
ceph health
ceph status
ceph osd status
```

### Performance Tuning
```bash
# Kernel parameters
sysctl -w vm.swappiness=10
sysctl -w vm.vfs_cache_pressure=50

# Disk I/O scheduler
echo deadline > /sys/block/sda/queue/scheduler
```

---

## 10. Updates & Subscription

### Update Proxmox VE
```bash
# Standard update (no subscription)
apt update
apt full-upgrade

# GUI update
# Datacenter -> Updates -> Refresh -> Upgrade

# Update with no GUI
pveupdate                                 # Refresh package lists
pveupgrade                                # Upgrade packages

# Update specific packages
apt install pve-kernel-6.2
apt install pve-header-6.2
```

### Repository Management
```bash
# Edit repositories
cat /etc/apt/sources.list                # System repos
cat /etc/apt/sources.list.d/pve-enterprise.list  # Enterprise
cat /etc/apt/sources.list.d/pve-no-subscription.list  # No-sub

# Switch to no-subscription (for testing)
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list

# Disable enterprise repo (if no subscription)
mv /etc/apt/sources.list.d/pve-enterprise.list /etc/apt/sources.list.d/pve-enterprise.list.disabled

# Update after repo change
apt update
apt full-upgrade
```

### Subscription Management
```bash
# Check subscription status
pvesubscription get

# Add subscription key
pvesubscription set <key>

# Remove subscription
pvesubscription delete
```

### Ceph Updates (if installed)
```bash
# Update Ceph
apt update
apt install ceph

# Ceph repository
cat /etc/apt/sources.list.d/ceph.list
```

### Kernel Management
```bash
# List installed kernels
dpkg -l | grep pve-kernel

# List available kernels
apt list | grep pve-kernel

# Remove old kernels
pvekclean                                # Remove old kernels
apt autoremove                           # Auto remove

# Boot kernel selection
ls /boot                                 # Available kernels
update-grub                              # Update GRUB
```

### Package Management
```bash
# List installed PVE packages
dpkg -l | grep pve

# Check for broken packages
dpkg --configure -a
apt check

# Reinstall package
apt install --reinstall pve-manager

# Remove package
apt remove pve-kernel-5.15
```

---

## 11. High Availability

### HA Configuration
```bash
# Configure HA
pvesh create /cluster/ha/resources --vmid 100 --group ha-group1
pvesh create /cluster/ha/groups --group ha-group1 --nodes node1,node2 --comment "My HA Group"

# List HA resources
pvesh get /cluster/ha/resources
pvesh get /cluster/ha/groups

# Remove HA resource
pvesh delete /cluster/ha/resources/100
```

### HA Groups
```bash
# Create HA group
pvesh create /cluster/ha/groups \
  --group ha-group1 \
  --nodes node1,node2,node3 \
  --comment "Production HA"

# Set group properties
pvesh set /cluster/ha/groups --group ha-group1 --nofailback 1
pvesh set /cluster/ha/groups --group ha-group1 --restricted 1
```

### HA Management
```bash
# Enable HA for VM
pvesh create /cluster/ha/resources --vmid 100 --group ha-group1

# Disable HA
pvesh delete /cluster/ha/resources/100

# Manual migration
pvesh create /cluster/ha/migrate --vmid 100 --node node2

# Failover
systemctl restart pve-ha-lrm              # Restart HA manager
```

### HA Status
```bash
# Check HA status
pvesh get /cluster/ha/status
systemctl status pve-ha-crm               # CRM status
systemctl status pve-ha-lrm               # LRM status

# HA logs
journalctl -u pve-ha-crm                  # CRM logs
journalctl -u pve-ha-lrm                  # LRM logs
```

### Resource Management
```bash
# View managed resources
pvesh get /cluster/resources --type vm

# Start/stop resource
pvesh create /cluster/ha/resources --vmid 100 --group ha-group1
pvesh delete /cluster/ha/resources/100

# Move resource
pvesh create /cluster/ha/migrate --vmid 100 --node node2
```

---

## 12. Useful Utilities

### Command-Line Tools
```bash
pvesh                                    # PVE shell/API
pvesm                                    # Storage management
pveca                                    # Cluster administration
pvecfg                                   # Cluster configuration
pvebackup                                # Backup tool
pveceph                                  # Ceph management
pvekvm                                   # KVM management
vzlist                                   # List containers
vzctl                                    # Container control
```

### API Access
```bash
# Using curl
curl -k -u admin@pam:password https://localhost:8006/api2/json/cluster/resources

# Using pvesh (local)
pvesh get /cluster/resources
pvesh get /nodes/localhost/status
pvesh create /nodes/localhost/status --reboot

# API token usage
curl -k -H "Authorization: PVEAPIToken=admin@pam!token=UUID" https://localhost:8006/api2/json/cluster/resources
```

### Scripting Examples
```bash
#!/bin/bash
# List all VMs and their status
for vmid in $(qm list | awk 'NR>1 {print $1}'); do
  name=$(qm config $vmid | grep name: | cut -d' ' -f2)
  status=$(qm status $vmid)
  echo "VM $vmid ($name): $status"
done

#!/bin/bash
# Start all stopped VMs
for vmid in $(qm list | grep stopped | awk '{print $1}'); do
  echo "Starting VM $vmid..."
  qm start $vmid
done
```

### Troubleshooting Commands
```bash
# Network issues
ip addr show
ip route show
bridge fdb show
brctl show
ping gateway
ping 8.8.8.8
nslookup google.com

# VM issues
qm status 100
qm config 100
qm monitor 100
qm showcmd 100
cat /var/log/pve/tasks.log

# Cluster issues
pvecm status
pvecm nodes
corosync-cfgtool -s
systemctl status corosync
journalctl -u corosync

# Storage issues
pvesm status
pvesm list local
ls -la /var/lib/vz/images
lvdisplay

# Performance issues
top
htop
iotop
iftop
netdata
```

### Emergency Commands
```bash
# Reset root password
# Reboot, press 'e' at GRUB, add init=/bin/bash
# mount -o remount,rw /
# passwd root

# Force quorum
pvecm expected 1

# Stop cluster
systemctl stop pve-cluster

# Start cluster
systemctl start pve-cluster

# Recover from split-brain
pvecm expected 2
pvecm delnode lost-node

# Emergency VM start
# Edit /etc/pve/local.conf/qemu-server/100.conf
# Add: onboot: 0 (to prevent auto-start, then manually start)
```

---

## Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| List VMs | `qm list` |
| List containers | `pct list` |
| Start VM | `qm start <id>` |
| Stop VM | `qm stop <id>` |
| Start container | `pct start <id>` |
| Stop container | `pct stop <id>` |
| Show VM config | `qm config <id>` |
| Enter container | `pct enter <id>` |
| Cluster status | `pvecm status` |
| Node status | `pvesh get /nodes/localhost/status` |

### Common VM Creation
```bash
# Basic VM
qm create 100 --name vm --memory 2048 --cores 2 --net0 virtio,bridge=vmbr0 --scsi0 local-lvm:32

# VM with cloud-init
qm create 101 --name cloud-vm --memory 4096 --cores 2 --scsi0 local-lvm:30 --ide2 local:cloudinit --net0 virtio,bridge=vmbr0

# VM from ISO
qm create 102 --name iso-vm --memory 2048 --cores 2 --cdrom local:iso/ubuntu.iso --scsi0 local-lvm:20
```

### Common Container Creation
```bash
# Basic container
pct create 200 --hostname ct --memory 1024 --cores 2 --net0 name=eth0,bridge=vmbr0 --rootfs local-lvm:10 --ostemplate local:vztmpl/ubuntu.tar.gz

# Container with Docker support
pct create 201 --hostname docker --memory 2048 --cores 4 --net0 name=eth0,bridge=vmbr0 --rootfs local-lvm:30 --ostemplate local:vztmpl/ubuntu.tar.gz --features keyctl=1
```

### Storage Commands
```bash
pvesm list local                         # List storage
pvesm status                             # Show status
pvesm free local                         # Free space
pvesm add lvmthin local --vgname pve --thin-pool data
```

### Backup Commands
```bash
vzdump 100                               # Backup VM
vzdump 200                               # Backup container
vzdump --all                             # Backup all
qmrestore /path/to/backup 101            # Restore VM
pct restore 201 /path/to/backup          # Restore container
```

### Emergency Recovery
```bash
# Reboot to recovery
# Use Proxmox VE Bootable USB
# Select "Rescue System"
# Mount volumes and chroot

# From live USB
mount /dev/pve/root /mnt
mount /dev/sda1 /mnt/boot
mount -t proc proc /mnt/proc
mount -t sysfs sys /mnt/sys
mount -o bind /dev /mnt/dev
chroot /mnt
```

---

## /etc Configuration Files

| File | Purpose |
|------|---------|
| `/etc/pve/user.cfg` | User permissions |
| `/etc/pve/domains.cfg` | Authentication realms |
| `/etc/pve/storage.cfg` | Storage configuration |
| `/etc/pve/cluster.conf` | Cluster configuration |
| `/etc/pve/nodes/*/qemu-server/*.conf` | VM configurations |
| `/etc/pve/nodes/*/lxc/*.conf` | Container configurations |
| `/etc/network/interfaces` | Network configuration |
| `/etc/hosts` | Hosts file |
| `/etc/apt/sources.list` | APT repositories |
| `/etc/vzdump.conf` | Backup configuration |
| `/etc/pve/firewall/*.fw` | Firewall rules |
| `/etc/corosync/corosync.conf` | Corosync config |
| `/etc/ceph/ceph.conf` | Ceph configuration |

---

## Troubleshooting Quick Fixes

### VM Won't Start
```bash
qm status 100
qm config 100
qm monitor 100
# Check for:
# - Disk space
# - Memory availability
# - Invalid config
# - Storage not mounted
journalctl -u pvedaemon
```

### Container Won't Start
```bash
pct status 200
pct config 200
pct enter 200
journalctl -u pve-daemon
# Check for:
# - Mount points
# - Disk space
# - Network config
```

### Network Issues
```bash
systemctl restart networking
ip link show
bridge fdb show
brctl show
ping 192.168.1.1 (gateway)
systemctl status pve-firewall
```

### Cluster Issues
```bash
pvecm status
pvecm nodes
systemctl status corosync
journalctl -u corosync
# Check for:
# - Network connectivity
# - Time sync (NTP)
# - Firewall rules
```

### Storage Issues
```bash
pvesm status
pvesm list local
lvdisplay
zpool status
# Check for:
# - NFS server availability
# - LVM volume groups
# - Disk space
```

### Slow Performance
```bash
top
htop
iotop -d 1
iftop -i eth0
# Check for:
# - High CPU usage
# - Memory pressure
# - Disk I/O bottlenecks
# - Network congestion
```

### Web UI Not Loading
```bash
systemctl status pveproxy
systemctl restart pveproxy
journalctl -u pveproxy
netstat -tulpn | grep 8006
# Check firewall: iptables -L
```

---

*Last Updated: January 2026*
*Generated for Proxmox VE 8.x*
