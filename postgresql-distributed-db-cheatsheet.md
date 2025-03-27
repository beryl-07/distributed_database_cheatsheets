# PostgreSQL Distributed Database Cheat Sheet

## 1. Creating Non-Admin Users
```sql
-- Create a new user
CREATE USER username WITH PASSWORD 'strong_password';

-- Create a user with specific permissions
CREATE USER username WITH 
    LOGIN 
    CREATEDB 
    NOCREATEUSER 
    PASSWORD 'strong_password';

-- Change user password
ALTER USER username WITH PASSWORD 'new_password';
```

## 2. Database Creation with Non-Admin Owner
```sql
-- Create database with a specific owner
CREATE DATABASE dbname 
    WITH OWNER username 
    ENCODING 'UTF8' 
    CONNECTION LIMIT -1;

-- Grant privileges to the user
GRANT ALL PRIVILEGES ON DATABASE dbname TO username;
```

## 3. Database Connection
```bash
# Command line connection
psql -h hostname -U username -d dbname

# Connection string format
postgresql://username:password@host:port/dbname
```

## 4. Table Creation
```sql
-- Basic table creation
CREATE TABLE tablename (
    id SERIAL PRIMARY KEY,
    column1 VARCHAR(255) NOT NULL,
    column2 INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Creating a table with constraints
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(50),
    salary NUMERIC(10,2) CHECK (salary > 0),
    hire_date DATE DEFAULT CURRENT_DATE
);
```

## 5. Loading SQL Files
```bash
# From command line
psql -U username -d dbname -f path/to/script.sql

# Within psql console
\i path/to/script.sql
```

## 6. Loading Data from CSV
```sql
-- Copy data from CSV
COPY tablename(column1, column2)
FROM '/path/to/file.csv' 
WITH (FORMAT csv, HEADER true);

-- Alternative method
\copy tablename FROM '/path/to/file.csv' WITH CSV HEADER;
```

## 7. Remote Database Connection
```sql
-- Basic connection string
postgres://username:password@remote_host:port/dbname

-- Using pg_hba.conf for remote access configuration
# In pg_hba.conf
host    all     all     remote_ip/24     md5
```

## 8. Creating Views
```sql
-- Regular View
CREATE VIEW view_name AS
SELECT column1, column2
FROM table_name
WHERE condition;

-- Materialized View
CREATE MATERIALIZED VIEW mat_view_name AS
SELECT column1, column2
FROM table_name
WITH DATA;

-- Refresh Materialized View
REFRESH MATERIALIZED VIEW mat_view_name;
```

## 9. Remote Database View with DBLink
```sql
-- Install dblink extension
CREATE EXTENSION dblink;

-- Create view from remote database
CREATE VIEW remote_view AS
SELECT * 
FROM dblink('dbname=remote_db host=remote_host user=username password=pass',
    'SELECT * FROM remote_table')
AS remote_data(id int, name text);
```

## 10. Foreign Tables with postgres_fdw
```sql
-- Install foreign data wrapper
CREATE EXTENSION postgres_fdw;

-- Create foreign server
CREATE SERVER foreign_server 
    FOREIGN DATA WRAPPER postgres_fdw
    OPTIONS (host 'remote_host', port '5432', dbname 'remote_db');

-- Create foreign table
CREATE FOREIGN TABLE foreign_table (
    id INT,
    name TEXT
)
SERVER foreign_server
OPTIONS (schema_name 'public', table_name 'remote_table');
```

## 11. Table Partitioning
```sql
-- Range Partitioning
CREATE TABLE sales (
    sale_date DATE NOT NULL,
    amount DECIMAL
) PARTITION BY RANGE (sale_date);

CREATE TABLE sales_2023 PARTITION OF sales
    FOR VALUES FROM ('2023-01-01') TO ('2023-12-31');

-- Hash Partitioning
CREATE TABLE users (
    user_id INT
) PARTITION BY HASH (user_id);

CREATE TABLE users_0 PARTITION OF users
    FOR VALUES WITH (modulus 4, remainder 0);
```

## 12. Performance Testing with pgbench
```bash
# Initialize benchmark database
pgbench -i -s 10 benchmark_db

# Run benchmark test
pgbench -c 10 -j 2 -t 1000 benchmark_db

# Custom test scenario
pgbench -c 20 -j 4 -t 5000 -f custom_script.sql benchmark_db
```

## 13. Connection Pooling
### PgBouncer Configuration (pgbouncer.ini)
```ini
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
pool_mode = transaction
max_client_conn = 100
default_pool_size = 20
```

### Pgpool-II Basic Configuration
```conf
backend_hostname0 = 'primary_host'
backend_port0 = 5432
backend_weight0 = 1

num_init_children = 32
max_pool = 4
```

## 14. Trigger for Master-Slave Replication
```sql
-- Create a trigger function
CREATE OR REPLACE FUNCTION sync_to_slave()
RETURNS TRIGGER AS $$
BEGIN
    -- Replication logic here
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Attach trigger to table
CREATE TRIGGER sync_changes
AFTER INSERT OR UPDATE OR DELETE ON master_table
FOR EACH ROW EXECUTE FUNCTION sync_to_slave();
```

## Pro Tips
- Always use connection pooling for high-traffic applications
- Implement proper indexing strategies
- Use EXPLAIN ANALYZE to understand query performance
- Regularly vacuum and analyze tables
- Implement proper access controls and network security
```

This comprehensive cheat sheet covers the key topics in your distributed database course with PostgreSQL. It provides practical examples and syntax for each topic, from user management to advanced replication techniques.

Would you like me to elaborate on any specific section or provide more detailed explanations for any of these topics?
