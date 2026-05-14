# [PG] Install HA PostgreSQL cluster in linux using repmgr and keepalived on RHEL

Owner: Nam Tran
Last edited time: March 23, 2026 2:19 PM

1. Install `postgresql17-server` , `repmgr_17` , `keepalived` on both node:
    - Quick install with Internet on both Postgresql node
        
        ```bash
        # Install the repository RPM:
        sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
        
        # Disable thebuilt-in PostgreSQL module:
        sudo dnf -qy module disable postgresql
        
        # Install PostgreSQL:
        sudo dnf install -y postgresql17-server repmgr_17 keepalived
        ```
        
    - Manual install offline
        - Get package from another RHEL 8.10 machine has internet connection
            
            ```bash
            dnf install dnf-utils
            mkdir package_offline
            repotrack --destdir ./package_offline postgresql17-server repmgr_17 keepalived
            tar cf package_offline.tar package_offline/
            ```
            
        - Install on both Postgresql node
            
            ```bash
            tar xfv package_offline.tar
            cd package_offline
            dnf install *.rpm --disablerepo=* --cacheonly
            ```
            
        - Initialize the Database
            
            ```bash
            /usr/pgsql-17/bin/initdb -D /var/lib/pgsql/17/data/
            ```
            
2. Disable SELinux on both node
    
    ```bash
    sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config
    setenforce 0
    ```
    
3. Config and check firewall on both node
    
    ```bash
    firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 0 -o lo -j ACCEPT
    firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 0 -m state --state ESTABLISHED,RELATED -j ACCEPT
    firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 1 -p tcp --dport 22 -j ACCEPT
    firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 1 -p tcp --dport 5432 -j ACCEPT
    firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 1 -p vrrp -j ACCEPT
    firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 1 -p udp --dport 53 -j ACCEPT
    firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 1 -p tcp --dport 53 -j ACCEPT
    firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 2 -j DROP
    firewall-cmd --permanent --zone=public --remove-service=cockpit
    firewall-cmd --permanent --zone=public --remove-service=dhcpv6-client
    firewall-cmd --permanent --zone=public --add-service=ssh
    firewall-cmd --permanent --zone=public --add-port=5432/tcp
    firewall-cmd --permanent --zone=public --add-protocol=vrrp
    firewall-cmd --reload
    firewall-cmd --direct --get-all-rules
    firewall-cmd --list-protocols
    firewall-cmd --list-all
    ```
    
4. Check status and enable `postgresql-17` service on both node
    
    ```bash
    systemctl status postgresql-17.service
    systemctl enable --now postgresql-17.service
    systemctl start postgresql-17.service
    ```
    
5. Check folder `/var/log/repmgr/` on both node
    
    ```bash
    ls -lhd /var/log/repmgr/
    	drwx------. 2 postgres postgres 6 Jan 21 22:17 /var/log/repmgr/
    
    chmod -R 755 /var/log/repmgr/
    ```
    
6. Create folder `/var/lib/postgresql/16/backup/archivelog` on two node
    
    ```bash
    sudo -u postgres mkdir -p /var/lib/pgsql/17/backups/archivelog
    ```
    
7. Set password `postgres` user on both node
    
    ```bash
    passwd postgres
    ```
    
8. Set password database user `postgres` on both node
    
    ```bash
    sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'Unit@032010';"
    ```
    
9. Give permission for `postgres` user on both node. Edit the `*/etc/sudoers.d/postgres*` file:
    
    ```bash
    postgres ALL=(ALL) NOPASSWD: /bin/systemctl start postgresql-17.service
    postgres ALL=(ALL) NOPASSWD: /bin/systemctl stop postgresql-17.service
    postgres ALL=(ALL) NOPASSWD: /bin/systemctl restart postgresql-17.service
    postgres ALL=(ALL) NOPASSWD: /bin/systemctl reload postgresql-17.service
    ```
    
10. Create folder `conf.d`  on both node
    
    ```bash
    sudo -u postgres mkdir -p /var/lib/pgsql/17/data/conf.d
    ```
    
    and modify `/var/lib/pgsql/17/data/postgresql.conf` 
    
    ```bash
    include_dir = 'conf.d'
    ```
    
