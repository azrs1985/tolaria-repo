# [PG] Dùng pg_cron để tạo Schedule job

Owner: Nam Tran
Last edited time: October 22, 2025 11:29 AM

## INSTALL `edb-as16-pg-cron1` ON RHEL

Cài đặt `edb-as16-pg-cron1`

```sql
dnf install edb-as16-pg-cron1
```

Modify `/usr/edb/as16/data/postgresql.conf`

```bash
...
# - Shared Library Preloading - add 'pg_cron'
shared_preload_libraries = '$libdir/dbms_pipe,$libdir/edb_gen,$libdir/dbms_aq,pg_stat_statements,pg_cron'
...
# Add settings for extensions here
cron.database_name = 'edb'
cron.timezone = 'Asia/Ho_Chi_Minh'
```

Khởi động lại `edb-as-16.service`

```bash
 systemctl restart edb-as-16.service
```

### CREATE CRON JOB ON DATABASE

Tạo extension trong database

```sql
CREATE EXTENSION pg_cron;
```

Xem danh sách các job đã lên lịch

```sql
SELECT * FROM cron.job
```

Tạo cron job 

```sql
SELECT cron.schedule(
	'drop_conn_idle',                     -- jobname
  '*/30 * * * *',                       -- schedule
  $$SELECT pg_terminate_backend(pid)    -- command
    FROM pg_stat_activity
    WHERE state = 'idle'
      AND pid <> pg_backend_pid();$$
);
```

Xoá cron job

```sql
SELECT cron.unschedule(<jobname>);
-- or
SELECT cron.unschedule(<jobid>::bigint);
```

Kiểm tra lịch sử cron job đã chạy

```sql
SELECT jobid, status, start_time, end_time, return_message
FROM cron.job_run_details
ORDER BY start_time DESC
LIMIT 50;
```

Kiểm tra timezone của cron job

```sql
SHOW cron.timezone;
```