# Troubleshooting Guide

## Overview

This document contains the most common issues encountered during the deployment of **Zabbix 7.2** with **PostgreSQL**, **Nginx**, **PHP-FPM**, and **LVM** on **Ubuntu Server 22.04 LTS**.

Unlike a typical installation guide, this document is based on real deployment issues encountered during implementation. Each issue includes its symptoms, root cause, diagnostic steps, resolution, and prevention recommendations.

---

# Issue 1 — Wrong Filesystem Type

## Error

```text
mount: wrong fs type, bad option, bad superblock
```

## Symptoms

* Mount operation failed.
* `/var/lib/postgresql` was not mounted.
* PostgreSQL data directory was unavailable.

## Root Cause

The Logical Volume had either not been formatted or an incorrect device was being mounted.

## Diagnosis

```bash
lsblk -f
```

```bash
blkid
```

```bash
lvs
```

## Resolution

Format the Logical Volume:

```bash
sudo mkfs.ext4 /dev/vg_data/lv_pgsql
```

Mount it:

```bash
sudo mount /dev/vg_data/lv_pgsql /var/lib/postgresql
```

## Prevention

Always verify the filesystem before mounting.

---

# Issue 2 — Invalid UUID in /etc/fstab

## Error

```text
can't find UUID=xxxxxxxx
```

## Symptoms

* Boot warnings
* mount -a failed
* Filesystem was not mounted

## Root Cause

The UUID of the LVM Physical Volume was used instead of the filesystem UUID.

## Diagnosis

```bash
blkid
```

## Resolution

Replace the incorrect UUID with the ext4 filesystem UUID.

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Test:

```bash
sudo mount -a
```

## Prevention

Always use the UUID reported for the filesystem, not the LVM member.

---

# Issue 3 — systemd Still Uses Old fstab

## Error

```text
your fstab has been modified,
but systemd still uses the old version
```

## Root Cause

systemd had not reloaded the mount configuration.

## Resolution

```bash
sudo systemctl daemon-reload
```

---

# Issue 4 — PostgreSQL Database Does Not Exist

## Error

```text
FATAL:
database "zabbix" does not exist
```

## Root Cause

The database was never created or an incorrect database name was specified.

## Diagnosis

```bash
sudo -u postgres psql -l
```

## Resolution

Create the database.

```bash
sudo -u postgres createdb \
-o zabbix \
-E UTF8 \
zabbix
```

---

# Issue 5 — Schema Imported with Wrong User

## Error

```text
permission denied for table users
```

and

```text
database is not a Zabbix database
```

## Symptoms

* Zabbix Server immediately stopped.
* systemd continuously restarted the service.
* Login page never became available.

## Root Cause

The schema was imported using the PostgreSQL superuser instead of the Zabbix database owner.

As a result, all tables belonged to **postgres**.

## Diagnosis

```sql
SELECT tablename,
tableowner
FROM pg_tables
WHERE schemaname='public';
```

## Resolution

Drop the incorrect database.

Create a new database.

Import the schema as the **zabbix** user.

```bash
zcat /usr/share/zabbix/sql-scripts/postgresql/server.sql.gz | \
sudo -u zabbix psql zabbix
```

## Prevention

Always import the schema using the database owner.

---

# Issue 6 — REASSIGN OWNED Failed

## Error

```text
cannot reassign ownership
```

## Root Cause

System objects owned by PostgreSQL cannot be reassigned.

## Resolution

Recreate the database instead of attempting ownership repair.

---

# Issue 7 — Zabbix Server Failed to Start

## Error

```text
Failed with result 'protocol'
```

## Symptoms

```bash
systemctl status zabbix-server
```

showed repeated restart attempts.

## Diagnosis

```bash
tail -100 /var/log/zabbix/zabbix_server.log
```

## Root Cause

Database initialization failed.

## Resolution

Fix the PostgreSQL configuration and verify table ownership.

---

# Issue 8 — PID File Error

## Error

```text
Can't open PID file
```

## Root Cause

The Zabbix process exited immediately before creating the PID file.

The PID file error was only a symptom.

## Resolution

Investigate the server log instead of focusing on the PID file.

```bash
tail -100 /var/log/zabbix/zabbix_server.log
```

