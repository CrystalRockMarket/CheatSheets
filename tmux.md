# tmux Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Sessions](#2-sessions)
3. [Windows](#3-windows)
4. [Panes](#4-panes)
5. [Key Bindings](#5-key-bindings)
6. [Configuration](#6-configuration)
7. [Plugins](#7-plugins)
8. [Scripting](#8-scripting)
9. [Troubleshooting](#9-troubleshooting)
10. [Quick Reference](#10-quick-reference)

---

## 1. Introduction

tmux is a terminal multiplexer that allows you to create, manage, and detach from multiple terminal sessions.

### Key Features
| Feature | Description |
|---------|-------------|
| **Sessions** | Persistent terminal sessions |
| **Windows** | Tab-like interfaces |
| **Panes** | Split-screen terminals |
| **Copy Mode** | Buffer navigation |
| **Plugins** | Extend functionality |
| **Scripting** | Automate tasks |

### Architecture
```
┌─────────────────────────────────────┐
│         tmux Server                 │
│  ┌─────────────────────────────┐    │
│  │       Session 1            │    │
│  │  ┌─────┬─────┐            │    │
│  │  │Win1 │Win2 │            │    │
│  │  └─────┴─────┘            │    │
│  └─────────────────────────────┘    │
│  ┌─────────────────────────────┐    │
│  │       Session 2            │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
```

---

## 2. Sessions

### Session Management
```bash
# Start new session
tmux

# Start named session
tmux new -s mysession

# Attach to session
tmux attach -t mysession

# Attach to last session
tmux attach

# Detach from session
Ctrl+b d

# List sessions
tmux ls
tmux list-sessions

# Kill session
tmux kill-session -t mysession

# Kill all sessions
tmux kill-server
```

### Session Commands
```bash
# New session with name
tmux new -s dev -d

# Rename session
Ctrl+b $

# Switch session
Ctrl+b s

# Previous session
Ctrl+b (

# Next session
Ctrl+b )
```

---

## 3. Windows

### Window Management
```bash
# Create new window
Ctrl+b c

# Rename window
Ctrl+b ,

# List windows
Ctrl+b w

# Previous window
Ctrl+b p

# Next window
Ctrl+b n

# Select window by number
Ctrl+b 0-9

# Find window
Ctrl+b f

# Kill window
Ctrl+b &

# Swap windows
Ctrl+b {
Ctrl+b }
```

---

## 4. Panes

### Pane Creation
```bash
# Split vertically
Ctrl+b %

# Split horizontally
Ctrl+b "

# Split with size
Ctrl+b :split-window -h -p 30

# Maximize pane
Ctrl+b z

# Restore pane
Ctrl+b z (again)
```

### Pane Navigation
```bash
# Navigate panes
Ctrl+b arrow keys

# Cycle panes
Ctrl+b o

# Previous pane
Ctrl+b ;

# Jump to pane by number
Ctrl+b q 0-9
```

### Pane Management
```bash
# Resize panes
Ctrl+b :resize-pane -L 10
Ctrl+b :resize-pane -R 10
Ctrl+b :resize-pane -U 5
Ctrl+b :resize-pane -D 5

# Resize by 1
Ctrl+b Ctrl+arrow

# Kill pane
Ctrl+b x

# Display pane numbers
Ctrl+b q

# Swap panes
Ctrl+b {
Ctrl+b }
```

### Layouts
```bash
# Even horizontal
Ctrl+b Alt+1

# Even vertical
Ctrl+b Alt+2

# Main horizontal
Ctrl+b Alt+3

# Main vertical
Ctrl+b Alt+4

# Tiled
Ctrl+b Alt+5
```

---

## 5. Key Bindings

### Default Prefix
```bash
# Default prefix
Ctrl+b

# Change prefix
unbind C-b
set-option -g prefix C-a
bind C-a send-prefix
```

### Essential Bindings
| Binding | Action |
|---------|--------|
| `Ctrl+b d` | Detach |
| `Ctrl+b c` | New window |
| `Ctrl+b &` | Kill window |
| `Ctrl+b %` | Split vertical |
| `Ctrl+b "` | Split horizontal |
| `Ctrl+b o` | Next pane |
| `Ctrl+b ;` | Previous pane |
| `Ctrl+b x` | Kill pane |
| `Ctrl+b z` | Maximize pane |
| `Ctrl+b w` | List windows |
| `Ctrl+b ,` | Rename window |
| `Ctrl+b p` | Previous window |
| `Ctrl+b n` | Next window |
| `Ctrl+b [` | Copy mode |
| `Ctrl+b ]` | Paste buffer |
| `Ctrl+b ?` | Show key bindings |

### Copy Mode Bindings
```bash
# Enter copy mode
Ctrl+b [

# Navigation
h/j/k/l or arrows

# Word movement
b/w

# Line movement
gg/G

# Start selection
Space

# Copy selection
Enter

# Paste
Ctrl+b ]
```

---

## 6. Configuration

### Configuration File
```bash
# Location
~/.tmux.conf

# Create/reload
tmux source-file ~/.tmux.conf
```

### Basic Configuration
```bash
# ~/.tmux.conf

# Change prefix
unbind C-b
set-option -g prefix C-a
bind-key C-a send-prefix

# Enable mouse
set -g mouse on

# Set default terminal
set -g default-terminal "screen-256color"

# Status bar
set -g status-style bg=colour233,fg=colour251
set -g status-left-length 30
set -g status-right-length 50

# Panes
set -g pane-border-style fg=colour233
set -g pane-active-border-style fg=colour39

# Windows
set -g window-status-current-style fg=colour39,bold

# Messages
set -g message-style bg=colour233,fg=colour251
```

### Advanced Configuration
```bash
# Reload config
bind r source-file ~/.tmux.conf \; display "Reloaded!"

# Mouse configuration
set -g mouse on
bind -n WheelUpPane if-shell -F -t = "#{mouse_any_flag}" "send-keys -M" "if -Ft= "#{pane_in_mode}" "send-keys -M" "copy-mode -e"

# Copy to system clipboard
bind -T copy-mode-vi v send-keys -X copy-pipe-and-cancel "xclip -selection clipboard -i"

# Faster key repeat
set -g repeat-time 500

# Automatic renumbering
set -g renumber-windows on

# Focus events
set -g focus-events on
```

---

## 7. Plugins

### TPM (Plugin Manager)
```bash
# Install TPM
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# Add to .tmux.conf
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'

# Install plugins
Press prefix + I

# Update plugins
Press prefix + U

# Remove plugins
Press prefix + alt + u
```

### Essential Plugins
```bash
# ~/.tmux.conf

# TPM
set -g @plugin 'tmux-plugins/tpm'

# Sensible defaults
set -g @plugin 'tmux-plugins/tmux-sensible'

# Copy to system clipboard
set -g @plugin 'tmux-plugins/tmux-yank'

# Mouse support
set -g @plugin 'tmux-plugins/tmux-mouse-support'

# Continuum (auto-save)
set -g @plugin 'tmux-plugins/tmux-continuum'

# Resurrect (save environments)
set -g @plugin 'tmux-plugins/tmux-resurrect'

# Navigation (vim-tmux-navigator)
set -g @plugin 'christoomey/vim-tmux-navigator'
```

---

## 8. Scripting

### Basic Script
```bash
#!/bin/bash
# my-session.sh

tmux new-session -d -s dev -x 120 -y 40
tmux new-window -t dev -n 'vim'
tmux send-keys -t dev:vim 'vim' Enter
tmux split-window -t dev:vim -h
tmux send-keys -t dev:vim.2 'ls -la' Enter
tmux attach-session -t dev
```

### Advanced Script
```bash
#!/bin/bash
# create-dev-env.sh

SESSION="dev-env"
tmux has-session -t $SESSION 2>/dev/null

if [ $? -ne 0 ]; then
    # Create session
    tmux new-session -d -s $SESSION -x 120 -y 40
    
    # Editor window
    tmux rename-window -t $SESSION:1 'vim'
    tmux send-keys -t $SESSION:1 'vim' Enter
    
    # Server window
    tmux new-window -t $SESSION:2 'server'
    tmux send-keys -t $SESSION:2 'npm run dev' Enter
    
    # Database window
    tmux new-window -t $SESSION:3 'db'
    tmux send-keys -t $SESSION:3 'mysql.server start' Enter
    
    # Logs window
    tmux new-window -t $SESSION:4 'logs'
    tmux send-keys -t $SESSION:4 'tail -f /var/log/syslog' Enter
fi

tmux attach-session -t $SESSION
```

### Save/Restore
```bash
# Save session state
tmux save-session -t mysession /tmp/session.txt

# Restore session
tmux new-session -d -s restored
# Manually recreate from file
```

---

## 9. Troubleshooting

### Common Issues

#### Colors Not Working
```bash
# Set terminal
set -g default-terminal "screen-256color"

# In .tmux.conf
set -g default-terminal "xterm-256color"

# Check terminal
echo $TERM
```

#### Mouse Not Working
```bash
# Enable mouse
set -g mouse on

# Check plugin
set -g @plugin 'tmux-plugins/tmux-mouse-support'

# Reload config
tmux source-file ~/.tmux.conf
```

#### Copy Mode Issues
```bash
# Enable mouse
set -g mouse on

# Use vi mode
setw -g mode-keys vi

# Copy to clipboard
bind -T copy-mode-vi v send-keys -X copy-pipe-and-cancel "xclip -selection clipboard -i"
```

### Debug Commands
```bash
# Show version
tmux -V

# Show configuration
tmux show-options -g

# Check processes
ps aux | grep tmux

# Show sessions
tmux ls

# Check socket
ls /tmp/tmux-*/
```

---

## 10. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| New session | `tmux new -s name` |
| Attach | `tmux attach -t name` |
| Detach | `Ctrl+b d` |
| List sessions | `tmux ls` |
| New window | `Ctrl+b c` |
| Split vertical | `Ctrl+b %` |
| Split horizontal | `Ctrl+b "` |
| Navigate panes | `Ctrl+b arrows` |
| Copy mode | `Ctrl+b [` |
| Paste | `Ctrl+b ]` |

### Key Bindings Quick Reference
| Binding | Action |
|---------|--------|
| `d` | Detach |
| `c` | New window |
| `"` | Split horizontal |
| `%` | Split vertical |
| `o` | Next pane |
| `x` | Kill pane |
| `z` | Maximize pane |
| `w` | List windows |
| `[` | Copy mode |
| `]` | Paste |
| `?` | Help |

---

*Last Updated: January 2026*
