# Connexions aux Bases de Données Distantes

## Prérequis
- Accès superutilisateur sur les serveurs source et cible
- PostgreSQL 12+ installé
- Pare-feu configuré pour autoriser le trafic PostgreSQL (port 5432 par défaut)

## Configurer PostgreSQL pour l'accès distant

```bash
# Modifier postgresql.conf
listen_addresses = '*'

# Modifier pg_hba.conf
# TYPE  BASE DE DONNÉES   UTILISATEUR     ADRESSE              MÉTHODE
host    all              app_user        samenet               md5
host    all              app_user        192.168.1.0/24        md5
```

## Connexion à une Base de Données Distante

```sql
-- En tant que superutilisateur
CREATE EXTENSION postgres_fdw;

-- Créer la connexion comme serveur distant
CREATE SERVER remote_server
FOREIGN DATA WRAPPER postgres_fdw
OPTIONS (host 'remote_host', port '5432', dbname 'remote_db');

-- Créer le mapping utilisateur
CREATE USER MAPPING FOR local_user
SERVER remote_server
OPTIONS (user 'remote_user', password 'remote_password');
```

## Test des Connexions
```bash
# Tester la connexion avec psql
psql "postgresql://user:password@remote_host:5432/dbname"

# Tester avec pg_isready
pg_isready -h remote_host -p 5432
```
