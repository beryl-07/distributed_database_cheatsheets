# Database Operations

## Prerequisites

- PostgreSQL installed (`sudo apt-get install postgresql postgresql-contrib`)
- psql client installed (`sudo apt-get install postgresql-client`)
- Appropriate user permissions

## Database Connection

### As superuser (postgres)

```bash
sudo -i -u postgres psql
```

### As application user

```bash
# Using psql
psql -h localhost -p 5432 -U app_user -d application_db
```

## Table Creation

Requires CREATE privilege on the schema (usually granted by default to database owner)

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255),
    age INT NOT NULL,
    size DECIMAL(5, 2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id),
    total_amount DECIMAL(10, 2),
    order_date DATE
    customer_id INT REFERENCES customers(id),
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);
```
## Table Alteration

### Adding a Column

```sql
ALTER TABLE customers ADD COLUMN phone VARCHAR(20);
```

### Dropping a Column

```sql
ALTER TABLE customers DROP COLUMN phone;
```

### Renaming a Column

```sql
ALTER TABLE customers RENAME COLUMN phone TO mobile;
```

### Adding a Constraint

```sql
ALTER TABLE customers ADD CONSTRAINT unique_email UNIQUE (email);
```

### Dropping a Constraint

```sql
ALTER TABLE customers DROP CONSTRAINT unique_email;
```

### Adding a Default Value

```sql
ALTER TABLE customers ALTER COLUMN age SET DEFAULT 18;
```

### Dropping a Default Value

```sql

ALTER TABLE customers ALTER COLUMN age DROP DEFAULT;
```

### Adding a Foreign Key

```sql
ALTER TABLE orders ADD CONSTRAINT fk_customer_id
    FOREIGN KEY (customer_id) REFERENCES customers(id);
```

## Generating Data using Generators

### Using `generate_series`

```sql
INSERT INTO "master_vehicles" ("brand", "immatriculation", "user_id")
SELECT
    'Brand ' || i,
    'IM-' || i,
    floor(random() * 500000 + 1)::int
FROM
    generate_series(3, 500) AS i;
```


## Loading SQL Files

### Using psql

```bash
# As superuser
sudo -i -u postgres psql -d application_db -f /path/to/schema.sql

# As application user
psql -U app_user -d application_db -f /path/to/schema.sql
```

### Within psql

```sql
\i /path/to/schema.sql
```

## Loading CSV Data

### Using COPY (requires superuser or system user postgres)

```sql
COPY tablename FROM '/path/to/file.csv' DELIMITER ',' CSV HEADER;
```

### Using Bash script

```bash
#!/bin/bash
# load_data.sh
PGPASSWORD=your_password psql -U app_user -d application_db \
  -c "\copy customers(name, email) FROM '/path/to/customers.csv' WITH (FORMAT csv, HEADER true);"
```
