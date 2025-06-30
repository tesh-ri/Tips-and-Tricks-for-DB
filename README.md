
# 🧠 PostgreSQL Tips & Tricks – Developer Guide

A complete guide for developers and backend engineers to handle PostgreSQL efficiently, safely, and professionally.

---

## 📥 INSERT Tips

### ✅ Use `ON CONFLICT` (UPSERT)
```sql
INSERT INTO table_name (id, value)
VALUES (1, 'abc')
ON CONFLICT (id) DO UPDATE
SET value = EXCLUDED.value;
```

### ✅ Insert from Another Table
```sql
INSERT INTO target_table (col1, col2)
SELECT col1, col2 FROM source_table
WHERE condition;
```

### ✅ Insert with `RETURNING`
```sql
INSERT INTO users (name) VALUES ('Ritesh') RETURNING *;
```

### ✅ Bulk Insert from CSV
```sql
COPY users(name, age) FROM '/path/users.csv' CSV HEADER;
```

### ✅ Insert All Combinations (Cross Join)
```sql
INSERT INTO mapping_table (uid, bid)
SELECT u.id, b.id FROM users u CROSS JOIN badges b;
```

---

## ✏️ UPDATE Tips

### ✅ Safe Update
```sql
UPDATE users SET status = 'inactive'
WHERE last_login < NOW() - INTERVAL '1 year';
```

### ✅ Conditional Update
```sql
UPDATE accounts
SET balance = CASE WHEN balance IS NULL THEN 0 ELSE balance + 100 END
WHERE uid = 'abc';
```

### ✅ Use `RETURNING`
```sql
UPDATE users SET status = 'active' WHERE id = 1 RETURNING *;
```

---

## 🗑️ DELETE Tips

### ✅ Safe Delete
```sql
DELETE FROM logs WHERE created_at < NOW() - INTERVAL '30 days';
```

### ✅ Delete within last 5 minutes
```sql
DELETE FROM table_name
WHERE created_at >= NOW() - INTERVAL '5 minutes';
```

### ✅ Delete with `RETURNING`
```sql
DELETE FROM sessions WHERE expired = true RETURNING *;
```

### ✅ Batch Delete for Huge Tables
```sql
DELETE FROM logs WHERE created_at < NOW() - INTERVAL '30 days' LIMIT 1000;
```

---

## 🔄 Data Copying Between Tables

### ✅ Copy Table Data
```sql
INSERT INTO new_table (col1, col2)
SELECT col1, col2 FROM old_table;
```

### ✅ With Transformation
```sql
INSERT INTO archive_logs (log, archived_at)
SELECT log, NOW() FROM logs WHERE created_at < NOW() - INTERVAL '1 month';
```

---

## ⏱️ Time-based Operations

### ✅ Get Data in Last 5 Minutes
```sql
SELECT * FROM events
WHERE created_at >= NOW() - INTERVAL '5 minutes';
```

### ✅ Get 5-Minute Intervals
```sql
SELECT date_trunc('minute', created_at) as minute_slot, COUNT(*)
FROM logs
GROUP BY minute_slot
ORDER BY minute_slot DESC;
```

---

## 🧾 TRANSACTIONS

### ✅ Usage
```sql
BEGIN;

-- insert/update/delete statements

COMMIT;
-- or use ROLLBACK; to undo
```

### ✅ When to Use
- Critical updates across multiple tables
- Bulk insert with rollback safety
- Database migrations

---

## 🧨 What is TRUNCATE?

```sql
TRUNCATE TABLE logs;
```
- Fast, irreversible deletion of all rows.
- Bypasses `ON DELETE` triggers.
- Use with caution in production.

---

## 🧠 Safety Tips Before Modifying DB

1. Always **take a backup** before mass delete/update.
2. Run `SELECT` first to preview.
3. Use `WHERE` clauses to limit scope.
4. Avoid `TRUNCATE` unless you're sure.
5. Avoid heavy operations in peak traffic.
6. Use **transactions** when updating multiple related rows.

---

## 💾 Taking a Backup

### ✅ Dump SQL Backup
```bash
pg_dump -U postgres -d dbname -f backup.sql
```

### ✅ Restore SQL Backup
```bash
psql -U postgres -d dbname -f backup.sql
```

---

## 🧰 Table Management Best Practices

- Normalize data where needed
- Add indexes on search-heavy columns
- Use `FOREIGN KEY` and `ON DELETE CASCADE` wisely
- Avoid storing large blobs (images/videos) directly
- Archive old data to separate tables
- Monitor table size using `pg_stat_user_tables`

---

## ❌ What to Avoid in PostgreSQL

- Using `SELECT *` in production code
- Missing indexes on `JOIN` or `WHERE` columns
- Overusing triggers (can reduce performance)
- Keeping unused tables
- Not using connection pooling
- Storing unencrypted sensitive data

---

## 📌 Helpful Queries

### ✅ Check Table Size
```sql
SELECT relname AS table, pg_size_pretty(pg_total_relation_size(relid)) AS size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(relid) DESC;
```

### ✅ Find Unused Indexes
```sql
SELECT * FROM pg_stat_user_indexes WHERE idx_scan = 0;
```

---

## 🧑‍💻 Final Advice

- Always test on staging.
- Use `EXPLAIN ANALYZE` to optimize queries.
- Automate backups.
- Document your DB schema.
- Set up alerting/monitoring for long-running queries.


