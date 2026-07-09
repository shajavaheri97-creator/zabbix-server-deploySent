# Nginx and PHP-FPM Configuration

## Overview

This document describes the configuration of the Zabbix Web Frontend using **Nginx** and **PHP-FPM**.

Although installing the required packages is straightforward, several configuration issues prevented the frontend from loading correctly. This document explains both the deployment process and the troubleshooting steps that were necessary to achieve a working web interface.

---

# Architecture

```text
                Browser
                   │
                   ▼
              Nginx (Port 80)
                   │
                   ▼
          PHP-FPM (FastCGI Socket)
                   │
                   ▼
         Zabbix PHP Frontend
                   │
                   ▼
            PostgreSQL Database
```

---

# Install Required Packages

```bash
sudo apt install \
nginx \
php8.1-fpm \
zabbix-frontend-php \
zabbix-nginx-conf -y
```

---

# Verify Installed Packages

```bash
dpkg -l | grep nginx
```

```bash
dpkg -l | grep php
```

```bash
dpkg -l | grep zabbix
```

---

# Verify Required Services

```bash
systemctl status nginx
```

```bash
systemctl status php8.1-fpm
```

Both services should be active.

---

# Zabbix Nginx Configuration

The package installs the following configuration file:

```text
/etc/zabbix/nginx.conf
```

It is linked into:

```text
/etc/nginx/conf.d/zabbix.conf
```

Verify the symbolic link:

```bash
ls -l /etc/nginx/conf.d/
```

---

# Document Root

The Zabbix frontend is installed under:

```text
/usr/share/zabbix
```

Verify:

```bash
ls -l /usr/share/zabbix
```

The directory should contain:

* index.php
* ui/
* assets/
* vendor/

---

# PHP-FPM Socket

Verify the configured socket.

```bash
ls /var/run/php/
```

Typical output:

```text
zabbix.sock
```

The Nginx configuration should reference this socket:

```nginx
fastcgi_pass unix:/var/run/php/zabbix.sock;
```

---

# FastCGI Parameters

The following parameters must point to the correct document root.

```nginx
fastcgi_param DOCUMENT_ROOT /usr/share/zabbix;

fastcgi_param SCRIPT_FILENAME /usr/share/zabbix$fastcgi_script_name;

fastcgi_param PATH_TRANSLATED /usr/share/zabbix$fastcgi_script_name;
```

Incorrect paths may prevent PHP from executing properly.

---

# Verify Configuration

Test the configuration:

```bash
sudo nginx -t
```

Display the loaded configuration:

```bash
sudo nginx -T
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

---

# Real Deployment Issues

---

## Issue 1 — HTTP 404

### Symptom

Opening the server IP displayed:

```text
404 Not Found
```

instead of the Zabbix login page.

---

### Investigation

The Zabbix configuration file existed.

```text
/etc/nginx/conf.d/zabbix.conf
```

The frontend files also existed.

```text
/usr/share/zabbix
```

The PHP socket was available.

The problem was therefore not related to package installation.

---

### Root Cause

The root location used:

```nginx
try_files $uri $uri/ =404;
```

This caused Nginx to immediately return a 404 response whenever the requested URI did not exist as a physical file.

---

### Resolution

The configuration was updated to:

```nginx
location / {
    try_files $uri $uri/ /index.php?$args;
}
```

This allows requests to be routed through the Zabbix front controller.

---

## Issue 2 — Incorrect Listen Port

During deployment, the configuration contained:

```nginx
listen 8080;
```

This caused the frontend to listen on port **8080** instead of the default HTTP port.

### Resolution

The explicit listen directive was removed.

With the custom directive removed, Nginx served the application using the standard HTTP configuration on port **80**.

---

## Issue 3 — PHP Frontend Appeared Correctly Installed but Was Not Reachable

The following components were verified:

* Nginx service
* PHP-FPM service
* Zabbix package installation
* Document root
* PHP socket

Everything appeared to be installed correctly, yet the frontend still failed.

This highlighted the importance of validating the active Nginx configuration instead of assuming the package defaults were being used.

---

## Issue 4 — Running nginx Without sudo

Executing:

```bash
nginx -T
```

generated permission-related errors because Nginx could not access protected files.

### Resolution

Run administrative commands using sudo:

```bash
sudo nginx -T
```

---

## Issue 5 — Symbolic Link Verification

The deployment confirmed that:

```text
/etc/nginx/conf.d/zabbix.conf
```

was a symbolic link pointing to:

```text
/etc/zabbix/nginx.conf
```

Understanding this relationship simplified future troubleshooting because configuration changes only needed to be made in one location.

---

# Verification Checklist

Before continuing, verify that:

* Nginx is running.
* PHP-FPM is running.
* The Zabbix frontend files exist.
* The PHP socket exists.
* The symbolic link is valid.
* `nginx -t` reports a valid configuration.
* The Zabbix login page loads successfully.

---

# Lessons Learned

Several important lessons were learned during the frontend deployment.

* A running Nginx service does not guarantee that the application is correctly configured.
* The `try_files` directive has a significant impact on PHP applications.
* Always inspect the active configuration using `nginx -T`.
* Verify the PHP-FPM socket before assuming PHP is malfunctioning.
* Symbolic links in `/etc/nginx/conf.d/` determine which application configurations are actually loaded.
* Troubleshooting should begin by validating each layer independently: Nginx → PHP-FPM → Frontend files → Database.

---

# Key Takeaways

The web frontend depends on the successful interaction between Nginx, PHP-FPM, and the Zabbix PHP application.

Although the installation itself is relatively simple, small configuration errors—such as an incorrect `try_files` directive or a non-standard listen port—can completely prevent the frontend from loading.

A systematic verification process greatly reduces troubleshooting time and helps isolate configuration issues quickly.
