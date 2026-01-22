# Arch Linux Complete Cheat Sheet

## Table of Contents
1. [System Basics](#1-system-basics)
2. [Pacman Package Manager](#2-pacman-package-manager)
3. [AUR (Arch User Repository)](#3-aur-arch-user-repository)
4. [Systemd Services](#4-systemd-services)
5. [Networking](#5-networking)
6. [User Management](#6-user-management)
7. [File Operations](#7-file-operations)
8. [Process Management](#8-process-management)
9. [Kernel & Boot](#9-kernel--boot)
10. [Storage & Disks](#10-storage--disks)
11. [Hardware Information](#11-hardware-information)
12. [System Maintenance](#12-system-maintenance)
13. [Display & Audio](#13-display--audio)
14. [Security](#14-security)
15. [Useful Utilities](#15-useful-utilities)

---

## 1. System Basics

| Command | Description |
|---------|-------------|
| `reboot` | Reboot system |
| `shutdown now` | Shutdown immediately |
| `shutdown +10` | Shutdown in 10 minutes |
| `halt` | Halt system |
| `exit` | Exit current shell |
| `clear` | Clear terminal screen |
| `history` | Show command history |
| `uptime` | Show system uptime |
| `date` | Show current date/time |
| `timedatectl` | Manage system time |
| `hostnamectl` | Manage hostname |
| `localectl` | Manage locale settings |

### timedatectl Commands
```bash
timedatectl set-time "2025-01-16 12:00:00"  # Set date/time
timedatectl set-timezone America/New_York   # Set timezone
timedatectl set-ntp true                    # Enable NTP
timedatectl status                           # Show status
```

### hostnamectl Commands
```bash
hostnamectl set-hostname myarch              # Set hostname
hostnamectl status                           # Show status
```

---

## 2. Pacman Package Manager

### Package Installation & Removal
| Command | Description |
|---------|-------------|
| `pacman -Syu` | Sync and upgrade all packages |
| `pacman -Sy` | Refresh package databases only |
| `pacman -Su` | Upgrade packages without refresh |
| `pacman -S <package>` | Install package |
| `pacman -R <package>` | Remove package |
| `pacman -Rs <package>` | Remove with unused dependencies |
| `pacman -Rns <package>` | Remove with config files |
| `pacman -Rsc <package>` | Remove with dependencies |
| `pacman -Rdd <package>` | Remove ignoring dependencies |

### Package Search & Query
| Command | Description |
|---------|-------------|
| `pacman -Ss <query>` | Search remote packages |
| `pacman -Qs <query>` | Search installed packages |
| `pacman -Fi <package>` | Search remote package files |
| `pacman -Ql <package>` | List installed package files |
| `pacman -Qi <package>` | Show package info |
| `pacman -Si <package>` | Show remote package info |
| `pacman -Qo <file>` | Which package owns file |
| `pacman -F <file>` | Search remote for file |
| `pacman -Qdt` | List orphaned packages |
| `pacman -Qet` | List explicitly installed |

### Useful Pacman Flags
| Flag | Description |
|------|-------------|
| `-y` | Refresh sync databases |
| `-u` | Upgrade packages |
| `-r` | Root directory |
| `-n` | No modify (simulate) |
| `-p` | Print only |
| `-v` | Verbose output |
| `-i` | Show info |
| `-c` | Clean cache |
| `-w` | Download only |
| `--asdeps` | Install as dependency |
| `--asexplicit` | Install as explicit |
| `--needed` | Don't reinstall if up-to-date |
| `--ignore <package>` | Ignore package on upgrade |

### Cache Management
```bash
pacman -Sc                    # Clean unused package cache
pacman -Scc                   # Clean ALL package cache
paccache -r                   # Clean old versions (keep 3)
paccache -ruk0                # Remove all uninstalled
paccache -rk1                 # Keep only 1 version
```

### Database Operations
```bash
pacman -D --asdeps <package>  # Mark as dependency
pacman -D --asexplicit <package>  # Mark as explicit
```

### Parallel Downloads
Edit `/etc/pacman.conf`:
```ini
[options]
ParallelDownloads = 5
```

---

## 3. AUR (Arch User Repository)

### yay (Yet Another Yaourt)
```bash
yay -S <package>              # Install AUR package
yay -R <package>              # Remove package
yay -Rns <package>            # Remove with deps and config
yay -Ss <query>               # Search AUR
yay -Ps                      # Print system statistics
yay -Yc                      # Clean unneeded dependencies
yay -Y --combinedupgrade      # Combined upgrade
yay -Syu --aur               # Upgrade AUR packages only
```

### paru (AUR Helper)
```bash
paru -S <package>             # Install AUR package
paru -R <package>             # Remove package
paru -Ss <query>              # Search AUR
paru -Syu                     # Upgrade all (repo + AUR)
paru -Sua                     # Upgrade AUR only
paru -c                       # Clean AUR cache
paru -Qa                      # List AUR packages
paru -S --ask<number>         # Confirm level
paru --bottomup               # Bottom-up search
```

### Manual AUR Build
```bash
git clone https://aur.archlinux.org/<package>.git
cd <package>
makepkg -si                   # Build and install
makepkg -si --noconfirm       # Skip confirmations
makepkg -si --nobuild         # Download only
makepkg -si --skippgpcheck    # Skip PGP check
makepkg -c                    # Clean after build
```

### AUR Helper Comparison
| Feature | yay | paru |
|---------|-----|------|
| AUR support | Yes | Yes |
| Repo support | Yes | Yes |
| AUR upgrade | Yes | Yes |
| Search ordering | Vote count | Alphabetical |
| Configuration | ~/.config/yay | ~/.config/paru |

---

## 4. Systemd Services

### Service Management
```bash
systemctl start <service>             # Start service
systemctl stop <service>              # Stop service
systemctl restart <service>           # Restart service
systemctl reload <service>            # Reload config
systemctl status <service>            # Check status
systemctl is-active <service>         # Is running?
systemctl is-enabled <service>        # Is enabled?
systemctl enable <service>            # Enable at boot
systemctl disable <service>           # Disable at boot
systemctl enable --now <service>      # Enable and start
systemctl disable --now <service>     # Disable and stop
systemctl mask <service>              # Mask (prevent start)
systemctl unmask <service>            # Unmask
```

### Systemctl Query Commands
```bash
systemctl list-units                  # List active units
systemctl list-units --all            # List all units
systemctl list-unit-files             # List unit files
systemctl list-timers                 # List timers
systemctl list-dependencies <unit>    # Show dependencies
systemctl show <service>              # Show unit properties
systemctl cat <service>               # Show unit file
systemctl edit <service>              # Edit unit file
```

### Power Management
```bash
systemctl poweroff                    # Power off
systemctl reboot                      # Reboot
systemctl suspend                     # Suspend to RAM
systemctl hibernate                   # Hibernate to disk
systemctl hybrid-sleep                # Hybrid suspend
systemctl reboot --boot-loader-menu   # Boot menu
```

### Systemd Essentials
| Service | Description |
|---------|-------------|
| `systemd-networkd` | Network manager |
| `systemd-resolved` | DNS resolver |
| `systemd-timesyncd` | NTP client |
| `firewalld` | Firewall |
| `NetworkManager` | Network manager (alternative) |
| `iwd` | Wireless daemon |
| `cups` | Printer service |
| `bluetooth` | Bluetooth service |

---

## 5. Networking

### Network Configuration (ip)
```bash
ip addr show                    # Show IP addresses
ip addr add 192.168.1.10/24 dev eth0  # Add IP
ip link set eth0 up             # Bring interface up
ip link set eth0 down           # Bring interface down
ip route show                   # Show routing table
ip route add default via 192.168.1.1  # Add route
ip neigh show                   # Show ARP table
ip link set eth0 mtu 1500       # Set MTU
```

### NetworkManager (nmcli)
```bash
nmcli device status                   # Show devices
nmcli device connect eth0             # Connect interface
nmcli device disconnect eth0          # Disconnect
nmcli connection show                 # Show connections
nmcli connection up "Wired connection 1"
nmcli connection down "Wired connection 1"
nmcli connection add type ethernet con-name mycon ifname eth0
nmcli connection modify mycon ipv4.addresses 192.168.1.10/24
nmcli connection modify mycon ipv4.method manual
nmcli wifi list                       # List WiFi networks
nmcli wifi connect "SSID" password "PASS"
```

### iwd (iWireless Daemon)
```bash
iwctl station wlan0 show              # Show station
iwctl station wlan0 scan              # Scan networks
iwctl station wlan0 get-networks      # List networks
iwctl station wlan0 connect "SSID"    # Connect
iwctl device list                     # Show devices
```

### WiFi Tools
```bash
wpa_passphrase "SSID" "password" > config.conf
wpa_supplicant -B -i wlan0 -c config.conf
dhcpcd wlan0                          # Get IP
```

### Firewall (firewalld)
```bash
firewall-cmd --state                  # Check status
firewall-cmd --get-active-zones       # Active zones
firewall-cmd --list-all               # List all rules
firewall-cmd --list-services          # List services
firewall-cmd --list-ports             # List ports
firewall-cmd --permanent --add-service=ssh
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --permanent --remove-service=ssh
firewall-cmd --reload                 # Reload rules
firewall-cmd --runtime-to-permanent   # Save runtime
```

### netctl
```bash
netctl list                           # List profiles
netctl start profile                  # Start profile
netctl stop profile                   # Stop profile
netctl enable profile                 # Enable at boot
netctl disable profile                # Disable at boot
```

### DNS
```bash
resolvectl status                     # Show DNS status
resolvectl dns eth0 8.8.8.8           # Set DNS
cat /etc/resolv.conf                  # DNS servers
dig domain.com                        # DNS lookup
nslookup domain.com                   # DNS lookup
host domain.com                       # DNS lookup
```

### Curl & Wget
```bash
curl -O http://example.com/file       # Download
curl -L http://example.com            # Follow redirects
curl -I http://example.com            # Show headers
curl -d "user=test" http://example.com
wget -c http://example.com/file       # Continue download
wget -r -np http://example.com        # Recursive
```

---

## 6. User Management

### User Accounts
```bash
useradd -m username                   # Create user with home
useradd -m -s /bin/bash username      # With shell
useradd -m -G wheel username          # Add to group
useradd -r username                   # System user
userdel username                      # Delete user
userdel -r username                   # Delete with home
usermod -aG wheel username            # Add to group
usermod -l newname oldname            # Rename user
usermod -d /new/home username         # Change home
usermod -s /bin/zsh username          # Change shell
```

### Password Management
```bash
passwd username                       # Change password
passwd -l username                    # Lock account
passwd -u username                    # Unlock account
passwd -d username                    # Delete password
passwd -S username                    # Show status
```

### Groups
```bash
groupadd groupname                    # Create group
groupdel groupname                    # Delete group
gpasswd -a user groupname             # Add user to group
gpasswd -d user groupname             # Remove from group
gpasswd -A user groupname             # Set admin
groups username                       # Show user groups
id username                           # Show UID/GID
getent group groupname                # Show group members
```

### Sudo Configuration
```bash
pacman -S sudo                         # Install sudo
usermod -aG wheel username            # Add to wheel group
EDITOR=nano visudo                    # Edit sudoers
```

Sudoers file examples:
```
# Allow wheel group sudo
%wheel ALL=(ALL) ALL

# No password for wheel
%wheel ALL=(ALL) NOPASSWD: ALL

# Specific user no password
username ALL=(ALL) NOPASSWD: ALL

# Specific commands only
username ALL=(ALL) NOPASSWD: /bin/systemctl
```

---

## 7. File Operations

### File Permissions
| Command | Description |
|---------|-------------|
| `chmod 755 file` | Set permissions (rwxr-xr-x) |
| `chmod +x file` | Add execute permission |
| `chmod -x file` | Remove execute permission |
| `chmod 600 file` | Private (rw-------) |
| `chmod 644 file` | Public read (rw-r--r--) |
| `chmod 755 dir` | Directory permissions |
| `chmod -R 755 dir` | Recursive change |

### Ownership
```bash
chown user:group file         # Change owner:group
chown user file               # Change owner only
chown :group file             # Change group only
chown -R user:group dir       # Recursive
```

### File Operations
| Command | Description |
|---------|-------------|
| `ls -la` | List all files with details |
| `ls -lh` | Human readable sizes |
| `ls -R` | Recursive listing |
| `cp -r src dst` | Copy recursively |
| `mv old new` | Move/rename |
| `rm -rf dir` | Force remove directory |
| `mkdir -p a/b/c` | Create nested directories |
| `rmdir dir` | Remove empty directory |
| `touch file` | Create/更新文件 |
| `cat file` | 显示文件内容 |
| `less file` | 分页查看文件 |
| `head -n 20 file` | 显示前20行 |
| `tail -n 20 file` | 显示后20行 |
| `tail -f file` | 实时查看文件 |

### File Search
```bash
find /path -name "file"               # Find by name
find /path -type f                    # Find files
find /path -type d                    # Find directories
find /path -size +100M                # Find >100MB
find /path -mtime -7                  # Modified <7 days
find /path -perm 755                  # By permissions
find /path -user username             # By owner
find /path -exec rm {} \;             # Execute on results
locate file                           # Fast search (updatedb)
whereis command                       # Find binary/source/manual
which command                         # Find binary in PATH
```

### Compression
```bash
tar -cvf archive.tar dir/             # Create tar
tar -xvf archive.tar                  # Extract tar
tar -czvf archive.tar.gz dir/         # Create gzip
tar -xzvf archive.tar.gz              # Extract gzip
tar -cjvf archive.tar.bz2 dir/        # Create bzip2
tar -xjvf archive.tar.bz2             # Extract bzip2
tar -cJvf archive.tar.xz dir/         # Create xz
tar -xJvf archive.tar.xz              # Extract xz
zip -r archive.zip dir/               # Create zip
unzip archive.zip                     # Extract zip
7z a archive.7z dir/                  # Create 7z
7z x archive.7z                       # Extract 7z
```

### File Info
```bash
file filename                         # Show file type
stat filename                         # Show detailed info
wc -l file                            # Count lines
wc -w file                            # Count words
wc -c file                            # Count bytes
md5sum file                           # MD5 checksum
sha256sum file                        # SHA256 checksum
cksum file                            # Checksum
diff file1 file2                      # Compare files
cmp file1 file2                       # Byte comparison
```

---

## 8. Process Management

### Process Viewing
```bash
ps aux                                # All processes
ps aux | grep name                    # Filter processes
ps -ef                                # Full listing
pstree                                # Process tree
top                                   # Interactive view
htop                                  # Enhanced top (install)
btop                                  # Modern system monitor (install)
```

### Process Management
```bash
kill <PID>                            # Terminate process
kill -9 <PID>                         # Force kill
kill -1 <PID>                         # HUP signal
killall process_name                  # Kill all by name
pkill process_name                    # Kill by pattern
pgrep process_name                    # Find PID
nice -n 10 command                    # Start with priority
renice 10 <PID>                       # Change priority
```

### Background Jobs
```bash
command &                             # Run in background
Ctrl+Z                                # Suspend job
bg %1                                 # Resume background
fg %1                                 # Bring to foreground
jobs                                  # List jobs
disown %1                             # Detach from shell
nohup command &                       # Immune to hangup
```

### Priority Range
| Priority | Description |
|----------|-------------|
| -20 | Highest priority |
| 0 | Normal priority |
| 19 | Lowest priority |

---

## 9. Kernel & Boot

### Boot Management (bootctl)
```bash
bootctl status                        # Show boot status
bootctl install                       # Install systemd-boot
bootctl update                        # Update systemd-boot
bootctl list                          # List entries
efibootmgr                            # Show EFI entries
```

### GRUB
```bash
grub-mkconfig -o /boot/grub/grub.cfg  # Generate config
grub-install /dev/sda                  # Install GRUB
update-grub                            # Update GRUB (alias)
```

### Initramfs (mkinitcpio)
```bash
mkinitcpio -p linux                   # Generate initramfs
mkinitcpio -p linux-lts               # LTS kernel
mkinitcpio -c /etc/mkinitcpio.conf -p linux
mkinitcpio -M                         # List presets
```

### Kernel Modules
```bash
lsmod                                 # Show loaded modules
modprobe -a module                    # Load module
modprobe -r module                    # Unload module
modprobe module                       # Load with deps
modinfo module                        # Show module info
echo module > /sys/module/module/parameters/enable
```

### Kernel Parameters
```bash
cat /proc/cmdline                     # Current parameters
grub-mkconfig -o /boot/grub/grub.cfg  # Add to GRUB
# Add to /etc/default/grub:
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

### Kernel Information
```bash
uname -a                              # Kernel info
uname -r                              # Kernel release
hostnamectl                           # Kernel info
cat /proc/version                     # Kernel version
cat /proc/sys/kernel/hostname         # Hostname
ls /lib/modules/                      # Available kernels

```

### SystemD Boot
```bash
# /boot/loader/loader.conf
default arch.conf
timeout 5
console-mode max

# /boot/loader/entries/arch.conf
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options root=PARTUUID=xxx ro quiet
```

---

## 10. Storage & Disks

### Disk Information
```bash
lsblk                                 # List block devices
lsblk -f                              # Show filesystems
lsblk -m                              # Show sizes
fdisk -l                              # Show all partitions
parted -l                             # List partitions
df -h                                 # Disk usage (human)
df -i                                 # Inode usage
du -sh /path                          # Directory size
du -h --max-depth=1                   # Top-level sizes
pydf                                  # Better df (install)

```

### Partition Management
```bash
fdisk /dev/sda                        # Interactive partition
cfdisk /dev/sda                       # Curses-based
parted /dev/sda                       # GNU Parted
sfdisk -l /dev/sda                    # List partitions
sfdisk -d /dev/sda > partition.dump   # Backup partition table
sfdisk /dev/sda < partition.dump      # Restore partition table
```

### File System Creation
```bash
mkfs.ext4 /dev/sda1                   # Create ext4
mkfs.ext4 -L "Label" /dev/sda1        # With label
mkfs.btrfs /dev/sda1                  # Create btrfs
mkfs.xfs /dev/sda1                    # Create xfs
mkfs.fat -F 32 /dev/sda1              # Create FAT32
mkfs.vfat -F 32 /dev/sda1             # Create vfat
mkswap /dev/sda2                      # Create swap
mkfs.zfs /dev/sda1                    # Create ZFS
```

### Mount Operations
```bash
mount /dev/sda1 /mnt                  # Mount device
mount -o remount,rw /                 # Remount read-write
mount -a                              # Mount all from fstab
umount /mnt                           # Unmount
umount -l /mnt                        # Lazy unmount
umount -f /mnt                        # Force unmount
findmnt                               # Show mounts
lsblk --mountpoint                    # Mounted status
```

### Swap
```bash
swapon --show                         # Show swap
swapon /dev/sda2                      # Enable swap
swapoff /dev/sda2                     # Disable swap
swapon -p 1 /dev/sda2                 # Set priority
```

### LVM
```bash
pvcreate /dev/sda1                    # Create physical volume
vgcreate vg0 /dev/sda1 /dev/sda2      # Create volume group
lvcreate -L 20G vg0 -n lv_root        # Create logical volume
lvextend -L +10G /dev/vg0/lv_root     # Extend LV
lvreduce -L -5G /dev/vg0/lv_root      # Reduce LV
lvresize -l +100%FREE /dev/vg0/lv_root
pvs                                   # Show PVs
vgs                                   # Show VGs
lvs                                   # Show LVs
```

### Btrfs
```bash
btrfs filesystem show                 # Show btrfs devices
btrfs balance start /mnt              # Balance
btrfs scrub start /mnt                # Scrub
btrfs device add /dev/sdb /mnt        # Add device
btrfs device remove /dev/sda /mnt     # Remove device
btrfs subvolume create /mnt/@home     # Create subvolume
btrfs subvolume list /mnt             # List subvolumes
```

### RAID
```bash
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1
mdadm --detail /dev/md0               # Show RAID details
mdadm --manage /dev/md0 --add /dev/sdc1
mdadm --stop /dev/md0                 # Stop RAID
cat /proc/mdstat                      # RAID status
```

### fstab Configuration
```bash
# /etc/fstab
# UUID=xxx / ext4 defaults 0 1
# LABEL=Data /data ext4 defaults 0 2
# /dev/sda1 /mnt ext4 nofail 0 0

genfstab -U /mnt >> /etc/fstab        # Generate fstab
```

---

## 11. Hardware Information

### CPU
```bash
lscpu                                 # CPU info
cat /proc/cpuinfo                     # Detailed CPU info
nproc                                 # Number of CPUs
lscpu --cpi                           # CPU cache info
cpuid                                 # CPU info (install)
```

### Memory
```bash
free -h                               # Memory usage
free -m                               # In MB
cat /proc/meminfo                     # Detailed memory info
lsmem                                 # Memory blocks
dmidecode -t memory                   # DIMM info
```

### Storage Devices
```bash
lsblk                                 # Block devices
lsblk -f                              # With filesystems
hdparm -I /dev/sda                    # Disk info
smartctl -a /dev/sda                  # SMART info
smartctl -t short /dev/sda            # Short self-test
nvme list                             # NVMe devices
lsscsi                                # SCSI devices
```

### USB Devices
```bash
lsusb                                 # List USB devices
lsusb -v                              # Verbose
usb-devices                           # Device details
```

### PCI Devices
```bash
lspci                                 # List PCI devices
lspci -v                              # Verbose
lspci -nn                             # With IDs
lspci -k                              # With kernel modules
lspci -t                              # Tree view
```

### Audio
```bash
aplay -l                              # List audio devices
amixer -c 0                           # Mixer info
pactl list sinks                      # PulseAudio sinks
pw-cli list objects Node              # PipeWire nodes
```

### Graphics
```bash
lspci | grep -i vga                   # GPU info
xrandr                                # Display config
xrandr --query                        # Current state
nvidia-smi                            # NVIDIA GPU (if installed)
vkdiag                                # Vulkan info (install)
```

### System Information
```bash
inxi -Fxz                            # Full system info
neofetch                             # Show system info
hwinfo --short                       # Hardware summary
dmidecode                            # DMI/SMBIOS info
cat /etc/os-release                  # OS info
hostnamectl                          # System info
```

---

## 12. System Maintenance

### System Logs
```bash
journalctl                            # All logs
journalctl -u service                 # Unit logs
journalctl -p err                     # Error priority
journalctl -b                         # Current boot
journalctl -b -1                      # Previous boot
journalctl -f                         # Follow logs
journalctl --since "1 hour ago"
journalctl --since "2025-01-16"
journalctl --disk-usage               # Log size
journalctl --vacuum-size=500M         # Reduce size
journalctl --vacuum-time=7d           # Keep 7 days
journalctl --rotate
```

### Log Files
```bash
tail -f /var/log/pacman.log          # Pacman log
tail -f /var/log/syslog              # System log
tail -f /var/log/Xorg.0.log          # X11 log
cat /var/log/auth.log                # Authentication log
cat /var/log/boot.log                # Boot log
last                                 # Login history
lastb                                # Failed logins
```

### Performance Monitoring
```bash
top                                   # Task manager
htop                                  # Enhanced top
btop                                  # Modern monitor
glances                               # System overview (install)
bashtop                               # Resource monitor (install)
iotop                                 # I/O usage
iftop                                 # Network usage
nethogs                               # Per-process network
netdata                               # Real-time monitoring (install)
```

### System Maintenance Commands
```bash
pacman -Syu                           # Update system
pacman -Rns $(pacman -Qdtq)          # Remove orphans
paccache -r                           # Clean cache
pacman -Scc                           # Clean all cache
systemd-tmpfiles --clean             # Clean temp files
journalctl --vacuum-time=2weeks      # Clean old logs
fc-cache -fv                          # Font cache
mandb                                 # Man database
updatedb                              # Locate database
```

### Backup & Restore
```bash
# rsync backup
rsync -avh /source/ /backup/         # Archive backup
rsync -avh --delete /source/ /backup/
rsync -avh --exclude='*.tmp' /source/ /backup/

# tar backup
tar -czvf backup.tar.gz /path/
tar -xzvf backup.tar.gz

# System backup
dd if=/dev/sda of=/dev/sdb bs=4M status=progress
dd if=/dev/sda of=backup.img bs=4M count=10M
```

### Timeshift (Snapshot Tool)
```bash
sudo timeshift --create --comments "Before update"
sudo timeshift --restore
sudo timeshift --list
sudo timeshift --delete --snapshot-device /dev/sda3
```

---

## 13. Display & Audio

### X11 Configuration
```bash
X :0 &                               # Start X server
startx                               # Start X session
cat ~/.xinitrc                       # X startup file
nvidia-xconfig                       # NVIDIA config
xrandr --output HDMI-1 --auto        # Auto config output
xrandr --output HDMI-1 --mode 1920x1080
xrandr --output HDMI-1 --left-of eDP-1
xrandr --output HDMI-1 --off
```

### Display Managers
```bash
systemctl status lightdm             # Check status
systemctl start lightdm              # Start
systemctl enable lightdm             # Enable at boot
systemctl set-default graphical.target
```

### Audio (PipeWire)
```bash
pw-cli info 0                        # PipeWire info
pw-link -l                           # List links
pw-metadata                          # Show metadata
wireplumber --exit                   # Restart WirePlumber
systemctl --user restart pipewire
systemctl --user restart wireplumber
```

### Audio (PulseAudio)
```bash
pulseaudio -k                        # Kill PulseAudio
pulseaudio --start                   # Start PulseAudio
pactl info                           # Show info
pactl list sinks short               # List sinks
pactl set-sink-volume 0 +5%          # Increase volume
pactl set-sink-mute 0 toggle         # Toggle mute
```

### Brightness
```bash
ls /sys/class/backlight/             # Backlight devices
brightnessctl set 50%                # Set brightness
brightnessctl get                    # Get brightness
xbacklight -set 50                   # X11 backlight
```

### Input Devices
```bash
xinput list                          # List input devices
xinput set-prop "device" "libinput Accel Speed" 0.5
libinput list-devices                # libinput devices
```

---

## 14. Security

### Firewall (iptables)
```bash
iptables -L                          # List rules
iptables -L -n -v                    # Verbose
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -j DROP
iptables -F                          # Flush rules
iptables -X                          # Delete chains
iptables-save > /etc/iptables/rules.v4
iptables-restore < /etc/iptables/rules.v4
```

### Firewall (nftables)
```bash
nft list ruleset                     # List rules
nft add rule ip filter input tcp dport 22 accept
nft flush ruleset                    # Flush all
nft -f /etc/nftables.conf            # Load config
```

### SELinux
```bash
getenforce                           # Check mode
setenforce 0                         # Set permissive
setenforce 1                         # Set enforcing
sestatus                             # SELinux status
```

### AppArmor
```bash
aa-status                            # Show status
aa-enforce /etc/apparmor.d/usr.sbin.nginx
aa-complain /etc/apparmor.d/usr.sbin.nginx
```

### System Hardening
```bash
# Disable core dumps
echo "* hard core 0" >> /etc/security/limits.conf

# Kernel hardening
sysctl -w kernel.randomize_va_space=2
sysctl -w net.ipv4.conf.all.rp_filter=1
sysctl -p /etc/sysctl.d/99-hardening.conf

# /etc/sysctl.d/99-hardening.conf
kernel.randomize_va_space=2
net.ipv4.conf.all.rp_filter=1
net.ipv4.conf.default.rp_filter=1
net.ipv4.icmp_echo_ignore_broadcasts=1
fs.suid_dumpable=0
```

### File Permissions Audit
```bash
# Find world-writable files
find / -perm -2 ! -type l -ls

# Find setuid files
find / -perm -4000 -ls

# Find files without user
find / -nouser -ls

# Find files without group
find / -nogroup -ls
```

### SSH Hardening
```bash
# /etc/ssh/sshd_config
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
AllowUsers user1 user2
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
```

---

## 15. Useful Utilities

### Package Management
```bash
pacman -Syu                          # Full upgrade
pacman -S <pkg>                      # Install
pacman -R <pkg>                      # Remove
pacman -Qs <query>                   # Search installed
pacman -Ss <query>                   # Search remote
pacman -Qdt                          # Orphans
paccache -r                          # Clean cache
pacman -Scc                          # Clean all
yay -Syu                             # Full + AUR
paru -Syu                            # Full + AUR
```

### AUR Helpers
```bash
yay -S <aur-package>                 # Install AUR
yay -R <package>                     # Remove
yay -Ss <query>                      # Search AUR
yay -Yc                              # Clean deps
paru -S <aur-package>                # Install AUR
paru -Sua                            # Upgrade AUR only
paru -c                              # Clean AUR cache
```

### System Information
```bash
neofetch                             # System info
inxi -Fxz                            # Full info
hwinfo --short                       # Hardware
hostnamectl                          # Host info
uname -a                             # Kernel info
uptime                               # Uptime
```

### Terminal Enhancements
```bash
zsh                                  # Z shell
oh-my-zsh                            # Zsh framework
tmux                                 # Terminal multiplexer
ranger                               # File manager
fzf                                  # Fuzzy finder
exa                                  # Better ls
bat                                  # Better cat
fd                                   # Better find
ripgrep                              # Better grep
starship                             # Prompt (install)
```

### File Managers
```bash
ranger                               # Console file manager
nnn                                  # Lightweight file manager
lf                                   # Terminal file manager
```

### System Monitors
```bash
htop                                 # Process manager
btop                                 # Resource monitor
bashtop                              # Resource monitor
nvtop                                # GPU monitor
glances                              # System overview
netdata                              # Real-time monitoring
```

### Text Editors
```bash
vim                                  # Vi IMproved
nvim                                 # Neovim
nano                                 # Simple editor
emacs                                # Extensible editor
micro                                # Simple terminal editor
code                                 # VSCode
```

### Documentation
```bash
man command                          # Manual page
man -k keyword                       # Search man pages
info command                         # GNU info
tldr command                         # Simplified man
cheat command                        # Community cheatsheets
```

---

## Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| Update system | `sudo pacman -Syu` |
| Install package | `sudo pacman -S <pkg>` |
| Search package | `pacman -Ss <query>` |
| Remove package | `sudo pacman -R <pkg>` |
| Clean cache | `sudo paccache -r` |
| Check services | `systemctl status` |
| View logs | `journalctl -xe` |
| Disk usage | `df -h` |
| Process list | `htop` |
| Open ports | `ss -tuln` |

### Emergency Commands
| Task | Command |
|------|---------|
| Boot to recovery | Hold Shift at boot |
| Root shell | Add `init=/bin/bash` to kernel cmdline |
| Remount root read-write | `mount -o remount,rw /` |
| Chroot from live USB | `arch-chroot /mnt` |
| Reset root password | `passwd` in chroot |
| Fix broken pacman | `pacman -Syyu` |
| Force shutdown | Hold power button |

---

## /etc Configuration Files

| File | Purpose |
|------|---------|
| `/etc/pacman.conf` | Pacman configuration |
| `/etc/pacman.d/mirrorlist` | Package mirrors |
| `/etc/systemd/system.conf` | SystemD config |
| `/etc/systemd/user.conf` | User SystemD config |
| `/etc/hostname` | System hostname |
| `/etc/locale.conf` | Locale settings |
| `/etc/locale.gen` | Available locales |
| `/etc/vconsole.conf` | Console settings |
| `/etc/fstab` | Mount points |
| `/etc/mkinitcpio.conf` | Initramfs config |
| `/etc/modprobe.d/` | Kernel modules |
| `/etc/sysctl.d/` | Kernel parameters |
| `/etc/security/limits.conf` | Resource limits |
| `/etc/ssh/sshd_config` | SSH server config |
| `/etc/iptables/rules.v4` | iptables rules |
| `/etc/nftables.conf` | nftables rules |

---

## Troubleshooting Quick Fixes

### System Won't Boot
```bash
# Boot to live USB, then:
mount /dev/sdaX /mnt
arch-chroot /mnt
mkinitcpio -P
grub-mkconfig -o /boot/grub/grub.cfg
exit
reboot
```

### Network Not Working
```bash
ip link set eth0 up
dhcpcd eth0
systemctl restart NetworkManager
cat /etc/resolv.conf
```

### Sound Not Working
```bash
pulseaudio -k && pulseaudio --start
alsamixer
speaker-test -c 2
systemctl --user restart pipewire
```

### Graphics Issues
```bash
Xorg :1 &                             # Try new X server
cat /var/log/Xorg.0.log | grep EE     # Check errors
xrandr                                # Check outputs
```

### Package Issues
```bash
pacman -Syyu                         # Refresh all
pacman -S --overwrite "*" <pkg>      # Force reinstall
pacman -Qk                           # Check package files
rm -rf /var/lib/pacman/db.lck        # Remove lock
```

---

*Last Updated: January 2026*
*Generated for Arch Linux Rolling Release*
