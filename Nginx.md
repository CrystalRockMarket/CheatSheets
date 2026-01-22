# Nginx

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Core Concepts](#core-concepts)
4. [Configuration Structure](#configuration-structure)
5. [Server Blocks](#server-blocks)
6. [Location Blocks](#location-blocks)
7. [Load Balancing](#load-balancing)
8. [SSL/TLS](#ssltls)
9. [Security](#security)
10. [Performance](#performance)
11. [Common Patterns](#common-patterns)
12. [Commands and Management](#commands-and-management)
13. [Quick Reference](#quick-reference)

---

## Introduction

Nginx (engine x) is a high-performance HTTP server, reverse proxy, and load balancer with event-driven, non-blocking architecture.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Nginx Architecture                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    ┌─────────────────────────────────────────────────────────┐  │
│    │                     Master Process                       │  │
│    │   - Reads configuration                                 │  │
│    │   - Spawns worker processes                             │  │
│    │   - Reloads configuration                               │  │
│    └────────────────────────┬────────────────────────────────┘  │
│                             │                                      │
│         ┌───────────────────┼───────────────────┐                  │
│         ▼                   ▼                   ▼                  │
│    ┌─────────┐         ┌─────────┐         ┌─────────┐            │
│    │ Worker  │         │ Worker  │         │ Worker  │            │
│    │ Process │         │ Process │         │ Process │            │
│    │         │         │         │         │         │            │
│    │  Event  │         │  Event  │         │  Event  │            │
│    │  Loop   │         │  Loop   │         │  Loop   │            │
│    └────┬────┘         └────┬────┘         └────┬────┘            │
│         │                   │                   │                   │
│         └───────────────────┴───────────────────┘                   │
│                             │                                      │
│         ┌───────────────────┴───────────────────┐                  │
│         ▼                   ▼                   ▼                  │
│    ┌─────────┐         ┌─────────┐         ┌─────────┐            │
│    │  Cache  │         │   Fast   │         │  Proxy  │            │
│    │  (disk) │         │CGI, etc │         │ Connection│          │
│    └─────────┘         └─────────┘         └─────────┘            │
│                                                                 │
│    Worker handles thousands of connections per process           │
│    Non-blocking I/O with epoll (Linux), kqueue (BSD)            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Request Processing Flow

```
Client Request
      │
      ▼
┌─────────────┐
│   Listen    │  Listen on port 80/443
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Server    │  Match server block (server_name)
│   Matching  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Location   │  Match location block
│  Matching   │  (prefix, regex, =)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Handler   │  Process request
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Response  │  Send to client
└─────────────┘
```

---

## Installation

### Package Installation

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install nginx

# RHEL/CentOS/Fedora
sudo yum install epel-release
sudo yum install nginx
sudo systemctl enable --now nginx

# Arch Linux
sudo pacman -S nginx

# macOS
brew install nginx

# Start service
sudo systemctl start nginx
sudo systemctl enable nginx
```

### From Source

```bash
# Install dependencies
sudo apt install build-essential libpcre3-dev zlib1g-dev libssl-dev

# Download and compile
wget https://nginx.org/download/nginx-1.25.0.tar.gz
tar -xzf nginx-1.25.0.tar.gz
cd nginx-1.25.0
./configure --prefix=/usr/local/nginx --with-http_ssl_module
make
sudo make install
```

### Verify Installation

```bash
# Check version
nginx -v
nginx -V

# Test configuration
sudo nginx -t

# Check status
sudo systemctl status nginx

# Test default page
curl http://localhost
```

---

## Core Concepts

### Directives and Contexts

```nginx
# Main context (global settings)
worker_processes auto;
error_log /var/log/nginx/error.log;

# Events context (connection processing)
events {
    worker_connections 1024;
}

# HTTP context (web server)
http {
    include /etc/nginx/mime.types;
    
    # Server context (virtual host)
    server {
        listen 80;
        server_name example.com;
        
        # Location context (path matching)
        location / {
            root /var/www/html;
            index index.html;
        }
    }
}
```

### Directive Hierarchy

| Context | Description | Children |
|---------|-------------|----------|
| `main` | Global settings | events, http, mail |
| `events` | Connection settings | - |
| `http` | HTTP settings | server, upstream |
| `server` | Virtual host | location |
| `location` | Path handling | location |

### Key Directives

```nginx
# Worker processes
worker_processes auto;          # Number of workers
worker_processes 4;             # Fixed number
worker_processes $((2*$(nproc)));  # Dynamic

# Connection handling
worker_connections 1024;        # Max connections per worker
multi_accept on;                # Accept multiple connections
use epoll;                      # Use efficient method

# File handling
sendfile on;
tcp_nopush on;
tcp_nodelay on;
open_file_cache max=1000 inactive=20s;
open_file_cache_valid 30s;
open_file_cache_min_uses 2;
```

---

## Configuration Structure

### Main Configuration File

```nginx
# /etc/nginx/nginx.conf
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;

events {
    worker_connections 768;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### Include Files

```nginx
# MIME types
include /etc/nginx/mime.types;

# Site configurations
include /etc/nginx/sites-enabled/*;

# Additional config
include /etc/nginx/conf.d/*.conf;

# Snippets
include /etc/nginx/snippets/ssl-params.conf;
```

### mime.types

```nginx
types {
    text/html                             html htm shtml;
    text/css                              css;
    text/xml                              xml;
    image/gif                             gif;
    image/jpeg                            jpg jpeg;
    application/javascript                js;
    application/json                      json;
    application/pdf                       pdf;
    application/zip                       zip;
    audio/mpeg                            mp3;
    video/mp4                             mp4;
}
```

---

## Server Blocks

### Basic Server Block

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    
    root /var/www/html;
    index index.html index.htm;
    
    access_log /var/log/nginx/example.com_access.log;
    error_log /var/log/nginx/example.com_error.log;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Server Name Matching

```nginx
# Exact match
server_name example.com;

# Wildcard
server_name *.example.com;

# Regex (case insensitive)
server_name ~^www\.(.+)\.com$;

# Multiple names
server_name example.com www.example.com *.example.com;

# Default catch-all
server_name _;

# Catch-all for any domain
server_name "";
```

### Complete Server Example

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    
    root /var/www/example.com/html;
    index index.html index.htm;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # Logs
    access_log /var/log/nginx/example.com_access.log combined;
    error_log /var/log/nginx/example.com_error.log warn;
    
    # Root and index
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # Deny access to hidden files
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
}
```

### Redirects

```nginx
# HTTP to HTTPS redirect
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}

# www to non-www
server {
    listen 80;
    server_name www.example.com;
    return 301 http://example.com$request_uri;
}

# Custom redirect
server {
    listen 80;
    server_name old.example.com;
    return 301 https://new.example.com$request_uri;
}
```

---

## Location Blocks

### Location Matching

```nginx
# Prefix match (most common)
location / { }
location /api/ { }
location /static/ { }

# Exact match
location = /api/status { }

# Regex match (case sensitive)
location ~ \.php$ { }

# Regex match (case insensitive)
location ~* \.(jpg|jpeg|png|gif)$ { }

# Negative lookahead
location !~ \.exe$ { }

# Preferential prefix
location ^~ /admin/ { }
```

### Location Directive Options

```nginx
location / {
    # Document root
    root /var/www/html;
    
    # Default file
    index index.html index.htm;
    
    # Try files
    try_files $uri $uri/ /index.php?$query_string;
    
    # Allow/Deny
    allow 192.168.1.0/24;
    deny all;
    
    # Timeouts
    client_max_body_size 16M;
    client_body_timeout 60;
    client_header_timeout 60;
}

location /api/ {
    # Proxy settings
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

location ~ \.php$ {
    # FastCGI pass
    fastcgi_pass unix:/var/run/php/php-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

### Root vs Alias

```nginx
# root: appends location to path
location /static/ {
    root /var/www;
    # Serves /var/www/static/
}

# alias: replaces location with path
location /static/ {
    alias /var/www/static-files/;
    # Serves /var/www/static-files/
}
```

---

## Load Balancing

### Upstream Blocks

```nginx
upstream backend {
    least_conn;  # Least connections
    server backend1.example.com weight=5;
    server backend2.example.com;
    server 192.168.1.100:8080;
    server 192.168.1.101:8080 backup;
    server 192.168.1.102:8080 down;
}

upstream api_servers {
    ip_hash;  # Sticky sessions
    server api1.example.com:8080;
    server api2.example.com:8080;
    server api3.example.com:8080;
}
```

### Load Balancing Methods

```nginx
# Round Robin (default)
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
}

# Least Connections
upstream backend {
    least_conn;
    server backend1.example.com;
    server backend2.example.com;
}

# IP Hash (sticky sessions)
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
}

# Weighted
upstream backend {
    server backend1.example.com weight=5;
    server backend2.example.com weight=1;
}

# Random
upstream backend {
    random;
    server backend1.example.com;
    server backend2.example.com;
}
```

### Using Upstream

```nginx
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Health Checks

```nginx
upstream backend {
    server backend1.example.com max_fails=3 fail_timeout=30s;
    server backend2.example.com max_fails=3 fail_timeout=30s;
    server backend3.example.com backup;
}

# Custom health check (ngx_http_upstream_module)
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    
    health_check;
    health_check interval=5s fails=3 passes=2 uri=/health;
}
```

---

## SSL/TLS

### SSL Configuration

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com;
    
    # SSL Certificate
    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    
    # SSL Configuration
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;
    
    # Modern SSL configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;
    
    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options DENY always;
    add_header X-Content-Type-Options nosniff always;
    add_header X-XSS-Protection "1; mode=block" always;
}
```

### SSL Snippet (ssl-params.conf)

```nginx
# /etc/nginx/snippets/ssl-params.conf
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers off;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
ssl_ecdh_curve secp384r1;
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;
ssl_session_tickets off;

# OCSP Stapling
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;

# Security headers
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
add_header X-Frame-Options DENY always;
add_header X-Content-Type-Options nosniff always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
```

### HTTP to HTTPS Redirect

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    
    # HSTS preload (optional)
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    
    return 301 https://$host$request_uri;
}
```

---

## Security

### Basic Security

```nginx
# Hide nginx version
server_tokens off;

# Hide server header
more_set_headers "Server:";

# Disable content-type sniffing
add_header X-Content-Type-Options "nosniff" always;

# X-Frame-Options
add_header X-Frame-Options "SAMEORIGIN" always;

# XSS Protection
add_header X-XSS-Protection "1; mode=block" always;

# Referrer Policy
add_header Referrer-Policy "strict-origin-when-cross-origin" always;

# Permissions Policy
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;

# CSP
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' https://cdn.example.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; img-src 'self' data: https:; font-src 'self' https://fonts.gstatic.com; connect-src 'self' https://api.example.com; frame-ancestors 'self';" always;
```

### Rate Limiting

```nginx
# Define rate limit zone
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

# Apply rate limiting
location /api/ {
    limit_req zone=api burst=20 nodelay;
    proxy_pass http://backend;
}

# Connection limiting
limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

location /download/ {
    limit_conn conn_limit 1;
    limit_rate 100k;
}
```

### Access Control

```nginx
# Allow by IP
location /admin/ {
    allow 192.168.1.0/24;
    allow 10.0.0.0/8;
    deny all;
}

# Basic auth
location /private/ {
    auth_basic "Restricted Area";
    auth_basic_user_file /etc/nginx/.htpasswd;
}

# Generate htpasswd
htpasswd -c /etc/nginx/.htpasswd admin
htpasswd /etc/nginx/.htpasswd user2
```

### Deny Access

```nginx
# Deny hidden files
location ~ /\. {
    deny all;
    access_log off;
    log_not_found off;
}

# Deny specific files
location = /wp-config.php {
    deny all;
}

# Deny specific patterns
location ~* \.(env|git|htaccess|yml)$ {
    deny all;
}
```

---

## Performance

### Gzip Compression

```nginx
gzip on;
gzip_vary on;
gzip_proxied any;
gzip_comp_level 6;
gzip_min_length 256;
gzip_types
    text/plain
    text/css
    text/xml
    text/javascript
    application/javascript
    application/json
    application/xml
    application/xml+rss
    image/svg+xml;
```

### Caching

```nginx
# Static file caching
location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2|ttf|eot)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
    access_log off;
}

# CSS/JS with longer cache
location ~* \.(css|js)$ {
    expires 7d;
    add_header Cache-Control "public, immutable";
}

# HTML no cache
location ~* \.html$ {
    expires -1;
    add_header Cache-Control "no-store, no-cache, must-revalidate";
}
```

### Buffer and Timeout Settings

```nginx
http {
    # Buffer sizes
    client_body_buffer_size 16K;
    client_max_body_size 100M;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 16k;
    
    # Timeouts
    client_body_timeout 60;
    client_header_timeout 60;
    keepalive_timeout 65;
    keepalive_requests 100;
    send_timeout 60;
    
    # File handling
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    
    # Connection handling
    worker_rlimit_nofile 65535;
}
```

### FastCGI (PHP-FPM)

```nginx
location ~ \.php$ {
    fastcgi_pass unix:/var/run/php/php-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    
    # FastCGI settings
    fastcgi_connect_timeout 60s;
    fastcgi_send_timeout 60s;
    fastcgi_read_timeout 60s;
    fastcgi_buffer_size 128k;
    fastcgi_buffers 4 256k;
    fastcgi_busy_buffers_size 256k;
}
```

---

## Common Patterns

### WordPress

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/wordpress;
    index index.php;
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 30d;
    }
    
    location = /favicon.ico {
        access_log off;
        log_not_found off;
    }
}
```

### Node.js Proxy

```nginx
server {
    listen 80;
    server_name nodeapp.example.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Static Files with Fallback

```nginx
server {
    listen 80;
    server_name app.example.com;
    root /var/www/app/current;
    index index.html;
    
    # Static assets with long cache
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }
    
    # SPA routing - serve index.html for all non-file routes
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### Redirect All to Main Domain

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://example.com$request_uri;
}

server {
    listen 443 ssl http2;
    server_name www.example.com;
    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    return 301 https://example.com$request_uri;
}
```

---

## Commands and Management

### Service Commands

```bash
# Start nginx
sudo systemctl start nginx

# Stop nginx
sudo systemctl stop nginx

# Restart nginx
sudo systemctl restart nginx

# Reload configuration (graceful)
sudo systemctl reload nginx

# Check configuration
sudo nginx -t

# Test configuration with details
sudo nginx -T

# Reload without restart
sudo nginx -s reload

# Stop gracefully
sudo nginx -s quit

# Stop immediately
sudo nginx -s stop

# Reopen log files
sudo nginx -s reopen
```

### Signal Control

```bash
# Send signal directly
kill -HUP $(cat /var/run/nginx.pid)

# Test configuration
kill -QUIT $(cat /var/run/nginx.pid)

# Roll log files
kill -USR1 $(cat /var/run/nginx.pid)

# Upgrade binary
kill -USR2 $(cat /var/run/nginx.pid)
```

### Log Management

```bash
# Default log
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# Custom log format
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for"';

# Access log with custom format
access_log /var/log/nginx/access.log main;

# Buffer logs
access_log /var/log/nginx/access.log main buffer=16k flush=5s;
```

---

## Quick Reference

### Common Directives

| Directive | Description | Default |
|-----------|-------------|---------|
| `listen` | Listen address/port | 80 |
| `server_name` | Domain name | - |
| `root` | Document root | html |
| `index` | Default index file | index.html |
| `location` | Path matching | - |
| `proxy_pass` | Backend URL | - |
| `fastcgi_pass` | PHP-FPM socket | - |
| `try_files` | File existence check | - |
| `return` | Return response code | - |
| `rewrite` | URL rewriting | - |

### Common Patterns

```nginx
# Redirect
return 301 $scheme://example.com$request_uri;

# Rewrite
rewrite ^/old/(.*)$ /new/$1 permanent;

# Proxy
proxy_pass http://backend;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;

# Cache
expires 7d;
add_header Cache-Control "public";
```

### File Locations

| Path | Purpose |
|------|---------|
| `/etc/nginx/nginx.conf` | Main config |
| `/etc/nginx/sites-available/` | Site configs |
| `/etc/nginx/sites-enabled/` | Enabled sites |
| `/etc/nginx/snippets/` | Reusable snippets |
| `/var/log/nginx/access.log` | Access log |
| `/var/log/nginx/error.log` | Error log |
| `/usr/share/nginx/html/` | Default content |

### Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |

---

## See Also

- [Nginx Documentation](https://nginx.org/en/docs/)
- [Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html)
- [Admin Guide](https://nginx.org/en/docs/admin-guide.html)
- [SSL Configuration](https://ssl-config.mozilla.org/)
