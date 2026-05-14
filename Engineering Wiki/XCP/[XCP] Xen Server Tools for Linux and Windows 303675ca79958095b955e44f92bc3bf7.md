# [XCP] Xen Server Tools for Linux and Windows

Owner: Nam Tran
Last edited time: April 13, 2026 9:30 AM

```bash
curl -L https://downloads.xenserver.com/vm-tools-linux/10.0.0-1-149/LinuxGuestTools-10.0.0-1.tar.gz -O

tar xvf LinuxGuestTools-10.0.0-1.tar.gz

cd LinuxGuestTools-10.0.0-1/

rpm -i xe-guest-utilities-8.4.0-1.x86_64.rpm xe-guest-utilities-xenstore-8.4.0-1.x86_64.rpm

systemctl status xe-linux-distribution.service
```

[https://www.xenserver.com/downloads](https://www.xenserver.com/downloads)

[https://downloads.xenserver.com/vm-tools-windows/9.4.2/managementagentx64-9.4.2.msi](https://downloads.xenserver.com/vm-tools-windows/9.4.2/managementagentx64-9.4.2.msi)

[https://downloads.xenserver.com/vm-tools-linux/10.0.0-1-149/LinuxGuestTools-10.0.0-1.tar.gz](https://downloads.xenserver.com/vm-tools-linux/10.0.0-1-149/LinuxGuestTools-10.0.0-1.tar.gz)