---

# Issue 9 — Repository Returned HTTP 404

## Error

```text
404 Not Found
```

while downloading the repository package.

## Root Cause

Network restrictions prevented access to the official repository.

## Resolution

Download the repository package from another machine and transfer it to the server.

Alternatively, configure a trusted Ubuntu package mirror for supported packages.

---

# Issue 10 — Unsupported Ubuntu Version

## Symptoms

Repository packages could not be installed.

## Root Cause

Ubuntu 26.04 (Resolute) was not officially supported by Zabbix.

## Resolution

Downgrade to Ubuntu 22.04 LTS.

---

# Issue 11 — TimescaleDB Extension Not Available

## Error

```text
extension "timescaledb" is not available
```

## Root Cause

The TimescaleDB package was not installed.

## Resolution

Continue the installation without TimescaleDB or install the extension from the official repository before enabling it.

---

# Issue 12 — HTTP 404 on Zabbix Frontend

## Symptoms

The web server displayed:

```text
404 Not Found
```

## Root Cause

The Nginx configuration used:

```nginx
try_files $uri $uri/ =404;
```

instead of forwarding requests to the PHP front controller.

## Resolution

Update the configuration.

```nginx
location / {
    try_files $uri $uri/ /index.php?$args;
}
```

Reload Nginx.

```bash
sudo systemctl reload nginx
```

---

# Issue 13 — Incorrect Nginx Listen Port

## Symptoms

The frontend was only available on port **8080**.

## Root Cause

The Zabbix virtual host contained:

```nginx
listen 8080;
```

## Resolution

Remove the custom listen directive and allow the server to use the standard HTTP configuration on port **80**.

---

# Issue 14 — PHP Socket Verification

## Symptoms

PHP pages were not processed correctly.

## Diagnosis

```bash
ls /var/run/php/
```

Verify the socket configured in Nginx.

```nginx
fastcgi_pass unix:/var/run/php/zabbix.sock;
```

## Resolution

Ensure that the configured socket matches the socket created by PHP-FPM.

---

# Issue 15 — Incorrect FastCGI Document Root

## Symptoms

PHP files could not be executed.

## Root Cause

The document root did not match the Zabbix frontend directory.

## Resolution

Use:

```nginx
fastcgi_param DOCUMENT_ROOT /usr/share/zabbix;

fastcgi_param SCRIPT_FILENAME /usr/share/zabbix$fastcgi_script_name;

fastcgi_param PATH_TRANSLATED /usr/share/zabbix$fastcgi_script_name;
```

---

# Diagnostic Commands

The following commands proved useful during troubleshooting.

## Storage

```bash
lsblk -f
```

```bash
blkid
```

```bash
pvs
```

```bash
vgs
```

```bash
lvs
```

```bash
df -h
```

---

## PostgreSQL

```bash
sudo -u postgres psql -l
```

```bash
sudo -u postgres psql
```

```sql
\dt
```

---

## Zabbix

```bash
systemctl status zabbix-server
```

```bash
journalctl -xeu zabbix-server
```

```bash
tail -100 /var/log/zabbix/zabbix_server.log
```

---

## Nginx

```bash
sudo nginx -t
```

```bash
sudo nginx -T
```

```bash
systemctl status nginx
```

---

## PHP

```bash
systemctl status php8.1-fpm
```

```bash
ls /var/run/php/
```

---

# Lessons Learned

* Read the application log before changing configuration files.
* Validate one layer at a time (Storage → Database → Zabbix → PHP → Nginx).
* Most service failures are secondary symptoms rather than the root cause.
* Verify ownership after importing any database schema.
* Always validate `/etc/fstab` before rebooting.
* Use UUIDs instead of device names.
* Test every configuration change before moving to the next step.
* Keep a record of every error and its resolution; it becomes an invaluable operational knowledge base.

---

# Final Thoughts

Deploying Zabbix is more than installing packages. A reliable production deployment requires understanding how storage, databases, web services, PHP, and the monitoring server interact.

The troubleshooting experience documented here demonstrates that systematic diagnosis is more valuable than trial-and-error. By identifying the true root cause, validating assumptions, and verifying each layer independently, complex deployment issues can be resolved efficiently and with confidence.
