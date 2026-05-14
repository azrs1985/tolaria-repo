# [XCP] Install XEN Guest Tools on CoreOS

Owner: Nam Tran
Last edited time: March 19, 2026 12:14 AM

```bash
curl -O -L [https://github.com/xenserver/xe-guest-utilities/releases/download/v8.4.0/xe-guest-utilities-8.4.0-1.x86_64.rpm](https://github.com/xenserver/xe-guest-utilities/releases/download/v8.4.0/xe-guest-utilities-8.4.0-1.x86_64.rpm)

curl -O -L [https://github.com/xenserver/xe-guest-utilities/releases/download/v8.4.0/xe-guest-utilities-xenstore-8.4.0-1.x86_64.rpm](https://github.com/xenserver/xe-guest-utilities/releases/download/v8.4.0/xe-guest-utilities-xenstore-8.4.0-1.x86_64.rpm)

curl -O -L https://github.com/xenserver/xe-guest-utilities/releases/download/v10.0.0/xe-guest-utilities-10.0.0-1.x86_64.rpm && curl -O -L https://github.com/xenserver/xe-guest-utilities/releases/download/v10.0.0/xe-guest-utilities-xenstore-10.0.0-1.x86_64.rpm
```

```bash
# Remove old xen-guest package
rm -f xe-guest-utilities-*
sudo rpm-ostree uninstall xe-guest-utilities-8.4.0-1.x86_64 xe-guest-utilities-xenstore-8.4.0-1.x86_64

# Install new xen-guest and xenstore package
sudo rpm-ostree install xe-guest-utilities-10.0.0-1.x86_64.rpm xe-guest-utilities-xenstore-10.0.0-1.x86_64.rpm
```