# [EDB] Advance Server - Install

Owner: Nam Tran
Last edited time: March 11, 2026 12:08 PM

- Set up the EDB repository.
    
    Setting up the repository is a one-time task. If you already set up your repository, you don't need to perform this step.
    
    To determine if your repository exists, enter:
    
    `dnf repolist | grep enterprisedb`
    
    If no output is generated, the repository is installed.
    
    To set up the EDB repository:
    
    1. Go to [EDB repositories](https://www.enterprisedb.com/repos-downloads).
    2. Select the button that provides access to the EDB repository.
    3. Select the platform and software that you want to download.
    4. Follow the instructions for setting up the EDB repository.
- Install the EPEL repository:
    
    ```bash
    sudo dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    ```
    
- If you are also installing PostGIS, enable additional repositories to resolve dependencies:
    
    ```bash
    ARCH=$( /bin/arch ); subscription-manager repos --enable "codeready-builder-for-rhel-9-${ARCH}-rpms"
    ```
    
    <aside>
    💡
    
    **Note**
    
    If you are using a public cloud RHEL image, `subscription manager` may not be enabled and enabling it may incur unnecessary charges. Equivalent packages may be available under a different name such as `codeready-builder-for-rhel-9-rhui-rpms`. Consult the documentation for the RHEL image you are using to determine how to install `codeready-builder`.
    
    </aside>
    

## **Install the package**

```bash
sudo dnf -y install edb-as<xx>-server
```

Where `<xx>` is the version of the EDB Postgres Advanced Server you're installing. For example, if you're installing version 18, the package name is `edb-as16-server`.

```bash
[root@edb-pg-dev02 ~]# mkdir -p /usr/edb/as16/data
[root@edb-pg-dev02 ~]# chown -R enterprisedb:enterprisedb /usr/edb/as16/data/
[root@edb-pg-dev02 ~]# chmod -R 0700 /usr/edb/as16/data/
[root@edb-pg-dev02 ~]# unlink /var/lib/edb/as16/data
[root@edb-pg-dev02 ~]# sudo -u enterprisedb ln -s /usr/edb/as16/data/ /var/lib/edb/as16/data

[root@edb-pg-dev02 ~]# sudo -u enterprisedb /usr/edb/as16/bin/initdb -D /usr/edb/as16/data/
The files belonging to this database system will be owned by user "enterprisedb".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.
Transparent data encryption is disabled.

fixing permissions on existing directory /usr/edb/as16/data ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Asia/Ho_Chi_Minh
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
creating edb sys ... ok
loading edb contrib modules ...
edb_redwood_bytea.sql
edb_redwood_date.sql
dbms_alert_public.sql
dbms_alert.plb
dbms_job_public.sql
dbms_job.plb
dbms_lob_public.sql
dbms_lob.plb
dbms_output_public.sql
dbms_output.plb
dbms_pipe_public.sql
dbms_pipe.plb
dbms_rls_public.sql
dbms_rls.plb
dbms_sql_public.sql
dbms_sql.plb
dbms_utility_public.sql
dbms_utility.plb
dbms_aqadm_public.sql
dbms_aqadm.plb
dbms_aq_public.sql
dbms_aq.plb
dbms_profiler_public.sql
dbms_profiler.plb
dbms_random_public.sql
dbms_random.plb
dbms_redact_public.sql
dbms_redact.plb
dbms_lock_public.sql
dbms_lock.plb
dbms_scheduler_public.sql
dbms_scheduler.plb
dbms_crypto_public.sql
dbms_crypto.plb
dbms_mview_public.sql
dbms_mview.plb
dbms_session_public.sql
dbms_session.plb
dbms_privilege_capture_public.sql
dbms_privilege_capture.plb
edb_bulkload.sql
edb_gen.sql
edb_objects.sql
edb_redwood_casts.sql
edb_redwood_strings.sql
edb_redwood_views.sql
utl_encode_public.sql
utl_encode.plb
utl_http_public.sql
utl_http.plb
utl_file.plb
edb_ht_public.sql
edb_ht.plb
utl_tcp_public.sql
utl_tcp.plb
utl_smtp_public.sql
utl_smtp.plb
utl_mail_public.sql
utl_mail.plb
utl_url_public.sql
utl_url.plb
utl_raw_public.sql
utl_raw.plb
edb_gen_redwood.sql
waitstates.sql
installing extension edb_dblink_libpq ... ok
installing extension edb_dblink_oci ... ok
snap_tables.sql
snap_functions.sql
dblink_ora.sql
sys_stats.sql
ok
finalizing initial databases ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
initdb: hint: You can change this by editing pg_hba.conf or using the option -A, or --auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

   /usr/edb/as16/bin/pg_ctl -D /usr/edb/as16/data/ -l logfile start
```

```bash
[root@edb-pg-dev02 ~\]# systemctl daemon-reload
[root@edb-pg-dev02 ~\]# systemctl start edb-as-16.service
```

```bash
[root@edb-pg-dev02 ~\]# systemctl status edb-as-16.service
● edb-as-16.service - EDB Postgres Advanced Server 16
     Loaded: loaded (/usr/lib/systemd/system/edb-as-16.service; enabled; preset: disabled)
     Active: active (running) since Wed 2025-06-04 14:30:05 +07; 5s ago
    Process: 183067 ExecStartPre=/usr/edb/as16/bin/edb-as-16-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
   Main PID: 183072 (edb-postgres)
      Tasks: 8 (limit: 22988)
     Memory: 23.6M
        CPU: 85ms
     CGroup: /system.slice/edb-as-16.service
             ├─183072 /usr/edb/as16/bin/edb-postgres -D /var/lib/edb/as16/data
             ├─183073 "postgres: logger "
             ├─183074 "postgres: checkpointer "
             ├─183075 "postgres: background writer "
             ├─183077 "postgres: walwriter "
             ├─183078 "postgres: autovacuum launcher "
             ├─183079 "postgres: dbms_aq launcher "
             └─183080 "postgres: logical replication launcher "

Jun 04 14:30:05 edb-pg-dev02.unit.local systemd\[1\]: Starting EDB Postgres Advanced Server 16...
Jun 04 14:30:05 edb-pg-dev02.unit.local edb-postgres\[183072\]: 2025-06-04 14:30:05 +07 LOG: redirecting log output to logging collector process
Jun 04 14:30:05 edb-pg-dev02.unit.local edb-postgres\[183072\]: 2025-06-04 14:30:05 +07 HINT: Future log output will appear in directory "log".
Jun 04 14:30:05 edb-pg-dev02.unit.local systemd\[1\]: Started EDB Postgres Advanced Server 16.
```