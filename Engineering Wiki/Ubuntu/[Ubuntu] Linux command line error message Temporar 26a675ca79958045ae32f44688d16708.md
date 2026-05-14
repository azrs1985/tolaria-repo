# [Ubuntu] Linux command line error message: Temporary failure in name resolution

Owner: Nam Tran
Last edited time: December 26, 2025 5:16 PM

Auto command

```bash
sudo systemctl disable --now systemd-resolved.service && \
sudo systemctl stop systemd-resolved.service && \
sudo rm -f /etc/resolv.conf && \
echo -e "nameserver 10.66.1.1\nnameserver 10.66.1.2\noptions edns0 trust-ad\nsearch unit.local" | \
sudo tee /etc/resolv.conf 

# Change /etc/hosts
127.0.0.1  <hostname>

#Fix systemd-networkd-wait-online.service
sudo systemctl disable systemd-networkd-wait-online.service && \
sudo systemctl daemon-reload
```

Using Ubuntu, first disable `systemd-resolved` service.

```bash
sudo systemctl disable systemd-resolved.service
```

Stop the service

```bash
sudo systemctl stop systemd-resolved.service
```

Then, remove the link to `/run/systemd/resolve/stub-resolv.conf` in `/etc/resolv.conf`

```bash
sudo rm /etc/resolv.conf
```

Add a manually created `resolv.conf` in `/etc/`

```bash
sudo nano /etc/resolv.conf
```

Add your prefered DNS server there

```bash
nameserver 10.66.1.1
nameserver 10.66.1.2
options edns0 trust-ad
search unit.local
```

I've tested this with success.