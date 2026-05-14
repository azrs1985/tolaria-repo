# [OCP] Install and upgrade n8n by heml chart

Owner: Nam Tran
Last edited time: September 9, 2025 10:28 AM

### Install

for Prod:

`helm upgrade --install n8n community-charts/n8n -n n8n -f prod-override-values.yml`

for Dev

`helm upgrade --install n8n-dev community-charts/n8n -f dev-override-values.yml -n n8n`

### Upgrade

for Prod:

`helm upgrade n8n community-charts/n8n -n n8n -f prod-override-values.yml`

for Dev

`helm upgrade n8n-dev community-charts/n8n -f dev-override-values.yml -n n8n`