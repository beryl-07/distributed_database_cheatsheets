# Database Operations

## Prerequisites
- PostgreSQL installed (`sudo apt-get install postgresql postgresql-contrib`)
- psql client installed (`sudo apt-get install postgresql-client`)
- Appropriate user permissions

## Database Connection

### As superuser (postgres)
```bash
sudo -u postgres psql
```

### As application user
```bash
# Using psql
psql -h localhost -p 5432 -U app_user -d application_db

# Using environment variables
export PGUSER=app_user
export PGPASSWORD=your_password
export PGDATABASE=application_db
psql

# Connection URL format
postgresql://app_user:password@localhost:5432/application_db
```

## Table Creation
Requires CREATE privilege on the schema (usually granted by default to database owner)

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Loading SQL Files

### Using psql
```bash
# As superuser
sudo -u postgres psql -d application_db -f schema.sql

# As application user
psql -U app_user -d application_db -f schema.sql
```

### Within psql
```sql
\i /path/to/schema.sql
```

## Loading CSV Data

### Using COPY (requires superuser or system user postgres)
```sql
COPY customers(name, email)
FROM '/path/to/customers.csv'
WITH (FORMAT csv, HEADER true);
```

### Using \copy (works with regular users)
```sql
\copy customers(name, email) FROM '/path/to/customers.csv' WITH (FORMAT csv, HEADER true);
```

### Using Bash script
```bash
#!/bin/bash
# load_data.sh
PGPASSWORD=your_password psql -U app_user -d application_db \
  -c "\copy customers(name, email) FROM '/path/to/customers.csv' WITH (FORMAT csv, HEADER true);"
```
