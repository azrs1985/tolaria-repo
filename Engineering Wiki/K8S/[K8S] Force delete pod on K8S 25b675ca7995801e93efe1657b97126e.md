# [K8S] Force delete pod on K8S

Owner: Nam Tran
Last edited time: March 24, 2026 2:26 PM

```bash
kubectl delete pod <pod_name> -n <namespace> --grace-period=0 --force
```