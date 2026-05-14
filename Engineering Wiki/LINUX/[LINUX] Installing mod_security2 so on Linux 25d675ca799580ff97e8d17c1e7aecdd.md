# [LINUX] Installing mod_security2.so on Linux

Owner: Nam Tran
Last edited time: August 28, 2025 3:02 PM

Installing `mod_security2.so` on Linux typically involves installing the ModSecurity Apache module. The specific steps vary slightly depending on your Linux distribution.

**1. Install Apache (if not already installed):**

Code 

```bash
*# For Debian/Ubuntu*
sudo apt update && sudo apt install apache2
*# For CentOS/RHEL*
sudo yum install httpd
```

**2. Install ModSecurity Apache module:**

Code

```bash
*# For Debian/Ubuntu*
sudo apt install libapache2-mod-security2
*# For CentOS/RHEL*
sudo yum install mod_security
```

**3. Enable the ModSecurity module (if not automatically enabled):**

Code

```bash
*# For Debian/Ubuntu*
sudo a2enmod security2
*# For CentOS/RHEL (usually enabled automatically after installation)*
# You might need to add "LoadModule security2_module modules/mod_security2.so" to your httpd.conf
```

**4. Restart Apache:**

Code

```bash
*# For Debian/Ubuntu*
sudo systemctl restart apache2
*# For CentOS/RHEL*
sudo systemctl restart httpd
```

**5. Configure ModSecurity (Optional but Recommended):**

Copy the recommended ModSecurity configuration file.

Code

```bash
  *# For Debian/Ubuntu*
  sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
  *# For CentOS/RHEL*
  sudo cp /etc/httpd/conf.d/modsecurity.conf-recommended /etc/httpd/conf.d/modsecurity.conf
```

- Edit `modsecurity.conf` to adjust settings and enable/disable features as needed.
- Consider installing and configuring the OWASP Core Rule Set (CRS) for comprehensive protection.

**6. Verify Installation:**

- Check the Apache error logs for any ModSecurity-related messages.
- Test your web application to see if ModSecurity rules are being applied (e.g., by intentionally triggering a rule).
- Examine ModSecurity audit logs (e.g., `/var/log/modsec_audit.log`) to see triggered rules.