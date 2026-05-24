## 📘 Part 3 — Working With the Borrow/Return System
In this section, students will:
- Insert sample data into the borrowed and returned tables
- Observe how the `inventory_availability` view updates automatically
- Understand how PostgreSQL enforces business rules through stored procedures
- Prepare the database for integration with the .NET MAUI application

This part builds directly on the stored procedures created earlier.

## 🧩 9 - Testing the Stored Procedures
Now that the stored procedures are created, we will test them to ensure:
- Borrowing respects remaining quantity
- Returning respects borrowed quantity
- IDs auto‑generate correctly (BUR-00001, RET-00001)

The inventory_availability view updates automatically

```sql
INSERT INTO inventory (name, quantity)
VALUES ('Database Systems Book', 10);
```

Verify:
```sql
SELECT * FROM inventory
```

### 🔍 9.2 — Check Initial Availability

```sql
SELECT * FROM inventory_availability;
```

Expected:
- `remaining_quantity = 10`
- `latest_transaction_type = NULL (no activity yet)`

### 🔍 9.3 — Test Borrow Procedure

Borrow 3 units:
```sql
SELECT safe_insert_borrowed_item(
    p_inventory_id := '<your-inventory-uuid>',
    p_user_id := 1,
    p_quantity := 3
);
```
Replace `<your-inventory-uuid>` with the actual UUID from the inventory table.

**Check availability again:**
```sql
SELECT * FROM inventory_availability;
```
Expected:
- `remaining_quantity = 7`
- `latest_transaction_type = 'Borrowed'`

### 🔍 9.4 — Test Over‑Borrowing (Should Fail)
**Check availability again:**
```sql
SELECT safe_insert_borrowed_item(
    p_inventory_id := '<your-inventory-uuid>',
    p_user_id := 1,
    p_quantity := 20
);
```

Expected error:
```sql
ERROR: Cannot borrow more than remaining quantity (7). 
```
This confirms the procedure is working.

## 🧩 Part 10 — Borrow/Return Sample Data
Now we will simulate a full borrow/return cycle.
### 📦 10.1 — Borrow Items
Borrow 2 more units:
```sql
SELECT safe_insert_borrowed_item(
    p_inventory_id := '<your-inventory-uuid>',
    p_user_id := 2,
    p_quantity := 2
);
```
Borrow 1 more unit:
```sql
SELECT safe_insert_borrowed_item(
    p_inventory_id := '<your-inventory-uuid>',
    p_user_id := 3,
    p_quantity := 1
);
```
Check availability:
```sql
SELECT * FROM inventory_availability;
```

Expected:
- Total borrowed = 3 + 2 + 1 = 6
- Remaining = 10 − 6 = 4

### 🔄 10.2 — Return Items
Return 1 unit from the first borrow:
```sql
SELECT safe_insert_returned_item(
    p_borrow_id := 'BUR-00001',
    p_inventory_id := '<your-inventory-uuid>',
    p_user_id := 1,
    p_quantity := 1
);
```
Check availability:
```sql
SELECT * FROM inventory_availability;
```

Expected:
- Remaining = 10 − 6 + 1 = 5
- Latest transaction = `Returned`

### ❌ 10.3 — Test Over‑Returning (Should Fail)
```sql
SELECT safe_insert_returned_item(
    p_borrow_id := 'BUR-00001',
    p_inventory_id := '<your-inventory-uuid>',
    p_user_id := 1,
    p_quantity := 10
);
```

Expected error:
```
ERROR: Cannot return more than borrowed.
```

### 📊 10.4 — Verify All Tables
Borrowed records:
```sql
SELECT * FROM borrowed;
```
Returned records:
```sql
SELECT * FROM returned;
```

Availability view:
```sql
SELECT * FROM inventory_availability;
```
Everything should now reflect:
- Correct remaining quantity
- Correct latest transaction
- Correct totals
