# Gestion des utilisateurs dans PostgreSQL

## Création d'utilisateurs Non-Admin

En tant que superutilisateur (postgres), vous pouvez créer des utilisateurs soit via SQL, soit via bash :

### Utilisation de SQL (psql)

```sql
-- Créer un nouvel utilisateur sans privilèges superutilisateur
CREATE USER app_user WITH PASSWORD 'secure_password';

-- Accorder des privilèges
CREATE DATABASE mydb WITH OWNER app_user;
```

### Utilisation de Bash (alternative plus simple)

```bash
# En tant que root ou utilisateur avec privilèges sudo
sudo -i -u postgres createuser app_user

# Définir le mot de passe
sudo -u postgres psql -c "ALTER USER app_user WITH PASSWORD 'secure_password';"

# Accorder des privilèges
sudo -i -u postgres createdb --owner=app_user mydb

sudo -i -u postgres createdb -O app_user mydb
```

## Vérification de la création des utilisateurs et bases de données

```bash
# Lister tous les utilisateurs
sudo -u postgres psql -c "\du"

# Lister toutes les bases de données et leurs propriétaires
sudo -u postgres psql -c "\l"
```
