# User Management in PostgreSQL

## Creating Non-Admin Users

As a superuser (postgres), you can create users either through SQL or bash:

### Using SQL (psql)
```sql
-- Create a new user without superuser privileges
CREATE USER app_user WITH PASSWORD 'secure_password';

-- Grant specific privileges
GRANT CONNECT ON DATABASE mydb TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
```

### Using Bash (easier alternative)
```bash
# As root or user with sudo privileges
sudo -u postgres createuser --no-superuser --no-createdb --no-createrole app_user

# Set password
sudo -u postgres psql -c "ALTER USER app_user WITH PASSWORD 'secure_password';"

# Grant privileges
sudo -u postgres psql -d mydb -c "GRANT CONNECT ON DATABASE mydb TO app_user;"
sudo -u postgres psql -d mydb -c "GRANT USAGE ON SCHEMA public TO app_user;"
sudo -u postgres psql -d mydb -c "GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;"
```

## Database Creation with Owner

### Using SQL (psql)
```sql
-- Must be executed as superuser
CREATE DATABASE application_db WITH OWNER app_user;
```

### Using Bash
```bash
# As root or user with sudo privileges
sudo -u postgres createdb --owner=app_user application_db
```

## Verifying User and Database Creation

```bash
# List all users
sudo -u postgres psql -c "\du"

# List all databases and their owners
sudo -u postgres psql -c "\l"
```
