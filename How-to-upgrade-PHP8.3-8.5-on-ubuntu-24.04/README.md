# PHP 8.3 to PHP 8.5 Migration for LibreNMS on Ubuntu

## Overview

This guide documents the migration of **PHP 8.3 to PHP 8.5** for a LibreNMS server running on Ubuntu.

The main issue encountered during the migration was:

```text
502 Bad Gateway
```

The root cause was that **Nginx was configured to use a dedicated LibreNMS PHP-FPM socket**, while PHP 8.5-FPM was not initially creating that socket.

The final architecture should be:

```text
Client Browser
      │
      ▼
    Nginx
      │
      │ fastcgi_pass
      ▼
/run/php-fpm-librenms.sock
      │
      ▼
PHP 8.5-FPM
      │
      ▼
LibreNMS
      │
      ▼
MariaDB
```

---

# 1. Check the Current PHP Version

Check the installed CLI version:

```bash
php -v
```

Also check PHP 8.5 directly:

```bash
php8.5 -v
```

Verify the required PHP extensions:

```bash
php8.5 -m | grep -E 'mysqli|pdo_mysql|curl|gd|mbstring|xml|zip|bcmath|intl|Zend OPcache'
```

Expected extensions include:

```text
bcmath
curl
gd
intl
mbstring
mysqli
pdo_mysql
xml
xmlreader
xmlwriter
zip
Zend OPcache
```

---

# 2. Install PHP 8.5 Extensions

Install the required PHP 8.5 extensions:

```bash
sudo apt install php8.5-mysql php8.5-gd php8.5-curl php8.5-mbstring php8.5-xml php8.5-zip php8.5-bcmath php8.5-intl -y
```

> **Note:** `php8.5-opcache` may not be available as a separate package on the system. OPcache is already shown by:
>
> ```bash
> php8.5 -m | grep OPcache
> ```
>
> If it shows `Zend OPcache`, it is installed and enabled.

---

# 3. Check PHP-FPM Versions

Check the available PHP-FPM sockets:

```bash
ls -l /run/php/
```

During the migration, the server contained sockets such as:

```text
php8.3-fpm.sock
php8.4-fpm.sock
php8.5-fpm.sock
php-fpm.sock
```

The important point is that **the generic PHP socket does not necessarily indicate which PHP-FPM pool LibreNMS is using**.

LibreNMS was using a dedicated socket instead:

```text
/run/php-fpm-librenms.sock
```

---

# 4. Identify the Nginx PHP-FPM Socket

Check the Nginx configuration:

```bash
sudo nginx -T 2>/dev/null | grep -n "fastcgi_pass"
```

The LibreNMS configuration showed:

```text
fastcgi_pass unix:/run/php-fpm-librenms.sock;
```

Therefore, Nginx expects:

```text
/run/php-fpm-librenms.sock
```

This is important.

Do **not** assume that changing:

```text
/run/php/php-fpm.sock
```

will fix LibreNMS.

Nginx must be able to communicate with the exact PHP-FPM socket configured in its `fastcgi_pass` directive.

---

# 5. Check the PHP 8.5 LibreNMS Pool

PHP 8.5-FPM uses:

```text
/etc/php/8.5/fpm/pool.d/
```

Check the directory:

```bash
ls -la /etc/php/8.5/fpm/pool.d/
```

The required LibreNMS pool should be present:

```text
librenms.conf
```

The important configuration is:

```ini
[librenms]

listen = /run/php-fpm-librenms.sock
listen.owner = www-data
listen.group = www-data
```

Optionally, the socket mode can be explicitly defined:

```ini
listen.mode = 0660
```

Therefore:

```bash
sudo vi /etc/php/8.5/fpm/pool.d/librenms.conf
```

Verify:

```bash
sudo grep -nE '^\[|^listen|^listen\.' /etc/php/8.5/fpm/pool.d/librenms.conf
```

Expected:

```text
[librenms]
listen = /run/php-fpm-librenms.sock
listen.owner = www-data
listen.group = www-data
listen.mode = 0660
```

---

# 6. Understand the `www.conf` and `librenms.conf` Difference

PHP 8.5 initially contained:

```text
/etc/php/8.5/fpm/pool.d/www.conf
```

with:

```ini
listen = /run/php/php8.5-fpm.sock
```

The LibreNMS pool contained:

