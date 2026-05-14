# [PG] Determine the number of users connected to a PostgreSQL database

Owner: Nam Tran
Last edited time: October 28, 2025 3:17 PM

To determine the number of users connected to a PostgreSQL database, you can query the `pg_stat_activity` system view. (*Để xác định số người dùng đang kết nối tới cơ sở dữ liệu PostgreSQL, bạn có thể truy vấn hệ quan sát `pg_stat_activity`).*

To get a count of all current connections, regardless of the user or database (*Để đếm tất cả các kết nối hiện tại, không phân biệt người dùng hay cơ sở dữ liệu*):

```sql
SELECT count(*)
FROM pg_stat_activity;

-- with state = 'idle'
SELECT count(*) 
FROM pg_stat_activity
WHERE state = 'idle';
```

To see the number of connections per user (*Để xem số lượng kết nối theo từng người dùng*):

```sql
SELECT usename, COUNT(*) FROM pg_stat_activity GROUP BY usename;

-- with state = 'idle'
SELECT usename, COUNT(*)
FROM pg_stat_activity
WHERE state = 'idle'
GROUP BY usename;
```

To see the number of connections per database (*Để xem số lượng kết nối theo từng cơ sở dữ liệu*):

```sql
SELECT datname, COUNT(*) FROM pg_stat_activity GROUP BY datname;
```

List and count a user's connections to each database, including client address and application name (*Liệt kê và đếm số kết nối của một người dùng tới từng cơ sở dữ liệu, kèm địa chỉ client và tên ứng dụng*).

```sql
-- list and count connections of a username to databases
SELECT datname, client_addr, application_name, COUNT(*) AS connection_count
FROM pg_stat_activity
WHERE usename = '<db_user>'
GROUP BY datname, client_addr, application_name
ORDER BY datname ASC, connection_count DESC;
```

Terminate all connections of a user that are in the idle state (*Ngắt tất cả kết nối của một người dùng khi trạng thái là idle*).

```sql
-- terminate all idle connections of a given user
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE usename = '<db_user>'
AND state = 'idle'
  AND pid <> pg_backend_pid();
```

Terminate all connections that are in the idle state (*Ngắt tất cả kết nối có trạng thái idle*).

```sql
-- terminate all idle connections
-- Ngắt tất cả kết nối có trạng thái idle
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE state = 'idle'
  AND pid <> pg_backend_pid();
```