11. Create file `/var/lib/pgsql/17/data/conf.d/custom.conf` on both node
    
    ```bash
    sudo -u postgres vi /var/lib/pgsql/17/data/conf.d/custom.conf
    
    listen_addresses = '*'
    wal_level = replica
    wal_log_hints = on
    archive_mode = on
    archive_command = 'test ! -f /var/lib/pgsql/17/backup/archivelog/%f && cp %p /var/lib/pgsql/17/backup/archivelog/%f'
    restore_command = 'cp /var/lib/pgsql/17/backup/archivelog/%f %p'
    archive_cleanup_command = 'pg_archivecleanup /var/lib/pgsql/17/backup/archivelog %r'
    max_wal_senders = 10
    #wal_keep_segments = 10
    wal_keep_size = '1024MB'
    shared_preload_libraries= 'repmgr'
    ```
    
12. Edit file `/var/lib/pgsql/17/data/pg_hba.conf` on both node
    
    ```bash
    sudo -u postgres vi /var/lib/pgsql/17/data/pg_hba.conf
    
    host    repmgr          repmgr          192.168.13.12/32         trust
    host    repmgr          repmgr          192.168.13.13/32        trust
    host    replication     repmgr          192.168.13.12/32         trust
    host    replication     repmgr          192.168.13.13/32        trust
    host    all             postgres        0.0.0.0/0               scram-sha-256
    ```
    
13. Restart service `postgres` on both node
    
    ```bash
    systemctl restart postgresql-17.service
    ```
    
14. On both node run these commands:
    
    ```
    sudo -u postgres createuser --superuser repmgr
    sudo -u postgres createdb --owner=repmgr repmgr
    sudo -u postgres psql -c "ALTER USER repmgr WITH REPLICATION;"
    sudo -u postgres psql -c "ALTER USER repmgr SET search_path TO repmgr, public;"
    sudo -u postgres psql -d repmgr -U postgres -c "CREATE EXTENSION repmgr;"
    sudo -u postgres psql -d repmgr -U postgres -c "\dx"
    ```
    
15. Setup SSH Passwordless on both node
    
    ```bash
    # Node01
    su - postgres
    ssh-keygen -t rsa -b 4096
    ssh-copy-id postgres@192.168.13.13
    
    # Node02
    su - postgres
    ssh-keygen -t rsa -b 4096
    ssh-copy-id postgres@192.168.13.12
    ```
    
16. Edit the `/etc/repmgr/17/repmgr.conf` file on both nodes
    
    ```bash
    # Node01 Custom config
    node_id=1
    node_name='pg-rhel-01'
    conninfo='host=192.168.13.12 user=repmgr dbname=repmgr connect_timeout=2'
    use_replication_slots=true
    data_directory='/var/lib/pgsql/17/data'
    log_file='/var/log/repmgr/repmgrd.log'
    log_status_interval=20
    ssh_options='-q -o ConnectTimeout=10'
    
    failover='automatic'
    promote_command='/var/lib/pgsql/17/scripts/repmgr_promote.sh'
    follow_command='/usr/pgsql-17/bin/repmgr standby follow --log-to-file --upstream-node-id=%n'
    monitor_interval_secs=2
    connection_check_type='ping'
    reconnect_attempts=5
    reconnect_interval=4
    standby_disconnect_on_failover=true
    
    service_start_command = 'sudo /bin/systemctl start postgresql-17.service'
    service_stop_command = 'sudo /bin/systemctl stop postgresql-17.service'
    service_restart_command = 'sudo /bin/systemctl restart postgresql-17.service'
    service_reload_command = 'sudo /bin/systemctl reload postgresql-17.service'
    
    # Node02 Custom config
    node_id=2
    node_name='pg-rhel-02'
    conninfo='host=192.168.13.13 user=repmgr dbname=repmgr connect_timeout=2'
    use_replication_slots=true
    data_directory='/var/lib/pgsql/17/data'
    log_file='/var/log/repmgr/repmgrd.log'
    log_status_interval=20
    ssh_options='-q -o ConnectTimeout=10'
    
    failover='automatic'
    promote_command='/var/lib/pgsql/17/scripts/repmgr_promote.sh'
    follow_command='/usr/pgsql-17/bin/repmgr standby follow --log-to-file --upstream-node-id=%n'
    monitor_interval_secs=2
    connection_check_type='ping'
    reconnect_attempts=5
    reconnect_interval=4
    standby_disconnect_on_failover=true
    
    service_start_command = 'sudo /bin/systemctl start postgresql-17.service'
    service_stop_command = 'sudo /bin/systemctl stop postgresql-17.service'
    service_restart_command = 'sudo /bin/systemctl restart postgresql-17.service'
    service_reload_command = 'sudo /bin/systemctl reload postgresql-17.service'
    ```
    
