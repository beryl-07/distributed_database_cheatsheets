# Foreign Data Wrappers (FDW)

## Prérequis
- Privilèges superutilisateur pour l'installation des extensions
- Accès réseau entre les serveurs
- Identifiants du serveur distant

## Configuration de postgres_fdw

```sql
-- Créer l'extension
CREATE EXTENSION postgres_fdw;

-- Créer le serveur distant
CREATE SERVER foreign_server
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'remote_host', port '5432', dbname 'remote_db');

-- Créer le mapping utilisateur
CREATE USER MAPPING FOR local_user
SERVER foreign_server
OPTIONS (user 'remote_user', password 'remote_pass');

-- Créer la table distante
CREATE FOREIGN TABLE foreign_products (
    id integer,
    name varchar(100),
    price numeric
)
SERVER foreign_server
OPTIONS (schema_name 'public', table_name 'products');
```

## Interroger les Tables Distantes

```sql
-- Interroger la table distante comme une table locale
SELECT * FROM foreign_products;

-- Joindre avec des tables locales
SELECT l.order_id, f.name, f.price
FROM local_orders l
JOIN foreign_products f ON l.product_id = f.id;
```

## Surveillance

```sql
-- Expliquer les requêtes distantes
EXPLAIN (VERBOSE, COSTS)
SELECT * FROM foreign_products;
```
