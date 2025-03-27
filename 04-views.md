# Views and Materialized Views

## Prerequisites
- Database owner or superuser privileges for creating materialized views
- dblink extension requires superuser for installation

## Regular Views

```sql
-- Crée une vue simple
CREATE VIEW active_customers AS
SELECT * FROM customers 
WHERE status = 'active';

-- Utilisation de la vue
SELECT * FROM active_customers;
```

## Materialized Views

```sql
-- Création d'une vue materialisée
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT * FROM customers 
WHERE status != 'active';

-- Refresh materialized view
REFRESH MATERIALIZED VIEW monthly_sales;
```

## DBLink Views

```sql
-- Install dblink extension (requires superuser)
CREATE EXTENSION dblink;

-- Crée view à pqrtir de données distantes
CREATE VIEW remote_products AS
SELECT *
FROM dblink('host=remote_server port=5432 dbname=remote_db user=remote_user password=remote_pass',
    'SELECT id, name, price FROM products')
AS remote_products(id INT, name VARCHAR, price DECIMAL);
```

## Remote Views Management
```sql
-- Crée l'extension (requires superuser)
CREATE EXTENSION IF NOT EXISTS dblink;

-- Crée une vue à partir de données distantes
CREATE OR REPLACE VIEW remote_products
AS
SELECT * FROM dblink('host=remote_server port=5432 dbname=remote_db user=remote_user password=remote_pass',
    'SELECT * FROM products')
AS remote_products(id INT, name VARCHAR, price DECIMAL);

-- DBLink_Execute pour executer une requête
SELECT * FROM dblink_exec('host=remote_server port=5432 dbname=remote_db user=remote_user password=remote_pass',
    'SELECT * FROM products');

-- Monitor view status
SELECT schemaname, viewname, definition 
FROM pg_views 
WHERE viewowner = current_user;
```