17. Now you can register your primary node
    
    ```bash
    ## Register primary
    sudo -u postgres /usr/pgsql-17/bin/repmgr primary register
    
    ## Start service repmgr-17.service
    systemctl start repmgr-17.service
    
    ## Active Repmgr Daemon (Auto-Failover)
    sudo -u postgres /usr/pgsql-17/bin/repmgrd  --daemonize --verbose
    ```
    
18. On the standby node, you have to delete the contents of the postgresql data folder, so you can replicate the primary database.
    
    ```bash
    systemctl stop postgresql-17.service
    cd /var/lib/pgsql/17/data/
    rm -rf ./*
    ## Run test before clone
    sudo -u postgres /usr/pgsql-17/bin/repmgr -F -h 192.168.13.12 -U repmgr -d repmgr standby clone --dry-run
    	NOTICE: destination directory "/var/lib/postgresql/16/main" provided
    	INFO: connecting to source node
    	DETAIL: connection string is: host=192.168.13.12 user=repmgr dbname=repmgr
    	DETAIL: current installation size is 29 MB
    	INFO: "repmgr" extension is installed in database "repmgr"
    	INFO: parameter "max_replication_slots" set to 10
    	INFO: parameter "max_wal_senders" set to 10
    	NOTICE: checking for available walsenders on the source node (2 required)
    	INFO: sufficient walsenders available on the source node
    	DETAIL: 2 required, 10 available
    	NOTICE: checking replication connections can be made to the source server (2 required)
    	INFO: required number of replication connections could be made to the source server
    	DETAIL: 2 replication connections required
    	INFO: replication slots will be created by user "repmgr"
    	NOTICE: standby will attach to upstream node 1
    	HINT: consider using the -c/--fast-checkpoint option
    	INFO: would execute:
    	  pg_basebackup -l "repmgr base backup"  -D /var/lib/postgresql/16/main -h 192.168.13.12 -p 5432 -U repmgr -X stream -S repmgr_slot_2 
    	INFO: all prerequisites for "standby clone" are met
    
    ## If dry test success, start clone
    sudo -u postgres /usr/pgsql-17/bin/repmgr -F -h 192.168.13.12 -U repmgr -d repmgr standby clone
    
    ## Start postgres service
    systemctl start postgresql-17.service
    
    ## Register standby
    sudo -u postgres /usr/pgsql-17/bin/repmgr standby register -F
    
    ## Start service repmgr-17.service
    systemctl start repmgr-17.service
    
    ## Active Repmgr Daemon (Auto-Failover)
    sudo -u postgres /usr/pgsql-17/bin/repmgrd  --daemonize --verbose
    
    ## Check cluster
    sudo -u postgres /usr/pgsql-17/bin/repmgr cluster show
    ```
    
19. Edit `/etc/keepalived/keepalived.conf` on both node
    
    ```bash
    # Primary node
    global_defs {
        router_id pg-rhel-01
        script_user postgres
        enable_script_security
        #max_auto_priority 99
    }
    
    vrrp_script pg_check {
          script "/var/lib/pgsql/17/scripts/check_pg_status.sh"
          interval 2
          #timeout 5
          weight 50
          fall 2
          rise 2
    }
    
    vrrp_instance VI_1 {
        state BACKUP
        interface eth0
        virtual_router_id 234
        advert_int 1
        priority 100
        authentication {
            auth_type PASS
            auth_pass Unit@203
        }
        virtual_ipaddress {
            192.168.13.14/20 dev eth0
        }
        track_script {
            pg_check
        }
    }
    
    # Standby node
    global_defs {
        router_id pg-rhel-02
        script_user postgres
        enable_script_security
        #max_auto_priority 99
    }
    
    vrrp_script pg_check {
          script "/var/lib/pgsql/17/scripts/check_pg_status.sh"
          interval 2
          #timeout 5
          weight 50
          fall 2
          rise 2
    }
    
    vrrp_instance VI_1 {
        state BACKUP
        interface eth0
        virtual_router_id 234
        advert_int 1
        priority 100
        authentication {
            auth_type PASS
            auth_pass Unit@203
        }
        virtual_ipaddress {
            192.168.13.14/20 dev eth0
        }
        track_script {
            pg_check
        }
    }
    ```
    
