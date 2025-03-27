# Gestion des utilisateurs dans PostgreSQL

## Créqtion d'utilisateurs Non-Admin

En tant que superutilisateur (postgres), vous pouvez créer des utilisateurs soit via SQL, soit via bash :

### Using SQL (psql)

```sql
-- Create a new user without superuser privileges
CREATE USER app_user WITH PASSWORD 'secure_password';

-- Grant privileges
CREATE DATABASE mydb WITH OWNER app_user;
```

### Using Bash (easier alternative)

```bash
# As root or user with sudo privileges
sudo -i -u postgres createuser app_user

# Set password
sudo -u postgres psql -c "ALTER USER app_user WITH PASSWORD 'secure_password';"

# Grant privileges
sudo -i -u postgres createdb --owner=app_user mydb

sudo -i -u postgres createdb -O app_user mydb
```

## Verifying User and Database Creation

```bash
# List all users
sudo -u postgres psql -c "\du"

# List all databases and their owners
sudo -u postgres psql -c "\l"
```
