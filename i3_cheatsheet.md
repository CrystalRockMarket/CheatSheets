# i3 Window Manager Cheat Sheet

## Mod Key
- Default: `Mod` = `Windows` key or `Alt`
- Configure in `~/.config/i3/config`: `set $mod Mod4`

## Basic Window Management

| Action | Shortcut |
|--------|----------|
| Open terminal | `$mod + Enter` |
| Close window | `$mod + Shift + q` |
| Kill window | `$mod + Shift + c` |
| Focus window | `j` (left), `k` (down), `l` (up), `;` (right) |
| Move window | `$mod + Shift + j/k/l/;` |
| Split mode | `$mod + h` (horizontal), `$mod + v` (vertical) |
| Fullscreen | `$mod + f` |
| Toggle floating | `$mod + Shift + Space` |
| Float resize | `$mod + right-click` |

## Workspace Management

| Action | Shortcut |
|--------|----------|
| Switch workspace | `$mod + [1-9]` |
| Move window to workspace | `$mod + Shift + [1-9]` |
| Next workspace | `$mod + n` |
| Previous workspace | `$mod + p` |
| Next workspace (all monitors) | `$mod + Ctrl + n` |
| Previous workspace (all monitors) | `$mod + Ctrl + p` |

## Layout Modes

| Action | Shortcut |
|--------|----------|
| Split toggle | `$mod + e` |
| Stacking | `$mod + s` |
| Tabbed | `$mod + w` |
| Split vertical | `$mod + v` |

## Resize Mode
- Enter: `$mod + r`
- Exit: `Esc` or `Enter`
- Resize with `j/k/l/;`

## Scratchpad
| Action | Shortcut |
|--------|----------|
| Move to scratchpad | `$mod + Shift + -` |
| Show scratchpad | `$mod + -` |

## i3bar
| Action | Shortcut |
|--------|----------|
| Toggle bar | `$mod + b` |

## Configuration Commands
```bash
# Reload config
$mod + Shift + c

# Restart i3
$mod + Shift + r

# Exit i3
$mod + Shift + e

# Check config syntax
i3 -c ~/.config/i3/config --validate
```

## Useful CLI Commands
```bash
i3-msg               # Send commands to i3
i3-dmenu-desktop     # App launcher
i3-swap              # Swap windows
i3-input             # Custom prompts
```

## Common Config Directives
```bash
# Gaps
gaps inner 10
gaps outer 5

# Border
new_window pixel 2
new_float pixel 2

# Workspaces
set $ws1 "1: term"
set $ws2 "2: web"

# Assignments
assign [class="Firefox"] → $ws2
```

## Key Symbols
- `$mod` - Mod4 (Windows) or Mod1 (Alt)
- `Mod` - Same as $mod
- `Ctrl` - Control
- `Shift` - Shift
- `Enter` - Return
- `Space` - Spacebar
- `Esc` - Escape

## Window States
- `tiled` - Default layout
- `floating` - Free-floating
- `fullscreen` - Fullscreen mode
- `none` - No decorations

## Monitor Setup
```bash
# In config
xrandr --output HDMI-1 --primary
exec --no-startup-id xrandr --output DP-1 --left-of HDMI-1
```

## Quick Reference
- `$mod + d` - dmenu
- `$mod + r` - Resize mode
- `$mod + Shift + Space` - Toggle float
- `$mod + f` - Fullscreen
- `$mod + Shift + e` - Exit i3
- `$mod + Ctrl + r` - Restart i3

---
*Generated: January 2026*
