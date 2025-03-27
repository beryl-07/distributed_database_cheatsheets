# Remote Database Connections

## Prerequisites
- Superuser access on both source and target servers
- PostgreSQL 12+ installed
- Firewall configured to allow PostgreSQL traffic (default port 5432)

## Configure PostgreSQL for Remote Access

```bash
# Edit postgresql.conf
listen_addresses = '*'

# Edit pg_hba.conf
# TYPE  DATABASE        USER            ADDRESS               METHOD
host    all             app_user        samenet               md5
host    all             app_user        192.168.1.0/24        md5
```

## Connecting to Remote Database

```sql
-- AS superuser
CREATE EXTENSION postgres_fdw;

-- Create connection as foreign server
CREATE SERVER remote_server
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'remote_host', port '5432', dbname 'remote_db');

-- Create user mapping
CREATE USER MAPPING FOR local_user
SERVER remote_server
OPTIONS (user 'remote_user', password 'remote_password');
```

## Testing Connections
```bash
# Test connection using psql
psql "postgresql://user:password@remote_host:5432/dbname"

# Test using pg_isready
pg_isready -h remote_host -p 5432
```
