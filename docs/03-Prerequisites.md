# Prerequisites

## Overview

Before deploying the Zabbix monitoring platform, ensure that the target environment meets the minimum hardware, software, storage, and network requirements described in this document.

This deployment is based on **Ubuntu Server 22.04 LTS (Jammy Jellyfish)** and follows production-oriented practices, including dedicated database storage, Logical Volume Management (LVM), and service separation.

---

# Hardware Requirements

| Component     | Minimum Requirement        | Recommended      |
| ------------- | -------------------------- | ---------------- |
| CPU           | 2 vCPUs                    | 4 vCPUs          |
| Memory        | 4 GB RAM                   | 8 GB RAM         |
| System Disk   | 20 GB                      | 40 GB            |
| Database Disk | 15 GB                      | 50 GB+           |
| Network       | Stable Internet Connection | Gigabit Ethernet |

---

# Operating System

The deployment described in this repository was performed using:

| Component        | Version                 |
| ---------------- | ----------------------- |
| Operating System | Ubuntu Server 22.04 LTS |
| Architecture     | x86_64                  |
| Init System      | systemd                 |

Verify the operating system version:

```bash
lsb_release -a
```

---

# Software Requirements

Install or verify the following packages before starting the deployment:

* PostgreSQL
* Nginx
* PHP 8.1 FPM
* PHP PostgreSQL Extension
* LVM2
* Git
* curl
* wget
* unzip

Useful verification commands:

```bash
nginx -v
```

```bash
php -v
```

```bash
psql --version
```

```bash
systemctl --version
```

---

# Storage Requirements

A dedicated disk is recommended for PostgreSQL.

Example storage layout:

```text
/dev/sda
├── /
└── /boot

/dev/sdb
└── Volume Group (vg_data)
    └── Logical Volume (lv_pgsql)
        └── ext4
            └── /var/lib/postgresql
```

Using a dedicated logical volume improves maintainability, storage scalability, and backup management.

---

# Network Requirements

The following ports should be available.

| Port  | Service       | Purpose                         |
| ----- | ------------- | ------------------------------- |
| 22    | SSH           | Remote administration           |
| 80    | HTTP          | Zabbix Web Interface            |
| 443   | HTTPS         | Secure Web Interface (Optional) |
| 10050 | Zabbix Agent  | Agent Communication             |
| 10051 | Zabbix Server | Server Communication            |
| 5432  | PostgreSQL    | Database Service                |

---

# User Accounts

The deployment relies on several system accounts.

| User     | Purpose               |
| -------- | --------------------- |
| root     | System administration |
| postgres | PostgreSQL management |
| zabbix   | Zabbix Server service |
| www-data | Nginx/PHP web service |

---

# Important Directories

| Directory           | Description                 |
| ------------------- | --------------------------- |
| /etc/zabbix         | Zabbix configuration files  |
| /usr/share/zabbix   | Zabbix Web Frontend         |
| /var/log/zabbix     | Zabbix log files            |
| /var/lib/postgresql | PostgreSQL database storage |
| /etc/nginx          | Nginx configuration         |
| /etc/php            | PHP configuration           |
| /run/php            | PHP-FPM socket location     |

---

# Required Services

The following services are required for a successful deployment.

| Service               | Description                 |
| --------------------- | --------------------------- |
| postgresql.service    | Database engine             |
| zabbix-server.service | Zabbix monitoring server    |
| php8.1-fpm.service    | PHP FastCGI Process Manager |
| nginx.service         | Web server                  |

Verify service status:

```bash
systemctl status postgresql
```

```bash
systemctl status nginx
```

```bash
systemctl status php8.1-fpm
```

```bash
systemctl status zabbix-server
```

---

# Initial Environment Verification

Before beginning the installation, verify the storage and system configuration.

Check available disks:

```bash
lsblk -f
```

Display Volume Groups:

```bash
vgs
```

Display Logical Volumes:

```bash
lvs
```

Check mounted filesystems:

```bash
df -h
```

Verify filesystem UUIDs:

```bash
blkid
```

---

# Best Practices

To ensure a stable deployment, follow these recommendations:

* Use UUIDs instead of device names in `/etc/fstab`.
* Allocate a dedicated disk for PostgreSQL data.
* Store database files on an LVM logical volume.
* Verify every configuration change before restarting services.
* Keep system packages updated.
* Document all configuration changes during deployment.

---

# Key Takeaways

* Verify the operating system before deployment.
* Separate database storage from the operating system whenever possible.
* Ensure all required services and packages are installed.
* Validate storage, networking, and service readiness before beginning the installation.
* Proper preparation significantly reduces troubleshooting during deployment.
