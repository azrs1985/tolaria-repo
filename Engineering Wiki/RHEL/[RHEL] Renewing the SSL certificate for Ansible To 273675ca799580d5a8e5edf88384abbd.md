# [RHEL] Renewing the SSL certificate for Ansible Tower

Owner: Nam Tran
Last edited time: September 19, 2025 5:00 PM

Renewing the SSL certificate for Ansible Tower (now Ansible Automation Platform) involves replacing the existing certificate and key files with new ones. This process ensures secure communication with the platform.

## Here are the steps to renew the SSL certificate:

- **Backup Existing Files:** Create backups of the current SSL certificate and key files.

```bash
    cp /etc/tower/tower.cert /etc/tower/tower.cert-$(date +%F)
    cp /etc/tower/tower.key /etc/tower/tower.key-$(date +%F)
```

- **Obtain New Certificate and Key:** Acquire a new SSL certificate and its corresponding private key. These can be generated through a Certificate Authority (CA) or a service like Let's Encrypt.
- **Replace Certificate and Key:** Copy the new SSL certificate to `/etc/tower/tower.cert` and the new private key to `/etc/tower/tower.key`, overwriting the old files.
- **Restore SELinux Context:** Ensure the correct SELinux context is applied to the new files.

```bash
    restorecon -v /etc/tower/tower.cert /etc/tower/tower.key
```

- **Set File Permissions:** Assign appropriate ownership and permissions to the certificate and key files.

```bash
    chown root:awx /etc/tower/tower.cert /etc/tower/tower.key
    chmod 0600 /etc/tower/tower.cert /etc/tower/tower.key
```

- **Test NGINX Configuration:** Verify that the NGINX configuration is still valid after the changes.

```bash
    nginx -t
```

- **Reload NGINX:** Apply the new certificate by reloading the NGINX service.

```bash
    systemctl reload nginx.service
```

- **Verify Installation:** Confirm that the new SSL certificate and key are active.

```bash
    true | openssl s_client -showcerts -connect ${CONTROLLER_FQDN}:443
```

Replace `${CONTROLLER_FQDN}` with the fully qualified domain name of your Ansible Automation Platform instance.