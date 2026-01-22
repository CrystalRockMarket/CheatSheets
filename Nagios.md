# Nagios Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Configuration Basics](#3-configuration-basics)
4. [Nagios Core Commands](#4-nagios-core-commands)
5. [Service Monitoring](#5-service-monitoring)
6. [Host Monitoring](#6-host-monitoring)
7. [Notifications](#7-notifications)
8. [Plugins](#8-plagins)
9. [NRPE (Nagios Remote Plugin Executor)](#9-nrpe-nagios-remote-plugin-executor)
10. [NSCA (Nagios Service Check Acceptor)](#10-nsca-nagios-service-check-acceptor)
11. [Web Interface](#11-web-interface)
12. [Troubleshooting](#12-troubleshooting)
13. [Best Practices](#13-best-practices)
14. [Quick Reference](#14-quick-reference)

---

## 1. Introduction

### What is Nagios?
Nagios is an open-source monitoring system that monitors hosts and services, alerting administrators when things go wrong and when they recover.

### Key Components
| Component | Description |
|-----------|-------------|
| `nagios` | Core monitoring daemon |
| `nagios.cfg` | Main configuration file |
| `nagios.log` | Main log file |
| `cgi.cfg` | Web interface configuration |
| `objects.*` | Object definition files |

### Directory Structure
```bash
/usr/local/nagios/
├── etc/                    # Configuration files
│   ├── nagios.cfg         # Main config
│   ├── cgi.cfg            # CGI config
│   ├── resource.cfg       # Resource macros
│   └── objects/           # Object definitions
│       ├── commands.cfg   # Command definitions
│       ├── contacts.cfg   # Contact definitions
│       ├── timeperiods.cfg # Time periods
│       ├── templates.cfg  # Object templates
│       ├── hosts.cfg      # Host definitions
│       └── services.cfg   # Service definitions
├── bin/                    # Executables
│   ├── nagios             # Core daemon
│   ├── nagios.cfg         # Config validator
│   └── nagiostats         # Statistics tool
├── sbin/                   # CGI scripts
├── libexec/                # Plugins
├── var/                    # Runtime data
│   ├── nagios.log         # Main log
│   ├── retention.dat      # State retention
│   └── objects.cache      # Compiled objects
└── share/                  # Web interface
    └── nagiosxi/          # Nagios XI (commercial)
```

---

## 2. Installation

### Ubuntu/Debian Installation
```bash
# Install from repository
sudo apt update
sudo apt install nagios4 nagios-plugins

# Or install from source
wget https://github.com/NagiosEnterprises/nagioscore/releases/download/nagios-4.4.14/nagios-4.4.14.tar.gz
tar xzf nagios-4.4.14.tar.gz
cd nagios-4.4.14
./configure --with-nagios-user=nagios --with-nagios-group=nagios
make all
sudo make install
sudo make install-init
sudo make install-commandmode
sudo make install-config
```

### RHEL/CentOS/Fedora Installation
```bash
# Install EPEL repository
sudo dnf install epel-release
sudo dnf install nagios nagios-plugins-all

# Start services
sudo systemctl enable nagios
sudo systemctl start nagios
```

### Verify Installation
```bash
# Check Nagios version
nagios --version

# Check configuration
nagios -v /usr/local/nagios/etc/nagios.cfg

# Check service status
systemctl status nagios
```

### Post-Installation Steps
```bash
# Set proper permissions
sudo usermod -a -G nagios www-data
sudo chown -R nagios:nagios /usr/local/nagios/var/

# Restart Nagios
sudo systemctl restart nagios

# Access web interface
# http://your-server/nagios
# Default credentials: nagiosadmin / nagiosadmin
```

---

## 3. Configuration Basics

### Main Configuration File (nagios.cfg)
```bash
# /usr/local/nagios/etc/nagios.cfg

# Log file
log_file=/usr/local/nagios/var/nagios.log

# Object configuration directory
cfg_dir=/usr/local/nagios/etc/objects

# Resource file (for sensitive macros)
resource_file=/usr/local/nagios/etc/resource.cfg

# Status file
status_file=/usr/local/nagios/var/status.dat

# Retention file
retention_file=/usr/local/nagios/var/retention.dat

# Comment file
comment_file=/usr/local/nagios/var/comments.dat

# Downtime file
downtime_file=/usr/local/nagios/var/downtime.dat

# Temp file
temp_file=/usr/local/nagios/var/nagios.tmp

# Lock file
lock_file=/usr/local/nagios/var/nagios.lock

# Log rotation
log_archive_path=/usr/local/nagios/var/archives

# User and group
nagios_user=nagios
nagios_group=nagios

# External command check
check_external_commands=1
command_check_interval=-1
command_file=/usr/local/nagios/var/rw/nagios.cmd

# Performance data
process_performance_data=1
host_perfdata_file=/usr/local/nagios/var/host-perfdata
service_perfdata_file=/usr/local/nagios/var/service-perfdata
perfdata_timeout=5

# Notification settings
notification_timeout=30
notification_warning_buffer=5
notification_critical_buffer=5

# Intervals
check_interval=5
retry_interval=1
max_check_attempts=3
check_period=24x7
notification_interval=30

# Flapping detection
enable_flap_detection=1
low_service_flap_threshold=20.0
high_service_flap_threshold=30.0
low_host_flap_threshold=20.0
high_host_flap_threshold=30.0

# State retention
retain_state_information=1
use_retained_program_state=1
use_retained_scheduling_info=1
```

### Object Configuration Files

#### Commands Definition (commands.cfg)
```bash
# /usr/local/nagios/etc/objects/commands.cfg

# Host check command
define command {
    command_name    check-host-alive
    command_line    /usr/local/nagios/libexec/check_ping -H $HOSTADDRESS$ -w 3000.0,100% -c 5000.0,100% -p 1
}

# Service check commands
define command {
    command_name    check-local-disk
    command_line    /usr/local/nagios/libexec/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$
}

define command {
    command_name    check-local-load
    command_line    /usr/local/nagios/libexec/check_load -w $ARG1$ -c $ARG2$
}

define command {
    command_name    check_local_procs
    command_line    /usr/local/nagios/libexec/check_procs -w $ARG1$ -c $ARG2$ -s $ARG3$
}

# Notification commands
define command {
    command_name    notify-host-by-email
    command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$" | /usr/bin/mail -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$
}

define command {
    command_name    notify-service-by-email
    command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$" | /usr/bin/mail -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
}

# NRPE check command
define command {
    command_name    check_nrpe
    command_line    /usr/local/nagios/libexec/check_nrpe -H $HOSTADDRESS$ -c $ARG1$ -a $ARG2$
}

# HTTP check command
define command {
    command_name    check_http
    command_line    /usr/local/nagios/libexec/check_http -H $HOSTADDRESS$ -u $ARG1$ -w $ARG2$ -c $ARG3$
}

# PING check command
define command {
    command_name    check_ping
    command_line    /usr/local/nagios/libexec/check_ping -H $HOSTADDRESS$ -w $ARG1$ -c $ARG2$ -p 5
}

# SSH check command
define command {
    command_name    check_ssh
    command_line    /usr/local/nagios/libexec/check_ssh -H $HOSTADDRESS$
}
```

#### Time Periods Definition (timeperiods.cfg)
```bash
# /usr/local/nagios/etc/objects/timeperiods.cfg

define timeperiod {
    timeperiod_name 24x7
    alias           24 Hours A Day, 7 Days A Week
    sunday          00:00-24:00
    monday          00:00-24:00
    tuesday         00:00-24:00
    wednesday       00:00-24:00
    thursday        00:00-24:00
    friday          00:00-24:00
    saturday        00:00-24:00
}

define timeperiod {
    timeperiod_name workhours
    alias           Normal Work Hours
    monday          09:00-17:00
    tuesday         09:00-17:00
    wednesday       09:00-17:00
    thursday        09:00-17:00
    friday          09:00-17:00
}

define timeperiod {
    timeperiod_name nonworkhours
    alias           Non-Work Hours
    monday          00:00-09:00,17:00-24:00
    tuesday         00:00-09:00,17:00-24:00
    wednesday       00:00-09:00,17:00-24:00
    thursday        00:00-09:00,17:00-24:00
    friday          00:00-09:00,17:00-24:00
    saturday        00:00-24:00
    sunday          00:00-24:00
}

define timeperiod {
    timeperiod_name only_monday
    alias           Only Mondays
    monday          00:00-24:00
}

define timeperiod {
    timeperiod_name never
    alias           Never
}
```

#### Contacts Definition (contacts.cfg)
```bash
# /usr/local/nagios/etc/objects/contacts.cfg

define contact {
    contact_name                    nagiosadmin
    alias                           Nagios Admin
    email                           admin@example.com
    host_notification_period        24x7
    service_notification_period     24x7
    host_notification_commands      notify-host-by-email
    service_notification_commands   notify-service-by-email
    host_notification_options       d,u,r,f
    service_notification_options    w,u,c,r,f
    can_submit_commands             1
}

define contact {
    contact_name                    ops-team
    alias                           Operations Team
    email                           ops@example.com
    host_notification_period        workhours
    service_notification_period     workhours
    host_notification_commands      notify-host-by-email
    service_notification_commands   notify-service-by-email
    service_notification_options    w,u,c,r,f
}

define contactgroup {
    contactgroup_name               admins
    alias                           Nagios Administrators
    members                         nagiosadmin
}

define contactgroup {
    contactgroup_name               ops
    alias                           Operations Team
    members                         ops-team
}
```

---

## 4. Nagios Core Commands

### Service Management
```bash
# Start Nagios
sudo systemctl start nagios

# Stop Nagios
sudo systemctl stop nagios

# Restart Nagios (graceful)
sudo systemctl restart nagios

# Reload configuration (without restart)
sudo systemctl reload nagios

# Check Nagios status
systemctl status nagios

# Check configuration validity
nagios -v /usr/local/nagios/etc/nagios.cfg

# View compiled objects
nagios -s /usr/local/nagios/etc/nagios.cfg

# Show object list
nagios -o /usr/local/nagios/etc/nagios.cfg
```

### Log Management
```bash
# View main log
tail -f /usr/local/nagios/var/nagios.log

# View alerts only
grep "ALERT" /usr/local/nagios/var/nagios.log

# View notifications only
grep "NOTIFICATION" /usr/local/nagios/var/nagios.log

# View external commands
grep "EXTERNAL COMMAND" /usr/local/nagios/var/nagios.log

# View errors only
grep "ERROR" /usr/local/nagios/var/nagios.log

# View service checks
grep "SERVICE CHECK" /usr/local/nagios/var/nagios.log

# View host checks
grep "HOST CHECK" /usr/local/nagios/var/nagios.log

# View flapping events
grep "FLAPPING" /usr/local/nagios/var/nagios.log
```

### Statistics
```bash
# Show statistics
nagiostats --datum=/usr/local/nagios/var/status.dat

# JSON output
nagiostats --datum=/usr/local/nagios/var/status.dat --json

# Show specific stats
nagiostats --datum=/usr/local/nagios/var/status.dat --data=METRICS

# Available metrics
nagiostats --datum=/usr/local/nagios/var/status.dat --list
```

---

## 5. Service Monitoring

### Basic Service Definition
```bash
# /usr/local/nagios/etc/objects/services.cfg

define service {
    name                            generic-service
    active_checks_enabled           1
    passive_checks_enabled          0
    parallelize_check               1
    obsess_over_service             1
    check_freshness                 0
    notifications_enabled           1
    event_handler_enabled           1
    flap_detection_enabled          1
    process_perf_data               1
    retain_status_information       1
    retain_nonstatus_information    1
    is_volatile                     0
    check_period                    24x7
    normal_check_interval           5
    retry_check_interval            1
    max_check_attempts              3
    notification_interval           30
    notification_period             24x7
    notification_options            w,u,c,r
    contact_groups                  admins
    register                        0
}

# Specific service definitions
define service {
    use                             generic-service
    host_name                       web-server
    service_description             HTTP
    check_command                   check_http
    check_interval                  5
    retry_interval                  1
    max_check_attempts              3
}

define service {
    use                             generic-service
    host_name                       web-server
    service_description             PING
    check_command                   check_ping!100.0,20%!500.0,60%
    check_interval                  5
    retry_interval                  1
    max_check_attempts              3
}

define service {
    use                             generic-service
    host_name                       database-server
    service_description             MySQL
    check_command                   check_mysql!nagios!password!localhost
    check_interval                  5
    retry_interval                  1
    max_check_attempts              3
}
```

### Service Templates
```bash
define service {
    name                            http-service
    use                             generic-service
    service_description             HTTP
    check_command                   check_http
    check_interval                  5
    notification_options            w,c,r
    register                        0
}

define service {
    name                            ping-service
    use                             generic-service
    service_description             PING
    check_command                   check_ping!100.0,20%!500.0,60%
    check_interval                  5
    notification_options            r
    register                        0
}

define service {
    name                            ssh-service
    use                             generic-service
    service_description             SSH
    check_command                   check_ssh
    check_interval                  5
    notification_options            w,c,r
    register                        0
}

define service {
    name                            disk-service
    use                             generic-service
    service_description             Disk Usage
    check_command                   check_local_disk!20%!10%!/
    check_interval                  10
    notification_options            w,c
    register                        0
}

define service {
    name                            load-service
    use                             generic-service
    service_description             CPU Load
    check_command                   check_local_load!5.0,4.0,3.0!10.0,6.0,4.0
    check_interval                  5
    notification_options            w,c
    register                        0
}
```

### Service Dependencies
```bash
define servicedependency {
    dependent_service_description   HTTP
    host_name                       web-server
    service_description             TCP_Ports
    host_name                       web-server
    execution_failure_criteria      u,c,w
    notification_failure_criteria   u,c,w
}

define servicedependency {
    dependent_service_description   MySQL
    host_name                       database-server
    service_description             TCP_Ports
    host_name                       database-server
    execution_failure_criteria      u,c,w
    notification_failure_criteria   u,c,w
}
```

### Service Escalations
```bash
define serviceescalation {
    host_name                       web-server
    service_description             HTTP
    first_notification              3
    last_notification               5
    notification_interval           15
    contact_groups                  admins,ops
}

define serviceescalation {
    host_name                       database-server
    service_description             MySQL
    first_notification              1
    last_notification               3
    notification_interval           5
    contact_groups                  admins,dba
}
```

---

## 6. Host Monitoring

### Basic Host Definition
```bash
# /usr/local/nagios/etc/objects/hosts.cfg

define host {
    host_name                       web-server
    alias                           Web Server 1
    address                         192.168.1.10
    max_check_attempts              3
    check_period                    24x7
    notification_interval           30
    notification_period             24x7
    notification_options            d,u,r
    contact_groups                  admins
    register                        1
}

define host {
    host_name                       database-server
    alias                           Database Server
    address                         192.168.1.20
    max_check_attempts              5
    check_period                    24x7
    notification_interval           60
    notification_period             workhours
    notification_options            d,u,r
    contact_groups                  admins,dba
    notes                           "Primary PostgreSQL server"
    icon_image                      database.png
    statusmap_image                 database.gd2
}

define host {
    host_name                       mail-server
    alias                           Mail Server
    address                         192.168.1.30
    max_check_attempts              3
    check_period                    24x7
    notification_interval           30
    notification_period             24x7
    notification_options            d,u,r
    contact_groups                  admins
    parents                         gateway
}
```

### Host Templates
```bash
define host {
    name                            generic-host
    active_checks_enabled           1
    passive_checks_enabled          0
    obsess_over_host                1
    check_freshness                 0
    notifications_enabled           1
    event_handler_enabled           1
    flap_detection_enabled          1
    process_perf_data               1
    retain_status_information       1
    retain_nonstatus_information    1
    max_check_attempts              3
    check_interval                  5
    retry_interval                  1
    notification_interval           30
    notification_period             24x7
    notification_options            d,u,r,f
    contacts                        nagiosadmin
    register                        0
}

define host {
    name                            linux-server
    use                             generic-host
    check_command                   check-host-alive
    check_interval                  5
    retry_interval                  1
    icon_image                      linux40.png
    statusmap_image                 linux40.gd2
    register                        0
}

define host {
    name                            windows-server
    use                             generic-host
    check_command                   check-host-alive
    check_interval                  5
    retry_interval                  1
    icon_image                      win40.png
    statusmap_image                 win40.gd2
    register                        0
}

define host {
    name                            router
    use                             generic-host
    check_command                   check-host-alive
    check_interval                  10
    retry_interval                  2
    icon_image                       router40.png
    statusmap_image                 router40.gd2
    notes                           "Network Router"
    register                        0
}
```

### Host Groups
```bash
define hostgroup {
    hostgroup_name                  web-servers
    alias                           Web Servers
    members                         web-server1,web-server2,web-server3
}

define hostgroup {
    hostgroup_name                  database-servers
    alias                           Database Servers
    members                         db-primary,db-replica1,db-replica2
}

define hostgroup {
    hostgroup_name                  mail-servers
    alias                           Mail Servers
    members                         mail-server
}

define hostgroup {
    hostgroup_name                  linux-servers
    alias                           Linux Servers
    members                         web-server,database-server,mail-server
}
```

### Host Dependencies
```bash
define hostdependency {
    dependent_host_name             web-server
    host_name                       database-server
    execution_failure_criteria      u,d
    notification_failure_criteria   u,d
}

define hostdependency {
    dependent_host_name             mail-server
    host_name                       gateway
    execution_failure_criteria      u,d
    notification_failure_criteria   u,d
}
```

### Host Escalations
```bash
define hostescalation {
    host_name                       web-server
    first_notification              3
    last_notification               5
    notification_interval           60
    contact_groups                  admins,ops
}
```

---

## 7. Notifications

### Notification Commands
```bash
# Email notification (already in commands.cfg)
define command {
    command_name    notify-host-by-email
    command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$" | /usr/bin/mail -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$
}

# SMS notification (using Twilio)
define command {
    command_name    notify-host-by-sms
    command_line    /usr/local/nagios/scripts/sms_notification.py "$NOTIFICATIONTYPE$" "$HOSTNAME$" "$HOSTSTATE$" "$CONTACTPAGER$"
}

# Slack notification
define command {
    command_name    notify-service-by-slack
    command_line    /usr/local/nagios/scripts/slack_notification.py "$NOTIFICATIONTYPE$" "$HOSTNAME$" "$SERVICEDESC$" "$SERVICESTATE$" "$SERVICEOUTPUT$" "$CONTACTEMAIL$"
}

# PagerDuty notification
define command {
    command_name    notify-by-pagerduty
    command_line    /usr/local/nagios/scripts/pagerduty.py "$HOSTNAME$" "$SERVICEDESC$" "$SERVICESTATE$" "$CONTACTPAGER$"
}
```

### Notification Configuration
```bash
# Extended host information for notifications
define hostextinfo {
    host_name                       web-server
    notes                           Production web server running Apache
    notes_url                       http://wiki.example.com/web-server
    icon_image                      web.png
    icon_image_alt                  Web Server
    vrml_image                      web.png
    statusmap_image                 web.gd2
    2d_coords                       100,200
    3d_coords                       100.0,200.0,0.0
}

define serviceextinfo {
    host_name                       web-server
    service_description             HTTP
    notes                           Main production website
    notes_url                       http://wiki.example.com/http-service
    icon_image                      http.png
    icon_image_alt                  HTTP Service
}
```

---

## 8. Plugins

### Standard Plugin Commands
```bash
# Check CPU Load
check_load -w 5.0,4.0,3.0 -c 10.0,6.0,4.0

# Check Disk Usage
check_disk -w 20% -c 10% -p /

# Check Memory
check_mem -w 80 -c 90 -C

# Check Processes
check_procs -w 150 -c 200 -s RSZDT

# Check HTTP
check_http -H www.example.com -w 5 -c 10

# Check SSH
check_ssh -H www.example.com

# Check PING
check_ping -H www.example.com -w 100,10% -c 200,20%

# Check MySQL
check_mysql -H localhost -u nagios -p password

# Check PostgreSQL
check_pgsql -H localhost -d postgres -u nagios -p password

# Check TCP Ports
check_tcp -H www.example.com -p 443

# Check DNS
check_dns -H www.example.com -s 192.168.1.10

# Check SMTP
check_smtp -H mail.example.com

# Check IMAP
check_imap -H mail.example.com

# Check POP3
check_pop -H mail.example.com

# Check LDAP
check_ldap -H ldap.example.com -b "dc=example,dc=com"

# Check Disk I/O
check_diskio -w 100 -c 200 -d /dev/sda

# Check Network Interface
check_interface -w 80 -c 90 -n eth0

# Check Users
check_users -w 5 -c 10
```

### Custom Plugin Example
```bash
#!/bin/bash
# /usr/local/nagios/libexec/check_example

# Nagios plugin example
PROGNAME=$(basename $0)
PROGPATH=$(echo $0 | sed -e 's,[\\/][^\\/][^\\/]*$,,')
REVISION="1.0"

. $PROGPATH/utils.sh

print_usage() {
    echo "Usage: $PROGNAME -w warning -c critical"
    echo "Usage: $PROGNAME --help"
    print_help
}

print_help() {
    echo ""
    echo "$PROGNAME - Check example service"
    echo ""
    echo "This plugin checks example service status"
    echo ""
    print_usage
    echo ""
    echo "Options:"
    echo "  -w, --warning)"
    echo "     Warning threshold"
    echo "  -c, --critical)"
    echo "     Critical threshold"
    echo "  -h, --help)"
    echo "     Print this help message"
    echo ""
    echo "Example:"
    echo "  $PROGNAME -w 100 -c 200"
    echo ""
}

while [ -n "$1" ]; do
    case "$1" in
        -h | --help)
            print_help
            exit $STATE_OK
            ;;
        -w | --warning)
            shift
            WARNING=$1
            ;;
        -c | --critical)
            shift
            CRITICAL=$1
            ;;
        *)
            echo "Unknown argument: $1"
            print_usage
            exit $STATE_UNKNOWN
            ;;
    esac
    shift
done

# Check if thresholds are set
if [ -z "$WARNING" ] || [ -z "$CRITICAL" ]; then
    echo "Warning and Critical thresholds required"
    exit $STATE_UNKNOWN
fi

# Perform check
VALUE=$(some_command)

if [ $VALUE -ge $CRITICAL ]; then
    echo "CRITICAL: Value is $VALUE"
    exit $STATE_CRITICAL
elif [ $VALUE -ge $WARNING ]; then
    echo "WARNING: Value is $VALUE"
    exit $STATE_WARNING
else
    echo "OK: Value is $VALUE"
    exit $STATE_OK
fi
```

---

## 9. NRPE (Nagios Remote Plugin Executor)

### NRPE Installation on Client
```bash
# Ubuntu/Debian
sudo apt install nagios-nrpe-server nagios-plugins

# RHEL/CentOS
sudo dnf install nrpe nagios-plugins-all

# Configure NRPE
sudo nano /etc/nagios/nrpe.cfg

# Add Nagios server IP
allowed_hosts=127.0.0.1,192.168.1.100

# Start NRPE
sudo systemctl enable nrpe
sudo systemctl start nrpe
```

### NRPE Configuration
```bash
# /etc/nagios/nrpe.cfg

# Server configuration
server_port=5666
server_address=0.0.0.0
allowed_hosts=127.0.0.1,192.168.1.100

# Connection settings
nrpe_user=nagios
nrpe_group=nagios
pid_file=/var/run/nrpe/nrpe.pid
log_facility=daemon
debug=0
command_timeout=60
connection_timeout=300

# Include additional commands
include_dir=/etc/nrpe.d/

# Command definitions
command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
command[check_load]=/usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,25,20
command[check_disk]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/sda1
command[check_mem]=/usr/local/nagios/libexec/check_mem -w 80 -c 90 -C
command[check_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200
command[check_cpu_stats]=/usr/local/nagios/libexec/check_cpu_stats
command[check_diskio]=/usr/local/nagios/libexec/check_diskio -w 100 -c 200
command[check_swap]=/usr/local/nagios/libexec/check_swap -w 20% -c 10%
```

### NRPE Commands from Nagios Server
```bash
# Add NRPE check command
define command {
    command_name    check_nrpe
    command_line    /usr/local/nagios/libexec/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}

# Add NRPE service checks
define service {
    use                             generic-service
    host_name                       remote-server
    service_description             CPU Load
    check_command                   check_nrpe!check_load
}

define service {
    use                             generic-service
    host_name                       remote-server
    service_description             Disk Usage
    check_command                   check_nrpe!check_disk
}

define service {
    use                             generic-service
    host_name                       remote-server
    service_description             Memory
    check_command                   check_nrpe!check_mem
}

define service {
    use                             generic-service
    host_name                       remote-server
    service_description             Processes
    check_command                   check_nrpe!check_procs
}
```

### NRPE with Arguments
```bash
# Client side (nrpe.cfg)
command[check_disk_custom]=/usr/local/nagios/libexec/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$

# Nagios server side
define command {
    command_name    check_nrpe_args
    command_line    /usr/local/nagios/libexec/check_nrpe -H $HOSTADDRESS$ -c check_disk_custom -a $ARG1$ $ARG2$ $ARG3$
}

define service {
    use                             generic-service
    host_name                       remote-server
    service_description             Disk /var
    check_command                   check_nrpe_args!20%!10%!/var
}
```

---

## 10. NSCA (Nagios Service Check Acceptor)

### NSCA Server Configuration
```bash
# Install NSCA server
sudo apt install nsca

# Configure NSCA
sudo nano /etc/nsca.cfg

# Configuration options
server_port=5667
server_address=0.0.0.0
nsca_user=nagios
nsca_group=nagios
password=secret_password
decryption_method=1

# Start NSCA
sudo systemctl enable nsca
sudo systemctl start nsca
```

### NSCA Client Configuration
```bash
# Install NSCA client
sudo apt install nsca-client

# Configure NSCA client
sudo nano /etc/nsca_client.cfg

# Configuration options
server_port=5667
server_address=192.168.1.100
password=secret_password
encryption_method=1
hostname=client-server

# Send passive check result
echo -e "example.com\tHTTP\t0\tOK: HTTP is up" | /usr/sbin/send_nsca -H 192.168.1.100 -c /etc/send_nsca.cfg
```

### Passive Check Configuration
```bash
# Define passive service
define service {
    use                             generic-service
    host_name                       remote-server
    service_description             Passive HTTP Check
    check_command                   check_dummy!0
    active_checks_enabled           0
    passive_checks_enabled          1
    check_period                    24x7
    notification_interval           60
    notification_options            w,u,c,r
}

# Define command for sending passive results
define command {
    command_name    process-service-check-result
    command_line    /usr/bin/printf "%b\t%s\t%s\t%s\n" "$HOSTNAME$" "$SERVICEDESC$" "$SERVICESTATE$" "$SERVICEOUTPUT$" | /usr/sbin/send_nsca -H 192.168.1.100 -c /etc/send_nsca.cfg
}
```

---

## 11. Web Interface

### Access and Authentication
```bash
# Default URL
http://your-nagios-server/nagios

# Default credentials
Username: nagiosadmin
Password: nagiosadmin

# Change password
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

### Web Interface Navigation
| Tab | Description |
|-----|-------------|
| Home | Dashboard overview |
| Tactical Overview | Current status summary |
| Services | Service status grid |
| Hosts | Host status grid |
| Hostgroups | Host group views |
| Servicegroups | Service group views |
| Comments | View/add comments |
| Downtime | Schedule downtime |
| Problems | Current problems |
| Reports | Historical reports |
| Trends | Performance trends |
| Availability | Service availability |
| History | Event history |
| Notifications | Notification log |
| Process Info | Nagios process info |
| Configuration | Configuration viewer |
| System | System settings |

### CGI Configuration (cgi.cfg)
```bash
# /usr/local/nagios/etc/cgi.cfg

# Main configuration file
main_config_file=/usr/local/nagios/etc/nagios.cfg

# Authentication
use_authentication=1
default_user_name=guest

# Authorized for full command access
authorized_for_system_information=nagiosadmin
authorized_for_configuration_information=nagiosadmin
authorized_for_system_commands=nagiosadmin
authorized_for_all_services=nagiosadmin
authorized_for_all_hosts=nagiosadmin

# Read-only users
authorized_for_read_only=nagiosadmin,readonlyuser

# URL styles
show_context_help=1
add_notice_duration=15
action_url_target=_blank
notes_url_target=_blank
enable_splunk_integration=0
```

### Status Visualization
```bash
# View host status
http://your-server/nagios/cgi-bin/status.cgi?hostgroup=all&style=hostdetail

# View service status
http://your-server/nagios/cgi-bin/status.cgi?host=all&servicestatustypes=28

# View problems
http://your-server/nagios/cgi-bin/status.cgi?host=all&type=detail&servicestatustypes=28

# View history
http://your-server/nagios/cgi-bin/history.cgi?host=your-server

# View notifications
http://your-server/nagios/cgi-bin/notification.cgi
```

---

## 12. Troubleshooting

### Common Issues and Solutions

#### Nagios Won't Start
```bash
# Check configuration errors
nagios -v /usr/local/nagios/etc/nagios.cfg

# Check log for errors
tail -f /usr/local/nagios/var/nagios.log | grep ERROR

# Check permissions
ls -la /usr/local/nagios/var/
ls -la /usr/local/nagios/etc/

# Check port availability
netstat -tulpn | grep 5666

# Check SELinux/AppArmor
getenforce
aa-status
```

#### Services Not Checking
```bash
# Check external commands
tail -f /usr/local/nagios/var/nagios.log | grep "EXTERNAL COMMAND"

# Check command file permissions
ls -la /usr/local/nagios/var/rw/nagios.cmd

# Verify command definitions
cat /usr/local/nagios/etc/objects/commands.cfg | grep command_name

# Check plugin permissions
ls -la /usr/local/nagios/libexec/check_*
```

#### No Notifications
```bash
# Check notification settings
grep "notification" /usr/local/nagios/etc/nagios.cfg

# Verify contact configuration
cat /usr/local/nagios/etc/objects/contacts.cfg

# Test email notifications
echo "Test message" | mail -s "Nagios Test" admin@example.com

# Check notification period
cat /usr/local/nagios/etc/objects/timeperiods.cfg

# View notification log
tail -f /usr/local/nagios/var/nagios.log | grep NOTIFICATION
```

#### High Latency
```bash
# Check performance metrics
nagiostats --datum=/usr/local/nagios/var/status.dat

# View service check times
tail -f /usr/local/nagios/var/nagios.log | grep "SERVICE CHECK"

# Check system load
top
htop

# Increase check workers
max_parallel_service_checks=100

# Optimize check intervals
# Reduce check frequency for less critical services
```

#### NRPE Connection Issues
```bash
# Test NRPE connectivity
/usr/local/nagios/libexec/check_nrpe -H remote-server

# Check NRPE on remote server
ps aux | grep nrpe
netstat -tulpn | grep 5666

# Verify firewall rules
iptables -L -n | grep 5666

# Check NRPE configuration
cat /etc/nagios/nrpe.cfg | grep allowed_hosts

# Test NRPE commands manually
/usr/local/nagios/libexec/check_nrpe -H remote-server -c check_load
```

### Debug Commands
```bash
# Verbose logging
tail -f /usr/local/nagios/var/nagios.log

# Show all events
tail -f /usr/local/nagios/var/nagios.log | grep -v INFO

# Check service check output
tail -f /usr/local/nagios/var/nagios.log | grep "SERVICE CHECK OUTPUT"

# Check host check output
tail -f /usr/local/nagios/var/nagios.log | grep "HOST CHECK OUTPUT"

# View performance data
tail -f /usr/local/nagios/var/nagios.log | grep "PERFDATA"

# Monitor external commands
tail -f /usr/local/nagios/var/nagios.log | grep "EXTERNAL COMMAND"

# Check for flapping
tail -f /usr/local/nagios/var/nagios.log | grep FLAPPING
```

### Log Analysis
```bash
# View recent alerts
grep "ALERT" /usr/local/nagios/var/nagios.log | tail -50

# View service state changes
grep "SERVICE STATE" /usr/local/nagios/var/nagios.log | tail -50

# View host state changes
grep "HOST STATE" /usr/local/nagios/var/nagios.log | tail -50

# View downtime
grep "DOWNTIME" /usr/local/nagios/var/nagios.log

# View acknowledgments
grep "ACKNOWLEDGEMENT" /usr/local/nagios/var/nagios.log

# Search for specific host/service
grep "web-server" /usr/local/nagios/var/nagios.log

# Search for errors
grep "ERROR" /usr/local/nagios/var/nagios.log

# Check for timeouts
grep "TIMEOUT" /usr/local/nagios/var/nagios.log
```

---

## 13. Best Practices

### Configuration Management
```bash
# Use templates for consistency
# Define reusable templates for hosts and services
# Use hostgroups and servicegroups for organization
# Keep configuration modular with cfg_dir

# Use version control
cd /usr/local/nagios/etc
git init
git add .
git commit -m "Initial Nagios configuration"
```

### Monitoring Strategy
```bash
# Follow the 3-2-1 rule
# 3 different check methods for critical services
# 2 different check locations
# 1 out-of-band monitoring method

# Example HTTP check redundancy
# 1. check_http - Basic HTTP response
# 2. check_tcp - TCP port check
# 3. check_http_content - Content verification
```

### Performance Optimization
```bash
# Use active checks sparingly
# Use passive checks for high-volume checks
# Enable check freshness for critical services
# Use host and service dependencies wisely
# Reduce check intervals for non-critical services
# Use commodity checks for bulk monitoring

# Example optimization
# For 100+ hosts, reduce check_interval to 10 minutes
# Use service dependency to skip checks when parent is down
```

### Alert Management
```bash
# Implement alert filtering
# Use appropriate notification periods
# Implement escalation paths
# Set up acknowledgments workflow
# Schedule regular maintenance windows

# Reduce alert fatigue
# Use flapping detection
# Set appropriate thresholds
# Group related alerts
# Use service dependencies to suppress cascading alerts
```

### Security Hardening
```bash
# Secure the web interface
# Use SSL/TLS for web access
# Implement strong authentication
# Restrict CGI access
# Use IP allowlisting

# Secure NRPE
# Use encryption for NRPE
# Restrict allowed hosts
# Use command argument validation

# Secure monitoring data
# Encrypt sensitive macros
# Use separate user permissions
# Audit access logs regularly
```

---

## 14. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| Validate config | `nagios -v /usr/local/nagios/etc/nagios.cfg` |
| Start Nagios | `systemctl start nagios` |
| Stop Nagios | `systemctl stop nagios` |
| Restart Nagios | `systemctl restart nagios` |
| Reload config | `systemctl reload nagios` |
| View logs | `tail -f /usr/local/nagios/var/nagios.log` |
| Check NRPE | `/usr/local/nagios/libexec/check_nrpe -H host` |

### State Types
| State | Description | Exit Code |
|-------|-------------|-----------|
| OK | Service is working | 0 |
| WARNING | Service is degraded | 1 |
| CRITICAL | Service is down | 2 |
| UNKNOWN | Check failed | 3 |

### Notification Types
| Type | Description |
|------|-------------|
| PROBLEM | Service/host went down |
| RECOVERY | Service/host recovered |
| ACKNOWLEDGEMENT | Alert acknowledged |
| FLAPPINGSTART | Started flapping |
| FLAPPINGSTOP | Stopped flapping |
| DOWNTIMESTART | Scheduled downtime started |
| DOWNTIMESTOP | Scheduled downtime ended |

### Macros Reference
| Macro | Description |
|-------|-------------|
| `$HOSTNAME$` | Host name |
| `$HOSTADDRESS$` | Host IP address |
| `$HOSTSTATE$` | Current host state |
| `$HOSTOUTPUT$` | Host check output |
| `$SERVICEDESC$` | Service description |
| `$SERVICESTATE$` | Current service state |
| `$SERVICEOUTPUT$` | Service check output |
| `$NOTIFICATIONTYPE$` | Type of notification |
| `$LONGDATETIME$` | Full date/time |
| `$CONTACTEMAIL$` | Contact email |
| `$CONTACTPAGER$` | Contact pager |

### Key Configuration Files
| File | Purpose |
|------|---------|
| `nagios.cfg` | Main configuration |
| `cgi.cfg` | Web interface config |
| `commands.cfg` | Command definitions |
| `contacts.cfg` | Contact definitions |
| `timeperiods.cfg` | Time period definitions |
| `templates.cfg` | Object templates |
| `hosts.cfg` | Host definitions |
| `services.cfg` | Service definitions |
| `hostgroups.cfg` | Host group definitions |
| `servicegroups.cfg` | Service group definitions |

### Plugin Exit Codes
| Exit Code | State | Meaning |
|-----------|-------|---------|
| 0 | OK | Everything is fine |
| 1 | WARNING | Warning threshold exceeded |
| 2 | CRITICAL | Critical threshold exceeded |
| 3 | UNKNOWN | Unable to determine status |

### Common Thresholds
| Resource | Warning | Critical |
|----------|---------|----------|
| CPU Load | 5.0 | 10.0 |
| Disk Usage | 20% | 10% |
| Memory Usage | 80% | 90% |
| PING RTT | 100ms | 500ms |
| HTTP Response | 5s | 10s |
| Process Count | 150 | 200 |

---

*Last Updated: January 2026*
*Generated for Nagios Core 4.4.x*
