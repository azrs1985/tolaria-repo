# [PG] Install HA PostgreSQL cluster in linux using repmgr and keepalived on Ubuntu

Owner: Nam Tran
Last edited time: January 22, 2026 2:00 PM

1. Install `postgresql-16` , `postgresql-16-repmgr` , `keepalived` on both node:
    - Quick install with Internet on both Postgresql node
        
        ```bash
        apt install postgresql-16 postgresql-16-repmgr keepalived
        ```
        
    - Manual install offline
        - Get package from another Ubuntu machine has internet connection
            
            ```bash
            sudo chown -R _apt:root /var/cache/apt/archives/
            cd /var/cache/apt/archives/
            sudo apt-get install --download-only postgresql-16 postgresql-16-repmgr keepalived -y
            sudo apt-get download perl perl-base perl-modules-5.38 libperl5.38t64
            mkdir ~/offline_postgres && mv *.deb ~/offline_postgres
            tar -cf ~/offline_postgres.tar ~/offline_postgres/
            ```
            
        - Install on both Postgresql node
            
            ```bash
            tar -xf ~/offline_postgres.tar
            cd ~/offline_postgres
            dpkg -i *.deb
            ```
            
2. Config and check firewall on both node
    
    ```bash
    ufw allow 5432/tcp
    ufw allow out 5432/tcp
    ufw allow 22/tcp
    ufw allow out 22/tcp
    ufw allow in from 192.168.13.24 proto vrrp
    ufw allow in from 224.0.0.18 proto vrrp
    ufw allow out to 192.168.13.24 proto vrrp
    ufw allow out to 224.0.0.18 proto vrrp
    ```
    
3. Check status and enable `repmgr` service on both node
    
    ```bash
    systemctl status postgresql@16-main.service
    systemctl status repmgrd.service
    systemctl enable --now repmgrd.service
    ```
    
4. Create folder `/var/log/repmgr/` on both node
    
    ```bash
    mkdir /var/log/repmgr/
    chown -R postgres:postgres /var/log/repmgr/
    chmod -R 755 /var/log/repmgr/
    ```
    
5. Create folder `/var/lib/postgresql/16/backup/archivelog` on both node
    
    ```bash
    sudo -u postgres mkdir -p /var/lib/postgresql/16/backup/archivelog
    ```
    
6. Set password `postgres` user on both node
    
    ```bash
    passwd postgres
    ```
    
7. Give permission for `postgres` user on both node
    
    Edit the `*/etc/sudoers.d/postgres*` file:
    
    ```bash
    postgres ALL=(ALL) NOPASSWD: /bin/systemctl start postgresql@16-main.service
    postgres ALL=(ALL) NOPASSWD: /bin/systemctl stop postgresql@16-main.service
    postgres ALL=(ALL) NOPASSWD: /bin/systemctl restart postgresql@16-main.service
    postgres ALL=(ALL) NOPASSWD: /bin/systemctl reload postgresql@16-main.service
    ```
    
8. Create file `/etc/postgresql/16/main/conf.d/custom.conf` on two node
    
    ```bash
    sudo -u postgres nano /etc/postgresql/16/main/conf.d/custom.conf
    
    listen_addresses = '*'
    wal_level = replica
    wal_log_hints = on
    archive_mode = on
    archive_command = 'test ! -f /var/lib/postgresql/16/backup/archivelog/%f && cp %p /var/lib/postgresql/16/backup/archivelog/%f'
    restore_command = 'cp /var/lib/postgresql/16/backup/archivelog/%f %p'
    archive_cleanup_command = 'pg_archivecleanup /var/lib/postgresql/16/backup/archivelog %r'
    max_wal_senders = 10
    #wal_keep_segments = 10
    wal_keep_size = '1024MB'
    shared_preload_libraries= 'repmgr'
    ```
    
