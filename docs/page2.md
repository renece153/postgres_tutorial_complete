# PostgreSQL Introduction  
### Creating Databases, Connecting, Creating Tables, and Understanding Backend File Structure

A student’s first real interaction with PostgreSQL begins the moment they create a database, connect to it, and define their first table. This section builds directly on the earlier concepts of relational structure — **Database → Schema → Database Objects** — which your Chapter 4.1 document explains as the universal hierarchy across relational systems.

---

## 1. Creating a Database in PostgreSQL

Creating a database is the first step in organizing your work. In PostgreSQL, a **database** is the top‑level container that holds schemas, tables, and all related objects.

> “The Database acts as the top‑level container, holding all data and metadata.”

### Creating a Database in pgAdmin (GUI)

1. Open **pgAdmin** and connect to your PostgreSQL server.  
2. In the left tree, right‑click **Databases → Create → Database**.  
3. Enter a name (e.g., `class_db`).  
4. Leave the owner as **postgres** unless you’ve created additional roles.  
5. Click **Save**.

### Creating a Database via SQL

```sql
CREATE DATABASE class_db;
```

## 2. Connecting to Databases

PostgreSQL allows multiple databases inside the same server instance. Each connection targets exactly one database at a time.

### Connecting in pgAdmin

1. Expand the server.
2. Double‑click the database you created.
3. pgAdmin will open a new connection tab for that database.

### Connecting via psql (CLI)

```bash
psql -U postgres -d class_db
```

- `-U` specifies the user
- `-d` specifies the database

## 3. Creating Your First Table

Your earlier chapter explains that schemas are _“logical containers for tables, views, functions”_ .
By default, PostgreSQL uses the public schema unless you specify otherwise.

1. Expand your database → Schemas → public → Tables.
2. Right‑click Tables → Create → Table.
3. Name the table (e.g., users).
4. Add columns:
   - id → SERIAL → Primary Key
   - name → VARCHAR(255)
   - age → INT
   - email → TEXT (Unique)

** Creating the Same Table via SQL **
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    age INT,
    email TEXT UNIQUE
);
```

This matches the example structure you used earlier in your CRUD handout.

## 4. PostgreSQL Backend File Structure

PostgreSQL stores data inside the data directory (PGDATA).

```
PGDATA/
│
├── base/          # One folder per database
├── global/        # Cluster-wide metadata
├── pg_wal/        # Write-Ahead Logs (durability)
├── pg_multixact/  # MVCC lock tracking
├── pg_tblspc/     # Tablespace links
└── postgresql.conf
```

**How a table becomes files**
- Each table = one or more files inside base/<db_oid>/
- Indexes = separate files
- Large rows = TOAST files

## 5. Quick Reference Commands

**List databases**
```sql
\l
```

**Switch database**
```sql
\c class_db
```

**List tables**
```sql
\dt
```

**Describe table**
```sql
\d users
```

