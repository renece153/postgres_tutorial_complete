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

## 🧩 Step 7 — Create the inventory_availability View
This view calculates:
- Total borrowed
- Total returned
- Remaining quantity
- Latest transaction (Borrowed or Returned)
```sql
CREATE OR REPLACE VIEW inventory_availability AS
WITH borrow_totals AS (
    SELECT 
        inventory_id,
        COALESCE(SUM(quantity), 0) AS total_borrowed
    FROM borrowed
    GROUP BY inventory_id
),
return_totals AS (
    SELECT 
        inventory_id,
        COALESCE(SUM(quantity), 0) AS total_returned
    FROM returned
    GROUP BY inventory_id
),
latest_activity AS (
    SELECT 
        inventory_id,
        GREATEST(
            COALESCE(MAX(transaction_date), '1900-01-01'),
            COALESCE((SELECT MAX(return_date) FROM returned r2 WHERE r2.inventory_id = b.inventory_id), '1900-01-01')
        ) AS latest_date,
        CASE 
            WHEN (SELECT MAX(transaction_date) FROM borrowed b2 WHERE b2.inventory_id = b.inventory_id) >=
                 (SELECT MAX(return_date) FROM returned r2 WHERE r2.inventory_id = b.inventory_id)
            THEN 'Borrowed'
            ELSE 'Returned'
        END AS latest_type
    FROM borrowed b
    GROUP BY inventory_id
)
SELECT 
    i.id AS inventory_id,
    i.name AS inventory_name,
    i.quantity AS max_quantity,
    (i.quantity 
        - COALESCE(bt.total_borrowed, 0) 
        + COALESCE(rt.total_returned, 0)
    ) AS remaining_quantity,
    la.latest_date AS latest_transaction_date,
    la.latest_type AS latest_transaction_type
FROM inventory i
LEFT JOIN borrow_totals bt ON bt.inventory_id = i.id
LEFT JOIN return_totals rt ON rt.inventory_id = i.id
LEFT JOIN latest_activity la ON la.inventory_id = i.id;
```

---

## 🧩 Step 8 — Create Stored Procedures
These stored procedures enforce business rules:
- No over‑borrowing
- No over‑returning
- Auto‑generate IDs (BUR-00001, RET-00001)

### 🔒 Safe Borrow Procedure
Prevents borrowing more than the remaining quantity.

```sql
CREATE OR REPLACE FUNCTION safe_insert_borrowed_item(
    p_inventory_id UUID,
    p_user_id BIGINT,
    p_quantity INT
)
RETURNS TEXT
LANGUAGE plpgsql
AS $$
DECLARE
    remaining INT;
    last_id TEXT;
    new_number INT;
    new_id TEXT;
BEGIN
    -- Get remaining quantity from the view
    SELECT remaining_quantity INTO remaining
    FROM inventory_availability
    WHERE inventory_id = p_inventory_id;

    IF remaining IS NULL THEN
        RAISE EXCEPTION 'Inventory item does not exist.';
    END IF;

    IF remaining <= 0 THEN
        RAISE EXCEPTION 'No stock available to borrow.';
    END IF;

    IF p_quantity > remaining THEN
        RAISE EXCEPTION 'Cannot borrow more than remaining quantity (%).', remaining;
    END IF;

    -- Generate BUR-00001 ID
    SELECT borrow_id INTO last_id
    FROM borrowed
    ORDER BY borrow_id DESC
    LIMIT 1;

    IF last_id IS NULL THEN
        new_number := 1;
    ELSE
        new_number := (regexp_replace(last_id, '\D', '', 'g'))::INT + 1;
    END IF;

    new_id := 'BUR-' || LPAD(new_number::TEXT, 5, '0');

    INSERT INTO borrowed (borrow_id, inventory_id, user_id, quantity)
    VALUES (new_id, p_inventory_id, p_user_id, p_quantity);

    RETURN new_id;
END;
$$;
```

### 🔄 Safe Return Procedure
Prevents returning more than what was borrowed.
```sql
CREATE OR REPLACE FUNCTION safe_insert_returned_item(
    p_borrow_id TEXT,
    p_inventory_id UUID,
    p_user_id BIGINT,
    p_quantity INT
)
RETURNS TEXT
LANGUAGE plpgsql
AS $$
DECLARE
    last_id TEXT;
    new_number INT;
    new_id TEXT;
    borrowed_qty INT;
    returned_qty INT;
BEGIN
    -- Get total borrowed for this borrow_id
    SELECT quantity INTO borrowed_qty
    FROM borrowed
    WHERE borrow_id = p_borrow_id;

    IF borrowed_qty IS NULL THEN
        RAISE EXCEPTION 'Borrow record does not exist.';
    END IF;

    -- Get total returned so far
    SELECT COALESCE(SUM(quantity), 0) INTO returned_qty
    FROM returned
    WHERE borrow_id = p_borrow_id;

    IF returned_qty + p_quantity > borrowed_qty THEN
        RAISE EXCEPTION 'Cannot return more than borrowed.';
    END IF;

    -- Generate RET-00001 ID
    SELECT return_id INTO last_id
    FROM returned
    ORDER BY return_id DESC
    LIMIT 1;

    IF last_id IS NULL THEN
        new_number := 1;
    ELSE
        new_number := (regexp_replace(last_id, '\D', '', 'g'))::INT + 1;
    END IF;

    new_id := 'RET-' || LPAD(new_number::TEXT, 5, '0');

    INSERT INTO returned (return_id, borrow_id, inventory_id, user_id, quantity)
    VALUES (new_id, p_borrow_id, p_inventory_id, p_user_id, p_quantity);

    RETURN new_id;
END;
$$;
```
