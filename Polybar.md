# Polybar Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Configuration](#3-configuration)
4. [Modules](#4-modules)
5. [Themes and Styling](#5-themes-and-styling)
6. [Launch Scripts](#6-launch-scripts)
7. [Troubleshooting](#7-troubleshooting)
8. [Quick Reference](#8-quick-reference)

---

## 1. Introduction

Polybar is a fast and easy-to-use status bar for Linux desktop environments, highly customizable with modules for system monitoring.

### Key Features
| Feature | Description |
|---------|-------------|
| **Modular** | Plug-and-play modules |
| **Customizable** | Full control over appearance |
| **Multi-monitor** | Independent bars per monitor |
| **Workspaces** | i3/sway integration |
| **Theming** | Built-in and custom themes |
| **IPC** | Control via scripts |

### Architecture
```
┌─────────────────────────────────────────────────────┐
│                    Polybar                            │
│  ┌─────────────────────────────────────────────────┐│
│  │                 Bar Module                        ││
│  │  [left]      [center]      [right]              ││
│  │  workspaces  window-title    date systray        ││
│  └─────────────────────────────────────────────────┘│
│                                                     │
│  ┌─────────────────────────────────────────────────┐│
│  │              Configuration                       ││
│  │  /etc/polybar/config.ini                        ││
│  │  ~/.config/polybar/config.ini                   ││
│  └─────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────┘
```

---

## 2. Installation

### Installation
```bash
# Ubuntu/Debian
sudo apt install polybar

# Arch Linux
sudo pacman -S polybar

# Fedora
sudo dnf install polybar

# From source
git clone https://github.com/polybar/polybar
cd polybar
mkdir build && cd build
cmake ..
make -j$(nproc)
sudo make install
```

### Running Polybar
```bash
# Default bar
polybar example

# List available bars
polybar -m | cut -d: -f1

# Check configuration
polybar -c /path/to/config.ini -r

# Validate configuration
polybar -c /path/to/config.ini -t
```

### Launch Scripts
```bash
# Create launch script
cat > ~/.config/polybar/launch.sh << 'EOF'
#!/bin/bash
killall polybar

if type "xrandr"; then
  for m in $(xrandr --query | grep " connected" | cut -d' ' -f1); do
    MONITOR=$m polybar --reload example &
  done
else
  polybar --reload example &
fi
EOF

chmod +x ~/.config/polybar/launch.sh
```

---

## 3. Configuration

### Main Configuration
```ini
; ~/.config/polybar/config.ini

[settings]
screenchange-reload = true
pseudo-transparency = true
override-redirect = false

[colors]
background = #282A2E
background-alt = #373B41
foreground = #C5C8C6
primary = #F0C674
secondary = #8ABEB7
alert = #A54242
disabled = #707880

[bar/example]
width = 100%
height = 28pt
radius = 6
background = ${colors.background}
foreground = ${colors.foreground}
line-size = 3pt
border-size = 4pt
border-color = #00000000
padding-left = 1
padding-right = 1
module-margin = 1
separator = |
separator-foreground = ${colors.disabled}
font-0 = JetBrains Mono;10
font-1 = JetBrains Mono;10
modules-left = xworkspaces xwindow
modules-center = date
modules-right = pulseaudio memory cpu battery wlan eth
tray = right
tray-padding = 8
tray-background = ${colors.background}
enable-ipc = true
wm-restack = i3
override-redirect = false
bottom = true
cursor-click = pointer
```

### Monitor Configuration
```ini
[bar/top]
width = 100%
height = 30pt
monitor = HDMI-1

[bar/bottom]
width = 50%
height = 25pt
monitor = DP-1
bottom = true
```

### Bar Position and Style
```ini
[bar/example]
; Position
top = false
bottom = true
offset-x = 0%
offset-y = 0%

; Size
width = 100%
height = 28pt

; Appearance
background = #282A2E
foreground = #C5C8C6
radius = 6
line-size = 3pt

; Borders
border-size = 4pt
border-color = #282A2E

; Fonts
font-0 = monospace:size=10
font-1 = FontAwesome:size=10
```

---

## 4. Modules

### Built-in Modules

#### Workspaces (i3/sway)
```ini
[module/xworkspaces]
type = internal/xworkspaces
label-active = %name%
label-active-background = ${colors.background-alt}
label-active-underline = ${colors.primary}
label-active-padding = 2
label-occupied = %name%
label-occupied-foreground = ${colors.foreground}
label-occupied-padding = 2
label-urgent = %name%
label-urgent-background = ${colors.alert}
label-urgent-padding = 2
label-empty = %name%
label-empty-foreground = ${colors.disabled}
label-empty-padding = 2
```

#### Window Title
```ini
[module/xwindow]
type = internal/xwindow
label = %title:0:60:...%
label-maxlen = 60
label-empty = Desktop
```

#### Date/Time
```ini
[module/date]
type = internal/date
interval = 1
time = %H:%M
time-alt = %Y-%m-%d %H:%M
format = <label>
format-prefix = " "
label = %date%
```

#### System Tray
```ini
[module/systray]
type = internal/systray
format = <tray>
tray-spacing = 8
tray-background = ${colors.background}
```

### System Modules

#### CPU
```ini
[module/cpu]
type = internal/cpu
interval = 1
format = <label>
format-prefix = " "
label = CPU %percentage:2%%
format-underline = ${colors.primary}
```

#### Memory
```ini
[module/memory]
type = internal/memory
interval = 1
format = <label>
format-prefix = " "
label = RAM %percentage_used:2%%
format-underline = ${colors.secondary}
```

#### Disk
```ini
[module/disk]
type = internal/disk
interval = 30
mount = /
format = <label-free><label-used>
format-free = " %free%"
format-used = " %used%"
format-prefix = " "
label-used = %percentage_used:2%%
label-free = %free:2%
```

#### Battery
```ini
[module/battery]
type = internal/battery
full-at = 100
low-at = 20
interval = 5
format = <label>
format-prefix = " "
format-underline = ${colors.alert}
label = %percentage%% [%status%]
status-charging = ⚡
status-discharging = 
```

#### Network
```ini
[module/wlan]
type = internal/network
interface = wlan0
interval = 3
format-connected = <label-connected>
format-connected-underline = ${colors.primary}
label-connected = %essid% %local_ip%
format-disconnected = <label-disconnected>
label-disconnected = offline
```

#### PulseAudio
```ini
[module/pulseaudio]
type = internal/pulseaudio
interval = 1
format = <label>
format-prefix = " "
label-volume = %percentage:2%%
format-muted = <label-muted>
label-muted = 🔇 muted
label-muted-foreground = ${colors.disabled}
```

### Custom Modules

#### Temperature
```ini
[module/temperature]
type = custom/script
exec = sensors | grep Core | awk '{print $3}' | cut -d'+' -f2 | head -1
interval = 5
format = <label>
format-prefix = " "
label = Temp: %output%
```

#### Weather
```ini
[module/weather]
type = custom/script
exec = curl -s wttr.in/London?format=1
interval = 600
format = <label>
format-prefix = " "
label = %output%
```

---

## 5. Themes and Styling

### Color Themes
```ini
[colors]
; Dracula
background = #282A2E
background-alt = #373B41
foreground = #F8F8F2
primary = #50FA7B
secondary = #8BE9FD
alert = #FF5555
disabled = #6272A4

; Gruvbox
background = #282828
background-alt = #3c3836
foreground = #ebdbb2
primary = #fabd2f
secondary = #8f3f71
alert = #cc241d
disabled = #928374

; Nord
background = #2E3440
background-alt = #3B4252
foreground = #D8DEE9
primary = #88C0D0
secondary = #81A1C1
alert = #BF616A
disabled = #4C566A
```

### Icons and Glyphs
```ini
[module/cpu]
format = <label>
format-prefix = "  "
label = %percentage:2%%

[module/memory]
format = <label>
format-prefix = "  "
label = %percentage_used:2%%

[module/battery]
status-charging = "  "
status-discharging = "  "
status-full = "  "

[module/wlan]
format-connected-prefix = "  "
format-disconnected-prefix = " ﲁ "
```

### Gradient Text
```ini
[module/gradient-text]
type = custom/text
content = Hello World
content-foreground = ${colors.foreground}
content-background = ${colors.background}
animation-gradient = true
gradient = ${colors.primary},${colors.secondary}
```

---

## 6. Launch Scripts

### i3/Sway Launch
```bash
#!/bin/bash
# ~/.config/polybar/launch.sh

killall polybar

# Primary bar
polybar example -c ~/.config/polybar/config.ini &

# Wait for bar
sleep 1

# Multiple monitors
for m in $(xrandr --query | grep " connected" | cut -d' ' -f1); do
    MONITOR=$m polybar secondary -c ~/.config/polybar/config.ini &
done

echo "Polybar launched..."
```

### i3 Config
```bash
# ~/.config/i3/config

# Launch polybar
exec_always --no-startup-id $HOME/.config/polybar/launch.sh

# Toggle polybar
bindsym $mod+Shift+b exec --no-startup-id polybar-msg cmd toggle
```

### Sway Config
```bash
# ~/.config/sway/config

# Launch polybar
exec_always --no-startup-id $HOME/.config/polybar/launch.sh

# Toggle polybar
bindsym $mod+Shift+b exec --no-startup-id polybar-msg cmd toggle
```

### IPC Control
```bash
# Toggle visibility
polybar-msg cmd toggle

# Show/hide specific bar
polybar-msg cmd show
polybar-msg cmd hide

# Reload configuration
polybar-msg cmd reload

# Get module values
polybar-msg get tagbar

# Action example
polybar-msg action "#menu.open"
```

---

## 7. Troubleshooting

### Common Issues

#### Bar Not Showing
```bash
# Check configuration
polybar -c ~/.config/polybar/config.ini -t

# Check for errors
polybar -c ~/.config/polybar/config.ini -r 2>&1 | head -50

# Check X11 connection
echo $DISPLAY

# Verify monitor names
xrandr | grep " connected"
```

#### Blank Bar
```bash
# Check modules
polybar-msg -m | grep modules

# Check font installation
fc-list | grep "JetBrains"

# Enable pseudo-transparency
pseudo-transparency = true
```

#### IPC Not Working
```bash
# Enable IPC
enable-ipc = true

# Check socket
ls /tmp/polybar_mqueue_*

# Restart bar
killall polybar
polybar example
```

#### Font Issues
```bash
# List installed fonts
fc-list | grep "FontAwesome"

# Install fonts
sudo apt install fonts-font-awesome

# Update font cache
fc-cache -fv

# Check font names in config
font-0 = FontAwesome;10
font-1 = materialdesignicons;10
```

### Debug Commands
```bash
# Trace mode
polybar -c config.ini -l trace

# Verbose output
polybar -c config.ini -v

# Check module output
polybar -c config.ini -r 2>&1 | grep "Output"

# Monitor IPC messages
polybar-msg listen
```

---

## 8. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| Run bar | `polybar example` |
| Check config | `polybar -t` |
| Reload | `polybar-msg cmd reload` |
| Toggle | `polybar-msg cmd toggle` |
| List bars | `polybar -m` |
| IPC listen | `polybar-msg listen` |

### Module Types
| Type | Description |
|------|-------------|
| internal/date | Date and time |
| internal/cpu | CPU usage |
| internal/memory | Memory usage |
| internal/battery | Battery status |
| internal/network | Network status |
| internal/pulseaudio | Volume |
| internal/xworkspaces | Workspaces |
| internal/xwindow | Window title |
| custom/script | Custom script |
| custom/ipc | Custom IPC |

### Common Fonts
| Font | Package |
|------|---------|
| FontAwesome | fonts-font-awesome |
| Material Icons | fonts-material-design-icons |
| Nerd Font | nerd-fonts |
| JetBrains Mono | fonts-jetbrains-mono |

---

*Last Updated: January 2026*
