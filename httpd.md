==#🌐 **APACHE HTTPD WEB SERVER CONFIGURATION**==

---

==**INSTALLATION**==
> Package name: httpd (RHEL/CentOS) or apache2 (Ubuntu/Debian)  
> Service name: httpd (RHEL/CentOS) or apache2 (Ubuntu/Debian)  
> Default port: 80 (HTTP), 443 (HTTPS)

**For RHEL/CentOS/Fedora:**
```bash
sudo dnf install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
sudo systemctl status httpd
```

**For Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl status apache2
```

**Open Firewall (if enabled):**
```bash
# For firewalld (RHEL/CentOS)
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload

# For UFW (Ubuntu/Debian)
sudo ufw allow 'Apache'
sudo ufw allow 'Apache Full'
```

---

==**IMPORTANT DIRECTORIES AND FILES**==

| Purpose | RHEL/CentOS Path | Ubuntu/Debian Path |
|---------|------------------|-------------------|
| Main config file | `/etc/httpd/conf/httpd.conf` | `/etc/apache2/apache2.conf` |
| Virtual host configs | `/etc/httpd/conf.d/` | `/etc/apache2/sites-available/` |
| Enabled sites (symlinks) | N/A (auto-loaded) | `/etc/apache2/sites-enabled/` |
| Default document root | `/var/www/html` | `/var/www/html` |
| Website directories | `/var/www/domain.com/public_html/` | `/var/www/domain.com/public_html/` |
| Access logs | `/var/log/httpd/access_log` | `/var/log/apache2/access.log` |
| Error logs | `/var/log/httpd/error_log` | `/var/log/apache2/error.log` |
| Modules directory | `/etc/httpd/modules/` | `/etc/apache2/mods-available/` |
| Configuration includes | `/etc/httpd/conf.d/` | `/etc/apache2/conf-available/` |

---

==**BASIC CONFIGURATION - SINGLE WEBSITE**==

**Create document root:**
```bash
sudo mkdir -p /var/www/html
sudo chown -R apache:apache /var/www/html  # For RHEL/CentOS
# OR
sudo chown -R www-data:www-data /var/www/html  # For Ubuntu/Debian
sudo chmod -R 755 /var/www/html
```

**Create a test page:**
```bash
echo "<h1>Welcome to Apache Web Server</h1>" | sudo tee /var/www/html/index.html
```

**Test in browser:**
```
http://your-server-ip
```

---

==**VIRTUAL HOSTS - MULTIPLE WEBSITES**==

**Directory Structure Setup:**
```bash
# Create directories for two websites
sudo mkdir -p /var/www/site1.com/public_html
sudo mkdir -p /var/www/site2.com/public_html

# Set ownership
sudo chown -R apache:apache /var/www  # RHEL/CentOS
# OR
sudo chown -R www-data:www-data /var/www  # Ubuntu/Debian

# Set permissions
sudo chmod -R 755 /var/www
```

**Create test pages:**
```bash
echo "<h1>Welcome to site1.com</h1>" | sudo tee /var/www/site1.com/public_html/index.html
echo "<h1>Welcome to site2.com</h1>" | sudo tee /var/www/site2.com/public_html/index.html
```

---

==**VIRTUAL HOST CONFIGURATION - SITE 1**==

**For RHEL/CentOS:**
`/etc/httpd/conf.d/site1.com.conf`
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@site1.com
    ServerName site1.com
    ServerAlias www.site1.com
    DocumentRoot /var/www/site1.com/public_html
    DirectoryIndex index.html index.php

    <Directory /var/www/site1.com/public_html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/site1.com-error.log
    CustomLog /var/log/httpd/site1.com-access.log combined
</VirtualHost>
```

**For Ubuntu/Debian:**
`/etc/apache2/sites-available/site1.com.conf`
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@site1.com
    ServerName site1.com
    ServerAlias www.site1.com
    DocumentRoot /var/www/site1.com/public_html
    DirectoryIndex index.html index.php

    <Directory /var/www/site1.com/public_html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/site1.com-error.log
    CustomLog ${APACHE_LOG_DIR}/site1.com-access.log combined
