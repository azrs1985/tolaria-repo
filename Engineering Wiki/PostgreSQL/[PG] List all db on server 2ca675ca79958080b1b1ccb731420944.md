# [PG] List all db on server

Owner: Nam Tran
Last edited time: December 15, 2025 10:29 AM

```sql
WITH db AS (
  SELECT
    d.oid,
    d.datname,
    r.rolname AS owner,
    d.encoding,
    d.datcollate,
    d.datctype
  FROM pg_database AS d
  JOIN pg_roles   AS r ON r.oid = d.datdba
  WHERE d.datistemplate = false
),
last_conn AS (
  -- Pick the most recent current session per database
  SELECT datid,
         client_addr,
         backend_start,
         row_number() OVER (
           PARTITION BY datid
           ORDER BY backend_start DESC NULLS LAST
         ) AS rn
  FROM pg_stat_activity
  WHERE client_addr IS NOT NULL
)
SELECT
  db.datname                               AS database_name,
  db.owner                                 AS owner,
  pg_size_pretty(pg_database_size(db.datname)) AS size,
  db.encoding,
  db.datcollate,
  db.datctype,
  lc.backend_start                          AS last_current_backend_start,
  lc.client_addr                            AS last_current_client_ip
FROM db
LEFT JOIN last_conn lc
  ON lc.datid = db.oid AND lc.rn = 1
ORDER BY db.datname;

```