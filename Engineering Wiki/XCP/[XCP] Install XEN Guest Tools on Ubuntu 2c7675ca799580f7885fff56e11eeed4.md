# [XCP] Install XEN Guest Tools on Ubuntu

Owner: Nam Tran
Last edited time: December 12, 2025 2:19 PM

```bash
apt install xe-guest-utilities

systemctl daemon-reload

systemctl enable xe-daemon.service

systemctl status xe-daemon.service

```