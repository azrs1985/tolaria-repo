# [RHEL] Permit sudo for all users

Owner: Nam Tran
Last edited time: August 28, 2025 2:04 PM

# Permit sudo for all users

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

### Uncomment the following line to implicitly trust users in the “wheel” group.

```
auth            sufficient      pam_wheel.so trust use_uid
...
```

```bash
[root@localhost ~]# groupadd nosu
[root@localhost ~]# usermod -a -G nosu root
[root@localhost ~]# usermod -a -G wheel sa
```