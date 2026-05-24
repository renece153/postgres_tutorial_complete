## 📘 Part 2 — Inserting Data Into PostgreSQL
In this section, we will insert sample data into the users table and then continue building our database by creating additional tables.
These steps should be executed inside PostgreSQL while following along with the reference document.

### 🧩 Step 6 — Insert 5 Rows Into users
We begin by inserting one row:

```sql
INSERT INTO users (name, email, age)
VALUES ('Brian Griffin', 'brian@gmail.com', 62);
```

**Next, insert four more rows:**
```sql
INSERT INTO users (name, email, age)
VALUES 
('Kevin Mccoy', 'kevin@gmail.com', 35),
('John Doe', 'john.doe@gmail.com', 23),
('Michael Jackson', 'michael.jackson@gmail.com', 76),
('Lebron James Cruz', 'lebron_cruz@gmail.com', 41);
```

**Verify the inserted data:**
```sql
SELECT * FROM users;
```
---
### 🧩 Next — Creating Another Table
After inserting users, we continue building the system by creating additional tables.
These tables will be used later for borrowing and returning items.

**Borrowed Table**
```sql
CREATE TABLE borrowed (
    borrow_id TEXT PRIMARY KEY,
    inventory_id UUID REFERENCES inventory(id) ON DELETE CASCADE,
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,
    transaction_date TIMESTAMPTZ DEFAULT NOW(),
    quantity INT NOT NULL
);
```

**Returned Table**
```sql
CREATE TABLE returned (
    return_id TEXT PRIMARY KEY,
    borrow_id TEXT REFERENCES borrowed(borrow_id) ON DELETE CASCADE,
    inventory_id UUID REFERENCES inventory(id) ON DELETE CASCADE,
    user_id BIGINT REFERENCES users(id) ON DELETE CASCADE,
    return_date TIMESTAMPTZ DEFAULT NOW(),
    quantity INT NOT NULL
);
```

### 🧠 What You Should Observe
- PostgreSQL supports multi‑row inserts
- Foreign keys enforce relationships between tables
- `borrowed` and `returned` tables depend on both users and inventory
- These tables will later be used by your stored procedures and .NET MAUI app

