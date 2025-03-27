# Table Partitioning

## Prerequisites
- PostgreSQL 12+ for declarative partitioning
- Superuser or table owner privileges

## Creating Partitioned Tables

```sql
-- Create master partitioned table
CREATE TABLE orders (
    order_id SERIAL,
    order_date DATE NOT NULL,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (order_date);

-- Create partition tables
CREATE TABLE orders_2023 PARTITION OF orders
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');
CREATE TABLE orders_2024 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

-- Create trigger for automatic routing
CREATE OR REPLACE FUNCTION orders_insert_trigger()
RETURNS TRIGGER AS $$
BEGIN
    IF (NEW.order_date >= '2023-01-01' AND NEW.order_date < '2024-01-01') THEN
        INSERT INTO orders_2023 VALUES (NEW.*);
    ELSIF (NEW.order_date >= '2024-01-01' AND NEW.order_date < '2025-01-01') THEN
        INSERT INTO orders_2024 VALUES (NEW.*);
    ELSE
        RAISE EXCEPTION 'Date out of range';
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER insert_orders_trigger
    BEFORE INSERT ON orders
    FOR EACH ROW EXECUTE FUNCTION orders_insert_trigger();
```

## List Partitioning
```sql
CREATE TABLE products (
    product_id SERIAL,
    category VARCHAR(50),
    name VARCHAR(100)
) PARTITION BY LIST (category);

CREATE TABLE products_electronics PARTITION OF products 
    FOR VALUES IN ('electronics');
CREATE TABLE products_clothing PARTITION OF products 
    FOR VALUES IN ('clothing');
```

## Partition Management
```sql
-- Monitor partition usage
SELECT schemaname, tablename, partition_strategy, partition_expression
FROM pg_partitioned_table pt
JOIN pg_class c ON pt.partrelid = c.oid
JOIN pg_namespace n ON c.relnamespace = n.oid;

-- Detach/Attach partitions
ALTER TABLE orders DETACH PARTITION orders_2023;
ALTER TABLE orders ATTACH PARTITION orders_2023
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');
```