```ini
listen = /run/php-fpm-librenms.sock
```

These are two different PHP-FPM pools.

The LibreNMS Nginx configuration specifically expects:

```text
/run/php-fpm-librenms.sock
```

Therefore, the **LibreNMS pool must provide that socket**.

If the default `www` pool is not required, it can be disabled:

```bash
sudo mv /etc/php/8.5/fpm/pool.d/www.conf /etc/php/8.5/fpm/pool.d/www.conf.disabled
```

This leaves the dedicated LibreNMS pool as the active pool.

---

# 7. Verify PHP-FPM Configuration

Before restarting PHP-FPM, test the configuration:

```bash
sudo php-fpm8.5 -tt
```

To specifically check the LibreNMS pool:

```bash
sudo php-fpm8.5 -tt 2>&1 | grep -A15 '\[librenms\]'
```

The important result is:

```text
[librenms]
user = librenms
group = librenms
listen = /run/php-fpm-librenms.sock
listen.owner = www-data
listen.group = www-data
```

And the final configuration test should say:

```text
configuration file /etc/php/8.5/fpm/php-fpm.conf test is successful
```

---

# 8. Verify PHP-FPM Includes the Pool

Check:

```bash
sudo grep -n "include" /etc/php/8.5/fpm/php-fpm.conf
```

The configuration should contain:

```text
include=/etc/php/8.5/fpm/pool.d/*.conf
```

This means PHP-FPM loads:

```text
/etc/php/8.5/fpm/pool.d/*.conf
```

including:

```text
librenms.conf
```

---

# 9. Restart PHP 8.5-FPM

Once the configuration test succeeds:

```bash
sudo systemctl restart php8.5-fpm
```

Check the service:

```bash
sudo systemctl status php8.5-fpm --no-pager
```

Expected:

```text
Active: active (running)
```

The process list should also show:

```text
php-fpm: pool librenms
```

For example:

```text
php-fpm: master process
php-fpm: pool librenms
php-fpm: pool librenms
```

This confirms that PHP 8.5-FPM has loaded the LibreNMS pool.

---

# 10. Verify the LibreNMS Socket

Check:

```bash
sudo ls -l /run/php-fpm-librenms.sock
```

The socket must exist.

You can also check listening Unix sockets:

```bash
sudo ss -lx | grep -E 'php|fpm'
```

The required socket is:

```text
/run/php-fpm-librenms.sock
```

---

# 11. Test Nginx

Before restarting Nginx:

