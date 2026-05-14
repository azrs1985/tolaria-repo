# [XCP] xo-server config.toml

Owner: Nam Tran
Last edited time: July 2, 2025 4:58 PM

```bash
sudo nano /opt/xen-orchestra/packages/xo-server/config.toml
...
[http.listen.0]
port = 443
cert = '/etc/ssl/private/xen.unit.local.crt'
key = '/etc/ssl/private/xen.unit.local.key'

[http.listen.1]
port = 80
```