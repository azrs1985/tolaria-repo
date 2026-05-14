# [SQL SERVER] Change Owner of Database

Owner: Nam Tran
Last edited time: December 31, 2025 11:48 AM

```sql
-- Kiểm tra <user_db> có tồn tại trong DB không
USE [<user_db>];
GO
SELECT name, type_desc, sid
FROM sys.database_principals
WHERE name = N'<user_db>';

-- Kiểm tra và bỏ membership khỏi các database roles (nếu có)
-- Liệt kê role membership của user ado_dev
SELECT dp1.name  AS db_role,
       dp2.name  AS db_user
FROM sys.database_role_members AS drm
JOIN sys.database_principals AS dp1 ON drm.role_principal_id = dp1.principal_id
JOIN sys.database_principals AS dp2 ON drm.member_principal_id = dp2.principal_id
WHERE dp2.name = N'<user_db>';

-- Nếu có role, gỡ từng cái:
EXEC sp_droprolemember N'db_datareader', N'<user_db>';
EXEC sp_droprolemember N'db_datawriter', N'<user_db>';
-- ... gỡ các role khác nếu liệt kê thấy (db_ddladmin, db_owner, v.v.)

-- Kiểm tra Schema ownership
SELECT s.name AS schema_name
FROM sys.schemas s
WHERE s.principal_id = USER_ID(N'<user_db>');

-- Drop <user_db> trong DB
DROP USER [<user_db>];

-- Đổi owner bằng ALTER AUTHORIZATION
ALTER AUTHORIZATION ON DATABASE::[<database>] TO [<user_db>];

-- Kiểm tra kết quả
SELECT name, SUSER_SNAME(owner_sid) AS owner_login
FROM sys.databases
WHERE name = N'<user_db>';
```