</VirtualHost>
```

---

==**VIRTUAL HOST CONFIGURATION - SITE 2**==

**For RHEL/CentOS:**
`/etc/httpd/conf.d/site2.com.conf`
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@site2.com
    ServerName site2.com
    ServerAlias www.site2.com
    DocumentRoot /var/www/site2.com/public_html
    DirectoryIndex index.html index.php

    <Directory /var/www/site2.com/public_html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/site2.com-error.log
    CustomLog /var/log/httpd/site2.com-access.log combined
</VirtualHost>
```

**For Ubuntu/Debian:**
`/etc/apache2/sites-available/site2.com.conf`
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@site2.com
    ServerName site2.com
    ServerAlias www.site2.com
    DocumentRoot /var/www/site2.com/public_html
    DirectoryIndex index.html index.php

    <Directory /var/www/site2.com/public_html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/site2.com-error.log
    CustomLog ${APACHE_LOG_DIR}/site2.com-access.log combined
</VirtualHost>
```

---

==**ENABLE VIRTUAL HOSTS**==

**For RHEL/CentOS:**
```bash
# Configuration files in /etc/httpd/conf.d/ are auto-loaded
# Just restart httpd
sudo systemctl restart httpd
```

**For Ubuntu/Debian:**
```bash
# Enable sites using a2ensite
sudo a2ensite site1.com.conf
sudo a2ensite site2.com.conf

# Disable default site (optional)
sudo a2dissite 000-default.conf

# Reload Apache
sudo systemctl reload apache2
```

---

==**DNS/HOSTS FILE CONFIGURATION**==

**For testing, add to `/etc/hosts` on client machines:**
```bash
192.168.1.100 site1.com www.site1.com
192.168.1.100 site2.com www.site2.com
```

**Or configure proper DNS records:**
```
site1.com.      IN  A   192.168.1.100
www.site1.com.  IN  A   192.168.1.100
site2.com.      IN  A   192.168.1.100
www.site2.com.  IN  A   192.168.1.100
```

---

==**SSL/HTTPS CONFIGURATION**==

**Install SSL module:**
```bash
# RHEL/CentOS
sudo dnf install mod_ssl -y

# Ubuntu/Debian
sudo a2enmod ssl
```

**Self-signed certificate (for testing):**
```bash
sudo mkdir -p /etc/httpd/ssl  # or /etc/apache2/ssl

sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/httpd/ssl/site1.com.key \
  -out /etc/httpd/ssl/site1.com.crt \
  -subj "/C=IN/ST=State/L=City/O=Company/CN=site1.com"
```

**SSL Virtual Host Configuration:**
`/etc/httpd/conf.d/site1.com-ssl.conf` or `/etc/apache2/sites-available/site1.com-ssl.conf`
```apache
<VirtualHost *:443>
    ServerAdmin webmaster@site1.com
    ServerName site1.com
    ServerAlias www.site1.com
    DocumentRoot /var/www/site1.com/public_html

    SSLEngine on
    SSLCertificateFile /etc/httpd/ssl/site1.com.crt
    SSLCertificateKeyFile /etc/httpd/ssl/site1.com.key

    <Directory /var/www/site1.com/public_html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/site1.com-ssl-error.log
    CustomLog /var/log/httpd/site1.com-ssl-access.log combined
</VirtualHost>
```

---

==**USEFUL APACHE COMMANDS**==

```bash
# Test configuration syntax
sudo apachectl configtest      # or: sudo httpd -t
sudo apache2ctl configtest     # Ubuntu/Debian

# Restart service
sudo systemctl restart httpd   # RHEL/CentOS
sudo systemctl restart apache2 # Ubuntu/Debian

