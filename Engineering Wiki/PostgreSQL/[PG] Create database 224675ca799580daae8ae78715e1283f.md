# [PG] Create database

Owner: Nam Tran
Last edited time: September 3, 2025 12:43 PM

<aside>

Database name: `*<database_name>*`

Login user: *`<login_user>`*

Login password: `*<login_user_password>*`

Database user: `<db_user>`

</aside>

```sql
--Create database
CREATE DATABASE *<database_name>*
WITH
OWNER = user_ies
ENCODING = 'UTF8'
LC_COLLATE = 'en_US.UTF-8'
LC_CTYPE = 'en_US.UTF-8'
TABLESPACE = pg_default
CONNECTION LIMIT = -1;
```