# PostgreSQL

Owner: Nam Tran
Last edited time: January 21, 2026 10:48 PM

**POSTGRESQL**

[[PG] Drop database](PostgreSQL/%5BPG%5D%20Drop%20database%20224675ca799580fa9765f72e535509fe.md)

[[PG] List all db on server](PostgreSQL/%5BPG%5D%20List%20all%20db%20on%20server%202ca675ca79958080b1b1ccb731420944.md)

[[PG] Create database](PostgreSQL/%5BPG%5D%20Create%20database%20224675ca799580daae8ae78715e1283f.md)

[[PG] Grant permission](PostgreSQL/%5BPG%5D%20Grant%20permission%20224675ca7995804d83c0cdd8c1ee99e1.md)

[[PG] Determine the number of users connected to a PostgreSQL database](PostgreSQL/%5BPG%5D%20Determine%20the%20number%20of%20users%20connected%20to%20a%20%2025d675ca799580d7bd06f55a22032d8b.md)

[[PG] Set default schema](PostgreSQL/%5BPG%5D%20Set%20default%20schema%20292675ca79958061be7bd14a318f4876.md)

[[PG] Dùng pg_cron để tạo Schedule job](PostgreSQL/%5BPG%5D%20D%C3%B9ng%20pg_cron%20%C4%91%E1%BB%83%20t%E1%BA%A1o%20Schedule%20job%20293675ca799580d58430d9594d2a9dfd.md)

[[PG] Create monitoring user](PostgreSQL/%5BPG%5D%20Create%20monitoring%20user%20224675ca799580ba9c47e38b73a7a474.md)

[[PG] Check and fix issue overload cpu](PostgreSQL/%5BPG%5D%20Check%20and%20fix%20issue%20overload%20cpu%2025d675ca799580adb24ee78525f56c4d.md)

[[PG] Install HA **PostgreSQL cluster in linux using repmgr and keepalived on Ubuntu**](PostgreSQL/%5BPG%5D%20Install%20HA%20PostgreSQL%20cluster%20in%20linux%20using%20%202e3675ca79958085bf9cf19d7259859c.md)

[[PG] Install HA **PostgreSQL cluster in linux using repmgr and keepalived on RHEL**](PostgreSQL/%5BPG%5D%20Install%20HA%20PostgreSQL%20cluster%20in%20linux%20using%20%202ef675ca7995800c99e4fe661a4382a1.md)

The error you’re encountering (ValidationFailedException) occurs because the checksum of a changeset in Liquibase has changed. This typically happens when the changeset definition (e.g., the file contents, structure, or metadata) has been modified after it was initially executed.

Here’s how to address this issue:

Understand the Problem

- Checksum in Liquibase:
- When a changeset is executed, Liquibase calculates and stores its checksum in the DATABASECHANGELOG table.
- If the changeset’s contents change (e.g., modifications to the META-INF/jpa-changelog-14.0.0.xml file), the checksum recalculates and no longer matches the one stored in the database, causing the validation error.
- Possible Causes:
- The file jpa-changelog-14.0.0.xml was manually edited.
- The database state and Liquibase changelog are out of sync.
- Changeset definition changes, such as renaming columns or altering table structures.

If the changeset is correct and should be revalidated, update the checksum in the database.

1. Connect to the database:

```sql
psql -h <host> -p <port> -U <username> -d <database>
```

2.	Find the offending changeset in the DATABASECHANGELOG table:

```sql
SELECT * FROM DATABASECHANGELOG WHERE ID = '14.0.0-KEYCLOAK-18286-supported-dbs';
```

3.	Update the checksum:

```sql
UPDATE DATABASECHANGELOG
SET MD5SUM = '8:f43dfba07ba249d5d932dc489fd2b886'
WHERE ID = '14.0.0-KEYCLOAK-18286-supported-dbs'
AND FILENAME = 'META-INF/jpa-changelog-14.0.0.xml';
```

**Set `datestyle`**

```sql
alter database n8ndb set datestyle to 'iso, mdy';
```