# htop Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Interface Overview](#3-interface-overview)
4. [Navigation](#4-navigation)
5. [Function Keys](#5-function-keys)
6. [Process Management](#6-process-management)
7. [Customization](#7-customization)
8. [Columns/Meters](#8-columnsmeters)
9. [Filters and Search](#9-filters-and-search)
10. [Tree View](#10-tree-view)
11. [Colors and Themes](#11-colors-and-themes)
12. [Setup Options](#12-setup-options)
13. [Command-Line Options](#13-command-line-options)
14. [Mouse Controls](#14-mouse-controls)
15. [Quick Reference](#15-quick-reference)

---

## 1. Introduction

### What is htop?
htop is an interactive process viewer for Linux and Unix systems that provides a real-time view of system processes with color coding and additional information not available in the traditional `top` command.

### Key Features
| Feature | Description |
|---------|-------------|
| **Color Coding** | Color-coded CPU, memory, and swap usage |
| **Scrollable Process List** | Navigate vertically and horizontally |
| **Tree View** | Show parent-child process relationships |
| **Mouse Support** | Click to select and perform actions |
| **Kill Processes** | Easy process termination |
| **Customizable Columns** | Choose which information to display |
| **Per-CPU Graphs** | Visual CPU usage representation |
| **Easy Navigation** | Keyboard shortcuts for quick access |

### Difference from top
| Feature | top | htop |
|----------|-----|------|
| Color output | No | Yes |
| Mouse support | Limited | Full |
| Scrollable | Vertical only | Both directions |
| Tree view | No | Yes |
| Process signals | Limited | All signals |
| Customization | Limited | Extensive |
| Easy navigation | Arrow keys | Arrow + function keys |

---

## 2. Installation

### Debian/Ubuntu
```bash
sudo apt update
sudo apt install htop
```

### RHEL/CentOS/Fedora
```bash
# RHEL/CentOS
sudo dnf install epel-release
sudo dnf install htop

# Fedora
sudo dnf install htop
```

### Arch Linux
```bash
sudo pacman -S htop
```

### macOS
```bash
# Using Homebrew
brew install htop

# Using MacPorts
sudo port install htop
```

### From Source
```bash
wget https://github.com/htop-dev/htop/releases/download/3.2.2/htop-3.2.2.tar.gz
tar xzf htop-3.2.2.tar.gz
cd htop-3.2.2
./configure
make
sudo make install
```

### Verify Installation
```bash
htop --version
htop --help
```

---

## 3. Interface Overview

### Main Screen Layout
```
  1  ████████  2  ████████████  3  ████████████  4  Tasks: 125/500 (25%)  Uptime: 5d 12h 30m
  ████████████████  ████████████████████████████  ████████████████████████████  Mem: 8.00G/16.0G  Swp: 2.00G/8.00G

  PID    USER   PRI   NI   VIRT   RES   SHR   S   CPU%   MEM%   TIME+   Command
  1234   root     20    0   1234M  456M   120M   R    45.2    2.8    12:34.56  /usr/bin/python3 app.py
  5678   mike     20    0    890M   234M    89M   S    12.1    1.5     3:21.45  /usr/bin/node server.js
  9012   www-data 20    0    567M   123M    56M   S     8.3    0.8     1:12.34  nginx: worker process
  3456   root     20    0    234M    45M    22M   S     5.2    0.3     0:45.67  /usr/lib/systemd/systemd
  ...

F1Help  F2Setup  F3Search  F4Filter  F5Tree   F6SortBy F7Nice-  F8Nice+  F9Kill  F10Quit
```

### Header Sections

#### CPU/Memory/Swap Bars
```text
  CPU[|||||||||     ]  MEM[|||||       ]  SWP[||          ]
  45%                62%               25%

  Green: Normal priority processes
  Blue:  Low priority (nice) processes
  Red:   Kernel threads
  Yellow: IRQ time
  Magenta: Soft IRQ time
  Gray:  IO wait time
```

#### Task and Load Average
```
  Tasks: 125 total, 3 running, 122 sleeping, 0 stopped, 0 zombie
  Load average: 1.25  1.18  1.05  (1 min, 5 min, 15 min)
```

#### Uptime
```
  Uptime: 5 days, 12 hours, 30 minutes, 45 seconds
```

### Process Columns
| Column | Description |
|--------|-------------|
| **PID** | Process ID |
| **USER** | Process owner |
| **PR** | Priority (real) |
| **NI** | Nice value |
| **VIRT** | Virtual memory |
| **RES** | Resident memory |
| **SHR** | Shared memory |
| **S** | Status |
| **CPU%** | CPU usage percentage |
| **MEM%** | Memory usage percentage |
| **TIME+** | CPU time |
| **COMMAND** | Command name/line |

### Process States
| State | Symbol | Description |
|-------|--------|-------------|
| Running | R | Currently running |
| Sleeping | S | Sleeping (wait for event) |
| Uninterruptible | D | Sleeping (I/O) |
| Zombie | Z | Terminated but not reaped |
| Stopped | T | Stopped by signal |
| Tracing | t | Being traced |

---

## 4. Navigation

### Keyboard Navigation
| Key | Action |
|-----|--------|
| `Arrow Up/Down` | Move selection up/down |
| `Arrow Left/Right` | Move selection left/right |
| `Page Up/Down` | Move one page up/down |
| `Home/End` | Go to beginning/end of list |
| `Tab` | Switch between process list and tree |
| `Ctrl + A` | Go to beginning |
| `Ctrl + E` | Go to end |

### Quick Navigation
| Key | Action |
|-----|--------|
| `u` | Show only processes of current user |
| `H` | Hide/show userland threads |
| `K` | Hide/show kernel threads |
| `F` | Follow process (keep selected) |
| `+` | Expand tree node |
| `-` | Collapse tree node |
| `*` | Toggle tree expansion |
| `Ctrl + L` | Refresh display |

### Search and Filter
| Key | Action |
|-----|--------|
| `Ctrl + F` | Search forward |
| `Ctrl + R` | Search backward |
| `/` | Search |
| `\` | Incremental search |
| `Space` | Tag/untag process |
| `U` | Untag all tagged |

---

## 5. Function Keys

### F1 - Help
```text
Displays help screen with all available commands and shortcuts.
```

### F2 - Setup
```text
┌─────────────────────────────────────────────────────────┐
│                    htop Setup                          │
├─────────────────────────────────────────────────────────┤
│  1. Display options                                    │
│  2. Colors                                             │
│  3. Columns                                            │
│  4. Meters                                             │
│  5. Layout                                             │
│  6. Behavior                                           │
│  7. Columns                                            │
├─────────────────────────────────────────────────────────┤
│  Press F10 to save and exit                            │
└─────────────────────────────────────────────────────────┘
```

### F3 - Search
```text
Search for a process by name or PID.
Use regular expressions for advanced matching.

Example searches:
  /nginx        - Find nginx processes
  /python       - Find Python processes
  /1234         - Find PID 1234
```

### F4 - Filter
```text
Filter processes by command name.
Only processes matching the filter will be displayed.

Example filters:
  nginx         - Only nginx processes
  python.*server - Python server processes
  ^apache       - Processes starting with apache
```

### F5 - Tree View
```text
Toggle between tree view and flat list view.
Tree view shows parent-child relationships.

Keys in tree view:
  +  - Expand node
  -  - Collapse node
  *  - Toggle all expansion
```

### F6 - Sort By
```text
┌─────────────────────────────────────────────────────┐
│               Sort by                              │
├─────────────────────────────────────────────────────┤
│  PID                                                 │
│  PPID                                                │
│  CPU%                                                │
│  MEM%                                                │
│  TIME                                                │
│  PERCENT_CPU                                         │
│  PERCENT_MEM                                         │
│  USER                                                │
│  PRIORITY                                            │
│  NICE                                                │
│  COMM                                                │
│  COMMAND                                             │
│  OOM                                                 │
│  IO_RATE                                             │
│  IO_READ_RATE                                        │
│  IO_WRITE_RATE                                       │
├─────────────────────────────────────────────────────┤
│  Press Enter to select, Esc to cancel                │
└─────────────────────────────────────────────────────┘
```

### F7 - Nice Decrease (Increase Priority)
```text
Decrease nice value (increase priority) of selected process.
Requires root privileges for system processes.

Current nice: 0 -> New nice: -1 (if permitted)
```

### F8 - Nice Increase (Decrease Priority)
```text
Increase nice value (decrease priority) of selected process.
Any user can increase nice value.

Current nice: 0 -> New nice: 1
```

### F9 - Kill
```text
┌─────────────────────────────────────────────────────┐
│                   Kill Process                      │
├─────────────────────────────────────────────────────┤
│  SIGTERM (15)  - Terminate gracefully               │
│  SIGKILL (9)   - Force terminate                    │
│  SIGINT (2)    - Interrupt                          │
│  SIGQUIT (3)   - Quit                               │
│  SIGSTOP (19)  - Stop                               │
│  SIGCONT (18)  - Continue                           │
│  SIGHUP (1)    - Hangup                             │
│  SIGUSR1 (10)  - User defined 1                     │
│  SIGUSR2 (12)  - User defined 2                     │
├─────────────────────────────────────────────────────┤
│  Press signal number or select and press Enter      │
└─────────────────────────────────────────────────────┘
```

### F10 - Quit
```text
Exit htop. Configuration changes are saved automatically.
```

---

## 6. Process Management

### Sending Signals
```bash
# From within htop: Press F9, select signal, press Enter

# Common signals:
SIGTERM (15)  - Graceful termination (default)
SIGKILL (9)   - Force kill (cannot be caught)
SIGINT (2)    - Interrupt (like Ctrl+C)
SIGSTOP (19)  - Stop process
SIGCONT (18)  - Continue stopped process
```

### Renicing Processes
```bash
# In htop: Press F7 (decrease nice) or F8 (increase nice)

# From command line:
nice -n 10 process_name        # Start with nice value
renice -n 5 -p 1234            # Change nice of PID 1234
renice -n -5 -u username       # Change nice of user's processes
```

### Killing Processes
```bash
# From command line (alternative to htop):
kill -TERM 1234                # Graceful termination
kill -KILL 1234                # Force kill
killall process_name           # Kill by name
pkill -9 process_name          # Kill by pattern
xkill                          # Kill X window
```

### Filtering by User
```bash
# In htop: Press 'u' and select user

# From command line:
ps -U username                 # Processes by user
ps -u username                 # Detailed by user
```

---

## 7. Customization

### Configuration File
```bash
# Location
~/.config/htop/htoprc

# Manual configuration
mkdir -p ~/.config/htop
htop -C                       # Generate default config
```

### Sample Configuration
```ini
# ~/.config/htop/htoprc

[settings]
delay=15
hide_kernel_threads=0
hide_userland_threads=0
shadow_other_users=0
show_thread_names=0
show_program_path=0
highlight_base_name=1
highlight_megabytes=1
highlight_tasks=1
find_comm_in_wrap=1
tree_view=0
all_branches_collapsed=0
header_margin=1
screen_buffer=8

[columns]
0:PID,1:USER,2:PR,3:NI,4:VIRT,5:RES,6:SHR,7:S,8:CPU%,9:MEM%,10:TIME+,11:COMMAND

[display]
hide_root_user=0
show_cpu_usage=1
show_cpu_frequency=0
show_cpu_temperature=0
show_memory_in_bytes=0
detailed_cpu_time=0
cpu_temperature_fahrenheit=0
temp_color=2
update_process_names=0
graph_cpus=0
graph_color=1
proc_color=2
proc_field_0=1
proc_field_1=1
proc_field_2=1
proc_field_3=1
proc_field_4=1
proc_field_5=1
proc_field_6=1
proc_field_7=1
proc_field_8=1
proc_field_9=1
proc_field_10=1
proc_field_11=1
```

### Command-Line Customization
```bash
# Set columns
htop -c PID,USER,CPU%,MEM%,TIME+,COMMAND

# Set sort column
htop -s PERCENT_CPU

# Tree view
htop -t

# User filter
htop -u username

# Delay refresh
htop -d 5       # 5 tenths of a second (0.5 seconds)

# Silent mode
htop -s

# Batch mode (useful for scripts)
htop -b -n 3 > output.txt
```

---

## 8. Columns/Meters

### Available Columns
| Column | Description |
|--------|-------------|
| **PID** | Process ID |
| **PPID** | Parent PID |
| **UID** | User ID |
| **USER** | Username |
| **PRI** | Priority (real) |
| **NI** | Nice value |
| **VIRT** | Virtual memory |
| **RES** | Resident memory |
| **SHR** | Shared memory |
| **SWAP** | Swap usage |
| **STATE** | Process state |
| **S** | State (single letter) |
| **CPU%** | CPU percentage |
| **MEM%** | Memory percentage |
| **TIME** | CPU time |
| **TIME+** | CPU time (hundredths) |
| **COMM** | Command name |
| **COMMAND** | Full command |
| **PPID** | Parent PID |
| **RPRVT** | Resident private memory |
| **RSHRD** | Resident shared memory |
| **RSIZE** | Resident size |
| **VPRVT** | Virtual private memory |
| **VSIZE** | Virtual size |

### Available Meters
| Meter | Description |
|-------|-------------|
| **CPU** | CPU usage bar |
| **Memory** | Memory usage bar |
| **Swap** | Swap usage bar |
| **Tasks** | Task summary |
| **Load** | Load average |
| **Uptime** | System uptime |
| **Battery** | Battery status |
| **Clock** | Current time |
| **AllCPUs** | Per-CPU usage |
| **AllCPUs2** | Per-CPU (two rows) |
| **DiskIO** | Disk I/O |
| **Network** | Network usage |
| **Sensors** | Temperature sensors |

### Column Configuration
```bash
# In htop: Press F2 -> Columns

# Available columns:
PID        - Process ID
PPID       - Parent PID
UID        - User ID
USER       - Username
PRI        - Priority
NI         - Nice value
VIRT       - Virtual memory
RES        - Resident memory
SHR        - Shared memory
SWAP       - Swap usage
CPU%       - CPU percentage
MEM%       - Memory percentage
TIME       - CPU time
TIME+      - CPU time (detailed)
COMM       - Command name
COMMAND    - Full command
OOM        - OOM score
IO_RATE    - I/O rate
IO_READ_RATE  - Read rate
IO_WRITE_RATE - Write rate
```

---

## 9. Filters and Search

### Search
```bash
# Press / or Ctrl+F
# Type search term
# Press Enter

# Examples:
/nginx      # Find nginx
/python3    # Find Python 3
/ssh        # Find SSH
```

### Incremental Search
```bash
# Press \ (backslash)
# Type as you go, results update in real-time
```

### Filter
```bash
# Press F4
# Type filter expression
# Only matching processes shown

# Examples:
nginx       # Only nginx
python.*server  # Python server
^d           # Starts with d
.*firefox.*  # Contains firefox
```

### Tag Processes
```bash
# Press Space on a process to tag
# Press U to untag all
# Perform action on all tagged
```

### Filter by User
```bash
# Press 'u'
# Select user from list
# Only that user's processes shown

# Show all: Press 'u', select <All>
```

---

## 10. Tree View

### Tree View Basics
```text
# Press F5 to toggle tree view

apache2(1234)
├── apache2(1235)
│   └── apache2(1236)
└── apache2(1237)

  Parent processes at left, children indented.
```

### Tree View Controls
| Key | Action |
|-----|--------|
| `F5` | Toggle tree/flat view |
| `+` | Expand node |
| `-` | Collapse node |
| `*` | Toggle all expansion |
| `Arrow Up/Down` | Navigate tree |
| `Tab` | Switch between list and tree |

### Tree View Options
```bash
# Show collapsed
htop --tree

# All branches expanded
htop -T

# Custom tree depth
# Not directly available, use collapse (*)
```

---

## 11. Colors and Themes

### Color Schemes
```bash
# In htop: Press F2 -> Colors

# Available themes:
Default
Light        - Light background
Midnight     - Dark blue background
Solarized    - Solarized colors
Gruvbox      - Gruvbox colors
Monochrome   - Black and white
Black on White
Grayscale
```

### Custom Colors
```bash
# In htop: Press F2 -> Colors -> [Select theme] -> [Customize]

# Color settings:
Small bar        - Used for small bars
Medium bar       - Used for medium bars
Large bar        - Used for large bars
Task             - Task name color
Task selected    - Selected task color
Task scrolling   - Scrolling task color
Shared library   - Library color
Thread           - Thread color
Thread selected  - Selected thread color
Thread scrolling - Scrolling thread color
Header text      - Header text color
Header highlights- Header highlight color
Border           - Border color
Graph CPU        - CPU graph color
Graph Mem        - Memory graph color
Graph Swap       - Swap graph color
Graph Nice       - Nice value color
Graph IRQ        - IRQ color
Graph SoftIRQ    - Soft IRQ color
Graph Steal      - Steal color
Graph IOWait     - I/O wait color
```

### Custom Theme Configuration
```ini
# ~/.config/htop/htoprc

[colors]
# Default theme colors
background= #1e1e1e
meter_initial= #555555
meter_text= #888888
meter_h1= #88b040
meter_h2= #a0a030
meter_h3= #b0b030
meter_h4= #c0c020
meter_h5= #d0d010
meter_h6= #e0e000
basic= #ffffff
title= #a0a0a0
spinner= #88b040
data= #ffffff
value= #ffffff
selected= #4a4a4a
border= #888888
header= #a0a0a0
table_header= #a0a0a0
highlight= #5a5a5a
hot= #d70000
```

---

## 12. Setup Options

### Display Options
```bash
# In htop: Press F2 -> Display options

Hide kernel threads=No
Hide userland threads=No
Shadow other users=No
Show thread names=No
Show program path=Yes
Highlight base name=Yes
Highlight megabytes=Yes
Highlight tasks=Yes
Find command in wrap=Yes
Tree view=No
All branches collapsed=No
Header margin=1
Screen buffer=8
```

### Meters Setup
```bash
# In htop: Press F2 -> Meters

# Left side:
AllCPUs|1 25
Memory|1 25
Swap|1 25

# Right side:
Tasks|2 20
Load|2 20
Uptime|2 20
Clock|2 20
```

### Layout
```bash
# In htop: Press F2 -> Layout

# Header layout
Two columns      - Header in two columns
One column       - Header in one column
Small            - Small header
```

### Behavior
```bash
# In htop: Press F2 -> Behavior

Update process names=No
Update interval=1.5 seconds
Detailed CPU time=No
CPU temperature=Fahrenheit
Color mode=Automatic
```

---

## 13. Command-Line Options

### Basic Options
| Option | Description |
|--------|-------------|
| `-h, --help` | Show help |
| `-v, --version` | Show version |
| `-s, --sort` | Sort by column |
| `-u, --user` | Show only user |
| `-p, --pid` | Show only PID |
| `-t, --tree` | Tree view |
| `-d, --delay` | Set delay (tenths) |
| `-C, --no-color` | No colors |
| `-s, --sort` | Sort key |

### Advanced Options
| Option | Description |
|--------|-------------|
| `--no-color` | Disable colors |
| `--no-mouse` | Disable mouse |
| `--no-prefix` | No process prefix |
| `--show-comm` | Show command name |
| `--long` | Long columns |
| `--batch` | Batch mode |
| `--user-config=FILE` | Config file |
| `--pid=FILE` | PID file |
| `--stat=PATH` | Path to /proc |

### Examples
```bash
# Basic
htop

# Specific user
htop -u www-data

# Tree view
htop -t

# Custom delay
htop -d 10        # 1 second

# Sort by CPU
htop -s PERCENT_CPU

# Multiple options
htop -u nginx -s MEM% -t

# Batch mode for scripts
htop -b -n 10 > output.txt

# Tree with PID
htop -t -p 1234,5678

# Quiet mode
htop -s

# Custom config
htop --user-config=/path/to/htoprc
```

---

## 14. Mouse Controls

### Mouse Actions
| Mouse Action | Result |
|--------------|--------|
| Left-click | Select process |
| Left-click on header | Sort by column |
| Right-click | Process menu |
| Scroll up/down | Scroll list |
| Click F-keys | Function key actions |

### Right-Click Menu
```text
When right-clicking on a process:
  - Nice - (increase priority)
  - Nice + (decrease priority)
  - Kill (terminate process)
  - IO priority (set I/O nice)
  - Filter (filter by command)
  - Search (search for command)
  - Tree (toggle tree)
```

### Scroll Wheel
```bash
# Scroll up/down in process list
# Scroll horizontally in wide columns
```

---

## 15. Quick Reference

### Essential Keyboard Shortcuts
| Key | Action |
|-----|--------|
| `F1` | Help |
| `F2` | Setup |
| `F3` | Search |
| `F4` | Filter |
| `F5` | Tree view |
| `F6` | Sort menu |
| `F7` | Nice - (increase priority) |
| `F8` | Nice + (decrease priority) |
| `F9` | Kill menu |
| `F10` | Quit |
| `Arrow Keys` | Navigate |
| `Space` | Tag process |
| `U` | Untag all |
| `u` | Filter by user |
| `k` | Kill menu |
| `+` | Expand tree |
| `-` | Collapse tree |
| `*` | Toggle tree |
| `Ctrl + L` | Refresh |
| `Ctrl + F` | Search forward |
| `/` | Search |
| `\` | Incremental search |
| `q` | Quit |

### Process States Quick Reference
| Symbol | State | Description |
|--------|-------|-------------|
| R | Running | Active process |
| S | Sleeping | Waiting for event |
| D | Uninterruptible | I/O wait |
| Z | Zombie | Terminated |
| T | Stopped | By signal |

### Color Coding
| Color | Meaning |
|-------|---------|
| Green | Normal processes |
| Blue | Low priority (nice) |
| Red | Kernel threads |
| Yellow | IRQ |
| Magenta | Soft IRQ |
| Gray | I/O wait |

### Status Bar Indicators
| Indicator | Description |
|-----------|-------------|
| Tasks | Total/running/sleeping/stopped/zombie |
| Load | 1/5/15 minute load average |
| Uptime | System uptime |
| Mem | Memory used/total |
| Swp | Swap used/total |

### Common Commands
| Task | Command |
|------|---------|
| Install htop | `apt install htop` |
| Run htop | `htop` |
| Sort by CPU | `htop -s PERCENT_CPU` |
| Filter user | `htop -u username` |
| Tree view | `htop -t` |
| Batch output | `htop -b -n 3` |
| Custom config | `htop -c /path/config` |

---

*Last Updated: January 2026*
*Generated for htop 3.2.x*
