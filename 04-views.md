# Views and Materialized Views

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
