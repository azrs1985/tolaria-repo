# [PG] Create monitoring user

Owner: Nam Tran
Last edited time: December 30, 2025 10:13 AM

```sql
-- Tạo user mới với password (thay 'your_password' bằng mật khẩu mạnh)
CREATE ROLE monitoring WITH LOGIN PASSWORD 'Unit@032010';

-- Grant role read-only toàn server
GRANT pg_read_all_data TO monitoring;

-- Nếu PostgreSQL 10 trở lên (hiện tại 2025 nên chắc chắn có)
GRANT pg_monitor TO monitoring;

-- Hoặc cụ thể hơn, chỉ cấp quyền đọc stats (không bao gồm một số quyền khác của pg_monitor)
GRANT pg_read_all_stats TO monitoring;
```