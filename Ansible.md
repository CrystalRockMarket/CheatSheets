# Ansible Complete Cheat Sheet

## Table of Contents
1. [Introduction](#1-introduction)
2. [Installation](#2-installation)
3. [Inventory](#3-inventory)
4. [Ad-Hoc Commands](#4-ad-hoc-commands)
5. [Playbooks](#5-playbooks)
6. [Variables](#6-variables)
7. [Handlers](#7-handlers)
8. [Conditionals](#8-conditionals)
9. [Loops](#9-loops)
10. [Templates](#10-templates)
11. [Roles](#11-roles)
12. [Vault](#12-vault)
13. [Galaxy](#13-galaxy)
14. [Best Practices](#14-best-practices)
15. [Quick Reference](#15-quick-reference)

---

## 1. Introduction

### What is Ansible?
Ansible is an open-source automation platform that enables infrastructure as code, configuration management, application deployment, and orchestration.

### Key Features
| Feature | Description |
|---------|-------------|
| **Agentless** | No agents required on managed nodes |
| **YAML** | Human-readable playbooks |
| **Idempotent** | Safe to run multiple times |
| **Push-based** | Pushes configuration to nodes |
| **SSH** | Secure connection by default |
| **Modules** | 2000+ built-in modules |
| **Plugins** | Extensible architecture |
| **Multi-tier** | Manages entire infrastructure |

### Architecture
```
┌─────────────────────────────────────────────────────────────────────┐
│                        Ansible Control Node                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────┐ │
│  │  Playbooks  │  │   Modules   │  │   Inventory                 │ │
│  │  (YAML)     │  │ (Python)    │  │   (Hosts)                   │ │
│  └─────────────┘  └─────────────┘  └─────────────────────────────┘ │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                    Execution Engine                     │ │
│  └─────────────────────────────────────────────────────────┘ │
│                            │ SSH                              │
└─────────────────────────────────────────────────────────────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│  Managed     │   │  Managed     │   │  Managed     │
│  Node 1      │   │  Node 2      │   │  Node 3      │
│  (SSH)       │   │  (SSH)       │   │  (SSH)       │
└──────────────┘   └──────────────┘   └──────────────┘
```

---

## 2. Installation

### Control Node Installation
```bash
# pip (recommended)
pip install ansible

# pip with requirements
pip install ansible[core]

# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# RHEL/CentOS/Fedora
sudo dnf install ansible

# macOS
brew install ansible

# Check version
ansible --version
```

### Managed Node Requirements
```bash
# Python 2.7+ or Python 3.5+
# SSH access
# sudo privileges (for most tasks)

# For Python 2 on older systems
sudo apt install python
```

### Quick Setup
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "ansible@control-node"

# Copy SSH key to managed nodes
ssh-copy-id user@managed-node-1
ssh-copy-id user@managed-node-2

# Test connection
ansible all -m ping
```

---

## 3. Inventory

### Static Inventory (INI Format)
```ini
# /etc/ansible/hosts
# Basic inventory

[webservers]
web1.example.com
web2.example.com
web3.example.com

[databases]
db1.example.com
db2.example.example

[loadbalancers]
lb.example.com

# Variables for group
[webservers:vars]
http_port=80
max_connections=1000

# Variables for host
web1.example.com ansible_host=192.168.1.10 ansible_port=2222
```

### Static Inventory (YAML Format)
```yaml
# inventory.yml
all:
  hosts:
    web1.example.com:
      ansible_host: 192.168.1.10
    web2.example.com:
      ansible_host: 192.168.1.11
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
      vars:
        http_port: 80
    databases:
      hosts:
        db1.example.com:
      vars:
        db_port: 5432
```

### Dynamic Inventory
```bash
# AWS EC2
ansible-inventory -i ec2.yml --list

# OpenStack
ansible-inventory -i openstack.yml --list

# Azure
ansible-inventory -i azure_rm.yml --list

# VMware
ansible-inventory -i vmware_vm_inventory.yml --list

# Custom script
ansible-inventory -i custom_inventory.py --list
```

### Inventory Parameters
| Parameter | Description | Example |
|-----------|-------------|---------|
| `ansible_host` | Hostname/IP | `ansible_host=192.168.1.10` |
| `ansible_port` | SSH port | `ansible_port=2222` |
| `ansible_user` | SSH user | `ansible_user=deploy` |
| `ansible_password` | SSH password | (use SSH keys instead) |
| `ansible_connection` | Connection type | `ssh`, `local`, `docker` |
| `ansible_become` | Enable privilege escalation | `yes` |
| `ansible_become_user` | User to become | `root` |
| `ansible_become_method` | Escalation method | `sudo` |
| `ansible_ssh_private_key_file` | SSH key | `/path/to/key` |

### Group Patterns
```bash
# All hosts
all
*

# Webservers group
webservers

# Multiple groups
webservers:dbservers

# Intersection (in both)
webservers:&production

# Union (in either)
webservers:!development

# Regex pattern
~web[0-9]+\.example\.com
```

---

## 4. Ad-Hoc Commands

### Basic Commands
```bash
# Ping all hosts
ansible all -m ping

# Execute command
ansible all -m command -a "uptime"

# Execute shell
ansible all -m shell -a "df -h"

# Copy file
ansible all -m copy -a "src=/local/file dest=/remote/file"

# Install package
ansible webservers -m apt -a "name=nginx state=present"

# Start service
ansible webservers -m service -a "name=nginx state=started"

# Create user
ansible all -m user -a "name=testuser password=<password_hash>"

# Create directory
ansible all -m file -a "path=/tmp/test state=directory"
```

### Common Modules
| Module | Description | Example |
|--------|-------------|---------|
| **command** | Execute command | `-m command -a "ls"` |
| **shell** | Execute shell | `-m shell -a "ls | grep .py"` |
| **copy** | Copy files | `-m copy -a "src=file dest=file"` |
| **fetch** | Fetch files | `-m fetch -a "src=file dest=dir"` |
| **file** | Manage files | `-m file -a "path=file state=absent"` |
| **apt** | APT packages | `-m apt -a "name=nginx"` |
| **yum** | YUM packages | `-m yum -a "name=nginx"` |
| **pip** | Python pip | `-m pip -a "name=flask"` |
| **service** | Services | `-m service -a "name=nginx"` |
| **user** | User management | `-m user -a "name=user"` |
| **group** | Group management | `-m group -a "name=admin"` |
| **git** | Git operations | `-m git -a "repo=url dest=/path"` |
| **template** | Template files | `-m template -a "src=j2 dest=file"` |
| **debug** | Print message | `-m debug -a "msg=Hello"` |
| **wait_for** | Wait for condition | `-m wait_for -a "port=80"` |

### Module Options
```bash
# Check mode (dry run)
ansible all -m apt -a "name=nginx" --check

# Diff mode
ansible all -m copy -a "src=file dest=file" --diff

# Become (sudo)
ansible all -m command -a "whoami" --become

# Specify user
ansible all -m command -a "whoami" -u deploy

# Limit hosts
ansible webservers -m ping

# Limit pattern
ansible web* -m ping

# Fork count
ansible all -m ping -f 10

# Timeout
ansible all -m command -a "sleep 60" --timeout=120
```

### Parallel Execution
```bash
# Run on 20 hosts in parallel (default)
ansible all -m ping

# Specify number of forks
ansible all -m ping -f 50

# Forks in ansible.cfg
[defaults]
forks = 50
```

---

## 5. Playbooks

### Basic Playbook
```yaml
# site.yml
---
- name: Install and configure web server
  hosts: webservers
  become: yes
  vars:
    http_port: 80
    ssl_enabled: no

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes

    - name: Copy nginx config
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
        mode: '0644'
      notify: Restart nginx

    - name: Ensure nginx is running
      service:
        name: nginx
        state: started

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

### Play Structure
```yaml
---
- name: Play name                    # Descriptive name
  hosts: webservers                  # Target hosts
  become: yes                        # Privilege escalation
  gather_facts: yes                  # Gather system info
  vars:                              # Play variables
    var1: value1
  vars_files:                        # Variable files
    - vars/secrets.yml
  vars_prompt:                       # Prompt for input
    - name: admin_password
      prompt: "Enter admin password"
      private: yes
  pre_tasks:                         # Tasks before roles
    - name: Pre-task message
      debug:
        msg: "Starting play..."
  roles:                             # Roles to include
    - common
    - webserver
  tasks:                             # Tasks
    - name: Task name
      module:
        option1: value1
  post_tasks:                        # Tasks after roles
    - name: Post-task message
      debug:
        msg: "Play complete"
  handlers:                          # Handlers
    - name: Handler name
      module:
        option: value
```

### Task Options
```yaml
tasks:
  - name: Task with options
    apt:
      name: nginx
      state: present
    become: yes
    become_user: root
    ignore_errors: yes
    ignore_unreachable: yes
    check_mode: no
    delegate_to: localhost
    run_once: no
    any_errors_fatal: no
    retries: 3
    delay: 5
    until: result.rc == 0
    register: task_result
    when: ansible_facts['os_family'] == "Debian"
```

---

## 6. Variables

### Variable Types
```yaml
# Variables
vars:
  app_name: myapp
  app_version: 1.0.0
  enabled: yes

# List
packages:
  - nginx
  - mysql
  - python

# Dictionary
config:
  host: localhost
  port: 5432
  database: myapp
```

### Variable Precedence
```bash
# From lowest to highest precedence:
# 1. Command line (--extra-vars or -e)
# 2. Role defaults (roles/*/defaults/main.yml)
# 3. Group vars (group_vars/*)
# 4. Host vars (host_vars/*)
# 5. Facts
# 6. Register variables
# 7. Task variables (set with set_fact or include_vars)
# 8. Block variables
# 9. Role variables (roles/*/vars/main.yml)
# 10. Include parameters
# 11. Include variables
```

### Defining Variables
```yaml
# In play
- name: Example play
  hosts: all
  vars:
    http_port: 8080

# In separate file
- name: Example play
  hosts: all
  vars_files:
    - vars/main.yml

# From command line
ansible-playbook site.yml -e "http_port=9090"

# From file
ansible-playbook site.yml -e @vars.yml

# From inventory
[webservers:vars]
http_port=8080

# In host_vars
# host_vars/web1.example.com.yml
http_port: 8080

# In group_vars
# group_vars/webservers.yml
http_port: 8080
```

### Accessing Variables
```yaml
# Simple variable
{{ variable_name }}

# Dictionary
{{ config.port }}

# List
{{ packages[0] }}

# Facts
{{ ansible_facts['hostname'] }}
{{ ansible_facts['default_ipv4']['address'] }}

# Host-specific
{{ hostvars['web1.example.com']['ansible_facts']['hostname'] }}
```

### set_fact
```yaml
tasks:
  - name: Set fact
    set_fact:
      my_fact: "computed_value"

  - name: Use fact
    debug:
      msg: "{{ my_fact }}"
```

### include_vars
```yaml
tasks:
  - name: Include variables
    include_vars:
      file: app_config.yml

  - name: Use included variables
    debug:
      msg: "{{ app_config.port }}"
```

---

## 7. Handlers

### Handler Syntax
```yaml
handlers:
  - name: Restart nginx
    service:
      name: nginx
      state: restarted
    listen: "web service restart"

  - name: Restart PHP-FPM
    service:
      name: php-fpm
      state: restarted
    listen: "web service restart"

  - name: Reload nginx
    command: nginx -s reload
```

### Triggering Handlers
```yaml
tasks:
  - name: Copy nginx config
    copy:
      src: files/nginx.conf
      dest: /etc/nginx/nginx.conf
    notify:
      - Restart nginx
      - "web service restart"

  - name: Configure PHP
    template:
      src: templates/php.ini.j2
      dest: /etc/php.ini
    notify:
      - Restart PHP-FPM
```

### Handler Flush
```yaml
# Force handlers to run
- name: Flush handlers immediately
  meta: flush_handlers

tasks:
  - name: First task
    ...
    notify: Restart nginx

  - name: Second task
    ...
    notify: Restart nginx

  - meta: flush_handlers

  - name: Final task
    debug:
      msg: "Handlers have run"
```

---

## 8. Conditionals

### When Statement
```yaml
tasks:
  - name: Install Apache on RedHat
    yum:
      name: httpd
      state: present
    when: ansible_facts['os_family'] == "RedHat"

  - name: Install Apache on Debian
    apt:
      name: apache2
      state: present
    when: ansible_facts['os_family'] == "Debian"

  - name: Install on specific version
    apt:
      name: nginx
      state: present
    when: ansible_facts['distribution_major_version'] | int == 20
```

### Complex Conditions
```yaml
tasks:
  - name: Run when both conditions are true
    debug:
      msg: "Production and webserver"
    when:
      - "'production' in group_names"
      - "'webservers' in group_names"

  - name: Run when either condition is true
    debug:
      msg: "Production or staging"
    when: "'production' in group_names or 'staging' in group_names"

  - name: Run when variable is defined
    debug:
      msg: "Variable is defined"
    when: my_variable is defined

  - name: Run when variable is not defined
    debug:
      msg: "Variable is not defined"
    when: my_variable is not defined

  - name: Run when variable is truthy
    debug:
      msg: "Enabled"
    when: enabled | bool

  - name: Run when string contains
    debug:
      msg: "Contains target"
    when: "'target' in string_var"
```

### Changed Result
```yaml
tasks:
  - name: Check if file exists
    stat:
      path: /etc/config.conf
    register: config_file

  - name: Only create backup if file exists
    copy:
      src: /etc/config.conf
      dest: /etc/config.conf.bak
    when: config_file.stat.exists
```

---

## 9. Loops

### Basic Loops
```yaml
tasks:
  - name: Install multiple packages
    apt:
      name: "{{ item }}"
      state: present
    loop:
      - nginx
      - mysql-server
      - python

  - name: Create users
    user:
      name: "{{ item }}"
      state: present
    loop:
      - user1
      - user2
      - user3

  - name: Create directories
    file:
      path: "/opt/{{ item }}"
      state: directory
    loop:
      - app1
      - app2
      - app3
```

### With_* Loops (Legacy)
```yaml
tasks:
  - name: Install packages with_items
    apt:
      name: "{{ item }}"
      state: present
    with_items:
      - nginx
      - mysql

  - name: Iterate over hostvars
    debug:
      msg: "{{ item }}"
    with_items: "{{ groups['webservers'] }}"

  - name: Subelements
    debug:
      msg: "{{ item.0.name }} - {{ item.1 }}"
    with_subelements:
      - users
      - permissions
```

### Loop Control
```yaml
tasks:
  - name: Loop with label
    debug:
      msg: "{{ item }}"
    loop:
      - item1
      - item2
    loop_control:
      label: "{{ item }}"

  - name: Extended loop
    debug:
      msg: "Index: {{ loop.index }}, Value: {{ item }}"
    loop:
      - a
      - b
      - c
    loop_control:
      index_var: my_index

  - name: Pause between iterations
    debug:
      msg: "{{ item }}"
    loop:
      - item1
      - item2
    loop_control:
      pause: 5
```

### Complex Loops
```yaml
# Loop over dictionary
vars:
  users:
    alice: 1000
    bob: 1001
    charlie: 1002

tasks:
  - name: Create users from dict
    user:
      name: "{{ item.key }}"
      uid: "{{ item.value }}"
    loop: "{{ dict(users) | dict2items }}"

# Loop with condition
- name: Install packages conditionally
  apt:
    name: "{{ item }}"
    state: present
  loop: "{{ packages }}"
  when: item in available_packages

# Loop until (with retries)
- name: Wait for port to be ready
  wait_for:
    port: 8080
    timeout: 60
  register: result
  retries: 10
  until: result.elapsed < 30
```

---

## 10. Templates

### Jinja2 Template
```jinja2
{# /etc/nginx/sites-available/default.conf.j2 #}
server {
    listen {{ http_port }};
    listen [::]:{{ http_port }};

    server_name {{ server_name }};

    root {{ document_root }};
    index index.html index.htm;

    access_log {{ log_path }}/access.log;
    error_log {{ log_path }}/error.log;

    {% if ssl_enabled %}
    listen 443 ssl;
    ssl_certificate {{ ssl_cert_path }};
    ssl_certificate_key {{ ssl_key_path }};
    {% endif %}

    location / {
        try_files $uri $uri/ =404;
    }

    {% for port in upstream_ports %}
    upstream backend_{{ port }} {
        server 127.0.0.1:{{ port }};
    }
    {% endfor %}
}
```

### Template Module
```yaml
tasks:
  - name: Copy nginx config
    template:
      src: templates/nginx.conf.j2
      dest: /etc/nginx/nginx.conf
      mode: '0644'
      owner: root
      group: root
      validate: nginx -t -c %s
    notify: Restart nginx

  - name: Generate app config
    template:
      src: templates/app.yml.j2
      dest: /opt/myapp/config.yml
      backup: yes
```

### Template Filters
```jinja2
{# String filters #}
{{ name | upper }}
{{ name | lower }}
{{ name | capitalize }}
{{ name | default('default_value') }}
{{ name | length }}
{{ name | trim }}
{{ name | replace('old', 'new') }}
{{ name | regex_replace('pattern', 'replacement') }}

{# Number filters #}
{{ value | round(2) }}
{{ value | int }}
{{ value | float }}
{{ value | abs }}
{{ value | max }}
{{ value | min }}

{# List filters #}
{{ list | first }}
{{ list | last }}
{{ list | join(',') }}
{{ list | unique }}
{{ list | sort }}
{{ list | length }}

{# Data filters #}
{{ dict | dict2items }}
{{ dict | items2dict }}
{{ json_string | from_json }}
{{ data | to_nice_yaml }}
```

---

## 11. Roles

### Role Structure
```
roles/
└── webserver/
    ├── defaults/
    │   └── main.yml          # Default variables (lowest priority)
    ├── files/
    │   └── config.txt        # Static files
    ├── handlers/
    │   └── main.yml          # Handlers
    ├── meta/
    │   └── main.yml          # Role metadata/dependencies
    ├── tasks/
    │   └── main.yml          # Tasks
    ├── templates/
    │   └── config.j2         # Jinja2 templates
    └── vars/
        └── main.yml          # Variables (highest priority)
```

### Create Role
```bash
# Using ansible-galaxy
ansible-galaxy init roles/webserver

# Manual creation
mkdir -p roles/webserver/{tasks,handlers,templates,files,vars,defaults,meta}
```

### Role Tasks
```yaml
# roles/webserver/tasks/main.yml
---
- name: Install nginx
  apt:
    name: nginx
    state: present

- name: Copy config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    mode: '0644'
  notify: Restart nginx

- name: Ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
```

### Role Defaults
```yaml
# roles/webserver/defaults/main.yml
---
http_port: 80
https_port: 443
ssl_enabled: false
server_name: localhost
document_root: /var/www/html
log_path: /var/log/nginx
```

### Role Handlers
```yaml
# roles/webserver/handlers/main.yml
---
- name: Restart nginx
  service:
    name: nginx
    state: restarted

- name: Reload nginx
  service:
    name: nginx
    state: reloaded
```

### Role Dependencies
```yaml
# roles/webserver/meta/main.yml
---
dependencies:
  - role: common
    vars:
      timezone: UTC
  - role: firewall
    when: firewall_enabled | bool
```

### Use Role in Playbook
```yaml
# site.yml
---
- name: Configure web servers
  hosts: webservers
  become: yes

  roles:
    - webserver
    - role: database
      db_name: myapp
      db_user: appuser
    - common
```

---

## 12. Vault

### Create Encrypted File
```bash
# Create new encrypted file
ansible-vault create secrets.yml

# Encrypt existing file
ansible-vault encrypt secrets.yml

# Edit encrypted file
ansible-vault edit secrets.yml

# View encrypted file
ansible-vault view secrets.yml

# Decrypt file
ansible-vault decrypt secrets.yml

# Re-encrypt with new password
ansible-vault rekey secrets.yml
```

### Vault Passwords
```bash
# Interactive password
ansible-vault create secrets.yml

# Password file
ansible-vault create --vault-password-file ~/.vault_pass secrets.yml

# Environment variable
export ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass
ansible-vault view secrets.yml
```

### Encrypted Variables
```yaml
# secrets.yml (encrypted)
---
db_password: my_secret_password
api_key: "your-api-key"
admin_password: "{{ vault_admin_password }}"
```

### Use Vault in Playbook
```yaml
# Playbook with vault
- name: Deploy application
  hosts: all
  vars_files:
    - vars/secrets.yml

  tasks:
    - name: Use vaulted variable
      debug:
        msg: "Database password is {{ db_password }}"
```

### Vault in Command Line
```bash
# Run playbook with vault
ansible-playbook site.yml --ask-vault-pass

# With password file
ansible-playbook site.yml --vault-password-file ~/.vault_pass

# With multiple vaults
ansible-playbook site.yml \
  --vault-password-file ~/.vault_pass_prod \
  -e @secrets_dev.yml \
  --vault-password-file ~/.vault_pass_dev \
  -e @secrets_prod.yml
```

---

## 13. Galaxy

### Basic Commands
```bash
# Search for role
ansible-galaxy search "nginx"

# Get role info
ansible-galaxy info nginxinc.nginx

# Install role
ansible-galaxy install nginxinc.nginx

# Install specific version
ansible-galaxy install nginxinc.nginx,1.0.0

# Install from requirements file
ansible-galaxy install -r requirements.yml

# List installed roles
ansible-galaxy list

# Remove role
ansible-galaxy remove nginxinc.nginx
```

### requirements.yml
```yaml
# requirements.yml
roles:
  # From Ansible Galaxy
  - name: nginxinc.nginx
    version: "1.0.0"
    src: nginxinc.nginx

  # From GitHub
  - name: custom_role
    src: https://github.com/user/ansible-role-custom
    version: main

  # From GitLab
  - name: internal_role
    src: git@gitlab.com:user/ansible-role-internal.git
    version: production

  # From local path
  - name: local_role
    src: /path/to/local/role

collections:
  - name: community.general
  - name: azure.azcollection
    version: ">=1.0.0"
```

### Create Role for Galaxy
```bash
# Initialize role
ansible-galaxy init my_role

# Structure
my_role/
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── README.md
├── tasks/
│   └── main.yml
├── templates/
└── vars/
    └── main.yml
```

### Publish to Galaxy
```bash
# Create GitHub repository
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/username/ansible-role-my_role.git
git push -u origin main

# Import to Galaxy
ansible-galaxy role import github_user github_repo
```

---

## 14. Best Practices

### Project Structure
```
production/
├── inventory.yml
├── group_vars/
│   ├── all.yml
│   ├── webservers.yml
│   └── databases.yml
├── host_vars/
│   └── web1.example.com.yml
├── roles/
│   └── webserver/
├── site.yml
├── web.yml
└── requirements.yml

staging/
└── (same structure)
```

### Directory Layout Best Practices
```
ansible-project/
├── ansible.cfg              # Ansible config
├── inventory/               # Inventories
│   ├── production/
│   └── staging/
├── group_vars/              # Group variables
├── host_vars/               # Host variables
├── plugins/                 # Custom plugins
├── modules/                 # Custom modules
├── filters/                 # Custom filters
├── library/                 # Module library
├── roles/                   # Roles
│   ├── common/
│   ├── webserver/
│   └── database/
├── collections/
│   └── requirements.yml
├── playbooks/               # Playbooks
│   ├── site.yml
│   ├── apps/
│   └── backup/
├── files/                   # Static files
├── templates/               # Templates
└── vars/                    # Variable files
```

### Naming Conventions
```yaml
# Playbooks: action_target.yml
- deploy-app.yml
- configure-database.yml
- setup-monitoring.yml

# Roles: noun_component.yml
- nginx
- mysql
- postgresql
- redis
- docker
- firewall

# Variables: descriptive
http_port: 8080
ssl_certificate_path: /etc/ssl/certs/
admin_email: admin@example.com
```

### Performance Tips
```yaml
# ansible.cfg
[defaults]
# Increase forks for parallel execution
forks = 50

# Disable gathering facts when not needed
gathering = explicit

# Pipelining for faster SSH
pipelining = True

# Increase host key checking timeout
host_key_checking = False

# Control persistent connections
[ssh_connection]
pipelining = True
ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s
```

### Security Best Practices
```bash
# Use vault for secrets
ansible-vault create group_vars/all/secrets.yml

# Don't commit secrets to git
# .gitignore
*.pem
*.key
secrets.yml
vault_pass

# Use SSH keys, not passwords
ssh-keygen -t ed25519 -C "ansible@host"

# Limit sudo access
# /etc/sudoers
%ansible ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx
```

---

## 15. Quick Reference

### Essential Commands
| Task | Command |
|------|---------|
| Run playbook | `ansible-playbook site.yml` |
| Check mode | `ansible-playbook site.yml --check` |
| Check syntax | `ansible-playbook site.yml --syntax-check` |
| Limit hosts | `ansible-playbook site.yml -l web1` |
| Dry run | `ansible-playbook site.yml --check --diff` |
| Verbose | `ansible-playbook site.yml -v` |
| More verbose | `ansible-playbook site.yml -vvv` |
| Inventory | `ansible-inventory -i inventory.yml --list` |
| Ad-hoc ping | `ansible all -m ping` |
| Ad-hoc command | `ansible all -a "command"` |

### Module Quick Reference
| Module | Description | Example |
|--------|-------------|---------|
| **apt** | APT packages | `apt: name=nginx state=present` |
| **yum** | YUM packages | `yum: name=nginx state=present` |
| **copy** | Copy files | `copy: src=file dest=file` |
| **template** | Template files | `template: src=conf.j2 dest=conf` |
| **file** | File attributes | `file: path=/dir state=directory` |
| **service** | Service management | `service: name=nginx state=started` |
| **user** | User management | `user: name=user state=present` |
| **group** | Group management | `group: name=admin state=present` |
| **command** | Execute command | `command: ls /tmp` |
| **shell** | Shell command | `shell: ls | grep .py` |
| **debug** | Print message | `debug: msg=Hello` |
| **include** | Include tasks | `include: tasks.yml` |
| **import** | Import tasks | `import_playbook: playbook.yml` |
| **set_fact** | Set variable | `set_fact: var=value` |
| **register** | Capture output | `register: result` |

### Configuration File
```ini
# ansible.cfg
[defaults]
inventory = inventory.yml
roles_path = roles:/etc/ansible/roles
host_key_checking = False
pipelining = True
forks = 50
gathering = explicit
deprecation_warnings = False
stdout_callback = yaml

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
pipelining = True
ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s
```

### Variable Filters Quick Reference
| Filter | Example | Result |
|--------|---------|--------|
| `default` | `{{ x \| default(0) }}` | 0 if x undefined |
| `upper` | `{{ name \| upper }}` | UPPERCASE |
| `lower` | `{{ name \| lower }}` | lowercase |
| `capitalize` | `{{ name \| capitalize }}` | Capitalized |
| `first` | `{{ list \| first }}` | First item |
| `last` | `{{ list \| last }}` | Last item |
| `join` | `{{ list \| join(',') }}` | Joined string |
| `length` | `{{ list \| length }}` | Count |
| `replace` | `{{ s \| replace('a','b') }}` | Replaced |
| `regex_replace` | `{{ s \| regex_replace('pattern','repl') }}` | Regex replace |
| `to_yaml` | `{{ data \| to_yaml }}` | YAML string |
| `to_json` | `{{ data \| to_json }}` | JSON string |
| `from_yaml` | `{{ yaml \| from_yaml }}` | YAML dict |

### Exit Codes
| Code | Meaning |
|------|---------|
| 0 | Successful |
| 1 | Error |
| 2 | Unreachable host |
| 3 | Parse error |
| 4 | Command line error |

---

*Last Updated: January 2026*
*Generated for Ansible 2.17.x*
