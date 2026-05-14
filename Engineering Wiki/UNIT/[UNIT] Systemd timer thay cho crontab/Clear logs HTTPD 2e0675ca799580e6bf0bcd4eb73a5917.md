# Clear logs HTTPD

Owner: Nam Tran
Last edited time: January 6, 2026 5:46 PM

`/etc/systemd/system/clearlogs.service`

```bash
[Unit]
Description=Clear HTTPD logs exceeding size limit

[Service]
Type=oneshot
ExecStart=/usr/bin/flock -n /var/run/clear_httpd_logs.lock /bin/bash /opt/scripts/clear_large_logs.sh

User=root
Group=root
Nice=10

```

`/etc/systemd/system/clearlogs.timer`

```bash
[Unit]
Description=Run Clear HTTPD log script at every hours

[Timer]
OnCalendar=hourly
Persistent=true
AccuracySec=1min
RandomizedDelaySec=5m
Unit=clearlogs.service

[Install]
WantedBy=timers.target
```

`systemctl daemon-reexec`

`systemctl daemon-reload`

`systemctl enable --now clearlogs.timer`