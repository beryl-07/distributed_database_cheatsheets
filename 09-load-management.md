# Load Management with PgBouncer and PgPool-II

## PgBouncer Configuration

```ini
[databases]
* = host=localhost port=5432

[pgbouncer]
listen_port = 6432
listen_addr = *
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20
```

## PgPool-II Setup

```ini
# pgpool.conf
listen_addresses = '*'
port = 5433
backend_hostname0 = 'master.db'
backend_port0 = 5432
backend_weight0 = 1
backend_hostname1 = 'replica1.db'
backend_port1 = 5432
backend_weight1 = 1

# Load Balancing Settings
load_balance_mode = on
master_slave_mode = on
master_slave_sub_mode = 'stream'
```

## Hash Partitioning

```sql
-- Create hash partitioned table
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    order_date DATE
) PARTITION BY HASH (customer_id);

-- Create partitions
CREATE TABLE orders_0 PARTITION OF orders 
    FOR VALUES WITH (modulus 4, remainder 0);
CREATE TABLE orders_1 PARTITION OF orders 
    FOR VALUES WITH (modulus 4, remainder 1);
CREATE TABLE orders_2 PARTITION OF orders 
    FOR VALUES WITH (modulus 4, remainder 2);
CREATE TABLE orders_3 PARTITION OF orders 
    FOR VALUES WITH (modulus 4, remainder 3);
```
