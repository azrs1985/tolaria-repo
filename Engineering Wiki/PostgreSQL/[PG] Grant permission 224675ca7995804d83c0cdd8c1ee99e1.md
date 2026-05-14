# [PG] Grant permission

Owner: Nam Tran
Last edited time: September 3, 2025 12:46 PM

<aside>

Database name: `*<database_name>*`

Login user: *`<login_user>`*

Login password: `*<login_user_password>*`

Database user: `<db_user>`

</aside>

```sql
--Grant permission
ALTER DATABASE *<database_name>* OWNER TO <db_user>;
GRANT TEMPORARY, CONNECT ON DATABASE *<database_name>* TO PUBLIC;
GRANT CREATE, CONNECT ON DATABASE *<database_name>* TO <db_user>;
GRANT USAGE, SELECT, CREATE ON ALL SEQUENCES IN SCHEMA public TO <db_user>;
GRANT TEMPORARY ON DATABASE *<database_name>* TO <db_user> WITH GRANT OPTION;
GRANT pg_read_all_data TO <db_user>;
GRANT pg_write_all_data TO <db_user>;
```