20. Create a scripts directory and scripts help us manage the whole failover process
    
    ```bash
    sudo -u postgres mkdir -p /var/lib/pgsql/17/scripts
    ```
    
21. Create the file `/var/lib/pgsql/17/scripts/setEnv.sh` on both node
    
    ```bash
    
    sudo -u postgres vi /var/lib/pgsql/17/scripts/setEnv.sh
    
    # Primary node
    export user='postgres'
    export node='192.168.13.12'
    export othernode='192.168.13.13'
    
    # Standby node
    export user='postgres'
    export node='192.168.13.13'
    export othernode='192.168.13.12'
    ```
    
22. Create the `/var/lib/pgsql/17/scripts/repmgr_promote.sh` on both node
    
    ```bash
    sudo -u postgres vi /var/lib/pgsql/17/scripts/repmgr_promote.sh
    
    #!/bin/bash
    source /var/lib/pgsql/17/scripts/setEnv.sh
    
    nodereachable=false
    logfile='/var/log/repmgr/repmgrd.log'
    
    if /bin/nc -w 1 -z $othernode 22 2>/dev/null; then
        echo "["`date "+%Y-%m-%d %H:%M:%S"`"] [INFO] ############### the other node ($othernode) is reachable! " >> ${logfile}
        nodereachable=true
    else
        echo "["`date "+%Y-%m-%d %H:%M:%S"`"] [WARNING] ############### the other node ($othernode) is not reachable on port 22! " >> ${logfile}
        nodereachable=false
    fi
    
    # need to promote local node!
    /usr/pgsql-17/bin/repmgr standby promote --log-to-file
    sleep 20
    #TODO need to denote other node!
    if $nodereachable
    then
        echo "["`date "+%Y-%m-%d %H:%M:%S"`"] [INFO] ############## rejoin other node as standby" >> ${logfile}
        /bin/ssh -tt ${user}@${othernode} "/usr/pgsql-17/bin/repmgr node rejoin -d 'host=${node} user=repmgr dbname=repmgr connect_timeout=2' --force-rewind='/usr/pgsql-17/bin/pg_rewind'"
        sleep 20
        exit 0
    fi
    
    ```
    
23. Create the `/var/lib/pgsql/17/scripts/check_pg_status.sh` on both node
    
    ```bash
    sudo -u postgres vi /var/lib/pgsql/17/scripts/check_pg_status.sh
    
    #!/bin/bash
    # ------------------------------------------------------------------
    # check_pg_status.sh
    # Check if the local PostgreSQL node is the cluster PRIMARY
    # ------------------------------------------------------------------
    # Kiểm tra node có đang ở trạng thái Standby (Read-only) hay không
    # Nếu là Master: pg_is_in_recovery trả về 'f' (false)
    # Nếu là Standby: pg_is_in_recovery trả về 't' (true)
    
    IS_RECOVERY=$(psql -t -A -p 5432 -U postgres -c "SELECT pg_is_in_recovery();" 2>/dev/null)
    
    if [ "$IS_RECOVERY" == "f" ]; then
        # Đây là Primary Master
        exit 0
    else
        # Đây là Standby hoặc DB đang sập
        exit 1
    fi
    ```
    
24. Set permission for all scripts file
    
    ```bash
    chmod -R +x /var/lib/pgsql/17/scripts/
    ```
    
25. Check state of node
    
    ```bash
    sudo -u postgres /var/lib/pgsql/17/scripts/check_pg_status.sh
    echo $?
    ```
    
    If result is `0` a node is Primary, if result is `1` a node is Standby
    
26. Start keepalived service on both node
    
    ```bash
    systemctl start keepalived
    ```