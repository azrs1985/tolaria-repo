# [PG] Check and fix issue overload cpu

Owner: Nam Tran
Last edited time: December 30, 2025 1:19 PM

```sql
-- Check status DB
SELECT datname, pid, usename, application_name, client_addr, backend_start, query_start, state, query
FROM pg_stat_activity
WHERE state = 'active';
```

```sql
-- Terminate Active Sessions Immediately
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE usename = '<db_user>' AND pid <> pg_backend_pid();
```

```sql
-- Terminate pid
SELECT pg_cancel_backend(802535);
```