# Create new database and user

Owner: Nam Tran
Last edited time: February 26, 2026 11:09 AM

```sql
-- Tạo database
CREATE DATABASE *<database_name>*;

-- Tạo login, user, và cấp quyền đầy đủ trên một database
CREATE LOGIN *<login_user>* WITH PASSWORD = *<login_user_password>*, CHECK_POLICY = OFF;

USE *<database_name>*;

ALTER AUTHORIZATION ON DATABASE::[<database>] TO [<*login_user>*];

CREATE USER *<user_name>* FOR LOGIN *<login_user>*;
ALTER ROLE db_owner ADD MEMBER *<user_name>*;
ALTER SERVER ROLE [dbcreator] ADD MEMBER *<user_name>*;

-- Kiểm tra quyền
SELECT DP.name AS UserName, DR.name AS RoleName
FROM sys.database_role_members DRM
JOIN sys.database_principals DR ON DRM.role_principal_id = DR.principal_id
JOIN sys.database_principals DP ON DRM.member_principal_id = DP.principal_id
WHERE DP.name = *<user_name>*;
```