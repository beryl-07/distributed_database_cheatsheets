# Transparence Réseau avec l'Héritage

## Configuration de l'Héritage des Tables

```sql
-- Créer la table parent
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2),
    location VARCHAR(50)
);

-- Créer les tables enfants
CREATE TABLE products_local () INHERITS (products);
CREATE TABLE products_remote () INHERITS (products);

-- Créer les règles de routage
CREATE RULE products_insert_local AS
    ON INSERT TO products
    WHERE (location = 'local')
    DO INSTEAD
    INSERT INTO products_local VALUES (NEW.*);

CREATE RULE products_insert_remote AS
    ON INSERT TO products
    WHERE (location = 'remote')
    DO INSTEAD
    INSERT INTO products_remote VALUES (NEW.*);
```

<!-- ## Replication Setup
```sql
-- On primary server
ALTER SYSTEM SET wal_level = replica;
ALTER SYSTEM SET max_wal_senders = 10;
ALTER SYSTEM SET max_replication_slots = 10;

-- Create replication user
CREATE USER replicator WITH REPLICATION PASSWORD 'rep123';

-- On standby server (postgresql.conf)
primary_conninfo = 'host=primary_host port=5432 user=replicator password=rep123'
restore_command = 'cp /var/lib/postgresql/archive/%f %p'

-- Monitor replication
SELECT * FROM pg_stat_replication;
``` -->
