# [UNIT] Elastic Search

Owner: Nam Tran
Last edited time: September 24, 2025 5:14 PM

Get allocation on cluster

```bash
curl --request GET \
  --url 'https://192.168.2.17:9200/_cat/allocation?v=' \
  --header 'Authorization: Basic ZWxhc3RpYzpVbml0QCkjQCkhKQ=='
```

Get indices on cluster

```bash
curl --request GET \
  --url 'https://192.168.2.15:9200/_cat/indices?v=&health=green&index=.ds-logs-app-container-k8s-vvt1-dev-2025.09.17-000025' \
  --header 'Authorization: Basic ZWxhc3RpYzpVbml0QCkjQCkhKQ=='
```

Delete indices on cluster

```bash
curl --request DELETE \
  --url 'https://192.168.2.17:9200/.ds-logs-app-container-k8s-vvt1-dev-2025.09.17-000025?=' \
  --header 'Authorization: Basic ZWxhc3RpYzpVbml0QCkjQCkhKQ=='
```