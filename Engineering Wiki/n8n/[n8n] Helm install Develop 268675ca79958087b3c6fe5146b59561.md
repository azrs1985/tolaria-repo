# [n8n] Helm install Develop

Owner: Nam Tran
Last edited time: December 30, 2025 2:24 PM

```yaml
# -- This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/
# image:
#   tag: "1.88.0"

# -- This section builds out the service account more information can be found here: https://kubernetes.io/docs/concepts/security/service-accounts/
serviceAccount:
  name: "n8n"

# -- n8n log configurations
log:
  level: info

# -- n8n database configurations
db:
  type: postgresdb
  logging:
    # -- Whether database logging is enabled.
    enabled: true
    # -- Database logging level. Requires `maxQueryExecutionTime` to be higher than `0`. Valid values 'query' | 'error' | 'schema' | 'warn' | 'info' | 'log' | 'all'
    options: all
    # -- Maximum query execution time in milliseconds. If a query takes longer than this, it will be logged. Set to `0` to disable logging.
    maxQueryExecutionTime: 1000

versionNotifications:
  # -- Whether to request notifications about new n8n versions
  enabled: true
  # -- Endpoint to retrieve n8n version information from
  endpoint: "https://api.n8n.io/api/versions/"
  # -- URL for versions panel to page instructing user on how to update n8n instance
  infoUrl: "https://docs.n8n.io/hosting/installation/updating/"

main:
  # -- Whether to enable the PodDisruptionBudget for the main pod.
  pdb:
    # -- Whether to enable the PodDisruptionBudget
    enabled: false

  # -- This block is for setting up the resource management for the main pod more information can be found here: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
  resources:
    requests:
      cpu: 256m
      memory: 128Mi
    limits:
      cpu: 512m
      memory: 512Mi

  volumeMounts:
    - name: n8n-data
      mountPath: /home/node/.n8n
      subPath: home
    # - name: n8n-data
    #   mountPath: /usr/local/lib/node_modules/n8n
    #   subPath: lib

  volumes:
    - name: n8n-data
      persistentVolumeClaim:
        claimName: n8n-data-pvc

  # -- Extra environment variables
  extraEnvVars:
    VUE_APP_URL_BASE_API: "https://n8n.bpmhub.io/"
    N8N_HOST: "n8n.bpmhub.io"
    N8N_PROTOCOL: "https"
    N8N_EDITOR_BASE_URL: "https://n8n.bpmhub.io/"
    WEBHOOK_URL: "https://n8n.bpmhub.io/"
    N8N_PROXY_HOPS: 1
		N8N_BLOCK_ENV_ACCESS_IN_NODE: "false"
    N8N_EMAIL_MODE: "smtp"
    N8N_SMTP_HOST: "mail.unit.vn"
    N8N_SMTP_USER: "sysadmin@unit.vn"
    N8N_SMTP_PASS: "bk9NtjE$C@nFECPi"
    N8N_SMTP_PORT: 465
    N8N_SMTP_SSL: "true"
    N8N_SMTP_SENDER: "sysadmin@unit.vn"
    N8N_GIT_NODE_DISABLE_BARE_REPOS: "true"
    
# -- Worker node configurations
worker:
  mode: queue
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 6
  resources:
    requests:
      cpu: 512m
      memory: 256Mi
    limits:
      cpu: 1000m
      memory: 512Mi
  extraEnvVars:
    N8N_URL: "https://n8n.bpmhub.io/"
    WEBHOOK_URL: "https://n8n.bpmhub.io/"
    N8N_PROXY_HOPS: 1
		N8N_BLOCK_ENV_ACCESS_IN_NODE: "false"
    N8N_EMAIL_MODE: "smtp"
    N8N_SMTP_HOST: "mail.unit.vn"
    N8N_SMTP_USER: "sysadmin@unit.vn"
    N8N_SMTP_PASS: "bk9NtjE$C@nFECPi"
    N8N_SMTP_PORT: 465
    N8N_SMTP_SSL: "true"
    N8N_SMTP_SENDER: "sysadmin@unit.vn"
    N8N_GIT_NODE_DISABLE_BARE_REPOS: "true"
    
# webhook:
#   mode: queue
#   url: "https://n8n.bpmhub.io/"
#   resources:
#     requests:
#       cpu: 256m
#       memory: 128Mi
#     limits:
#       cpu: 512m
#       memory: 512Mi

# -- If you install n8n first time, you can keep this empty and it will be auto generated and never change again. If you already have a encryption key generated before, please use it here.
encryptionKey: "Unit@032010"

# -- The name of an existing secret with encryption key. The secret must contain a key with the name N8N_ENCRYPTION_KEY.
#existingEncryptionKeySecret: "Unit@032010"

# -- For instance, the Schedule node uses it to know at what time the workflow should start. Find you timezone from here: https://momentjs.com/timezone/
timezone: "Asia/Ho_Chi_Minh"

redis:
  # -- Enable redis
  enabled: true
  
#-- External PostgreSQL parameters
externalPostgresql:
  host: "edb-pg-dev.unit.local"
  username: "n8n"
  password: "Unit@032010"
  database: "n8ndb_test"
  port: 5444

```