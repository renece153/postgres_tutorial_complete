# 📘 Part 3 — Working With the Borrow/Return System
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

``sql
SELECT * FROM inventory_availability;
```

Expected:
- `remaining_quantity = 10`
- `latest_transaction_type = NULL (no activity yet)`

### 🔍 9.3 — Test Borrow Procedure

Borrow 3 units:
``sql
SELECT safe_insert_borrowed_item(
    p_inventory_id := '<your-inventory-uuid>',
    p_user_id := 1,
    p_quantity := 3
);
```
Replace `<your-inventory-uuid>` with the actual UUID from the inventory table.

**Check availability again:**
``sql
SELECT * FROM inventory_availability;
```
Expected:
- `remaining_quantity = 7`
- `latest_transaction_type = 'Borrowed'`

### 🔍 9.4 — Test Over‑Borrowing (Should Fail)
**Check availability again:**
``sql
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

