# Zabbix Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Configuration Basics](#3-configuration-basics)
4. [Host Management](#4-host-management)
5. [Items](#5-items)
6. [Triggers](#6-triggers)
7. [Actions](#7-actions)
8. [Templates](#8-templates)
9. [User Management](#9-user-management)
10. [Proxy](#10-proxy)
11. [API](#11-api)
12. [Monitoring Patterns](#12-monitoring-patterns)
13. [Troubleshooting](#13-troubleshooting)
14. [Quick Reference](#14-quick-reference)

---

## 1. Introduction

### What is Zabbix?
Zabbix is an enterprise-grade open-source monitoring solution for networks and applications, offering real-time monitoring, alerting, and visualization.

### Key Features
| Feature | Description |
|---------|-------------|
| **Agent-based & Agentless** | Supports Zabbix agent, SNMP, IPMI, JMX |
| **Distributed Monitoring** | Proxy support for large deployments |
| **Auto-discovery** | Automatic network and agent discovery |
| **Scalable** | Monitors thousands of devices |
| **Customizable** | Flexible item and trigger definitions |
| **Visualization** | Built-in graphing and dashboards |
| **Alerting** | Multiple notification channels |
| **Reporting** | Built-in and custom reports |

### Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                      Zabbix Server                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  Poller/    │  │  Trapper/   │  │   Database          │ │
│  │ alerter     │  │  Escalator  │  │   (MySQL/PostgreSQL)│ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  SNMP/      │  │  Java/      │  │   Web Interface     │ │
│  │  IPMI       │  │  Agentless  │  │   (PHP)             │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
          │                    │
          │                    │ Proxy
          │                    ▼
    ┌─────────────┐     ┌────────────────┐
    │  Zabbix     │     │  Zabbix Proxy  │
    │  Agent      │     │  (Optional)    │
    └─────────────┘     └────────────────┘
          │                    │
          ▼                    ▼
    ┌─────────────┐     ┌──────────────┐
    │  Monitored  │     │  Monitored   │
    │  Host       │     │  Host        │
    └─────────────┘     └──────────────┘
```

### Core Concepts
| Concept | Description |
|---------|-------------|
| **Host** | Device to monitor |
| **Item** | Individual metric to collect |
| **Trigger** | Condition that evaluates item value |
| **Action** | Response when trigger fires |
| **Template** | Reusable set of items/triggers |
| **Host Group** | Logical grouping of hosts |
| **Proxy** | Distributed monitoring component |
| **User** | Zabbix interface user |

---

## 2. Installation

### Prerequisites
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install apache2 mysql-server php php-mysql libapache2-mod-php mysql-client php-mysql php-cli php-xml php-mbstring php-bcmath php-curl php-zip

# RHEL/CentOS
sudo dnf install httpd mariadb-server php php-mysqlnd php-cli php-xml php-mbstring php-bcmath php-curl php-zip
```

### Database Setup
```bash
# Create database and user
mysql -u root -p

CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;

# Import schema
zcat /usr/share/zabbix-sql-scripts/mysql/create.sql.gz | mysql -u zabbix -p zabbix
```

### Zabbix Server Installation
```bash
# Ubuntu/Debian
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update
sudo apt install zabbix-server-mysql zabbix-agent zabbix-frontend-php

# RHEL/CentOS
sudo dnf install zabbix-server-mysql zabbix-agent zabbix-web-mysql

# Configure PHP timezone
sudo sed -i 's/;date.timezone =/date.timezone = UTC/' /etc/php/8.1/apache2/php.ini

# Start services
sudo systemctl restart apache2
sudo systemctl enable zabbix-server zabbix-agent
sudo systemctl start zabbix-server
sudo systemctl start zabbix-agent
```

### Zabbix Frontend Configuration
```php
# /etc/zabbix/web/zabbix.conf.php
<?php
$DB['TYPE']     = 'MYSQL';
$DB['SERVER']   = 'localhost';
$DB['PORT']     = '3306';
$DB['DATABASE'] = 'zabbix';
$DB['USER']     = 'zabbix';
$DB['PASSWORD'] = 'password';

$ZBX_SERVER      = 'localhost';
$ZBX_SERVER_PORT = '10051';
$ZBX_SERVER_NAME = 'Zabbix Server';

$IMAGE_FORMAT_DEFAULT = IMAGE_FORMAT_PNG;
?>
```

### Docker Installation
```bash
# Run Zabbix stack
docker run -d \
  --name zabbix-server \
  -p 10051:10051 \
  -e DB_SERVER_HOST="mysql-server" \
  -e MYSQL_DATABASE="zabbix" \
  -e MYSQL_USER="zabbix" \
  -e MYSQL_PASSWORD="password" \
  zabbix/zabbix-server-mysql:latest

docker run -d \
  --name zabbix-web \
  -p 80:8080 \
  -e ZBX_SERVER_HOST="zabbix-server" \
  -e DB_SERVER_HOST="mysql-server" \
  -e MYSQL_DATABASE="zabbix" \
  -e MYSQL_USER="zabbix" \
  -e MYSQL_PASSWORD="password" \
  zabbix/zabbix-web-nginx-mysql:latest

docker run -d \
  --name zabbix-agent \
  -p 10050:10050 \
  -e ZBX_SERVER_HOST="zabbix-server" \
  zabbix/zabbix-agent:latest
```

### Verify Installation
```bash
# Check server status
systemctl status zabbix-server

# Check server logs
tail -f /var/log/zabbix/zabbix_server.log

# Access web interface
# http://your-server/zabbix
# Default credentials: Admin / zabbix

# Check API
curl http://your-server/zabbix/api_jsonrpc.php \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc": "2.0", "method": "apiinfo.version", "params": [], "id": 1}'
```

---

## 3. Configuration Basics

### Server Configuration
```bash
# /etc/zabbix/zabbix_server.conf

# Database connection
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=password
DBSocket=/var/run/mysqld/mysqld.sock
DBPort=3306

# Zabbix server
ListenPort=10051
ListenIP=0.0.0.0
SourceIP=0.0.0.0

# Performance
StartPollers=5
StartPollersUnreachable=1
StartTrappers=5
StartPingers=1
StartDiscoverers=0
StartHTTPPollers=1

# Cache
CacheSize=8M
HistoryCacheSize=16M
TrendCacheSize=4M
ValueCacheSize=8M

# Timeout
Timeout=30
TrapperTimeout=300
UnreachablePeriod=45
UnavailableDelay=60
UnreachableDelay=15

# Logging
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=100
DebugLevel=3

# Proxy
ProxyConfigFrequency=3600

# Alerts
StartAlerters=3
```

### Agent Configuration
```bash
# /etc/zabbix/zabbix_agentd.conf

# Server connection
Server=127.0.0.1,192.168.1.100
ServerActive=127.0.0.1
Hostname=localhost

# Logging
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=100
DebugLevel=3

# Performance
StartAgents=3
RefreshActiveChecks=120
BufferSize=100
BufferSend=5
MaxLinesPerSecond=100

# TLS
TLSAccept=unencrypted
TLSConnect=unencrypted

# User parameters
UserParameter=custom.cpu.util[*],/etc/zabbix/scripts/cpu_util.sh $1
UserParameter=custom.disk.io[*],/etc/zabbix/scripts/disk_io.sh $1

# Include custom configurations
Include=/etc/zabbix/zabbix_agentd.d/*.conf
```

### Frontend Configuration
```php
# /etc/zabbix/web/zabbix.conf.php
<?php
$DB['TYPE']             = 'MYSQL';
$DB['SERVER']           = 'localhost';
$DB['PORT']             = '0';
$DB['DATABASE']         = 'zabbix';
$DB['USER']             = 'zabbix';
$DB['PASSWORD']         = 'password';
$DB['SCHEMA']           = '';

$ZBX_SERVER             = 'localhost';
$ZBX_SERVER_PORT        = '10051';
$ZBX_SERVER_NAME        = 'Production Zabbix';

$IMAGE_FORMAT_DEFAULT   = IMAGE_FORMAT_PNG;

// Authentication
$ZBX_AUTH_TYPE          = 'basic';

// Session
$ZBX_SESSION_NAME       = 'ZBX_SESSION';
$ZBX_SESSION_REMEMBER   = 'zbx_remember';
$ZBX_SESSION_EXPIRY     = '0';

// Proxy
$PROXY_CONFIGURATION_FORCE = false;

// Maintenance
$MAINTENANCE_ERROR_IF_NO_DATA = false;
?>
```

---

## 4. Host Management

### Host Configuration
```json
{
  "host": "web-server-01",
  "name": "Web Server 01",
  "interfaces": [
    {
      "type": 1,
      "main": 1,
      "useip": 1,
      "ip": "192.168.1.10",
      "dns": "",
      "port": 10050
    }
  ],
  "groups": [
    {
      "groupid": 2
    }
  ],
  "templates": [
    {
      "templateid": 10001
    }
  ],
  "inventory_mode": 0,
  "inventory": {
    "os": "Ubuntu 22.04",
    "location": "US-East",
    "contact": "admin@example.com"
  },
  "macros": [
    {
      "macro": "{$SMTP_SERVER}",
      "value": "smtp.example.com"
    }
  ]
}
```

### Host Groups
```bash
# Create host group via API
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Web Servers"
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php" \
  -u "admin:zabbix"
```

### Host Discovery
```bash
# Network discovery rule
# Configure in: Configuration -> Discovery

# Discovery actions
# Configure in: Configuration -> Actions -> Discovery actions

# Discovery parameters
{
  "name": "Local Network Discovery",
  "iprange": "192.168.1.1-254",
  "delay": "3600",
  "proxy": 0,
  "checks": [
    {
      "type": 9,
      "key_": "system.hostname",
      "ports": "10050",
      "snmp_community": "",
      "snmpv3_contextname": "",
      "snmpv3_securityname": "",
      "snmpv3_securitylevel": 0,
      "snmpv3_authpassphrase": "",
      "snmpv3_privpassphrase": "",
      "snmp_oid": ""
    }
  ],
  "uniq": 0
}
```

### Host Import/Export
```xml
<?xml version="1.0" encoding="UTF-8"?>
<zabbix_export>
    <version> "6.4",
    <date>: "2023-01-01T00:00:00Z",
    <groups>
        <group>
            <name>Web Servers</name>
        </group>
    </groups>
    <hosts>
        <host>
            <host>web-01</host>
            <name>Web Server 01</name>
            <templates>
                <template>
                    <name>Linux by Zabbix agent</name>
                </template>
            </templates>
            <groups>
                <group>
                    <name>Web Servers</name>
                </group>
            </groups>
            <interfaces>
                <interface>
                    <ip>192.168.1.10</ip>
                    <dns>web-01.example.com</dns>
                    <useip>0</useip>
                    <port>10050</port>
                    <type>1</type>
                    <main>1</main>
                </interface>
            </interfaces>
        </host>
    </hosts>
</zabbix_export>
```

---

## 5. Items

### Item Types
| Type ID | Name | Description |
|---------|------|-------------|
| 0 | Zabbix agent | Agent-based checks |
| 1 | SNMPv1/v2c | SNMP checks |
| 2 | Zabbix trapper | Trapper items |
| 3 | Simple check | ICMP/TCP checks |
| 4 | SNMPv3 | SNMPv3 checks |
| 5 | External check | External scripts |
| 6 | ODBC database monitor | Database queries |
| 7 | IPMI agent | IPMI sensors |
| 10 | SSH agent | SSH commands |
| 11 | Telnet agent | Telnet commands |
| 12 | Calculated | Calculated items |
| 13 | JMX agent | Java JMX |
| 14 | SNMP trap | SNMP traps |
| 16 | Prometheus | Prometheus data |
| 18 | HTTP agent | HTTP requests |
| 19 | SNMP agent | SNMP agent items |

### Item Configuration
```json
{
  "name": "CPU Utilization",
  "key": "system.cpu.util[,idle]",
  "hostid": "10084",
  "type": 0,
  "value_type": 0,
  "delay": "60",
  "history": "7d",
  "trends": "365d",
  "triggers": [],
  "tags": [
    {
      "tag": "component",
      "value": "cpu"
    }
  ],
  "preprocessing": [
    {
      "type": 1,
      "params": "100",
      "error_handler": 1,
      "error_handler_params": ""
    }
  ]
}
```

### Built-in Items
```bash
# System information
system.cpu.util[,idle]           # CPU idle percentage
system.cpu.util[,user]           # CPU user percentage
system.cpu.util[,system]         # CPU system percentage
system.cpu.num                   # Number of CPUs
system.cpu.load[,avg1]           # 1-minute load average
system.cpu.load[,avg5]           # 5-minute load average
system.cpu.load[,avg15]          # 15-minute load average

# Memory
vm.memory.size[,total]           # Total memory
vm.memory.size[,free]            # Free memory
vm.memory.size[,available]       # Available memory
vm.memory.size[,pctavailable]    # Available percentage
vm.memory.size[,used]            # Used memory
vm.memory.size[,pctused]         # Used percentage

# Disk
vfs.fs.size[/,total]             # Root disk total
vfs.fs.size[/,free]              # Root disk free
vfs.fs.size[/,pfree]             # Root disk free percentage
vfs.fs.size[/,pused]             # Root disk used percentage
vfs.fs.inode[/,pfree]            # Inodes free percentage

# Network
net.if.in[eth0]                  # Inbound bytes on eth0
net.if.out[eth0]                 # Outbound bytes on eth0
net.if.discovery                 # Network interface discovery
net.tcp.port[,80]                # TCP port 80 check
net.udp.port[,161]               # UDP port 161 check

# Processes
proc.num[,,run]                  # Running processes
proc.num[,,sleep]                # Sleeping processes
proc.num[all]                    # All processes

# Filesystem
vfs.file.cksum[/etc/passwd]      # File checksum
vfs.file.regexp[/var/log/syslog,error]  # Regex match
vfs.file.exists[/path/to/file]   # File existence

# System
system.localtime                 # System time
system.uptime                    # System uptime
system.uname                     # System information
system.sw.os                     # OS information
system.sw.packages               # Installed packages
```

### User Parameters
```bash
# /etc/zabbix/zabbix_agentd.conf

# Simple user parameter
UserParameter=custom.disk.stats[*],/etc/zabbix/scripts/disk_stats.sh $1

# Multiple parameters
UserParameter=custom.cpu.util[*],/etc/zabbix/scripts/cpu_util.sh $1 $2

# Complex command
UserParameter=custom.mysql.status[*],mysql -u zabbix -p'$3' -e "SHOW GLOBAL STATUS LIKE '$1'" | grep "$1" | awk '{print $$2}'

# Script-based
UserParameter=custom.apache.status,/etc/zabbix/scripts/apache_status.py

# JSON parsing
UserParameter=custom.http.response[*],curl -s -o /dev/null -w "%{http_code}" http://$1/

# Active check
UserParameter=custom.vfs.dev.read.ops[*],/etc/zabbix/scripts/device_stats.sh read ops $1
```

### Item Preprocessing
```json
{
  "preprocessing": [
    {
      "type": 1,
      "params": "100",
      "error_handler": 1,
      "error_handler_params": ""
    },
    {
      "type": 20,
      "params": "1.00",
      "error_handler": 1,
      "error_handler_params": ""
    },
    {
      "type": 21,
      "params": "",
      "error_handler": 1,
      "error_handler_params": ""
    }
  ]
}
```

### Calculated Items
```json
{
  "name": "Memory Usage Percentage",
  "key": "custom.mem.usage.pct",
  "formula": "100 - (vm.memory.size[,available] / vm.memory.size[,total]) * 100"
}
```

```json
{
  "name": "Network Interface Errors",
  "key": "custom.net.errors",
  "formula": "last(net.if.in[eth0,errors]) + last(net.if.out[eth0,errors])"
}
```

```json
{
  "name": "Average Response Time (Last 5 Minutes)",
  "key": "custom.avg.response.time",
  "formula": "avg(last_5m,15s)"
}
```

---

## 6. Triggers

### Trigger Expressions
```bash
# Basic threshold
{host:item.last()} > 100

# Comparison
{host:item.min(5m)} < 0

# Multiple hosts
{host1:item.last()} > 50 OR {host2:item.last()} > 50

# Time-based
{host:item.change()} < 0

# Nodata detection
{host:item.nodata(5m)} = 1

# Delta calculation
{host:item.delta(5m)} > 1000

# Rate of change
{host:item.rate(5m)} > 50

# String matching
{host:item.str(backup)} = 1
```

### Trigger Configuration
```json
{
  "description": "High CPU Usage",
  "expression": "{web-server-01:system.cpu.util[,idle].avg(5m)} < 20",
  "recovery_expression": "{web-server-01:system.cpu.util[,idle].avg(5m)} > 80",
  "priority": 2,
  "status": 0,
  "type": 0,
  "dependencies": [],
  "tags": [
    {
      "tag": "severity",
      "value": "warning"
    }
  ],
  "url": "https://wiki.example.com/cpu-alerts",
  "manual_close": 0
}
```

### Trigger Severity
| ID | Severity | Color |
|----|----------|-------|
| 0 | Not classified | Gray |
| 1 | Information | Light blue |
| 2 | Warning | Yellow |
| 3 | Average | Orange |
| 4 | High | Red |
| 5 | Disaster | Dark red |

### Trigger Dependencies
```json
{
  "description": "Service Down",
  "expression": "{web-server-01:net.tcp.port[,8080].last()} = 0",
  "recovery_expression": "{web-server-01:net.tcp.port[,8080].last()} = 1",
  "dependencies": [
    {
      "triggerid": "12345"
    }
  ]
}
```

### Trigger Examples
```bash
# CPU high
{host:system.cpu.util[,idle].avg(5m)} < 20

# Memory low
{host:vm.memory.size[,pfree].min(5m)} < 10

# Disk full
{host:vfs.fs.size[/,pfree].min(5m)} < 10

# Service down
{host:net.tcp.port[,80].last()} = 0

# Too many processes
{host:proc.num[,,run].avg(5m)} > 200

# Load too high
{host:system.cpu.load[,avg1].min(5m)} > 10

# Temperature critical
{sensor.temp[/sys/class/thermal/thermal_zone0/temp].max(5m)} > 90000

# Disk I/O high
{host:vfs.dev.io[,sda].avg(5m)} > 50000

# Network traffic spike
{host:net.if.in[eth0].delta(1m)} > 100000000

# Backup failed
{host:vfs.file.regexp[/var/log/backup.log,ERROR].str(1)} = 1

# SSL certificate expiring soon
{certificate.expiry[https://example.com].now()} < 2592000
```

### Event Correlation
```json
{
  "name": "Service Recovery Correlation",
  "type": 0,
  "correlation_rule": {
    "type": 0,
    "operations": [
      {
        "type": 0,
        "filter": {
          "conditions": [
            {
              "type": 2,
              "operator": 0,
              "tag": "service"
            }
          ],
          "eval_type": 0
        }
      }
    ]
  }
}
```

---

## 7. Actions

### Action Configuration
```json
{
  "name": "Notify on High CPU",
  "eventsource": 0,
  "status": 0,
  "esc_period": 3600,
  "default_msg": 0,
  "recovery_msg": 1,
  "recovery_operation": 1,
  "ack_operation": 0,
  "trigger_hierarchy": [],
  "filter": {
    "evaltype": 0,
    "formula": "",
    "conditions": [
      {
        "conditiontype": 2,
        "operator": 0,
        "value": "High CPU Usage",
        "formulaid": "A"
      },
      {
        "conditiontype": 0,
        "operator": 0,
        "hostid": "10084",
        "formulaid": "B"
      }
    ],
    "eval_formula": "A and B"
  },
  "operations": [
    {
      "type": 0,
      "operationtype": 0,
      "esc_period": 0,
      "esc_step_from": 1,
      "esc_step_to": 1,
      "actionid": "1",
      "opmessage": {
        "default_msg": 1,
        "subject": "Problem: {EVENT.NAME}",
        "message": "Problem started at {EVENT.TIME} on {EVENT.DATE}\nTrigger: {TRIGGER.NAME}\nStatus: {TRIGGER.STATUS}\nSeverity: {TRIGGER.SEVERITY}\nHost: {HOST.NAME}\n\nURL: {TRIGGER.URL}",
        "mediatypeid": "1"
      },
      "opmessage_grp": [
        {
          "usrgrpid": "7"
        }
      ],
      "opmessage_usr": []
    }
  ],
  "recovery_operations": [
    {
      "type": 0,
      "operationtype": 1,
      "actionid": "1",
      "opmessage": {
        "default_msg": 1,
        "subject": "RESOLVED: {EVENT.NAME}",
        "message": "Problem resolved at {EVENT.RECOVERY.TIME} on {EVENT.RECOVERY.DATE}\nDuration: {EVENT.DURATION}\nTrigger: {TRIGGER.NAME}\nHost: {HOST.NAME}",
        "mediatypeid": "1"
      },
      "opmessage_grp": [
        {
          "usrgrpid": "7"
        }
      ],
      "opmessage_usr": []
    }
  ]
}
```

### Operation Types
| Type ID | Operation | Description |
|---------|-----------|-------------|
| 0 | Send message | Send notification |
| 1 | Remote command | Execute command |
| 2 | Add host | Add to host group |
| 3 | Remove from host group | Remove from group |
| 4 | Link to template | Link template |
| 5 | Unlink from template | Unlink template |
| 6 | Enable host | Enable host |
| 7 | Disable host | Disable host |
| 8 | Set host inventory | Set inventory |

### Remote Commands
```bash
# Restart service
systemctl restart httpd

# Clear disk space
find /var/log -name "*.log" -mtime +30 -delete

# Restart agent
systemctl restart zabbix-agent

# Drain and restart
docker stop container && docker start container

# Execute script
/etc/zabbix/scripts/alert-handler.sh {HOST.NAME} {TRIGGER.NAME}

# Send to Slack
curl -X POST -d 'payload={"text": "{TRIGGER.NAME}: {TRIGGER.STATUS}"}' https://hooks.slack.com/xxx

# Acknowledge
# Built-in operation
```

### Escalations
```json
{
  "name": "Critical Alert Escalation",
  "eventsource": 0,
  "esc_period": 1800,
  "default_msg": 0,
  "recovery_msg": 1,
  "recovery_operation": 1,
  "operations": [
    {
      "type": 0,
      "operationtype": 0,
      "esc_period": 0,
      "esc_step_from": 1,
      "esc_step_to": 1,
      "opmessage": {
        "default_msg": 0,
        "subject": "URGENT: {TRIGGER.NAME}",
        "message": "This is a critical alert!\nEscalation level 1\nPlease respond immediately.",
        "mediatypeid": "1"
      },
      "opmessage_grp": [
        {
          "usrgrpid": "7"
        }
      ]
    },
    {
      "type": 0,
      "operationtype": 0,
      "esc_period": 0,
      "esc_step_from": 2,
      "esc_step_to": 2,
      "opmessage": {
        "default_msg": 0,
        "subject": "ESCALATED: {TRIGGER.NAME}",
        "message": "Alert has not been acknowledged!\nEscalation level 2\nEscalating to management.",
        "mediatypeid": "2"
      },
      "opmessage_grp": [
        {
          "usrgrpid": "8"
        }
      ]
    },
    {
      "type": 1,
      "operationtype": 1,
      "esc_period": 0,
      "esc_step_from": 2,
      "esc_step_to": 2,
      "opcommand": {
        "type": 0,
        "command": "/etc/zabbix/scripts/escalate.sh {HOST.NAME} {TRIGGER.NAME}",
        "execute_on": 0
      }
    }
  ]
}
```

---

## 8. Templates

### Template Structure
```xml
<?xml version="1.0" encoding="UTF-8"?>
<zabbix_export>
    <version> "6.4",
    <date>: "2023-01-01T00:00:00Z",
    <templates>
        <template>
            <name>Linux Server Template</name>
            <description>Standard Linux server monitoring template</description>
            <groups>
                <group>
                    <name>Templates/Operating systems</name>
                </group>
            </groups>
            <items>
                <item>
                    <name>CPU Utilization</name>
                    <key>system.cpu.util[,idle]</key>
                    <delay>60</delay>
                    <history>7d</history>
                    <trends>365d</trends>
                    <value_type>0</value_type>
                </item>
            </items>
            <triggers>
                <trigger>
                    <name>High CPU Usage</name>
                    <expression>{host:system.cpu.util[,idle].avg(5m)} < 20</expression>
                    <priority>2</priority>
                </trigger>
            </triggers>
            <graphs>
                <graph>
                    <name>CPU Utilization</name>
                    <graph_items>
                        <graph_item>
                            <item>
                                <host>host</host>
                                <key>system.cpu.util[,idle]</key>
                            </item>
                            <color>00FF00</color>
                        </graph_item>
                    </graph_items>
                </graph>
            </graphs>
        </template>
    </templates>
</zabbix_export>
```

### Built-in Templates
| Template | Description |
|----------|-------------|
| Linux by Zabbix agent | Basic Linux monitoring |
| Windows by Zabbix agent | Basic Windows monitoring |
| SNMP Generic | Generic SNMP monitoring |
| VMware | VMware hypervisor monitoring |
| Ceph | Ceph storage monitoring |
| PostgreSQL | PostgreSQL database monitoring |
| MySQL | MySQL database monitoring |

### Creating Templates
```json
{
  "host": "Custom Application Template",
  "name": "Custom Application Template",
  "groups": [
    {
      "groupid": 1
    }
  ],
  "items": [
    {
      "name": "Application Health",
      "key": "app.health",
      "type": 0,
      "value_type": 3,
      "delay": "30"
    }
  ],
  "triggers": [
    {
      "description": "Application Unhealthy",
      "expression": "{host:app.health.last()} = 0",
      "priority": 3
    }
  ],
  "discovery_rules": [],
  "httptests": []
}
```

### Template Linking
```bash
# Link template to host via API
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "templates": [
      {"templateid": "10001"},
      {"templateid": "10002"}
    ],
    "hosts": [
      {"hostid": "10084"}
    ]
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php" -u "admin:zabbix"
```

---

## 9. User Management

### User Configuration
```json
{
  "username": "john.doe",
  "name": "John Doe",
  "surname": "Doe",
  "alias": "john.doe",
  "passwd": "password123",
  "url": "",
  "autologin": 0,
  "autologout": "0",
  "lang": "en_US",
  "theme": "default",
  "refresh": "30",
  "rows_per_page": "50",
  "timezone": "America/New_York",
  "roleid": "2",
  "usrgrps": [
    {
      "usrgrpid": "7"
    }
  ],
  "user_medias": [
    {
      "mediatypeid": "1",
      "sendto": "john.doe@example.com",
      "active": 0,
      "severity": 63,
      "period": "1-7,00:00-24:00"
    }
  ]
}
```

### User Groups
```json
{
  "name": "Linux Administrators",
  "gui_access": 0,
  "users_status": 0,
  "debug_mode": 0,
  "rights": [
    {
      "id": "1",
      "permission": "3"
    }
  ]
}
```

### Permissions
| Permission ID | Permission | Description |
|---------------|------------|-------------|
| 0 | None | No access |
| 1 | Read | Read-only access |
| 2 | Read/Write | Read and write |
| 3 | Admin | Full admin access |

### Media Types
```json
{
  "name": "Email",
  "type": 0,
  "smtp_server": "smtp.example.com",
  "smtp_helo": "zabbix.example.com",
  "smtp_email": "zabbix@example.com",
  "content_type": 1,
  "max_attempts": 3,
  "attempt_interval": "10s"
}
```

```json
{
  "name": "Slack",
  "type": 1,
  "exec_params": "{
    \"channel\": \"{ALERT.SENDTO}\",
    \"text\": \"{TRIGGER.NAME}\n{TRIGGER.STATUS}\n{HOST.NAME}\n{TRIGGER.URL}\"
  }",
  "exec_path": "/usr/local/bin/slack-notify.sh"
}
```

```json
{
  "name": "PagerDuty",
  "type": 19,
  "service_key": "",
  "trigger": "",
  "severity": "",
  "event_source": "trigger",
  "recipients": "",
  "client": "Zabbix"
}
```

### API Authentication
```bash
# Get auth token
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "user.login",
    "params": {
      "username": "admin",
      "password": "zabbix"
    },
    "id": 1
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php"

# Response: {"jsonrpc":"2.0","result":"abc123...","id":1}

# Use auth token in subsequent requests
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer abc123..." \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.get",
    "params": {},
    "id": 1
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php"
```

---

## 10. Proxy

### Proxy Installation
```bash
# Install Zabbix proxy
sudo apt install zabbix-proxy-mysql zabbix-proxy-sqlite3

# Configure proxy
# /etc/zabbix/zabbix_proxy.conf
Server=192.168.1.100
Hostname=proxy-01
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=password
ProxyLocalBuffer=3600
ProxyOfflineBuffer=7200
ConfigFrequency=60
DataSenderFrequency=1
StartPollers=5
StartPingers=1

# Start proxy
sudo systemctl start zabbix-proxy
sudo systemctl enable zabbix-proxy
```

### Proxy Configuration in Server
```bash
# Create proxy in Zabbix
# Configuration -> Proxies -> Create proxy

# Proxy config
{
  "host": "proxy-01",
  "status": 5,
  "tls_accept": 1,
  "tls_psk_identity": "proxy-01",
  "tls_psk": "a1b2c3d4e5f6..."
}
```

### Proxy Commands
```bash
# Get proxy configuration
zabbix_proxy --config /etc/zabbix/zabbix_proxy.conf --print-config

# Test connection
zabbix_proxy --config /etc/zabbix/zabbix_proxy.conf --test

# Force configuration reload
# In server: Administration -> Proxies -> Select proxy -> Force check

# View proxy status
# Monitoring -> Problems -> Proxy filter
```

---

## 11. API

### Common API Methods

#### Host
```bash
# Get all hosts
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.get",
    "params": {
      "output": ["hostid", "host", "name"],
      "selectInterfaces": ["ip"]
    },
    "auth": "AUTH_TOKEN",
    "id": 1
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php"

# Create host
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.create",
    "params": {
      "host": "new-server",
      "name": "New Server",
      "interfaces": [
        {
          "type": 1,
          "main": 1,
          "useip": 1,
          "ip": "192.168.1.50",
          "dns": "",
          "port": "10050"
        }
      ],
      "groups": [{"groupid": "2"}],
      "templates": [{"templateid": "10001"}]
    },
    "auth": "AUTH_TOKEN",
    "id": 1
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php"

# Update host
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.update",
    "params": {
      "hostid": "10084",
      "name": "Updated Server Name"
    },
    "auth": "AUTH_TOKEN",
    "id": 1
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php"

# Delete host
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "host.delete",
    "params": ["10084"],
    "auth": "AUTH_TOKEN",
    "id": 1
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php"
```

#### Item
```bash
# Get items
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "item.get",
    "params": {
      "hostids": "10084",
      "output": ["itemid", "name", "key_"],
      "selectTriggers": ["triggerid", "description"]
    },
    "auth": "AUTH_TOKEN",
    "id": 1
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php"
```

#### Trigger
```bash
# Get triggers
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "trigger.get",
    "params": {
      "hostids": "10084",
      "output": ["triggerid", "description", "priority", "status"],
      "expandDescription": true,
      "filter": {
        "value": 1
      }
    },
    "auth": "AUTH_TOKEN",
    "id": 1
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php"
```

#### Problem
```bash
# Get problems
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "problem.get",
    "params": {
      "output": ["eventid", "name", "severity", "time"],
      "selectHosts": ["host"],
      "filter": {
        "severity": [4, 5]
      },
      "recent": true
    },
    "auth": "AUTH_TOKEN",
    "id": 1
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php"
```

#### History
```bash
# Get history
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "history.get",
    "params": {
      "itemids": "23269",
      "history": 0,
      "output": "extend",
      "limit": 100,
      "sortfield": "clock",
      "sortorder": "DESC"
    },
    "auth": "AUTH_TOKEN",
    "id": 1
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php"
```

#### Event
```bash
# Get events
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "event.get",
    "params": {
      "output": ["eventid", "objectid", "clock", "value"],
      "selectHosts": ["host"],
      "limit": 100
    },
    "auth": "AUTH_TOKEN",
    "id": 1
  }' \
  "http://zabbix-server/zabbix/api_jsonrpc.php"
```

### Python Zabbix API Client
```python
from zabbix_api import ZabbixAPI

zapi = ZabbixAPI("http://zabbix-server/zabbix")
zapi.login("admin", "zabbix")

# Get hosts
hosts = zapi.host.get({
    "output": ["hostid", "host", "name"],
    "selectInterfaces": ["ip"]
})

for host in hosts:
    print(f"{host['name']} ({host['host']})")

# Get items
items = zapi.item.get({
    "hostids": "10084",
    "output": ["itemid", "name", "key_"]
})

# Get triggers
triggers = zapi.trigger.get({
    "hostids": "10084",
    "output": ["triggerid", "description", "priority"],
    "filter": {"value": 1}
})

# Get problems
problems = zapi.problem.get({
    "output": ["eventid", "name", "severity"],
    "filter": {"value": 1},
    "recent": True
})

# Create item
zapi.item.create({
    "name": "Custom Check",
    "key_": "custom.check",
    "hostid": "10084",
    "type": 0,
    "value_type": 3,
    "delay": "60"
})

# Update trigger
zapi.trigger.update({
    "triggerid": "12345",
    "description": "Updated Description"
})
```

---

## 12. Monitoring Patterns

### Linux Server Monitoring
```bash
# CPU Monitoring
Item: system.cpu.util[,idle] (Zabbix agent)
Trigger: {host:system.cpu.util[,idle].avg(5m)} < 20

# Memory Monitoring
Item: vm.memory.size[,pfree] (Zabbix agent)
Trigger: {host:vm.memory.size[,pfree].min(5m)} < 10

# Disk Usage Monitoring
Item: vfs.fs.size[/,pfree] (Zabbix agent)
Trigger: {host:vfs.fs.size[/,pfree].min(5m)} < 10

# Disk I/O Monitoring
Item: vfs.dev.io[,sda] (Zabbix agent)
Trigger: {host:vfs.dev.io[,sda].avg(5m)} > 50000

# Network Interface Monitoring
Item: net.if.in[eth0] (Zabbix agent)
Trigger: {host:net.if.in[eth0].max(5m)} > 100000000

# Process Monitoring
Item: proc.num[,,run] (Zabbix agent)
Trigger: {host:proc.num[,,run].avg(5m)} > 300

# User Sessions
Item: system.users.num (Zabbix agent)
Trigger: {host:system.users.num.last()} > 50
```

### Network Device Monitoring
```bash
# SNMP Interface Status
Item: snmp.ifOperStatus[ifIndex.1] (SNMP)
Trigger: {host:snmp.ifOperStatus[ifIndex.1].last()} = 2

# SNMP Interface Traffic
Item: snmp.ifInOctets[ifIndex.1] (SNMP)
Trigger: {host:snmp.ifInOctets[ifIndex.1].delta(5m)} > 100000000

# SNMP CPU
Item: ssCpuUser.0 (SNMP)
Trigger: {host:ssCpuUser.0.avg(5m)} > 90

# SNMP Memory
Item: hrStorageUsed.1 (SNMP)
Trigger: {host:hrStorageUsed.1.last()} > 1000000000
```

### Web Application Monitoring
```bash
# HTTP Response Time
Item: web.page.performance[http://example.com] (Simple check)
Trigger: {host:web.page.performance[http://example.com].last()} > 5000

# HTTP Status Code
Item: web.page.get[http://example.com] (Simple check)
Trigger: {host:web.page.get[http://example.com].regexp("200")} = 0

# HTTPS Certificate Expiry
Item: certificate.validity[https://example.com] (Zabbix agent)
Trigger: {host:certificate.validity[https://example.com].now()} < 2592000

# Database Connection
Item: net.tcp.port[,3306] (Zabbix agent)
Trigger: {host:net.tcp.port[,3306].last()} = 0
```

### Docker/Kubernetes Monitoring
```bash
# Container Status
Item: docker.container.info[container_name] (User parameter)
Trigger: {host:docker.container.info[container_name].last()} = 0

# Kubernetes Pod Status
Item: kube_pod.status_phase{pod="my-pod"} (Prometheus)
Trigger: {host: kube_pod.status_phase{pod="my-pod"}.last()} != "Running"

# Pod Memory Usage
Item: container_memory_working_set_bytes{pod="my-pod"} (Prometheus)
Trigger: {host:container_memory_working_set_bytes{pod="my-pod"}.avg(5m)} > 1073741824

# Pod CPU Usage
Item: rate(container_cpu_usage_seconds_total{pod="my-pod"}[5m]) (Prometheus)
Trigger: {host:rate(container_cpu_usage_seconds_total{pod="my-pod"}[5m]).avg(5m)} > 2
```

### Database Monitoring
```bash
# MySQL Connections
Item: mysql.connections (User parameter)
Trigger: {host:mysql.connections.last()} > 1000

# PostgreSQL Backends
Item: pg_stat_activity_count (User parameter)
Trigger: {host:pg_stat_activity_count.last()} > 100

# Query Execution Time
Item: mysql.slow_queries (User parameter)
Trigger: {host:mysql.slow_queries.delta(1h)} > 100

# Database Size
Item: mysql.db.size[dbname] (User parameter)
Trigger: {host:mysql.db.size[dbname].last()} > 10000000000
```

---

## 13. Troubleshooting

### Common Issues

#### Agent Not Connecting
```bash
# Check agent status
systemctl status zabbix-agent

# Check agent logs
tail -f /var/log/zabbix/zabbix_agentd.log

# Test agent connection
zabbix_get -s host -k agent.ping

# Check firewall
iptables -L -n | grep 10050
firewall-cmd --list-ports

# Verify configuration
cat /etc/zabbix/zabbix_agentd.conf | grep -E "^Server|^Hostname"

# Test from server
zabbix_get -s 192.168.1.10 -k system.cpu.load
```

#### Server Not Starting
```bash
# Check server logs
tail -f /var/log/zabbix/zabbix_server.log

# Check database connection
mysql -u zabbix -p -e "SELECT 1"

# Check configuration
zabbix_server --config /etc/zabbix/zabbix_server.conf --test

# Check port
netstat -tulpn | grep 10051

# Check SELinux/AppArmor
getenforce
aa-status
```

#### No Data
```bash
# Check item configuration
# Verify key is correct

# Check item is enabled
# Configuration -> Hosts -> Items

# Check trigger expression
# Test in: Latest data -> History

# Check preprocessing
# Verify preprocessing steps

# Check network connectivity
ping host
telnet host 10050

# Check permissions
ls -la /etc/zabbix/
```

#### Performance Issues
```bash
# Check server performance
# Administration -> System information

# Check database performance
mysqladmin -u zabbix -p status

# Optimize database
# Add indexes to frequently queried columns

# Increase cache size
# Edit zabbix_server.conf

# Reduce polling frequency
# Increase item delay

# Use proxy for distributed monitoring
```

### Debug Commands
```bash
# Test agent item
zabbix_get -s host -k "system.cpu.util[,idle]"

# Test SNMP item
snmpwalk -v2c -c public host .1.3.6.1.2.1.1.1.0

# Check database
mysql -u zabbix -p zabbix -e "SELECT * FROM hosts LIMIT 5;"

# View queue
# Administration -> Queue

# Check internal metrics
# Administration -> Internal checks

# Test web monitoring
curl -I http://example.com

# Check trigger expression
# Use expression constructor in UI
```

### Log Analysis
```bash
# View server errors
grep -i error /var/log/zabbix/zabbix_server.log | tail -50

# View agent errors
grep -i error /var/log/zabbix/zabbix_agentd.log | tail -50

# View web errors
tail -f /var/log/apache2/error.log

# View database errors
tail -f /var/log/mysql/error.log

# Search for specific issue
grep "item" /var/log/zabbix/zabbix_server.log | grep -i error
```

---

## 14. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| Start server | `systemctl start zabbix-server` |
| Start agent | `systemctl start zabbix-agent` |
| Check status | `systemctl status zabbix-server` |
| Test agent | `zabbix_get -s host -k agent.ping` |
| Import template | `zabbix_export -c template.xml` |
| View queue | `mysql -u zabbix -p zabbix -e "SELECT * FROM queue"` |
| Get history | `zabbix_get -s host -k item.key` |

### Item Keys Quick Reference
| Key | Description |
|-----|-------------|
| `agent.ping` | Agent availability |
| `agent.version` | Agent version |
| `system.cpu.util[,idle]` | CPU idle percentage |
| `system.cpu.load[,avg1]` | 1-minute load average |
| `vm.memory.size[,total]` | Total memory |
| `vfs.fs.size[/,pfree]` | Disk free percentage |
| `net.if.in[eth0]` | Network inbound |
| `net.tcp.port[,80]` | TCP port check |
| `proc.num[,,run]` | Running processes |
| `system.uptime` | System uptime |

### Trigger Functions
| Function | Description | Example |
|----------|-------------|---------|
| `last()` | Last value | `last()` |
| `min()` | Minimum value | `min(5m)` |
| `max()` | Maximum value | `max(5m)` |
| `avg()` | Average value | `avg(5m)` |
| `sum()` | Sum of values | `sum(5m)` |
| `count()` | Count of values | `count(5m, 0)` |
| `delta()` | Difference | `delta(5m)` |
| `change()` | Change since last | `change()` |
| `nodata()` | No data check | `nodata(5m)` |
| `diff()` | Compare to previous | `diff()` |

### Media Types
| Type | ID | Description |
|------|----|-------------|
| Email | 0 | Email notifications |
| Script | 1 | Custom scripts |
| SMS | 2 | SMS via modem |
| Jabber | 3 | Jabber/XMPP |
| Ez Texting | 4 | SMS via Ez Texting |
| Custom alert | 9 | Custom media |
| PagerDuty | 19 | PagerDuty integration |

### Trigger Severity Colors
| Severity | Color | Use Case |
|----------|-------|----------|
| Not classified | Gray | Uncategorized |
| Information | Light blue | FYI alerts |
| Warning | Yellow | Non-critical issues |
| Average | Orange | Service degraded |
| High | Red | Critical issue |
| Disaster | Dark red | System down |

### API Version Compatibility
| Zabbix Version | API Version |
|----------------|-------------|
| 6.4 | 6.4 |
| 6.0 | 6.0 |
| 5.4 | 5.4 |
| 5.0 | 5.0 |
| 4.4 | 4.4 |
| 4.0 | 4.0 |

### Ports
| Port | Service | Description |
|------|---------|-------------|
| 80/443 | HTTP/S | Web interface |
| 10050 | Zabbix agent | Agent (passive) |
| 10051 | Zabbix server | Server (active/proxy) |
| 3306 | MySQL | Database |
| 162 | SNMP trap | SNMP traps |

---

*Last Updated: January 2026*
*Generated for Zabbix 6.4*
