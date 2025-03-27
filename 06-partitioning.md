# Table Partitioning

## Prerequisites
- PostgreSQL 12+ for declarative partitioning
- Superuser or table owner privileges

## Creating Partitioned Tables

### Range Partitioning

```sql

-- Create master partitioned table
CREATE TABLE IF NOT EXISTS master_vehicles (
    id SERIAL PRIMARY KEY,
    brand VARCHAR(255) NOT NULL,
    immatriculation VARCHAR(255) NOT NULL,
    user_id INT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users (id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create child tables

CREATE TABLE IF NOT EXISTS slave1_vehicles (check (user_id < 125000)) INHERITS (master_vehicles);
CREATE TABLE IF NOT EXISTS slave2_vehicles (check ((user_id >= 125000) AND (user_id < 250000))) INHERITS (master_vehicles);

-- Create triggers for automatic routing
CREATE OR REPLACE FUNCTION insert_vehicle()
RETURNS TRIGGER AS $$
BEGIN
    IF (NEW.user_id < 125000) THEN
        INSERT INTO slave1_vehicles (brand, immatriculation, user_id) VALUES (NEW.brand, NEW.immatriculation, NEW.user_id);
    ELSIF ((NEW.user_id >= 125000) AND (NEW.user_id < 250000)) THEN
        INSERT INTO slave2_vehicles (brand, immatriculation, user_id) VALUES (NEW.brand, NEW.immatriculation, NEW.user_id);
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER insert_vehicle_trigger
    BEFORE INSERT ON master_vehicles
    FOR EACH ROW
    EXECUTE PROCEDURE insert_vehicle();

CREATE OR REPLACE FUNCTION update_vehicle()
RETURNS TRIGGER AS $$
BEGIN
    IF (NEW.user_id < 125000) THEN
        UPDATE slave1_vehicles SET brand = NEW.brand, immatriculation = NEW.immatriculation, user_id = NEW.user_id WHERE id = NEW.id;
    ELSIF ((NEW.user_id >= 125000) AND (NEW.user_id < 250000)) THEN
        UPDATE slave2_vehicles SET brand = NEW.brand, immatriculation = NEW.immatriculation, user_id = NEW.user_id WHERE id = NEW.id;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_vehicle_trigger
    BEFORE UPDATE ON master_vehicles
    FOR EACH ROW
    EXECUTE PROCEDURE update_vehicle();

CREATE OR REPLACE FUNCTION delete_vehicle()
RETURNS TRIGGER AS $$
BEGIN
    IF (OLD.user_id < 125000) THEN
        DELETE FROM slave1_vehicles WHERE id = OLD.id;
    ELSIF ((OLD.user_id >= 125000) AND (OLD.user_id < 250000)) THEN
        DELETE FROM slave2_vehicles WHERE id = OLD.id;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER delete_vehicle_trigger
    BEFORE DELETE ON master_vehicles
    FOR EACH ROW
    EXECUTE PROCEDURE delete_vehicle();
```

OR


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
<!-- 
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
``` -->

<!-- ## Partition Management
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
``` -->
