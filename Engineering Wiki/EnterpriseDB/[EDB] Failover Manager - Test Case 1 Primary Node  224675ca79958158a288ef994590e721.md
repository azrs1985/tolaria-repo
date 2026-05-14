# [EDB] Failover Manager - Test Case 1: Primary Node lost AS and EFM services

Owner: Nam Tran
Last edited time: March 11, 2026 3:13 PM

# Test Case 1: Primary Node lost AS and EFM services

```bash
[root@node01]# systemctl stop edb-efm-4.10.service && systemctl stop edb-as-16.service
```

```bash
[root@node03]# /usr/edb/efm-4.10/bin/efm cluster-status efm
Cluster Status: efm

        Agent Type  Address              DB       VIP
        ----------------------------------------------------------------
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
        Standby     192.168.11.27        0/90000D8          0/90000D8

        No primary database was found.

```

```bash
[root@node02]# /usr/edb/efm-4.10/bin/efm promote efm
The primary agent is not present but the VIP is in use. Before proceeding, you should ensure that the VIP is not in use by a node other than the standby node. Are you sure you want to continue? [y/N]: y
Would you like to attempt to promote the standby anyway [y/N]: y
Forcing this promotion may result in data loss - are you sure you want to continue? This is the last prompt. [y/N]: y
Promote command accepted by local agent. Proceeding with promotion. Run the 'cluster-status' command for information about the new cluster state.
```

```bash
[root@node03]# /usr/edb/efm-4.10/bin/efm cluster-status efm
Cluster Status: efm

        Agent Type  Address              DB       VIP
        ----------------------------------------------------------------
        Primary     192.168.11.27        UP       192.168.11.25*
        Witness     192.168.11.28        N/A      192.168.11.25

Allowed node host list:
        192.168.11.26 192.168.11.27 192.168.11.28

Membership coordinator: 192.168.11.28

Standby priority host list:
        (List is empty.)

Promote Status:

        DB Type     Address              WAL Received LSN   WAL Replayed LSN   Info
        ---------------------------------------------------------------------------
        Primary     192.168.11.27                           0/9000228

        No standby databases were found.

```

```bash
[root@node01]# systemctl start edb-as-16.service && systemctl start edb-efm-4.10.service
Job for edb-efm-4.10.service failed because the control process exited with error code.
See "systemctl status edb-efm-4.10.service" and "journalctl -xeu edb-efm-4.10.service" for details.
```

```bash
[root@node01]# systemctl status edb-efm-4.10.service
× edb-efm-4.10.service - EnterpriseDB Failover Manager 4.10
     Loaded: loaded (/usr/lib/systemd/system/edb-efm-4.10.service; disabled; preset: disabled)
     Active: failed (Result: exit-code) since Mon 2024-11-18 14:38:28 +07; 4min 0s ago
    Process: 140872 ExecStart=/bin/bash -c /usr/edb/efm-4.10/bin/runefm.sh start ${CLUSTER} (code=exited, status=1/FAILURE)
        CPU: 8.642s

Nov 18 14:38:21 node01 sudo[141124]: pam_unix(sudo:session): session opened for user enterprisedb(uid=991) by (uid=1000)
Nov 18 14:38:21 node01 sudo[141124]: pam_unix(sudo:session): session closed for user enterprisedb
Nov 18 14:38:22 node01 sudo[141147]:      efm : PWD=/ ; USER=root ; COMMAND=/usr/edb/efm-4.10/bin/efm_root_functions dbservicestatus efm
Nov 18 14:38:22 node01 sudo[141165]:      efm : PWD=/ ; USER=enterprisedb ; COMMAND=/usr/edb/efm-4.10/bin/efm_db_functions validatedatadir /etc/edb/efm-4.10/efm.properties
Nov 18 14:38:23 node01 sudo[141176]:      efm : PWD=/ ; USER=enterprisedb ; COMMAND=/usr/edb/efm-4.10/bin/efm_db_functions validatedbconf /etc/edb/efm-4.10/efm.properties
Nov 18 14:38:23 node01 sudo[141187]:      efm : PWD=/ ; USER=enterprisedb ; COMMAND=/usr/edb/efm-4.10/bin/efm_db_functions readpgversion /etc/edb/efm-4.10/efm.properties
Nov 18 14:38:28 node01 systemd[1]: edb-efm-4.10.service: Control process exited, code=exited, status=1/FAILURE
Nov 18 14:38:28 node01 systemd[1]: edb-efm-4.10.service: Failed with result 'exit-code'.
Nov 18 14:38:28 node01 systemd[1]: Failed to start EnterpriseDB Failover Manager 4.10.
Nov 18 14:38:28 node01 systemd[1]: edb-efm-4.10.service: Consumed 8.642s CPU time.

```