9. Edit file `/etc/postgresql/16/main/pg_hba.conf` on two node
    
    ```bash
    sudo -u postgres nano /etc/postgresql/16/main/pg_hba.conf
    
    host    repmgr          repmgr          192.168.13.2/32         trust
    host    repmgr          repmgr          192.168.13.13/32        trust
    host    replication     repmgr          192.168.13.2/32         trust
    host    replication     repmgr          192.168.13.13/32        trust
    host    all             postgres        0.0.0.0/0               scram-sha-256
    ```
    
10. On your **primary** node run these commands:
    
    ```
    sudo -u postgres createuser --superuser repmgr
    sudo -u postgres createdb --owner=repmgr repmgr
    sudo -u postgres psql -c "ALTER USER repmgr WITH REPLICATION;"
    sudo -u postgres psql -c "ALTER USER repmgr SET search_path TO repmgr, public;"
    ```
    
11. Setup SSH Passwordless for two node
    
    ```bash
    # Node01
    su - postgres
    ssh-keygen -t rsa -b 4096
    ssh-copy-id postgres@pg-ha-02
    
    # Node02
    su - postgres
    ssh-keygen -t rsa -b 4096
    ssh-copy-id postgres@pg-ha-01
    ```
    
12. Modify the `/etc/default/repmgrd` file on two node

```bash
REPMGRD_ENABLED=yes
REPMGRD_CONF="/etc/repmgr.conf"
```

1. Edit the `/etc/repmgr.conf` file on both nodes

```bash
# Node01
node_id=1
node_name='pg-ha-01'
conninfo='host=192.168.13.2 user=repmgr dbname=repmgr connect_timeout=2'
use_replication_slots=true
data_directory='/var/lib/postgresql/16/main'
log_file='/var/log/repmgr/repmgrd.log'
log_status_interval=20
ssh_options='-q -o ConnectTimeout=10'

failover='automatic'
promote_command='/var/lib/postgresql/scripts/repmgr_promote.sh'
follow_command='/usr/bin/repmgr standby follow --log-to-file --upstream-node-id=%n'
monitor_interval_secs=2
connection_check_type='ping'
reconnect_attempts=5
reconnect_interval=4
standby_disconnect_on_failover=true

service_start_command = 'sudo /bin/systemctl start postgresql@16-main.service'
service_stop_command = 'sudo /bin/systemctl stop postgresql@16-main.service'
service_restart_command = 'sudo /bin/systemctl restart postgresql@16-main.service'
service_reload_command = 'sudo /bin/systemctl reload postgresql@16-main.service'

# Node02
node_id=2
node_name='pg-ha-02'
conninfo='host=192.168.13.13 user=repmgr dbname=repmgr connect_timeout=2'
use_replication_slots=true
data_directory='/var/lib/postgresql/16/main'
log_file='/var/log/repmgr/repmgrd.log'
log_status_interval=20
ssh_options='-q -o ConnectTimeout=10'

failover='automatic'
promote_command='/var/lib/postgresql/scripts/repmgr_promote.sh'
follow_command='/usr/bin/repmgr standby follow --log-to-file --upstream-node-id=%n'
monitor_interval_secs=2
connection_check_type='ping'
reconnect_attempts=5
reconnect_interval=4
standby_disconnect_on_failover=true

service_start_command = 'sudo /bin/systemctl start postgresql@16-main.service'
service_stop_command = 'sudo /bin/systemctl stop postgresql@16-main.service'
service_restart_command = 'sudo /bin/systemctl restart postgresql@16-main.service'
service_reload_command = 'sudo /bin/systemctl reload postgresql@16-main.service'
```

1. Restart service `postgres` on primary node

```bash
systemctl restart postgresql@16-main.service
```

1. Now you can register your primary node

```bash
sudo -u postgres repmgr primary register

## Active Repmgr Daemon (Auto-Failover)
sudo -u postgres repmgrd  --daemonize --verbose
```

1. On the standby node, you have to delete the contents of the postgresql data folder, so you can replicate the primary database.

