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

## Monitoring and Failover
```sql
-- PgPool-II watchdog configuration
# In pgpool.conf
use_watchdog = on
wd_hostname = 'pool_node1'
delegate_IP = '10.0.0.100'
heartbeat_destination0 = 'pool_node2'
heartbeat_destination_port0 = 9694

-- Monitor PgPool-II status
show pool_nodes;
show pool_processes;
show pool_status;

-- Monitor PgBouncer
SHOW POOLS;
SHOW CLIENTS;
SHOW SERVERS;
SHOW STATS;
```
