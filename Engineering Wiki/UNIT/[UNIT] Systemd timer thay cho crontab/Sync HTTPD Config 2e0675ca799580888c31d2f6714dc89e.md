# Sync HTTPD Config

Owner: Nam Tran
Last edited time: January 6, 2026 5:46 PM

`/etc/systemd/system/synchttpd.service`

```bash
[Unit]
Description=Sync HTTPD Config

[Service]
Type=oneshot
ExecStart=/usr/bin/flock -n /var/run/sync_httpd.lock /bin/bash /opt/scripts/sync.sh

User=root
Group=root
Nice=10
```

`/etc/systemd/system/synchttpd.timer`

```bash
[Unit]
Description=Run Sync HTTPD Config script at every hours

[Timer]
OnCalendar=hourly
Persistent=true
AccuracySec=1min
RandomizedDelaySec=5m
Unit=synchttpd.service

[Install]
WantedBy=timers.target
```

`systemctl daemon-reexec`

`systemctl daemon-reload`

`systemctl enable --now synchttpd.timer`