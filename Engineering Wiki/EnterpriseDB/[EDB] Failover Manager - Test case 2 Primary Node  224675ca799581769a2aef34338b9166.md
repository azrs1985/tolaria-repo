# [EDB] Failover Manager - Test case 2: Primary Node lost AS services

Owner: Nam Tran
Last edited time: March 10, 2026 3:21 PM

# Test case 2: Primary Node lost AS services

```
[root@node02]# systemctl stop edb-as-16.services
```

```
[root@node03]# /usr/edb/efm-4.10/bin/efm cluster-status efm
Cluster Status: efm

        Agent Type  Address              DB       VIP
        ----------------------------------------------------------------
        Standby     192.168.11.26        UP       192.168.11.25
        Primary     192.168.11.27        UP       192.168.11.25*
        Witness     192.168.11.28        N/A      192.168.11.25

Allowed node host list:
        192.168.11.26 192.168.11.27 192.168.11.28

Membership coordinator: 192.168.11.28

Standby priority host list:
        192.168.11.26

Promote Status:

        DB Type     Address              WAL Received LSN   WAL Replayed LSN   Info
        ---------------------------------------------------------------------------
        UNKNOWN     192.168.11.27        UNKNOWN            UNKNOWN            Connection to 192.168.11.27:5444 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.
        Standby     192.168.11.26        0/D0001C0          0/D0001C0

        No primary database was found.
```

```
[root@node03]#  /usr/edb/efm-4.10/bin/efm cluster-status efm
Cluster Status: efm

        Agent Type  Address              DB       VIP
        ----------------------------------------------------------------
        Primary     192.168.11.26        UP       192.168.11.25*
        Idle        192.168.11.27        UNKNOWN  192.168.11.25
        Witness     192.168.11.28        N/A      192.168.11.25

Allowed node host list:
        192.168.11.26 192.168.11.27 192.168.11.28

Membership coordinator: 192.168.11.28

Standby priority host list:
        (List is empty.)

Promote Status:

        DB Type     Address              WAL Received LSN   WAL Replayed LSN   Info
        ---------------------------------------------------------------------------
        Primary     192.168.11.26                           0/D000310

        No standby databases were found.

Idle Node Status (idle nodes ignored in WAL LSN comparisons):

        Address              WAL Received LSN   WAL Replayed LSN   Info
        ---------------------------------------------------------------
        192.168.11.27        UNKNOWN            UNKNOWN            Connection to 192.168.11.27:5444 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.
```

```
[root@node02]# systemctl start edb-as-16.service
Job for edb-as-16.service failed because the control process exited with error code.
See "systemctl status edb-as-16.service" and "journalctl -xeu edb-as-16.service" for details.

[root@node02 /]# cat /usr/edb/as16/data/log/enterprisedb-2024-11-18_150454.log
2024-11-18 15:04:54 +07 LOG:  starting PostgreSQL 16.4 (EnterpriseDB Advanced Server 16.4.1) on x86_64-pc-linux-gnu, compiled by gcc (GCC) 11.4.1 20231218 (Red Hat 11.4.1-3), 64-bit
2024-11-18 15:04:54 +07 LOG:  listening on IPv4 address "0.0.0.0", port 5444
2024-11-18 15:04:54 +07 LOG:  listening on IPv6 address "::", port 5444
2024-11-18 15:04:54 +07 LOG:  listening on Unix socket "/tmp/.s.PGSQL.5444"
2024-11-18 15:04:54 +07 LOG:  database system was shut down at 2024-11-18 15:01:31 +07
2024-11-18 15:04:54 +07 FATAL:  using recovery command file "recovery.conf" is not supported
2024-11-18 15:04:54 +07 LOG:  startup process (PID 63042) exited with exit code 1
2024-11-18 15:04:54 +07 LOG:  aborting startup due to startup process failure
2024-11-18 15:04:54 +07 LOG:  database system is shut down
[root@node02]# rm -f /usr/edb/as16/data/recovery.conf
[root@node02]# systemctl start edb-as-16.service
```

```
[root@node03]# /usr/edb/efm-4.10/bin/efm cluster-status efm
Cluster Status: efm

        Agent Type  Address              DB       VIP
        ----------------------------------------------------------------
        Primary     192.168.11.26        UP       192.168.11.25*
        Idle        192.168.11.27        UNKNOWN  192.168.11.25
        Witness     192.168.11.28        N/A      192.168.11.25

Allowed node host list:
        192.168.11.26 192.168.11.27 192.168.11.28

Membership coordinator: 192.168.11.28

Standby priority host list:
        (List is empty.)

Promote Status:

        DB Type     Address              WAL Received LSN   WAL Replayed LSN   Info
        ---------------------------------------------------------------------------
        Primary     192.168.11.26                           0/D000310

        No standby databases were found.

Idle Node Status (idle nodes ignored in WAL LSN comparisons):

        Address              WAL Received LSN   WAL Replayed LSN   Info
        ---------------------------------------------------------------
        192.168.11.27                           0/D0001F8          DB is not in recovery.
```

```
[root@node02]# rm -rf /usr/edb/as16/data/
[root@node02]# pg_basebackup -h 192.168.11.26 -U repl -p 5444 -D /usr/edb/as16/data -Fp -Xs -P -R
84885/84885 kB (100%), 1/1 tablespace
[root@node02]# chown enterprisedb:enterprisedb -R /usr/edb/as16/data
[root@node02]# chmod 0700 /usr/edb/as16/data
[root@node02]# systemctl start edb-as-16.service && systemctl start edb-efm-4.10.service
```

```
[root@node03]# /usr/edb/efm-4.10/bin/efm cluster-status efm

Cluster Status: efm

        Agent Type  Address              DB       VIP
        ----------------------------------------------------------------
        Primary     192.168.11.26        UP       192.168.11.25*
        Standby     192.168.11.27        UP       192.168.11.25
        Witness     192.168.11.28        N/A      192.168.11.25

Allowed node host list:
        192.168.11.26 192.168.11.27 192.168.11.28

Membership coordinator: 192.168.11.28

Standby priority host list:
        192.168.11.27

Promote Status:

        DB Type     Address              WAL Received LSN   WAL Replayed LSN   Info
        ---------------------------------------------------------------------------
        Primary     192.168.11.26                           0/F000060
        Standby     192.168.11.27        0/F000060          0/F000060

        Standby database(s) in sync with primary. It is safe to promote.
```