# Reload configuration without restart
sudo systemctl reload httpd
sudo systemctl reload apache2

# Check service status
sudo systemctl status httpd
sudo systemctl status apache2

# Enable/disable sites (Ubuntu/Debian only)
sudo a2ensite site.conf        # Enable site
sudo a2dissite site.conf       # Disable site

# Enable/disable modules (Ubuntu/Debian)
sudo a2enmod rewrite           # Enable mod_rewrite
sudo a2dismod autoindex        # Disable directory listing

# View logs in real-time
sudo tail -f /var/log/httpd/access_log
sudo tail -f /var/log/apache2/access.log
```

---

==**COMMON APACHE MODULES**==

| Module | Purpose | Enable Command (Ubuntu) |
|--------|---------|------------------------|
| mod_rewrite | URL rewriting | `sudo a2enmod rewrite` |
| mod_ssl | SSL/TLS support | `sudo a2enmod ssl` |
| mod_headers | HTTP header manipulation | `sudo a2enmod headers` |
| mod_deflate | Compression | `sudo a2enmod deflate` |
| mod_proxy | Reverse proxy | `sudo a2enmod proxy proxy_http` |
| mod_php | PHP support | Auto-enabled with PHP |

**RHEL/CentOS:** Modules are in `/etc/httpd/modules/` and loaded in config files.

---

==**DIRECTORY OPTIONS EXPLAINED**==

| Option | Description |
|--------|-------------|
| `Options Indexes` | Show directory listing if no index file |
| `Options FollowSymLinks` | Allow following symbolic links |
| `AllowOverride All` | Allow `.htaccess` files to override settings |
| `Require all granted` | Allow access to all users |
| `Require all denied` | Deny access to all users |

---

==**HTACCESS FILE EXAMPLES**==

**Password Protection:**
`/var/www/site1.com/public_html/.htaccess`
```apache
AuthType Basic
AuthName "Restricted Area"
AuthUserFile /var/www/site1.com/.htpasswd
Require valid-user
```

**Create password file:**
```bash
sudo htpasswd -c /var/www/site1.com/.htpasswd username
```

**URL Rewriting (Clean URLs):**
```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php?url=$1 [L,QSA]
```

---

==**TROUBLESHOOTING**==

```bash
# Check if Apache is listening on port 80/443
sudo netstat -tlnp | grep httpd
sudo ss -tlnp | grep apache2

# Check SELinux context (RHEL/CentOS)
sudo ls -Z /var/www/html
sudo chcon -R -t httpd_sys_content_t /var/www/

# Set SELinux boolean for network connections
sudo setsebool -P httpd_can_network_connect 1

# Check Apache error logs
sudo tail -50 /var/log/httpd/error_log
sudo tail -50 /var/log/apache2/error.log

# Check configuration for errors
sudo apachectl configtest
sudo apache2ctl configtest

# Verify DNS resolution
dig site1.com
nslookup site1.com
```

---

==**PERFORMANCE TUNING**==

**Edit `/etc/httpd/conf/httpd.conf` or `/etc/apache2/apache2.conf`:**
```apache
# Adjust these based on server resources
<IfModule mpm_prefork_module>
    StartServers          5
    MinSpareServers       5
    MaxSpareServers      10
    MaxRequestWorkers   150
    MaxConnectionsPerChild 3000
</IfModule>

# Enable compression
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css text/javascript application/javascript
</IfModule>

# Browser caching
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType image/jpg "access plus 1 year"
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/gif "access plus 1 year"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
</IfModule>
```

---

==**IMPORTANT NOTES**==

- Always test configuration before restarting: `apachectl configtest`
- Use name-based virtual hosting for multiple sites on one IP
- Keep log files rotated to prevent disk space issues
- Set proper file permissions (755 for directories, 644 for files)
- Use SSL/TLS for sensitive data (Let's Encrypt for free certificates)
- Configure SELinux/AppArmor contexts properly for security
- Regular security updates are critical