# Table Partitioning

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
