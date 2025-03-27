# Aide-mémoire PostgreSQL pour Bases de Données Distribuées

## 1. Création d'Utilisateurs Non-Admin
```sql
-- Créer un nouvel utilisateur
CREATE USER username WITH PASSWORD 'strong_password';

-- Créer un utilisateur avec des permissions spécifiques
CREATE USER username WITH 
    LOGIN 
    CREATEDB 
    NOCREATEUSER 
    PASSWORD 'strong_password';

-- Changer le mot de passe de l'utilisateur
ALTER USER username WITH PASSWORD 'new_password';
```

## 2. Création de Base de Données avec Propriétaire Non-Admin
```sql
-- Créer une base de données avec un propriétaire spécifique
CREATE DATABASE dbname 
    WITH OWNER username 
    ENCODING 'UTF8' 
    CONNECTION LIMIT -1;

-- Accorder des privilèges à l'utilisateur
GRANT ALL PRIVILEGES ON DATABASE dbname TO username;
```

## 3. Connexion à la Base de Données
```bash
# Connexion en ligne de commande
psql -h hostname -U username -d dbname

# Format de la chaîne de connexion
postgresql://username:password@host:port/dbname
```

## 4. Création de Table
```sql
-- Création de table basique
CREATE TABLE tablename (
    id SERIAL PRIMARY KEY,
    column1 VARCHAR(255) NOT NULL,
    column2 INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Création d'une table avec des contraintes
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    department VARCHAR(50),
    salary NUMERIC(10,2) CHECK (salary > 0),
    hire_date DATE DEFAULT CURRENT_DATE
);
```

## 5. Chargement de Fichiers SQL
```bash
# Depuis la ligne de commande
psql -U username -d dbname -f path/to/script.sql

# Dans la console psql
\i path/to/script.sql
```

## 6. Chargement de Données depuis un CSV
```sql
-- Copier des données depuis un CSV
COPY tablename(column1, column2)
FROM '/path/to/file.csv' 
WITH (FORMAT csv, HEADER true);

-- Méthode alternative
\copy tablename FROM '/path/to/file.csv' WITH CSV HEADER;
```

## 7. Connexion à une Base de Données Distante
```sql
-- Chaîne de connexion basique
postgres://username:password@remote_host:port/dbname

-- Utilisation de pg_hba.conf pour la configuration de l'accès distant
# Dans pg_hba.conf
host    all     all     remote_ip/24     md5
```

## 8. Création de Vues
```sql
-- Vue régulière
CREATE VIEW view_name AS
SELECT column1, column2
FROM table_name
WHERE condition;

-- Vue matérialisée
CREATE MATERIALIZED VIEW mat_view_name AS
SELECT column1, column2
FROM table_name
WITH DATA;

-- Rafraîchir la vue matérialisée
REFRESH MATERIALIZED VIEW mat_view_name;
```

## 9. Vue de Base de Données Distante avec DBLink
```sql
-- Installer l'extension dblink
CREATE EXTENSION dblink;

-- Créer une vue depuis une base de données distante
CREATE VIEW remote_view AS
SELECT * 
FROM dblink('dbname=remote_db host=remote_host user=username password=pass',
    'SELECT * FROM remote_table')
AS remote_data(id int, name text);
```

## 10. Tables Étrangères avec postgres_fdw
```sql
-- Installer le wrapper de données étrangères
CREATE EXTENSION postgres_fdw;

-- Créer un serveur étranger
CREATE SERVER foreign_server 
    FOREIGN DATA WRAPPER postgres_fdw
    OPTIONS (host 'remote_host', port '5432', dbname 'remote_db');

-- Créer une table étrangère
CREATE FOREIGN TABLE foreign_table (
    id INT,
    name TEXT
)
SERVER foreign_server
OPTIONS (schema_name 'public', table_name 'remote_table');
```

## 11. Partitionnement de Table
```sql
-- Partitionnement par plage
CREATE TABLE sales (
    sale_date DATE NOT NULL,
    amount DECIMAL
) PARTITION BY RANGE (sale_date);

CREATE TABLE sales_2023 PARTITION OF sales
    FOR VALUES FROM ('2023-01-01') TO ('2023-12-31');

-- Partitionnement par hachage
CREATE TABLE users (
    user_id INT
) PARTITION BY HASH (user_id);

CREATE TABLE users_0 PARTITION OF users
    FOR VALUES WITH (modulus 4, remainder 0);
```

## 12. Test de Performance avec pgbench
```bash
# Initialiser la base de données de benchmark
pgbench -i -s 10 benchmark_db

# Exécuter le test de benchmark
pgbench -c 10 -j 2 -t 1000 benchmark_db

# Scénario de test personnalisé
pgbench -c 20 -j 4 -t 5000 -f custom_script.sql benchmark_db
```

## 13. Pooling de Connexions
### Configuration de PgBouncer (pgbouncer.ini)
```ini
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
pool_mode = transaction
max_client_conn = 100
default_pool_size = 20
```

### Configuration de Base de Pgpool-II
```conf
backend_hostname0 = 'primary_host'
backend_port0 = 5432
backend_weight0 = 1

num_init_children = 32
max_pool = 4
```

## 14. Déclencheur pour la Réplication Master-Slave
```sql
-- Créer une fonction de déclenchement
CREATE OR REPLACE FUNCTION sync_to_slave()
RETURNS TRIGGER AS $$
BEGIN
    -- Logique de réplication ici
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Attacher le déclencheur à la table
CREATE TRIGGER sync_changes
AFTER INSERT OR UPDATE OR DELETE ON master_table
FOR EACH ROW EXECUTE FUNCTION sync_to_slave();
```

## Conseils Pro
- Utilisez toujours le pooling de connexions pour les applications à fort trafic
- Implémentez des stratégies d'indexation appropriées
- Utilisez EXPLAIN ANALYZE pour comprendre les performances des requêtes
- Effectuez régulièrement des opérations de vacuum et d'analyse sur les tables
- Implémentez des contrôles d'accès et une sécurité réseau appropriés
```

Cette aide-mémoire complète couvre les sujets clés de votre cours sur les bases de données distribuées avec PostgreSQL. Il fournit des exemples pratiques et la syntaxe pour chaque sujet, de la gestion des utilisateurs aux techniques avancées de réplication.

Souhaitez-vous que j'élabore sur une section spécifique ou que je fournisse des explications plus détaillées pour l'un de ces sujets ?
