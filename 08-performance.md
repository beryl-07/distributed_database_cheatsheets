# Tests de Performance avec pgbench

## Configuration de pgbench

```sql
-- Initialiser les tables pgbench
pgbench -i -s 10 nom_de_votre_base

-- Test de benchmark basique
pgbench -c 10 -j 2 -t 1000 nom_de_votre_base

-- Script de benchmark personnalisé
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

## Analyse des Résultats

```sql
-- Vérifier les statistiques des tables
SELECT schemaname, relname, seq_scan, seq_tup_read, 
       idx_scan, idx_tup_fetch
FROM pg_stat_user_tables;

-- Vérifier l'utilisation des index
SELECT schemaname, relname, indexrelname, idx_scan, 
       idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes;
```

## Surveillance Étendue
```sql
-- Surveiller les requêtes actives
SELECT pid, query, state, wait_event, wait_event_type
FROM pg_stat_activity
WHERE state != 'idle';

-- Trouver les requêtes lentes
SELECT calls, total_exec_time/calls as avg_time, query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Métriques de pooling de connexions
SELECT * FROM pg_stat_database;

-- Efficacité des index
SELECT schemaname, tablename, indexrelname,
       idx_scan, idx_tup_read,
       idx_tup_fetch
FROM pg_stat_all_indexes
WHERE idx_scan = 0
AND schemaname NOT IN ('pg_catalog', 'pg_toast');
```
