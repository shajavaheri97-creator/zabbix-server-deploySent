# Project Overview

## Introduction

This project demonstrates the deployment of a production-style **Zabbix Monitoring Server** on **Ubuntu Server 22.04 LTS** using **PostgreSQL** as the backend database, **Nginx** as the web server, and **PHP-FPM** for processing the Zabbix frontend.

Unlike many installation guides that focus only on package installation, this repository documents the complete deployment lifecycle, including planning, storage design, service configuration, troubleshooting, and validation.

The goal is to provide both a deployment guide and a technical reference based on real-world implementation experience.

---

# Project Goals

The main objectives of this project are:

- Deploy a fully functional Zabbix Server.
- Configure PostgreSQL as the database backend.
- Store PostgreSQL data on a dedicated Logical Volume (LVM).
- Separate operating system storage from database storage.
- Configure persistent storage using UUID-based mounting.
- Configure Nginx and PHP-FPM to serve the Zabbix Web Interface.
- Verify that all services start automatically after system reboot.
- Document every issue encountered during deployment and explain its resolution.

---

# Why This Project?

Monitoring is one of the fundamental components of modern IT infrastructure.

Organizations rely on monitoring platforms to:

- Detect infrastructure failures
- Monitor server health
- Track service availability
- Generate alerts
- Collect historical metrics
- Improve incident response

Zabbix provides all these capabilities within a single open-source platform.

This project focuses on deploying Zabbix using infrastructure practices commonly found in enterprise environments.

---

# Design Decisions

Several architectural decisions were intentionally made during this deployment.

## Dedicated Database Storage

Instead of storing PostgreSQL data on the operating system partition, a separate disk was allocated for the database.

Benefits include:

- Better storage management
- Easier future expansion
- Isolation from operating system usage
- Simplified backup strategy

---

## Logical Volume Manager (LVM)

LVM was selected instead of directly partitioning the disk.

Advantages:

- Online storage expansion
- Flexible volume management
- Better administration
- Enterprise deployment practice

---

## PostgreSQL

PostgreSQL was selected because it is officially supported by Zabbix and offers:

- High reliability
- Excellent performance
- Strong data integrity
- Wide enterprise adoption

---

## Nginx

Nginx was selected as the web server because it provides:

- High performance
- Low memory usage
- Excellent FastCGI integration
- Simple reverse proxy capabilities

---

## PHP-FPM

PHP-FPM executes the PHP code used by the Zabbix frontend.

Its responsibilities include:

- Processing PHP requests
- Connecting the frontend to PostgreSQL
- Returning generated HTML pages to Nginx

---

# Deployment Environment

| Component | Specification |
|------------|---------------|
| Operating System | Ubuntu Server 22.04 LTS |
| Database | PostgreSQL |
| Monitoring Platform | Zabbix Server 7.2 |
| Web Server | Nginx |
| PHP Engine | PHP 8.1 FPM |
| Filesystem | ext4 |
| Storage Management | LVM2 |

---

# Project Scope

The scope of this repository includes:

- Operating system preparation
- LVM configuration
- Filesystem creation
- Database deployment
- Zabbix Server installation
- Web frontend configuration
- Service management using systemd
- Persistent filesystem mounting
- Deployment verification
- Troubleshooting

---

# Expected Outcome

After completing all documented steps, the deployment should provide:

- A running Zabbix Server service
- A configured PostgreSQL database
- A working Zabbix Web Interface
- Persistent database storage on LVM
- Automatic service startup
- A maintainable production-style monitoring environment

---

# Lessons Learned

During this deployment several real-world issues were encountered, including:

- Incorrect filesystem mounting
- Invalid UUID references inside `/etc/fstab`
- LVM mount failures
- Repository accessibility problems
- Missing PostgreSQL extensions
- Database ownership mistakes
- Zabbix schema import issues
- Permission errors on database tables
- Zabbix startup failures
- Nginx configuration problems
- Incorrect virtual host configuration
- PHP-FPM socket configuration issues

Each issue is fully documented later in the **Troubleshooting** section together with its root cause and solution.
## Key Takeaways

- Enterprise deployments benefit from separating database storage.
- LVM provides flexibility for future storage expansion.
- Proper service layering simplifies troubleshooting.
- Infrastructure documentation is as important as deployment itself.