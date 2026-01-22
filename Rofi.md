# Rofi Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Basic Usage](#3-basic-usage)
4. [Configuration](#4-configuration)
5. [Modes](#5-modes)
6. [Themes](#6-themes)
7. [Scripts and Actions](#7-scripts-and-actions)
8. [Troubleshooting](#8-troubleshooting)
9. [Quick Reference](#9-quick-reference)

---

## 1. Introduction

Rofi is a window switcher, application launcher, and dmenu replacement with support for custom modes and themes.

### Key Features
| Feature | Description |
|---------|-------------|
| **Window Switcher** | Quick window navigation |
| **App Launcher** | Launch applications |
| **SSH Client** | SSH connections |
| **Custom Modes** | User-defined modes |
| **Theming** | Full customization |
| **Scriptable** | Custom actions |

### Running Modes
```bash
# Window switcher
rofi -show window

# Application launcher
rofi -show drun

# SSH
rofi -show ssh

# Combined
rofi -show combi

# File browser
rofi -show filebrowser

# Calculator
rofi -calc
```

---

## 2. Installation

### Installation
```bash
# Ubuntu/Debian
sudo apt install rofi

# Arch Linux
sudo pacman -S rofi

# Fedora
sudo dnf install rofi

# macOS
brew install rofi
```

### Basic Launch
```bash
# Default (application launcher)
rofi

# Show specific mode
rofi -show drun

# Window switcher
rofi -show window

# SSH mode
rofi -show ssh

# File browser
rofi -show filebrowser
```

---

## 3. Basic Usage

### Keybindings
| Key | Action |
|-----|--------|
| `Ctrl+Space` | Complete selected |
| `Tab` | Complete entry |
| `Enter` | Select |
| `Shift+Enter` | Custom action |
| `Esc` | Quit |
| `Ctrl+c` | Quit |
| `Up/Down` | Navigate |
| `Page Up/Down` | Scroll |
| `Home/End` | First/Last |
| `Ctrl+v` | Paste |

### Command Options
```bash
# Show menu
rofi -show run

# Window mode
rofi -show window

# SSH mode
rofi -show ssh

# Password mode
rofi -show password

# Matching
rofi -matching fuzzy
rofi -matching regex

# Case sensitivity
rofi -case-sensitive

# Filter
rofi -filter "firefox"
```

### Input and Output
```bash
# From stdin
echo "item1\nitem2" | rofi -dmenu

# Output selected
rofi -dmenu < input.txt

# Multi-select
rofi -dmenu -multi-select

# Output format
rofi -dmenu -format "s"
```

---

## 4. Configuration

### Configuration File
```bash
# Location
~/.config/rofi/config.rasi

# Example configuration
@theme "/usr/share/rofi/themes/solarized.rasi"

configuration {
    modi: "window,run,ssh";
    width: 50;
    lines: 15;
    columns: 1;
    font: "JetBrains Mono 12";
    fixed-num-lines: true;
    show-icons: true;
    icon-theme: "Papirus";
}
```

### Global Settings
```rasi
configuration {
    modi: "drun,window,run,ssh";
    width: 50;
    height: 30;
    lines: 10;
    columns: 1;
    font: "DejaVu Sans 12";
    theme: "default";
    auto-select: false;
    filter: "";
    case-sensitive: false;
    cycle: true;
    sidebar-mode: false;
    show-icons: true;
    icon-theme: "Papirus";
}
```

### Window Switcher
```rasi
configuration {
    window-format: "{w:10} {t}";
    window-thumbnail: true;
    show-all-desktop-windows: true;
    skip-dialog: true;
    matching: "normal";
}
```

### App Launcher
```rasi
configuration {
    drun-show-actions: true;
    drun-match-fields: "name,generic,exec,keywords";
    drun-display-format: "{name}";
    show-exec-icon: true;
    run-command: "{exec}";
}
```

---

## 5. Modes

### Built-in Modes

#### drun (Application Launcher)
```bash
rofi -show drun

# With actions
rofi -show drun -drun-show-actions

# Specific category
rofi -show drun -drun-categories "Network;Office"
```

#### window (Window Switcher)
```bash
rofi -show window

# Format options
# {w} = window title
# {t} = window class
# {c} = window client
# {f} = flags
rofi -window-format "{w} - {t}"
```

#### run (Application Runner)
```bash
rofi -show run

# Custom command
rofi -run-command "{cmd}"
```

#### ssh (SSH Client)
```bash
rofi -show ssh

# SSH hosts from file
rofi -ssh-command "ssh {host}"
```

#### filebrowser (File Browser)
```bash
rofi -show filebrowser

# Start directory
rofi -filebrowser-executable
```

### Custom Modes
```bash
# Create custom mode
rofi -modes "drun,run,custom:/path/to/custom.sh"

# Custom mode script
#!/bin/bash
echo "Option 1"
echo "Option 2"
echo "Option 3"
```

### Mode Configuration
```rasi
configuration {
    modi: "drun,run,ssh,combi";
    combi-modi: "window,run";
}
```

---

## 6. Themes

### Theme Files
```bash
# System themes
/usr/share/rofi/themes/

# User themes
~/.config/rofi/themes/

# Apply theme
rofi -theme /path/to/theme.rasi

# Theme name only (from system)
rofi -theme solarized
```

### Theme Structure
```rasi
/* ~/.config/rofi/themes/my-theme.rasi */
* {
    background: #282a2e;
    foreground: #f8f8f2;
    border: #44475a;
    selected: #6272a4;
    highlight: #ff79c6;
    font: "JetBrains Mono 12";
}

window {
    width: 50%;
    background-color: @background;
    border: 2px;
    border-color: @border;
    border-radius: 8px;
}

listview {
    lines: 10;
    columns: 1;
    spacing: 5px;
}

element {
    background-color: @background;
    text-color: @foreground;
    border: 0px;
    border-radius: 4px;
    padding: 5px;
}

element selected {
    background-color: @selected;
    text-color: @foreground;
}

element active {
    background-color: @selected;
}

element urgent {
    background-color: @alert;
}

sidebar {
    background-color: @background;
}

inputbar {
    background-color: @background;
    text-color: @foreground;
    border: 0px 0px 2px 0px;
    border-color: @selected;
}

prompt {
    background-color: @background;
    text-color: @foreground;
}

entry {
    background-color: @background;
    text-color: @foreground;
}

scrollbar {
    background-color: @background;
    handle-color: @selected;
}
```

### Common Themes
```bash
# Dracula
rofi -theme /usr/share/rofi/themes/dracula.rasi

# Solarized
rofi -theme /usr/share/rofi/themes/solarized.rasi

# Arc
rofi -theme arc.rasi

# Adapta
rofi -theme adapta.rasi
```

### Color Schemes
```rasi
* {
    /* Dracula */
    background: #282a2e;
    foreground: #f8f8f2;
    border: #44475a;
    selected: #6272a4;
    highlight: #ff79c6;
    alert: #ff5555;
}
```

---

## 7. Scripts and Actions

### Script Mode
```bash
#!/bin/bash
# ~/.config/rofi/scripts/system-menu.sh

options="Lock\nLogout\nSuspend\nReboot\nShutdown"

chosen=$(echo -e $options | rofi -dmenu -p "System" -theme /path/to/theme.rasi)

case $chosen in
    Lock)
        dm-tool lock
        ;;
    Logout)
        i3-msg exit
        ;;
    Suspend)
        systemctl suspend
        ;;
    Reboot)
        systemctl reboot
        ;;
    Shutdown)
        systemctl poweroff
        ;;
esac
```

### Custom Actions
```bash
# SSH with custom command
rofi -ssh-command "xfce4-terminal -e 'ssh {host}'"

# App with custom action
rofi -drun-show-actions -actions "run,edit"

# Browser bookmarks
rofi -bookmarks -web-search
```

### Dmenu Scripts
```bash
#!/bin/bash
# Power menu
echo -e "Lock\nLogout\nSuspend\nReboot\nShutdown" | \
    rofi -dmenu -p "Power" -theme arc.rasi | \
    xargs -I {} systemctl {}
```

### Rofiwrap
```bash
# Wrapper script
#!/bin/bash
rofi -show drun \
    -theme /usr/share/rofi/themes/gruvbox.rasi \
    -font "JetBrains Mono 12" \
    -show-icons \
    -drun-icon-theme "Papirus" \
    -matching fuzzy \
    -filter "fire"
```

---

## 8. Troubleshooting

### Common Issues

#### Not Showing
```bash
# Check if running
ps aux | grep rofi

# Check display
echo $DISPLAY

# Check i3/sway config
# Ensure mod key is correct
```

#### Style Issues
```bash
# List available themes
ls /usr/share/rofi/themes/

# Check theme syntax
cat /usr/share/rofi/themes/default.rasi | head -10

# Reset to default
rofi -theme default
```

#### Performance
```bash
# Disable icons
rofi -no-icons

# Disable filtering
rofi -filter ""

# Lower font resolution
rofi -dpi 96
```

### Debug Commands
```bash
# Verbose output
rofi -v

# Trace mode
rofi -log-level trace

# Check X11 errors
xprop | head -20
```

---

## 9. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| App launcher | `rofi -show drun` |
| Window switcher | `rofi -show window` |
| SSH | `rofi -show ssh` |
| File browser | `rofi -show filebrowser` |
| Calculator | `rofi -calc` |
| Password | `rofi -password` |

### Command Options
| Option | Description |
|--------|-------------|
| `-show mode` | Show specific mode |
| `-width` | Width percentage |
| `-lines` | Number of lines |
| `-font` | Font and size |
| `-theme` | Theme file |
| `-icon-theme` | Icon theme |
| `-show-icons` | Show icons |
| `-filter` | Initial filter |
| `-case-sensitive` | Case sensitive |
| `-matching` | Matching algorithm |

### Keyboard Shortcuts
| Key | Action |
|-----|--------|
| `Enter` | Select |
| `Esc` | Quit |
| `Tab` | Complete |
| `Ctrl+Space` | Complete |
| `Shift+Enter` | Custom action |
| `Up/Down` | Navigate |
| `PgUp/PgDn` | Page scroll |
| `Home/End` | First/Last |

---

*Last Updated: January 2026*
