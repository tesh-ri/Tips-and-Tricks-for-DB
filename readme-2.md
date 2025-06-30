
# 🛠️ PostgreSQL Best Practices – Code Examples

This guide includes actionable code snippets for testing, performance analysis, automation, documentation, and monitoring in PostgreSQL.

---

## ✅ 1. Always Test on Staging

### a. Use `.env` for environment-specific config
```env
# .env.staging
DB_HOST=staging-db.example.com
DB_NAME=myapp_staging
```

### b. Use configuration in backend code (e.g., Node.js/NestJS)
```ts
const dbConfig = {
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
};
```

### c. Prevent destructive actions in production (PostgreSQL logic)
```sql
DO $$
BEGIN
  IF current_setting('server_version') != '15.0' THEN
    RAISE EXCEPTION 'Only allowed in staging';
  END IF;
END $$;
```

---

## ✅ 2. Use `EXPLAIN ANALYZE` to Optimize Queries

### Example
```sql
EXPLAIN ANALYZE
SELECT * FROM orders
WHERE customer_id = 123
AND order_date >= NOW() - INTERVAL '1 month';
```

This shows execution plans, index usage, and query costs. Use it for tuning performance.

---

## ✅ 3. Automate Backups

### a. Bash Script for Daily Backup (`backup.sh`)
```bash
#!/bin/bash

DB_NAME="your_db_name"
DATE=$(date +"%Y-%m-%d")
BACKUP_FILE="/backups/${DB_NAME}_backup_$DATE.sql"

pg_dump -U postgres -d $DB_NAME -F c -f "$BACKUP_FILE"
```

### b. Schedule via Cron
```bash
crontab -e
# Run backup every day at 2AM
0 2 * * * /path/to/backup.sh
```

---

## ✅ 4. Document Your DB Schema

### a. Schema-only SQL Dump
```bash
pg_dump -U postgres -d your_db --schema-only > schema.sql
```

### b. Query to View Tables and Columns
```sql
SELECT table_name, column_name, data_type
FROM information_schema.columns
WHERE table_schema = 'public'
ORDER BY table_name, ordinal_position;
```

### c. Use Tools
- [dbdocs.io](https://dbdocs.io)
- [SchemaSpy](http://schemaspy.org/)

---

## ✅ 5. Monitor Long-Running Queries

### a. Find Active Queries Running >1 Minute
```sql
SELECT pid, now() - pg_stat_activity.query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active'
  AND now() - query_start > interval '1 minute'
ORDER BY duration DESC;
```

### b. Use `pg_stat_statements` for Historical Query Stats
```sql
-- Enable in postgresql.conf:
shared_preload_libraries = 'pg_stat_statements';

-- Then:
CREATE EXTENSION pg_stat_statements;

-- Query top queries by execution time:
SELECT * FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;
```

### c. Use Monitoring Tools
- Prometheus + PostgreSQL Exporter
- pgMonitor (CrunchyData)
- pgAdmin dashboards

---