```bash
systemctl stop postgresql@16-main.service
cd /var/lib/postgresql/16/main/
rm -rf ./*
## Run test before clone
sudo -u postgres repmgr -F -h 192.168.13.2 -U repmgr -d repmgr standby clone --dry-run
	NOTICE: destination directory "/var/lib/postgresql/16/main" provided
	INFO: connecting to source node
	DETAIL: connection string is: host=192.168.13.2 user=repmgr dbname=repmgr
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
	  pg_basebackup -l "repmgr base backup"  -D /var/lib/postgresql/16/main -h 192.168.13.2 -p 5432 -U repmgr -X stream -S repmgr_slot_2 
	INFO: all prerequisites for "standby clone" are met

## If dry test success, start clone
sudo -u postgres repmgr -F -h 192.168.13.2 -U repmgr -d repmgr standby clone

## Start postgres service
systemctl start postgresql@16-main.service

## Register standby
sudo -u postgres repmgr standby register -F

## Active Repmgr Daemon (Auto-Failover)
sudo -u postgres repmgrd  --daemonize --verbose
```

1. Edit `*/etc/keepalived/keepalived.conf*` on both node

```bash
# Primary node
global_defs {
    router_id pg_node1
    script_user postgres
    enable_script_security
    #max_auto_priority 99
}

vrrp_script pg_check {
      script "/var/lib/postgresql/scripts/check_pg_status.sh"
      interval 2
      #timeout 5
      weight 50
      fall 2
      rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface enX0
    virtual_router_id 132
    advert_int 1
    priority 100
    authentication {
        auth_type PASS
        auth_pass Unit@203
    }
    virtual_ipaddress {
        192.168.11.132/20 dev enX0
    }
    track_script {
        pg_check
    }
}

# Standby node
global_defs {
    router_id pg_node2
    script_user postgres
    enable_script_security
    #max_auto_priority 99
}

vrrp_script pg_check {
      script "/var/lib/postgresql/scripts/check_pg_status.sh"
      interval 2
      #timeout 5
      weight 50
      fall 2
      rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface enX0
    virtual_router_id 132
    advert_int 1
    priority 100
    authentication {
        auth_type PASS
        auth_pass Unit@203
    }
    virtual_ipaddress {
        192.168.11.132/20 dev enX0
    }
    track_script {
        pg_check
    }
}
```

1. Create a scripts directory and scripts help us manage the whole failover process

```bash
sudo -u postgres mkdir -p /var/lib/postgresql/scripts
```

Create the file */var/lib/postgresql/scripts/setEnv.sh* on both node

```bash

sudo -u postgres nano /var/lib/postgresql/scripts/setEnv.sh

# Primary node
export user='postgres'
export node='192.168.13.2'
export othernode='192.168.13.13'

# Standby node
export user='postgres'
export node='192.168.13.13'
export othernode='192.168.13.2'
```

Create the */var/lib/postgresql/scripts/repmgr_promote.sh* on both node

```bash
sudo -u postgres nano /var/lib/postgresql/scripts/repmgr_promote.sh
#!/bin/bash

source /var/lib/postgresql/scripts/setEnv.sh

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
/usr/bin/repmgr standby promote --log-to-file
sleep 20
#TODO need to denote other node!
if $nodereachable
then
    echo "["`date "+%Y-%m-%d %H:%M:%S"`"] [INFO] ############## rejoin other node as standby" >> ${logfile}
    /bin/ssh -tt ${user}@${othernode} "/usr/bin/repmgr node rejoin -d 'host=${node} user=repmgr dbname=repmgr connect_timeout=2' --force-rewind='/usr/lib/postgresql/16/bin/pg_rewind'"
    sleep 20
    exit 0
fi

```

Create the */var/lib/postgresql/scripts/check_pg_status.sh* on both node

```bash
sudo -u postgres nano /var/lib/postgresql/scripts/check_pg_status.sh
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

Set permission for all scripts file

```bash
chmod -R +x /var/lib/postgresql/scripts/
```

Start keepalived service on both node

```bash
systemctl start keepalived
```