```bash
[root@node01]# cat /var/log/efm-4.10/efm.log
...
An unexpected error has occurred on this node. Please
check the agent log for more information. Error:
java.lang.IllegalStateException: channel is not connected
2024-11-18 14:38:27 com.enterprisedb.efm.nodes.EfmNode signalLeavingCluster ERROR: There was an unexpected problem sending shutdown signal: {}
java.lang.IllegalStateException: channel is not connected
        at org.jgroups.blocks.MessageDispatcher$ProtocolAdapter.down(MessageDispatcher.java:517)
        at org.jgroups.blocks.RequestCorrelator.sendAnycastRequest(RequestCorrelator.java:312)
        at org.jgroups.blocks.RequestCorrelator.sendMulticastRequest(RequestCorrelator.java:134)
        at org.jgroups.blocks.GroupRequest.sendRequest(GroupRequest.java:303)
        at org.jgroups.blocks.GroupRequest.sendRequest(GroupRequest.java:76)
        at org.jgroups.blocks.Request.execute(Request.java:48)
        at org.jgroups.blocks.MessageDispatcher.cast(MessageDispatcher.java:287)
        at org.jgroups.blocks.MessageDispatcher.castMessage(MessageDispatcher.java:231)
        at com.enterprisedb.efm.utils.ClusterUtils.sendToOtherNodes(ClusterUtils.java:417)
        at com.enterprisedb.efm.nodes.EfmNode.signalLeavingCluster(EfmNode.java:2420)
        at com.enterprisedb.efm.nodes.EfmNode.commonShutdown(EfmNode.java:1179)
        at com.enterprisedb.efm.nodes.EfmAgent.shutdown(EfmAgent.java:858)
        at com.enterprisedb.efm.nodes.EfmAgent.run(EfmAgent.java:229)
        at com.enterprisedb.efm.main.ServiceCommand.main(ServiceCommand.java:116)
2024-11-18 14:38:27 com.enterprisedb.efm.utils.MissingPrimaryNotifier stopNotifying INFO: 'Missing primary' notification service shutdown called.
2024-11-18 14:38:27 com.enterprisedb.efm.utils.VipMonitor stopMonitoring INFO: Stopping vip monitor
2024-11-18 14:38:27 com.enterprisedb.efm.nodes.EfmNode commonShutdown INFO: This node coordinator during cluster shutdown: false
2024-11-18 14:38:27 com.enterprisedb.efm.admin.AdminServer shutdown INFO: Stopping AdminServer...
2024-11-18 14:38:27 com.enterprisedb.efm.utils.Notifications lambda$sendMail$0 DEBUG: About to call Transport.send(). Subject: [SEVERE] EFM An unexpected error has occurred for cluster efm.
```

```bash
[root@node01]# cat /var/log/efm-4.10/efm.log
...
2024-11-18 14:38:26 -------------------------------------------------------------------
2024-11-18 14:38:26 GMS: address=node01-44951(192.168.11.26), cluster=efm, physical address=192.168.11.26:7800
2024-11-18 14:38:26 -------------------------------------------------------------------
2024-11-18 14:38:27 There is already a primary node in this cluster: node02-48361(192.168.11.27). Starting shutdown process.

```

```bash
[root@node01]# rm -rf /usr/edb/as16/data/
[root@node01]# pg_basebackup -h 192.168.11.27 -U repl -p 5444 -D /usr/edb/as16/data -Fp -Xs -P -R
84875/84875 kB (100%), 1/1 tablespace
[root@node01]# chown enterprisedb:enterprisedb -R /usr/edb/as16/data
[root@node01]# chmod 0700 /usr/edb/as16/data
[root@node01]# systemctl start edb-as-16.service && systemctl start edb-efm-4.10.service
```

```bash
[root@node03]# /usr/edb/efm-4.10/bin/efm cluster-status efm
Cluster Status: efm

        Agent Type  Address              DB       VIP
        ----------------------------------------------------------------
        Standby     192.168.11.26        UP       192.168.11.25*
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
        Primary     192.168.11.27                           0/D000060
        Standby     192.168.11.26        0/D000060          0/D000060

        Standby database(s) in sync with primary. It is safe to promote.
```