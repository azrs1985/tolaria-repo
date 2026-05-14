# [SQL SERVER] Create new database, user and Grant permissions

Owner: Nam Tran
Last edited time: October 27, 2025 2:07 PM

<aside>
<img src="notion://custom_emoji/2a84f521-0191-4a2f-9b66-dc4f1b3b0287/22a675ca-7995-80c5-98a8-007a80b2969f" alt="notion://custom_emoji/2a84f521-0191-4a2f-9b66-dc4f1b3b0287/22a675ca-7995-80c5-98a8-007a80b2969f" width="40px" />

Database name: `*<database_name>*`

Login user: *`<login_user>`*

Login password: `*<login_user_password>*`

Database user: `<db_user>`

</aside>

```sql
USE master;
-- Create database
CREATE DATABASE *<database_name>*;

-- Create login if it doesn't exist
IF NOT EXISTS (SELECT 1 FROM sys.server_principals WHERE name = '*<login_user>*')
BEGIN
    CREATE LOGIN *<login_user>* WITH PASSWORD = '*<login_user_password>*', CHECK_EXPIRATION = OFF, CHECK_POLICY = OFF;
END;

-- Create user in *<database_name>* and grant permissions
USE *<database_name>*;
IF NOT EXISTS (SELECT 1 FROM sys.database_principals WHERE name = '*<db_user>*')
BEGIN
  CREATE USER *<db_user>* FOR LOGIN *<login_user>*;
END;

GRANT SELECT, INSERT, UPDATE, ALTER, DELETE, REFERENCES ON DATABASE::*<database_name>* TO *<db_user>*;

-- Assign roles foe user
ALTER ROLE db_owner ADD MEMBER *<db_user>*;

-- Verify permissions
SELECT 
    p.name AS UserName,
    p.type_desc AS UserType,
    perm.permission_name AS Permission,
    perm.state_desc AS PermissionState,
    perm.class_desc AS PermissionClass,
    CASE 
        WHEN perm.class_desc = 'OBJECT' THEN OBJECT_NAME(perm.major_id)
        WHEN perm.class_desc = 'SCHEMA' THEN SCHEMA_NAME(perm.major_id)
        WHEN perm.class_desc = 'DATABASE' THEN DB_NAME()
        ELSE CAST(perm.major_id AS nvarchar(128))
    END AS Scope
FROM sys.database_principals p
LEFT JOIN sys.database_permissions perm 
    ON p.principal_id = perm.grantee_principal_id
WHERE p.name = '<*db_user*>'
ORDER BY perm.permission_name;

-- Verify roles
SELECT DP.name AS UserName, DR.name AS RoleName
FROM sys.database_role_members DRM
JOIN sys.database_principals DR ON DRM.role_principal_id = DR.principal_id
JOIN sys.database_principals DP ON DRM.member_principal_id = DP.principal_id
WHERE DP.name = '<*db_user*>';
```

[Project **DM01Y25DMS**](%5BSQL%20SERVER%5D%20Create%20new%20database,%20user%20and%20Grant%20p/Project%20DM01Y25DMS%20255675ca799580dd9aafc9cfa0585364.md)

[Project ADO](%5BSQL%20SERVER%5D%20Create%20new%20database,%20user%20and%20Grant%20p/Project%20ADO%2026b675ca7995800eacf1f732b76c60d5.md)