# Performance Testing with pgbench

## Setting up pgbench

```sql
-- Initialize pgbench tables
pgbench -i -s 10 your_database_name

-- Basic benchmark test
pgbench -c 10 -j 2 -t 1000 your_database_name

-- Custom benchmark script
CREATE TABLE test_table (
    id SERIAL PRIMARY KEY,
    data TEXT
);

\set aid random(1, 100000 * :scale)
\set bid random(1, 1 * :scale)
BEGIN;
UPDATE pgbench_accounts SET abalance = abalance + :bid WHERE aid = :aid;
SELECT abalance FROM pgbench_accounts WHERE aid = :aid;
COMMIT;
```

## Analyzing Results

```sql
-- Check table statistics
SELECT schemaname, relname, seq_scan, seq_tup_read, 
       idx_scan, idx_tup_fetch
FROM pg_stat_user_tables;

-- Check index usage
SELECT schemaname, relname, indexrelname, idx_scan, 
       idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes;
```

## Extended Monitoring
```sql
-- Monitor active queries
SELECT pid, query, state, wait_event, wait_event_type
FROM pg_stat_activity
WHERE state != 'idle';

-- Find slow queries
SELECT calls, total_exec_time/calls as avg_time, query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Connection pooling metrics
SELECT * FROM pg_stat_database;

-- Index efficiency
SELECT schemaname, tablename, indexrelname,
       idx_scan, idx_tup_read,
       idx_tup_fetch
FROM pg_stat_all_indexes
WHERE idx_scan = 0
AND schemaname NOT IN ('pg_catalog', 'pg_toast');
```
