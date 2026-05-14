# [XCP] Install XEN Guest Tools on RHEL

Owner: Nam Tran
Last edited time: August 26, 2025 4:48 PM

```bash
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-[version].noarch.rpm -y

dnf install xe-guest-utilities-latest -y

systemctl enable --now xe-linux-distribution
```