# Foreign Data Wrappers (FDW)

## Prerequisites
- Superuser privileges for extension installation
- Network access between servers
- Foreign server credentials

## Setting up postgres_fdw

```sql
-- Create the extension
CREATE EXTENSION postgres_fdw;

-- Create the foreign server
CREATE SERVER foreign_server
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'remote_host', port '5432', dbname 'remote_db');

-- Create user mapping
CREATE USER MAPPING FOR local_user
SERVER foreign_server
OPTIONS (user 'remote_user', password 'remote_pass');

-- Create foreign table
CREATE FOREIGN TABLE foreign_products (
    id integer,
    name varchar(100),
    price numeric
)
SERVER foreign_server
OPTIONS (schema_name 'public', table_name 'products');
```

## Query Foreign Tables

```sql
-- Query foreign table like a local table
SELECT * FROM foreign_products;

-- Join with local tables
SELECT l.order_id, f.name, f.price
FROM local_orders l
JOIN foreign_products f ON l.product_id = f.id;
```

## Monitoring

```sql
-- Explain foreign queries
EXPLAIN (VERBOSE, COSTS)
SELECT * FROM foreign_products;
```
