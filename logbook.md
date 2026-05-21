
```md
# Task 01 – Hetzner Backup & Restoration Logbook
**Author:** Group 2 
**Project:** Alienable Backup of Hetzner Cloud Server  
**Environment:** Ubuntu 24.04 (Hetzner → VirtualBox)  
**Status:** In Progress  

---

## 1. Objective
Create a fully alienable backup of a Hetzner Cloud server that can be:

- Generated directly from the Hetzner instance  
- Downloaded as a file  
- Restored on a non-Hetzner environment (VirtualBox)  
- Booted and used with minimal configuration changes  
- Capable of running the original software (WordPress)

---

## 2. Access to the Cloned Server
After restoring the disk image into a VirtualBox VM:

- Network configured in bridged mode  
- SSH access restored  
- Root login enabled temporarily for recovery  
- Verified that the system boots correctly outside Hetzner  

---

## 3. Service Stack Verification
Checked which services were active on the cloned machine:

| Service | Status | Notes |
|--------|--------|-------|
| Apache2 | Running | Primary web server |
| MySQL | Running | Contains `wordpress` database |
| Nginx | Not installed | Not required |
| MariaDB | Not installed | MySQL used instead |

---

## 4. WordPress Database Recovery
The cloned server contained the original WordPress database:

```
information_schema
mysql
performance_schema
sys
wordpress   ← original DB found
```

However, the WordPress MySQL user was missing, causing WordPress to show the installation screen.

### Recreated the missing MySQL user:

```sql
CREATE USER 'wordpress'@'localhost'
IDENTIFIED BY 'b340b3fe0e8a98a0b8d17ce9226eb4ac04355ace20d22d2a';

GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost';
FLUSH PRIVILEGES;
```

Connection test succeeded.

---

## 5. WordPress URL Mismatch
After restoring DB access, WordPress redirected to:

```
https://wordpress.multinomial.se
```

This happened because the original domain was still stored in the database and Apache configuration.

### Updated WordPress URLs:

```sql
UPDATE wp_options SET option_value='http://10.205.237.137'
WHERE option_name IN ('siteurl','home');
```

### Added forced URL override in `wp-config.php`:

```php
define('WP_HOME', 'http://10.205.237.137');
define('WP_SITEURL', 'http://10.205.237.137');
```

---

## 6. Apache Redirection Cleanup
Apache was still forcing a redirect to the old domain due to leftover Hetzner configs.

### Files identified:

```
/etc/apache2/sites-enabled/000-default.conf
/etc/apache2/sites-enabled/000-default-le-ssl.conf
```

### Issues found:

- `ServerName wordpress.multinomial.se`
- Rewrite rules forcing HTTPS redirection
- Let’s Encrypt SSL config referencing old domain

### Fixes applied:

- Removed SSL VirtualHost symlink:

```
rm /etc/apache2/sites-enabled/000-default-le-ssl.conf
```

- Cleaned `000-default.conf`:
  - Removed rewrite rules
  - Updated ServerName to the VM IP

- Restarted Apache:

```
systemctl restart apache2
```

### Verification:

```
curl -I http://10.205.237.137
```

Expected output:

```
HTTP/1.1 200 OK
```

---

## 7. Current Status
- Server boots correctly in VirtualBox  
- Apache + MySQL operational  
- WordPress database restored  
- WordPress URL corrected  
- Apache redirections removed  
- WordPress expected to load from:  
  `http://10.205.237.137`

---

## 8. Next Steps
- Final verification of WordPress frontend  
- Cleanup of cloud-init and Hetzner metadata  
- Optional: regenerate SSL for local testing  
- Document full backup → restore procedure  
- Prepare final report for Task 01  

```
