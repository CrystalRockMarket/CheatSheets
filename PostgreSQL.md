# PostgreSQL (psql)

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Connecting](#connecting)
4. [Basic Commands](#basic-commands)
5. [Database Operations](#database-operations)
6. [Table Operations](#table-operations)
7. [Data Manipulation](#data-manipulation)
8. [Users and Roles](#users-and-roles)
9. [Backups and Restore](#backups-and-restore)
10. [Performance](#performance)
11. [Troubleshooting](#troubleshooting)
12. [Quick Reference](#quick-reference)

---

## Introduction

PostgreSQL is a powerful, open-source object-relational database system known for its reliability, feature robustness, and standards compliance.

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     PostgreSQL Architecture                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      Client Applications                   │  │
│  │    (psql, pgAdmin, applications, ORMs, tools)             │  │
│  └────────────────────────┬────────────────────────────────┘   │
│                           │                                      │
│                           ▼                                      │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                      PostgreSQL Server                     │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌───────────────────┐  │  │
│  │  │   Parser    │  │  Rewriter   │  │    Planner        │  │  │
│  │  └─────────────┘  └─────────────┘  └───────────────────┘  │  │
│  │         │               │                  │               │  │
│  │         ▼               ▼                  ▼               │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │                    Executor                          │  │  │
│  │  │  (Sequential Scan, Index Scan, Nested Loop, etc.)   │  │  │
│  │  └────────────────────────┬───────────────────────────┘  │  │
│  └───────────────────────────┼──────────────────────────────┘  │
│                              │                                    │
│              ┌───────────────┴───────────────┐                   │
│              ▼                               ▼                   │
│    ┌─────────────────┐           ┌─────────────────┐             │
│    │  Shared Memory  │           │   Disk Storage  │             │
│    │  (buffers, WAL) │           │  (data files,   │             │
│    └─────────────────┘           │   logs, xlog)   │             │
│                                  └─────────────────┘             │
│                                                                  │
│  Process Model:                                                  │
│  - Postmaster (master process)                                  │
│  - Backend processes (one per connection)                       │
│  - Background processes (WAL writer, autovacuum, etc.)          │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Features

- ACID compliance
- Complex queries (JOINs, subqueries, CTEs)
- Foreign keys, triggers, views
- Stored procedures in multiple languages
- Full-text search
- JSON/JSONB support
- Spatial data (PostGIS)
- Replication (streaming, logical)
- Extensions support

---

## Installation

### Package Installation

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install postgresql postgresql-contrib

# RHEL/CentOS/Fedora
sudo yum install postgresql-server postgresql-contrib
sudo postgresql-setup initdb
sudo systemctl enable --now postgresql

# Arch Linux
sudo pacman -S postgresql

# macOS
brew install postgresql@15
brew services start postgresql@15

# Verify installation
psql --version
```

### Initial Setup

```bash
# Switch to postgres user
sudo -i -u postgres
psql

# Or connect directly
sudo -u postgres psql

# Create user and database
sudo -u postgres createuser myuser
sudo -u postgres createdb mydb
sudo -u postgres psql -c "ALTER USER myuser WITH PASSWORD 'secret';"
```

### Configuration Files

| File | Purpose |
|------|---------|
| `postgresql.conf` | Main configuration |
| `pg_hba.conf` | Client authentication |
| `pg_ident.conf` | User name mapping |

```bash
# Common locations
/etc/postgresql/15/main/postgresql.conf
/etc/postgresql/15/main/pg_hba.conf
/var/lib/postgresql/data/postgresql.conf
```

---

## Connecting

### Connection Methods

```bash
# Local connection as postgres user
sudo -u postgres psql

# Connect as specific user
psql -U username -d database
psql --username=username --dbname=database

# Connect to remote host
psql -h hostname -p 5432 -U username -d database

# Using socket
psql -h /var/run/postgresql -U username -d database

# Connection string
psql "postgresql://user:pass@host:5432/dbname"

# From environment variables
export PGHOST=localhost
export PGUSER=username
export PGDATABASE=dbname
export PGPASSWORD=secret
psql
```

### Connection Parameters

| Parameter | Short | Description |
|-----------|-------|-------------|
| `--host` | `-h` | Server hostname |
| `--port` | `-p` | Port number |
| `--username` | `-U` | Username |
| `--dbname` | `-d` | Database name |
| `--command` | `-c` | Execute command |
| `--file` | `-f` | Execute from file |
| `--quiet` | `-q` | Quiet mode |
| `--echo-queries` | `-e` | Echo queries |
| `--pset` | `--pset` | Set output format |

### psql Meta-Commands

```sql
-- Help
\h                 -- SQL help
\h CREATE TABLE    -- Help on specific command
\?                 -- psql help
\? commands        -- Backslash commands

-- Connection
\c database        -- Connect to database
\c user database   -- Connect as user
\conninfo          -- Connection info

-- Output format
\x                 -- Toggle expanded display
\a                 -- Toggle unaligned mode
\pset format       -- Set format (aligned, unaligned, html, latex)
\pset tuples_only  -- Show only tuples
\pset fieldsep     -- Field separator
\pset border       -- Border style

-- Query buffer
\e                 -- Edit in editor
\ef function       -- Edit function
\p                 -- Print buffer
\r                 -- Reset buffer
\g [file]          -- Execute and save to file
\gdesc             -- Describe query result
\watch [seconds]   -- Repeat query
```

---

## Basic Commands

### Getting Information

```sql
-- List databases
\l
\list
\l+

-- List tables
\dt
\dt+
\dt schema.*

-- List views
\dv
\dv+

-- List indexes
\di
\di+

-- List sequences
\ds
\ds+

-- List functions
\df
\df+

-- List schemas
\dn
\dn+

-- List users/roles
\du
\du+

-- Show table structure
\d table_name
\d+ table_name

-- Show index definition
\d index_name

-- Show function definition
\df+ function_name

-- Show view definition
\sv view_name
```

### Database Operations

```sql
-- Create database
CREATE DATABASE dbname;

-- Create database with owner
CREATE DATABASE dbname OWNER username;

-- Create database with template
CREATE DATABASE dbname TEMPLATE template0;

-- Create database with encoding
CREATE DATABASE dbname ENCODING 'UTF8' LC_COLLATE='en_US.utf8';

-- Connect to database
\c dbname

-- Rename database
ALTER DATABASE dbname RENAME TO newname;

-- Drop database
DROP DATABASE dbname;
DROP DATABASE IF EXISTS dbname;

-- Set default schema
SET search_path TO schema_name, public;
```

### Schema Operations

```sql
-- Create schema
CREATE SCHEMA schemaname;

-- Create schema with owner
CREATE SCHEMA schemaname AUTHORIZATION username;

-- Drop schema
DROP SCHEMA schemaname;
DROP SCHEMA IF EXISTS schemaname CASCADE;

-- Drop all objects in schema
DROP SCHEMA schemaname CASCADE;
```

---

## Table Operations

### Create Table

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash CHAR(60) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE
);

-- Create table with constraints
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    price DECIMAL(10,2) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT positive_price CHECK (price >= 0)
);

-- Create table with foreign key actions
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(id) ON DELETE CASCADE,
    product_id INTEGER REFERENCES products(id) ON DELETE RESTRICT,
    quantity INTEGER NOT NULL
);

-- Temporary table
CREATE TEMP TABLE temp_data (
    id SERIAL,
    value TEXT
);

-- Unlogged table (not WAL logged, faster)
CREATE UNLOGGED TABLE analytics_data (
    id SERIAL,
    data JSONB
);
```

### Table Modifications

```sql
-- Add column
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
ALTER TABLE users ADD COLUMN IF NOT EXISTS address TEXT;

-- Drop column
ALTER TABLE users DROP COLUMN phone;
ALTER TABLE users DROP COLUMN IF EXISTS phone CASCADE;

-- Rename column
ALTER TABLE users RENAME COLUMN username TO user_name;

-- Change column type
ALTER TABLE users ALTER COLUMN email TYPE VARCHAR(150);

-- Add constraint
ALTER TABLE users ADD CONSTRAINT users_email_key UNIQUE (email);
ALTER TABLE users ADD CONSTRAINT users_is_active CHECK (is_active IN (TRUE, FALSE));

-- Drop constraint
ALTER TABLE users DROP CONSTRAINT users_email_key;

-- Set default value
ALTER TABLE users ALTER COLUMN is_active SET DEFAULT FALSE;

-- Drop default
ALTER TABLE users ALTER COLUMN is_active DROP DEFAULT;

-- Set NOT NULL
ALTER TABLE users ALTER COLUMN email SET NOT NULL;

-- Drop NOT NULL
ALTER TABLE users ALTER COLUMN email DROP NOT NULL;

-- Rename table
ALTER TABLE users RENAME TO customers;
```

### Index Operations

```sql
-- Create index
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Create unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Create composite index
CREATE INDEX idx_orders_user_product ON orders(user_id, product_id);

-- Create index with conditions
CREATE INDEX idx_users_active ON users(email) WHERE is_active = TRUE;

-- Create index using specific method
CREATE INDEX idx_users_name ON users USING gin (username gin_trgm_ops);

-- Create index concurrently (no lock)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- Drop index
DROP INDEX idx_users_email;
DROP INDEX CONCURRENTLY IF EXISTS idx_users_email;

-- Reindex
REINDEX TABLE users;
REINDEX INDEX idx_users_email;

-- Index information
\d users
SELECT * FROM pg_indexes WHERE tablename = 'users';
```

### Table Maintenance

```sql
-- Analyze table (update statistics)
ANALYZE users;

-- Vacuum table (reclaim space)
VACUUM users;

-- Vacuum with full (exclusive lock)
VACUUM FULL users;

-- Vacuum with analyze
VACUUM ANALYZE users;

-- Autovacuum settings
ALTER TABLE users SET (autovacuum_enabled = true, 
                      autovacuum_vacuum_threshold = 50);

-- Check table size
SELECT pg_size_pretty(pg_total_relation_size('users'));
SELECT pg_size_pretty(pg_relation_size('users'));

-- List table sizes
SELECT relname, pg_size_pretty(pg_relation_size(relid))
FROM pg_stat_user_tables
ORDER BY pg_relation_size(relid) DESC;
```

---

## Data Manipulation

### INSERT

```sql
-- Insert single row
INSERT INTO users (username, email, password_hash)
VALUES ('john', 'john@example.com', 'hash123');

-- Insert multiple rows
INSERT INTO users (username, email, password_hash) VALUES
    ('alice', 'alice@example.com', 'hash1'),
    ('bob', 'bob@example.com', 'hash2'),
    ('charlie', 'charlie@example.com', 'hash3');

-- Insert from select
INSERT INTO active_users (username, email)
SELECT username, email FROM users WHERE is_active = TRUE;

-- Insert with on conflict
INSERT INTO users (username, email, password_hash)
VALUES ('john', 'john@example.com', 'new_hash')
ON CONFLICT (username) 
DO UPDATE SET email = EXCLUDED.email, 
              password_hash = EXCLUDED.password_hash,
              updated_at = CURRENT_TIMESTAMP;

-- Insert with returning
INSERT INTO users (username, email) VALUES ('dave', 'dave@example.com')
RETURNING id, username;
```

### SELECT

```sql
-- Basic select
SELECT * FROM users;
SELECT username, email FROM users;

-- With where clause
SELECT * FROM users WHERE is_active = TRUE;
SELECT * FROM users WHERE id IN (1, 2, 3);
SELECT * FROM users WHERE username LIKE 'j%';
SELECT * FROM users WHERE created_at > '2024-01-01';

-- With operators
SELECT * FROM users WHERE id BETWEEN 10 AND 20;
SELECT * FROM users WHERE email IS NULL;
SELECT * FROM users WHERE is_active IS NOT FALSE;

-- With order by
SELECT * FROM users ORDER BY created_at DESC;
SELECT * FROM users ORDER BY username ASC NULLS LAST;

-- With limit/offset
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;
SELECT * FROM users OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY;

-- With distinct
SELECT DISTINCT status FROM orders;
SELECT DISTINCT ON (status) * FROM orders ORDER BY status, created_at DESC;

-- With aggregates
SELECT COUNT(*) FROM users;
SELECT COUNT(DISTINCT user_id) FROM orders;
SELECT SUM(amount) FROM orders;
SELECT AVG(amount) FROM orders;
SELECT MIN(amount), MAX(amount) FROM orders;
SELECT status, COUNT(*) FROM orders GROUP BY status;
SELECT status, COUNT(*) FROM orders GROUP BY status HAVING COUNT(*) > 10;

-- With joins
SELECT u.username, o.id, o.total 
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

SELECT u.username, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;

SELECT * FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

SELECT * FROM users u
CROSS JOIN (SELECT generate_series(1,5) as n) n;

-- With subquery
SELECT * FROM users 
WHERE id IN (SELECT user_id FROM orders WHERE total > 100);

SELECT u.username, 
       (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) as order_count
FROM users u;

-- With CTE
WITH active_orders AS (
    SELECT user_id, COUNT(*) as order_count
    FROM orders WHERE status = 'completed'
    GROUP BY user_id
)
SELECT u.username, ao.order_count
FROM users u
LEFT JOIN active_orders ao ON u.id = ao.user_id;

-- With window functions
SELECT id, username,
       ROW_NUMBER() OVER (ORDER BY created_at DESC) as rn,
       RANK() OVER (ORDER BY id) as rank
FROM users;

SELECT id, status, amount,
       LAG(amount) OVER (PARTITION BY user_id ORDER BY created_at) as prev_amount,
       amount - LAG(amount) OVER (PARTITION BY user_id ORDER BY created_at) as diff
FROM orders;
```

### UPDATE

```sql
-- Basic update
UPDATE users SET is_active = FALSE WHERE id = 1;

-- Update multiple columns
UPDATE users SET 
    email = 'new@example.com',
    updated_at = CURRENT_TIMESTAMP
WHERE id = 1;

-- Update with returning
UPDATE users SET last_login = CURRENT_TIMESTAMP 
WHERE id = 1
RETURNING id, username, last_login;

-- Update with join
UPDATE orders o
SET status = 'cancelled'
FROM users u
WHERE o.user_id = u.id AND u.is_active = FALSE;

-- Update with subquery
UPDATE users u
SET last_login = (SELECT MAX(created_at) FROM orders WHERE user_id = u.id)
WHERE u.id = 1;
```

### DELETE

```sql
-- Basic delete
DELETE FROM users WHERE id = 1;

-- Delete with returning
DELETE FROM orders WHERE status = 'cancelled' RETURNING id, status;

-- Delete duplicates
DELETE FROM users a USING users b 
WHERE a.ctid < b.ctid AND a.email = b.email;

-- Truncate table
TRUNCATE users RESTART IDENTITY;
TRUNCATE users CASCADE;  -- Also truncate tables with foreign keys
```

---

## Users and Roles

### Create Users/Roles

```sql
-- Create login role (user)
CREATE ROLE username WITH LOGIN PASSWORD 'secret';

-- Create role with privileges
CREATE ROLE username WITH 
    LOGIN PASSWORD 'secret'
    CREATEDB CREATEROLE
    VALID UNTIL '2025-01-01';

-- Create superuser
CREATE ROLE username WITH SUPERUSER PASSWORD 'secret';

-- Create read-only user
CREATE ROLE readonly WITH LOGIN PASSWORD 'secret';
GRANT CONNECT ON DATABASE dbname TO readonly;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA public TO readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO readonly;

-- Create read-write user
CREATE ROLE readwrite WITH LOGIN PASSWORD 'secret';
GRANT CONNECT ON DATABASE dbname TO readwrite;
GRANT USAGE, CREATE ON SCHEMA public TO readwrite;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO readwrite;
```

### Modify Roles

```sql
-- Change password
ALTER USER username WITH PASSWORD 'newsecret';

-- Set role expiration
ALTER USER username VALID UNTIL '2025-12-31';

-- Lock/unlock user
ALTER USER username WITH LOGIN;     -- Enable login
ALTER USER username WITH NOLOGIN;   -- Disable login

-- Create superuser
ALTER USER username WITH SUPERUSER;

-- Remove superuser
ALTER USER username WITH NOSUPERUSER;

-- Set role attributes
ALTER USER username CREATEDB;
ALTER USER username NOCREATEDB;
ALTER USER username CREATEROLE;
ALTER USER username INHERIT;
ALTER USER username NOINHERIT;
```

### Permissions

```sql
-- Grant privileges
GRANT SELECT ON TABLE users TO readonly;
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE users TO readwrite;
GRANT ALL ON TABLE users TO admin;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT CREATE ON SCHEMA public TO readwrite;

-- Grant with grant option
GRANT SELECT ON TABLE users TO readonly WITH GRANT OPTION;

-- Revoke privileges
REVOKE SELECT ON TABLE users FROM readonly;
REVOKE ALL ON TABLE users FROM PUBLIC;

-- Set default privileges
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
    GRANT SELECT ON TABLES TO readonly;

-- Check permissions
\dp users
SELECT * FROM information_schema.table_privileges 
WHERE grantee = 'username';
```

### Drop Roles

```sql
-- Drop role
DROP ROLE username;

-- Drop role if exists
DROP ROLE IF EXISTS username;

-- Reassign owned objects
REASSIGN OWNED BY username TO new_owner;

-- Drop owned objects
DROP OWNED BY username CASCADE;
```

---

## Backups and Restore

### pg_dump

```bash
# Dump single database
pg_dump -U username -d dbname > dbname.sql

# Dump with compression
pg_dump -U username -d dbname | gzip > dbname.sql.gz

# Dump specific tables
pg_dump -U username -d dbname -t users -t orders > tables.sql

# Dump schema only
pg_dump -U username -d dbname --schema-only > schema.sql

# Dump data only
pg_dump -U username -d dbname --data-only > data.sql

# Dump with INSERT instead of COPY
pg_dump -U username -d dbname --inserts > insert.sql

# Dump with clean (DROP) commands
pg_dump -U username -d dbname -c > with_drop.sql

# Dump as custom format
pg_dump -U username -d dbname -Fc > dbname.dump

# Dump as directory (parallel)
pg_dump -U username -d dbname -Fd -j 4 -f backup_dir

# Exclude table data
pg_dump -U username -d dbname --exclude-table-data=logs > no_logs.sql
```

### pg_restore

```bash
# Restore from plain SQL
psql -U username -d dbname < dbname.sql

# Restore with drop
psql -U username -d newdb < dbname.sql

# Restore custom format
pg_restore -U username -d dbname dbname.dump

# Restore specific tables
pg_restore -U username -d dbname --table=users dbname.dump

# Restore with jobs (parallel)
pg_restore -U username -d dbname -j 4 dbname.dump

# Restore to different schema
pg_restore -U username -d dbname --schema=myschema dbname.dump

# Restore only schema
pg_restore -U username -d dbname --schema-only dbname.dump

# Restore only data
pg_restore -U username -d dbname --data-only dbname.dump
```

### Continuous Archiving (PITR)

```bash
# Configure WAL archiving in postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /var/lib/postgresql/wal_archive/%f'

# Base backup
pg_basebackup -D /backup/base -Ft -z -P

# Restore from WAL
# 1. Stop PostgreSQL
# 2. Clear data directory
# 3. Copy base backup
# 4. Create recovery signal
# 5. Configure recovery.conf
restore_command = 'cp /var/lib/postgresql/wal_archive/%f %p'
recovery_target_time = '2024-01-15 12:00:00'
# 6. Start PostgreSQL
```

### Backup Script

```bash
#!/bin/bash
# backup.sh
set -e

BACKUP_DIR="/backup/postgres"
DB_NAME="mydb"
DB_USER="postgres"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup
pg_dump -U $DB_USER -d $DB_NAME | gzip > "$BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz"

# Keep last 7 daily, 4 weekly, 12 monthly
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +7 -delete
find "$BACKUP_DIR" -name "*.sql.gz" -type f | sort -r | tail -n +5 | xargs -r rm
```

---

## Performance

### EXPLAIN

```sql
-- Explain query plan
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Explain with costs
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT) 
SELECT * FROM users WHERE email = 'test@example.com';

-- Explain analyze
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 1;

-- Output formats
EXPLAIN (FORMAT JSON) SELECT * FROM users;
EXPLAIN (FORMAT YAML) SELECT * FROM users;
```

### Query Optimization

```sql
-- Create index for query
CREATE INDEX idx_users_email ON users(email);

-- Check index usage
EXPLAIN (ANALYZE) SELECT * FROM users WHERE email = 'test@example.com';

-- Check for missing indexes
SELECT query, calls, total_time, mean_time 
FROM pg_stat_statements 
ORDER BY total_time DESC LIMIT 10;

-- Check slow queries
SELECT query, calls, mean_time 
FROM pg_stat_statements 
WHERE mean_time > 100 
ORDER BY mean_time DESC;

-- Check table statistics
SELECT * FROM pg_stat_user_tables WHERE relname = 'users';

-- Check index statistics
SELECT * FROM pg_stat_user_indexes WHERE relname = 'users';
```

### VACUUM and ANALYZE

```sql
-- Manual vacuum
VACUUM ANALYZE users;

-- Vacuum with verbose
VACUUM (VERBOSE, ANALYZE) users;

-- Check vacuum progress
SELECT * FROM pg_stat_progress_vacuum;

-- Check autovacuum settings
SELECT * FROM pg_settings WHERE name LIKE 'autovacuum%';

-- Set autovacuum
ALTER TABLE users SET (autovacuum_vacuum_scale_factor = 0.1);
```

### Connection Pooling

```bash
# Install PgBouncer
sudo apt install pgbouncer

# Configure pgbouncer.ini
[databases]
mydb = host=127.0.0.1 port=5432 dbname=mydb

[pgbouncer]
listen_addr = 127.0.0.1
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20
reserve_pool_size = 5
log_connections = 0
log_disconnections = 0
log_pooler_errors = 1

# Connect via PgBouncer
psql -h 127.0.0.1 -p 6432 -U user mydb
```

---

## Troubleshooting

### Connection Issues

```bash
# Check if PostgreSQL is running
sudo systemctl status postgresql

# Check listening ports
ss -tlnp | grep 5432

# Test connection
psql -h localhost -p 5432 -U postgres

# Check pg_hba.conf
sudo nano /etc/postgresql/15/main/pg_hba.conf

# Test from remote
psql -h server_ip -p 5432 -U user -d dbname

# Check firewall
sudo ufw status
```

### Common Errors

```sql
-- Duplicate key
ERROR: duplicate key value violates unique constraint
-- Solution: Use ON CONFLICT or check data

-- Foreign key violation
ERROR: insert or update on table violates foreign key constraint
-- Solution: Insert parent record first

-- Permission denied
ERROR: permission denied for table users
-- Solution: Grant permissions

-- Cannot connect
FATAL: no pg_hba.conf entry for host
-- Solution: Add entry to pg_hba.conf

-- Database does not exist
FATAL: database "dbname" does not exist
-- Solution: Create database first

-- Role does not exist
FATAL: role "username" does not exist
-- Solution: Create role first
```

### Log Analysis

```bash
# View PostgreSQL logs
tail -f /var/log/postgresql/postgresql-15-main.log

# Set log level in postgresql.conf
log_statement = 'all'
log_min_duration_statement = 1000
log_connections = on
log_disconnections = on

# Check lock waits
SELECT * FROM pg_locks WHERE NOT granted;
```

### Performance Issues

```sql
-- Check active connections
SELECT count(*) FROM pg_stat_activity WHERE state = 'active';

-- Check long-running queries
SELECT pid, now() - pg_stat_activity.query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active' AND now() - pg_stat_activity.query_start > interval '5 minutes';

-- Terminate long-running query
SELECT pg_terminate_backend(pid);

-- Check for idle transactions
SELECT pid, now() - state_change AS idle_duration
FROM pg_stat_activity
WHERE state = 'idle' AND now() - state_change > interval '10 minutes';
```

---

## Quick Reference

### psql Commands

| Command | Description |
|---------|-------------|
| `\c dbname` | Connect to database |
| `\l` | List databases |
| `\dt` | List tables |
| `\d table` | Describe table |
| `\du` | List roles |
| `\df` | List functions |
| `\x` | Toggle expanded display |
| `\?` | Help |
| `\h cmd` | SQL help |
| `\q` | Quit |

### SQL Commands

| Command | Description |
|---------|-------------|
| `CREATE DATABASE` | Create database |
| `CREATE TABLE` | Create table |
| `INSERT INTO` | Add rows |
| `SELECT` | Query data |
| `UPDATE` | Modify data |
| `DELETE` | Remove data |
| `ALTER TABLE` | Modify table |
| `DROP TABLE` | Remove table |
| `GRANT` | Give privileges |
| `REVOKE` | Remove privileges |

### Data Types

| Type | Description |
|------|-------------|
| `SERIAL` | Auto-increment integer |
| `INTEGER` | 4-byte integer |
| `BIGINT` | 8-byte integer |
| `DECIMAL(p,s)` | Exact numeric |
| `NUMERIC(p,s)` | Exact numeric |
| `VARCHAR(n)` | Variable string |
| `TEXT` | Unlimited string |
| `BOOLEAN` | True/false |
| `DATE` | Date |
| `TIMESTAMP` | Date/time |
| `JSON/JSONB` | JSON data |
| `ARRAY` | Array type |
| `UUID` | Universal unique ID |

### pg_dump Options

| Option | Description |
|--------|-------------|
| `-d` | Database name |
| `-U` | Username |
| `-h` | Host |
| `-p` | Port |
| `-t` | Table |
| `-s` | Schema only |
| `-a` | Data only |
| `-c` | Include DROP |
| `-Fc` | Custom format |
| `-Fd` | Directory format |
| `-j` | Parallel jobs |

---

## See Also

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [psql Reference](https://www.postgresql.org/docs/current/app-psql.html)
- [SQL Reference](https://www.postgresql.org/docs/current/sql.html)
- [Performance Tips](https://www.postgresql.org/docs/current/performance-tips.html)
