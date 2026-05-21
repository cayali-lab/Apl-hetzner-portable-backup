

---

```md
# Hetzner → VirtualBox Migration Guide  
**Author:** Group 2 
**Project:** Alienable Backup of Hetzner Cloud Server  
**Environment:** Ubuntu 24.04 (Hetzner → VirtualBox)  
**Status:** Work in Progress  

---

# 1. Overview
This guide documents the complete process used to:

1. Access a Hetzner Cloud server  
2. Create a raw disk image from the running system  
3. Compress and download the image  
4. Convert the image into a VirtualBox‑compatible format  
5. Boot the cloned system locally  
6. Fix networking, MySQL, Apache, and WordPress redirections  

The goal is to produce an **alienable backup** that can run outside Hetzner with minimal adjustments.

---

# 2. Accessing the Hetzner Server

### 2.1. SSH Access
Hetzner provides SSH keys for root access.

```bash
ssh -i <your_key>.pem root@<hetzner_public_ip>
```

Example:

```bash
ssh -i id_rsa root@wordpress.multinomial.se
```

Once inside, verify system info:

```bash
hostnamectl
lsblk
df -h
```

---

# 3. Creating a Raw Disk Image

We create a **bit‑for‑bit clone** of the server’s main disk.

### 3.1. Identify the disk
```bash
lsblk
```

Typical Hetzner disk:

```
/dev/sda
```

### 3.2. Create a raw image using `dd`
```bash
dd if=/dev/sda of=/root/hetzner.img bs=1M status=progress
```

This produces:

```
/root/hetzner.img
```

### 3.3. Compress the image
```bash
gzip /root/hetzner.img
```

Result:

```
/root/hetzner.img.gz
```

---

# 4. Downloading the Image

### 4.1. From your local machine:
```bash
scp -i <your_key>.pem root@<hetzner_public_ip>:/root/hetzner.img.gz .
```

Example:

```bash
scp -i id_rsa root@wordpress.multinomial.se:/root/hetzner.img.gz .
```

---

# 5. Preparing the Image for VirtualBox

### 5.1. Decompress the image
```bash
gunzip hetzner.img.gz
```

Now you have:

```
hetzner.img
```

### 5.2. Convert RAW → VDI (VirtualBox format)
```bash
VBoxManage convertfromraw hetzner.img hetzner.vdi --format VDI
```

### 5.3. Create a new VM in VirtualBox
- OS: Linux → Ubuntu (64‑bit)  
- Disk: Use existing disk → `hetzner.vdi`  
- Network: **Bridged Adapter** (important for WordPress)  

---

# 6. First Boot Fixes (VirtualBox)

### 6.1. Fix network interface names
Hetzner uses `ens3`, VirtualBox uses `enp0s3`.

Update Netplan:

```bash
nano /etc/netplan/*.yaml
```

Example:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
```

Apply:

```bash
netplan apply
```

### 6.2. Disable cloud-init (Hetzner metadata)
```bash
touch /etc/cloud/cloud-init.disabled
```

---

# 7. Restoring WordPress Functionality

## 7.1. Verify services
```bash
systemctl status apache2
systemctl status mysql
```

## 7.2. Confirm WordPress database exists
```bash
mysql -u root -p -e "SHOW DATABASES;"
```

Expected:

```
wordpress
```

## 7.3. Recreate missing MySQL user
The cloned system lost the original WordPress MySQL user.

```sql
CREATE USER 'wordpress'@'localhost'
IDENTIFIED BY 'b340b3fe0e8a98a0b8d17ce9226eb4ac04355ace20d22d2a';

GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost';
FLUSH PRIVILEGES;
```

Test:

```bash
mysql -u wordpress -p wordpress
```

---

# 8. Fixing WordPress URL Redirection

The cloned site kept redirecting to:

```
https://wordpress.multinomial.se
```

### 8.1. Update WordPress URLs in the database
```sql
UPDATE wp_options SET option_value='http://10.205.237.137'
WHERE option_name IN ('siteurl','home');
```

### 8.2. Force URL override in wp-config.php
```php
define('WP_HOME', 'http://10.205.237.137');
define('WP_SITEURL', 'http://10.205.237.137');
```

---

# 9. Removing Apache Redirections

Hetzner left behind SSL configs that forced HTTPS redirection.

### 9.1. Remove SSL VirtualHost
```bash
rm /etc/apache2/sites-enabled/000-default-le-ssl.conf
```

### 9.2. Clean HTTP VirtualHost
Edit:

```bash
nano /etc/apache2/sites-enabled/000-default.conf
```

Remove:

```
ServerName wordpress.multinomial.se
RewriteCond %{SERVER_NAME} =wordpress.multinomial.se
RewriteRule ^ https://wordpress.multinomial.se%{REQUEST_URI} [END,NE,R=permanent]
```

Replace with:

```
ServerName 10.205.237.137
```

Restart Apache:

```bash
systemctl restart apache2
```

### 9.3. Verify
```bash
curl -I http://10.205.237.137
```

Expected:

```
HTTP/1.1 200 OK
```

---

# 10. Current Status

- Hetzner server successfully cloned  
- Disk image downloaded and converted  
- VirtualBox VM boots correctly  
- Networking fixed  
- MySQL user restored  
- WordPress URL corrected  
- Apache redirections removed  
- WordPress loads from:  
  `http://10.205.237.137`

---

# 11. Next Steps

- Clean up Hetzner-specific packages  
- Regenerate SSL certificates (optional)  
- Document full backup → restore workflow  
- Automate the process with a script  
- Prepare final report for Task 01  

```

---
