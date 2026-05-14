# [K8S] Increase proxy body size Ingress Nginx

Owner: Nam Tran
Last edited time: August 28, 2025 2:11 PM

```yaml
ingress:
  enabled: true
  className: "nginx"
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: 20m
  hosts:
    - host: ies-eibeform.bpmhub.io
      paths:
        - path: /admin-api-uat
          pathType: Prefix
          svcPort: 8080
```