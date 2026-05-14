# [UNIT] Update Sonar

Owner: Nam Tran
Last edited time: December 11, 2025 11:10 AM

```bash
cd /opt/

wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-24.12.0.100206.zip
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.11.0.114957.zip

wget https://github.com/cnescatlab/sonar-cnes-report/releases/download/5.0.2/sonar-cnes-report-5.0.2.jar
wget https://github.com/dependency-check/dependency-check-sonar-plugin/releases/download/6.0.0/sonar-dependency-check-plugin-6.0.0.jar

unzip sonarqube-24.12.0.100206.zip -d /opt/

systemctl stop sonarqube.service

ln -s /opt/sonarqube-24.12.0.100206/ /opt/sonarqube
chown -R sonar:sonar /opt/sonarqube*
chmod -R 755 /opt/sonarqube*

nano /etc/systemd/system/sonarqube.service
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
LimitNOFILE=65536
LimitNPROC=4096
User=sonar
Group=sonar
Restart=on-failure

[Install]
WantedBy=multi-user.target

nano sonarqube/conf/sonar.properties
##Database details
sonar.jdbc.username=sonar
#sonar.jdbc.password=Unit@032010
sonar.jdbc.password=ToIeMvq6ULUBWX-8UQ7p
##sonar.jdbc.url=jdbc:postgresql://localhost/sonar_db
sonar.jdbc.url=jdbc:postgresql://edb-pg-prod.unit.local:5444/sonar_db

##How you will access SonarQube Web UI
sonar.web.host=0.0.0.0
sonar.web.port=9000

##Java options
#sonar.web.javaOpts=-Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError
#sonar.search.javaOpts=-Xmx512m -Xms512m -XX:MaxDirectMemorySize=256m -XX:+HeapDumpOnOutOfMemoryError

##Also uncomment the following Elasticsearch storage paths
sonar.path.data=data
sonar.path.temp=temp

# WEB SERVER
sonar.web.javaOpts=-Xmx2048m -Xms512m -XX:+HeapDumpOnOutOfMemoryError
sonar.web.javaAdditionalOpts=
#sonar.web.host=0.0.0.0
#sonar.web.port=9000
sonar.web.http.maxThreads=50
sonar.web.http.minThreads=5
sonar.web.http.acceptCount=25
#sonar.auth.jwtBase64Hs256Secret=
sonar.web.sessionTimeoutInMinutes=1440
#sonar.web.systemPasscode=

#--------------------------------------------------------------------------------------------------
# COMPUTE ENGINE
sonar.ce.javaOpts=-Xmx2048m -Xms512m -XX:+HeapDumpOnOutOfMemoryError
#sonar.ce.javaAdditionalOpts=

#--------------------------------------------------------------------------------------------------
# ELASTICSEARCH
sonar.search.javaOpts=-Xmx2048m -Xms2048m -XX:MaxDirectMemorySize=256m -XX:+HeapDumpOnOutOfMemoryError
#sonar.search.javaAdditionalOpts=
sonar.search.port=9001

```

```sql

SELECT n.nspname AS schema, c.relname AS object, c.relkind,
       pg_get_userbyid(c.relowner) AS owner
FROM pg_class c JOIN pg_namespace n ON n.oid=c.relnamespace
WHERE n.nspname='public'
ORDER BY c.relkind, c.relname;

SELECT n.nspname AS schema, p.proname AS routine,
       pg_get_function_identity_arguments(p.oid) AS args,
       pg_get_userbyid(p.proowner) AS owner
FROM pg_proc p JOIN pg_namespace n ON n.oid=p.pronamespace
WHERE n.nspname='public'
ORDER BY routine;

ALTER SCHEMA public OWNER TO pg_database_owner;

-- TABLES
DO $$
DECLARE r RECORD;
BEGIN
  FOR r IN
    SELECT c.oid, format('%I.%I', n.nspname, c.relname) AS fqname
    FROM pg_class c
    JOIN pg_namespace n ON n.oid = c.relnamespace
    WHERE n.nspname = 'public'
      AND c.relkind = 'r'  -- r = ordinary table
  LOOP
    EXECUTE format('ALTER TABLE %s OWNER TO %I', r.fqname, 'pg_database_owner');
  END LOOP;
END$$;

-- VIEWS
DO $$
DECLARE r RECORD;
BEGIN
  FOR r IN
    SELECT format('%I.%I', n.nspname, c.relname) AS fqname
    FROM pg_class c
    JOIN pg_namespace n ON n.oid = c.relnamespace
    WHERE n.nspname = 'public'
      AND c.relkind = 'v'  -- v = view
  LOOP
    EXECUTE format('ALTER VIEW %s OWNER TO %I', r.fqname, 'pg_database_owner');
  END LOOP;
END$$;

-- MATERIALIZED VIEWS
DO $$
DECLARE r RECORD;
BEGIN
  FOR r IN
    SELECT format('%I.%I', n.nspname, c.relname) AS fqname
    FROM pg_class c
    JOIN pg_namespace n ON n.oid = c.relnamespace
    WHERE n.nspname = 'public'
      AND c.relkind = 'm'  -- m = matview
  LOOP
    EXECUTE format('ALTER MATERIALIZED VIEW %s OWNER TO %I', r.fqname, 'pg_database_owner');
  END LOOP;
END$$;

-- SEQUENCES
DO $$
DECLARE r RECORD;
BEGIN
  FOR r IN
    SELECT format('%I.%I', n.nspname, c.relname) AS fqname
    FROM pg_class c
    JOIN pg_namespace n ON n.oid = c.relnamespace
    WHERE n.nspname = 'public'
      AND c.relkind = 'S'  -- S = sequence
  LOOP
    EXECUTE format('ALTER SEQUENCE %s OWNER TO %I', r.fqname, 'pg_database_owner');
  END LOOP;
END$$;

-- TYPE
DO $$
DECLARE r RECORD;
BEGIN
  FOR r IN
    SELECT format('%I.%I', n.nspname, t.typname) AS fqname
    FROM pg_type t
    JOIN pg_namespace n ON n.oid = t.typnamespace
    WHERE n.nspname = 'public'
      AND t.typtype IN ('c','d','e','r') -- composite, domain, enum, range
      AND t.typowner <> (SELECT oid FROM pg_roles WHERE rolname = 'pg_database_owner')
  LOOP
    EXECUTE format('ALTER TYPE %s OWNER TO %I', r.fqname, 'pg_database_owner');
  END LOOP;
END$$;

GRANT USAGE ON SCHEMA public TO sonar;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO sonar;
GRANT USAGE, SELECT, UPDATE ON ALL SEQUENCES IN SCHEMA public TO sonar;

-- tự động áp quyền cho đối tượng tạo mới:
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO sonar;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT USAGE, SELECT, UPDATE ON SEQUENCES TO sonar;

```