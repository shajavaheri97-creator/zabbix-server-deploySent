# Zabbix Installation

## Overview

This document describes the complete installation process of **Zabbix Server 7.2** on **Ubuntu Server 22.04 LTS** using **PostgreSQL**, **Nginx**, and **PHP-FPM**.

The deployment follows the official package-based installation method while documenting several real-world issues encountered during the implementation.

---

# Software Stack

| Component  | Version               |
| ---------- | --------------------- |
| Ubuntu     | 22.04 LTS             |
| PostgreSQL | 16                    |
| PHP        | 8.1 FPM               |
| Nginx      | Latest Ubuntu Package |
| Zabbix     | 7.2                   |

---

# Update Package Repository

```bash
sudo apt update
sudo apt upgrade -y
```

### Why?

Updating package indexes ensures that the latest package metadata is available before installing Zabbix.

---

# Install Required Packages

```bash
sudo apt install \
postgresql \
nginx \
php8.1-fpm \
php8.1-pgsql \
curl \
wget \
unzip -y
```

### Verify

```bash
systemctl status postgresql
systemctl status nginx
systemctl status php8.1-fpm
```

All services should be active.

---

# Add the Zabbix Repository

Download the official repository package.

```bash
wget https://repo.zabbix.com/zabbix/7.2/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.2+ubuntu22.04_all.deb
```

Install it.

```bash
sudo dpkg -i zabbix-release_latest_7.2+ubuntu22.04_all.deb
```

Update repositories.

```bash
sudo apt update
```

---

# Install Zabbix Components

```bash
sudo apt install \
zabbix-server-pgsql \
zabbix-frontend-php \
zabbix-nginx-conf \
zabbix-sql-scripts \
zabbix-agent -y
```

---

# Verify Installed Packages

```bash
dpkg -l | grep zabbix
```

Expected packages include:

* zabbix-server-pgsql
* zabbix-agent
* zabbix-frontend-php
* zabbix-nginx-conf
* zabbix-sql-scripts
* zabbix-release

---

# Verify Installation Files

Configuration files:

```text
/etc/zabbix/

/etc/zabbix/zabbix_server.conf

/etc/zabbix/nginx.conf
```

Web files:

```text
/usr/share/zabbix
```

Service:

```text
zabbix-server.service
```

---

# Verify Service Availability

```bash
systemctl status zabbix-server
```

At this stage, the service is expected to fail because the database has not yet been configured.

This behavior is normal.

---

# Real Deployment Issues

## Issue 1 — Repository Access Problems

### Symptom

The official Zabbix repository returned HTTP 404 errors.

### Root Cause

Repository access was blocked due to network restrictions.

### Resolution

The repository package was downloaded manually and transferred to the server before installation.

---

## Issue 2 — Unsupported Ubuntu Release

### Symptom

The system was initially running Ubuntu 26.04 (Resolute), which was not supported by the official Zabbix repository.

### Resolution

The operating system was downgraded to Ubuntu 22.04 LTS (Jammy Jellyfish), which is officially supported.

---

## Issue 3 — Repository Mirror Configuration

An ArvanCloud mirror was configured for Ubuntu packages to improve package download reliability.

---

## Verification Checklist

Verify that:

* Repository package is installed.
* All Zabbix packages are installed.
* Configuration files exist.
* Service unit has been created.
* Package installation completed without errors.

---

# Key Takeaways

* Always install Zabbix using the official repository whenever possible.
* Verify package installation before configuring the database.
* Ubuntu version compatibility is critical.
* Repository connectivity issues can be mitigated by manually downloading the repository package.
* Service startup failures at this stage are expected until database initialization is completed.
