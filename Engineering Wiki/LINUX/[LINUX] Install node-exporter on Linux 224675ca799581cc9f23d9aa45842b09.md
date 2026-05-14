# [LINUX] Install node-exporter on Linux

Owner: Nam Tran
Last edited time: March 4, 2026 7:30 PM

```bash
[root@linux ~]# cd /tmp/ && curl -O -L https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz && tar -xzf node_exporter-1.9.1.linux-amd64.tar.gz

[root@linux tmp]# mv node_exporter-1.9.1.linux-amd64/node_exporter /usr/local/bin/

[root@linux tmp]# useradd --no-create-home --shell /bin/false prometheus && chown prometheus:prometheus /usr/local/bin/node_exporter

[root@linux tmp]# rm -rf node_exporter-*
```

```bash
[root@linux tmp]# vi /etc/systemd/system/node-exporter.service

[Unit]
Description=Node Exporter
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target

```

```bash
[root@linux tmp]# sed -i 's/SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config && setenforce 0

[root@linux tmp]# firewall-cmd --permanent --add-port=9100/tcp && firewall-cmd --reload

[root@linux tmp]# systemctl daemon-reload && systemctl enable node-exporter && systemctl start node-exporter && systemctl status node-exporter

[root@linux tmp]# curl localhost:9100/metrics
```