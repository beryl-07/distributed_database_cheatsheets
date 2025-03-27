# Views and Materialized Views

## Prerequisites
- Database owner or superuser privileges for creating materialized views
- dblink extension requires superuser for installation

## Regular Views

```sql
-- Create a simple view
CREATE VIEW active_customers AS
SELECT * FROM customers 
WHERE status = 'active';

-- Using the view
SELECT * FROM active_customers;
```

## Materialized Views

```sql
-- Create materialized view
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT 
    date_trunc('month', sale_date) AS month,
    SUM(amount) as total_sales
FROM sales
GROUP BY 1
WITH DATA;

-- Refresh materialized view
REFRESH MATERIALIZED VIEW monthly_sales;
```

## Materialized Views with Indexes
```sql
-- Create materialized view with indexes
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT 
    date_trunc('month', sale_date) AS month,
    SUM(amount) as total_sales
FROM sales
GROUP BY 1
WITH DATA;

-- Add indexes to improve refresh performance
CREATE INDEX idx_monthly_sales_month ON monthly_sales(month);

-- Concurrent refresh (requires at least one unique index)
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_sales;
```

## DBLink Views

```sql
-- Install dblink extension
CREATE EXTENSION dblink;

-- Create view using remote data
CREATE VIEW remote_products AS
SELECT *
FROM dblink('host=remote_server port=5432 dbname=remote_db user=remote_user password=remote_pass',
    'SELECT id, name, price FROM products')
AS remote_products(id INT, name VARCHAR, price DECIMAL);
```

## Remote Views Management
```sql
-- Create extension (requires superuser)
CREATE EXTENSION IF NOT EXISTS dblink;

-- Create user mapping for dblink
CREATE USER MAPPING FOR CURRENT_USER
SERVER remote_server
OPTIONS (user 'remote_user', password 'remote_pass');

-- Monitor view status
SELECT schemaname, viewname, definition 
FROM pg_views 
WHERE viewowner = current_user;
```