```bash
sudo nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

Then restart:

```bash
sudo systemctl restart nginx
```

---

# 12. Test LibreNMS Locally

Test Nginx from the server itself:

```bash
curl -I http://127.0.0.1
```

Previously the result was:

```text
HTTP/1.1 502 Bad Gateway
```

This occurred because Nginx was trying to communicate with:

```text
/run/php-fpm-librenms.sock
```

while the socket did not exist.

After PHP 8.5-FPM correctly loads the LibreNMS pool and creates the socket, the `502 Bad Gateway` should disappear.

---

# 13. If You Still Get 502 Bad Gateway

Check the Nginx error log:

```bash
sudo tail -50 /var/log/nginx/error.log
```

Look for messages such as:

```text
connect() to unix:/run/php-fpm-librenms.sock failed
```

This immediately indicates a PHP-FPM socket problem.

Also verify:

```bash
sudo ls -l /run/php-fpm-librenms.sock
```

and:

```bash
sudo systemctl status php8.5-fpm --no-pager
```

---

# 14. Verify LibreNMS Using PHP 8.5

Run:

```bash
cd /opt/librenms
sudo -u librenms ./validate.php
```

The important result from the migration was:

```text
PHP | 8.5.9
```

This confirms that LibreNMS CLI validation is running under PHP 8.5.

---

# 15. Fix LibreNMS Database Credentials

The validation also reported:

```text
The database credentials are incorrect.
```

Specifically:

```text
Access denied for user 'librenms'@'localhost' (using password: NO)
```

This is **separate from the PHP-FPM/502 problem**.

Check the LibreNMS environment:

```bash
sudo vi /opt/librenms/.env
```

Verify the database settings, particularly:

```text
DB_DATABASE
DB_USERNAME
DB_PASSWORD
DB_HOST
```

The credentials must match the MariaDB/MySQL account created for LibreNMS.

After correcting the credentials:

```bash
cd /opt/librenms
sudo -u librenms ./validate.php
```

---

# 16. Fix PHP Timezone

LibreNMS also reported:

```text
The system timezone (IST) is different from the PHP configured timezone (UTC)
```

Since the server uses India Standard Time, configure PHP for:

```text
Asia/Kolkata
```

For PHP 8.5 CLI:

```bash
sudo vi /etc/php/8.5/cli/php.ini
```

Find:

```ini
;date.timezone =
```

Set:

```ini
date.timezone = Asia/Kolkata
```

For PHP-FPM:

```bash
sudo vi /etc/php/8.5/fpm/php.ini
```

Set:

```ini
date.timezone = Asia/Kolkata
```

Restart PHP-FPM:

```bash
sudo systemctl restart php8.5-fpm
```

Verify CLI:

```bash
php8.5 -i | grep "date.timezone"
```

---

# 17. Set PHP 8.5 as the Default CLI Version

Check available PHP versions:

```bash
sudo update-alternatives --config php
```

Select:

```text
/usr/bin/php8.5
```

Verify:

```bash
php -v
```

Expected:

```text
PHP 8.5.x
```

---

# 18. Do Not Confuse the Generic PHP-FPM Socket With the LibreNMS Socket

A major lesson from this migration is that these are different:

```text
/run/php/php-fpm.sock
```

and:

```text
/run/php-fpm-librenms.sock
```

The generic socket may point to:

```text
/run/php/php8.5-fpm.sock
```

while LibreNMS can use its own dedicated pool:

```text
/run/php-fpm-librenms.sock
```

Always check Nginx:

```bash
sudo nginx -T 2>/dev/null | grep -n "fastcgi_pass"
```

Then make sure the PHP-FPM pool has the same `listen` path.

---

# 19. Final Verification

Run all of the following:

```bash
php -v
```

```bash
sudo systemctl status php8.5-fpm --no-pager
```

```bash
sudo ls -l /run/php-fpm-librenms.sock
```

```bash
sudo nginx -t
```

```bash
sudo systemctl status nginx --no-pager
```

```bash
curl -I http://127.0.0.1
```

Then:

```bash
cd /opt/librenms
sudo -u librenms ./validate.php
```

The desired architecture is:

```text
                 ┌─────────────────┐
                 │     Browser     │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │      Nginx      │
                 └────────┬────────┘
                          │
                          │ fastcgi_pass
                          ▼
             /run/php-fpm-librenms.sock
                          │
                          ▼
                 ┌─────────────────┐
                 │ PHP 8.5-FPM     │
                 │                 │
                 │ librenms pool   │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │    LibreNMS     │
                 └────────┬────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │    MariaDB      │
                 └─────────────────┘
```

---

# 20. Removing PHP 8.3 — Only After Successful Migration

**Do not remove PHP 8.3 until:**

* PHP 8.5 CLI works
* PHP 8.5-FPM is running
* `librenms` FPM pool is running
* `/run/php-fpm-librenms.sock` exists
* Nginx configuration passes
* LibreNMS web interface opens
* `validate.php` reports PHP 8.5
* Database credentials are working

After everything is confirmed, PHP 8.3 packages can be removed:

```bash
sudo apt purge 'php8.3*' -y
```

Then:

```bash
sudo apt autoremove -y
```

If PHP 8.4 is also no longer required:

```bash
sudo apt purge 'php8.4*' -y
sudo apt autoremove -y
```

Finally:

```bash
sudo systemctl restart php8.5-fpm
sudo systemctl restart nginx
```

Run the final validation again:

```bash
cd /opt/librenms
sudo -u librenms ./validate.php
```

---

# Key Troubleshooting Lesson

The critical problem was **not simply "PHP 8.3 is installed."**

The actual chain was:

```text
Nginx
  │
  │ expects
  ▼
/run/php-fpm-librenms.sock
  │
  │ socket missing
  ▼
502 Bad Gateway
```

The solution was to ensure:

```text
Nginx:
fastcgi_pass unix:/run/php-fpm-librenms.sock;
```

matches:

```text
PHP 8.5-FPM:
[librenms]
listen = /run/php-fpm-librenms.sock
```

Once these two configurations match and PHP 8.5-FPM creates the socket, Nginx can communicate with the LibreNMS PHP 8.5 pool.
