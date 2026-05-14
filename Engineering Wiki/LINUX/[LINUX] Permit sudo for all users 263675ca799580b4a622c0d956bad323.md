# [LINUX] Permit sudo for all users

Owner: Nam Tran
Last edited time: April 20, 2026 9:29 AM

```bash
[root@localhost ~]# visudo
...
 Allows people in group wheel to run all commands
 %wheel        ALL=(ALL)       ALL
```

```bash
[root@localhost ~]# vi /etc/pam.d/su
...
auth [success=ignore default=1] pam_succeed_if.so user ingroup nosu
auth required pam_deny.so
```

### Uncomment the following line to implicitly trust users in the "wheel" group.

```bash
auth            sufficient      pam_wheel.so trust use_uid
...
```

```bash
[root@localhost ~]# groupadd nosu
[root@localhost ~]# usermod -a -G nosu root
[root@localhost ~]# usermod -a -G wheel sa
```

# Cho phép chạy lệnh admin nhưng KHÔNG cho sudo -i

stackops ALL=(ALL) ALL, \
!/usr/bin/sudo -i, \
!/usr/bin/sudo -s, \
!/usr/bin/sudo su, \
!/usr/bin/sudo su -, \
!/usr/bin/su, \
!/usr/bin/su -, \
!/bin/bash, \
!/bin/sh, \
!/usr/bin/bash, \
!/usr/bin/sh, \
!/usr/bin/passwd