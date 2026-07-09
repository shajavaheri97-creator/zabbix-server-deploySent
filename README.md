# Zabbix Server Deployment on Ubuntu 22.04 LTS

A complete deployment guide for **Zabbix Server 7.2** on **Ubuntu Server 22.04 LTS (Jammy Jellyfish)** using **PostgreSQL**, **Nginx**, **PHP-FPM**, and a dedicated **LVM volume** for database storage.

Unlike many installation guides, this repository focuses not only on deployment but also on **real-world troubleshooting** encountered during the implementation process.

---

## Project Objectives

The primary objectives of this project are:

- Deploy Zabbix Server 7.2 on Ubuntu Server 22.04.
- Store PostgreSQL data on a dedicated Logical Volume (LVM).
- Configure persistent storage using UUID-based mounting.
- Configure Nginx and PHP-FPM for serving the Zabbix Web Interface.
- Deploy a fully functional monitoring platform.
- Document every deployment challenge and its solution.

---

## Technology Stack

| Component | Version |
|------------|----------|
| Ubuntu Server | 22.04 LTS |
| Zabbix Server | 7.2 |
| PostgreSQL | 16 |
| Nginx | Latest Ubuntu Package |
| PHP | 8.1 FPM |
| Filesystem | ext4 |
| Volume Manager | LVM2 |

---

## Project Architecture

```text
                         +----------------+
                         |    Browser     |
                         +-------+--------+
                                 |
                                 |
                          HTTP / HTTPS
                                 |
                                 ▼
                       +------------------+
                       |      Nginx       |
                       +--------+---------+
                                |
                                | FastCGI
                                ▼
                       +------------------+
                       |     PHP-FPM      |
                       +--------+---------+
                                |
                                |
                                ▼
                  +----------------------------+
                  |  Zabbix Frontend (PHP UI)  |
                  +-------------+--------------+
                                |
                                |
                                ▼
                     +------------------------+
                     |     PostgreSQL DB      |
                     +------------+-----------+
                                  ▲
                                  |
                          Database Queries
                                  |
                     +------------+-----------+
                     |     Zabbix Server      |
                     +------------+-----------+
                                  ▲
                                  |
                          Agent / SNMP / API
                                  |
                     +------------+-----------+
                     |    Monitored Hosts     |
                     +------------------------+
```

---

## Repository Structure

```
zabbix-server-deployment/

├── README.md
├── docs/
├── configs/
├── scripts/
├── LICENSE
└── .gitignore
```

---

## Documentation

Detailed documentation is available in the **docs** directory.

| Document | Description |
|----------|-------------|
| 01-Project-Overview | Project description |
| 02-Architecture | Infrastructure architecture |
| 03-Prerequisites | Required software and hardware |
| 04-LVM-Configuration | Creating and mounting the database volume |
| 05-PostgreSQL-Configuration | Database installation and configuration |
| 06-Zabbix-Installation | Zabbix installation |
| 07-Nginx-PHP-FPM | Web interface configuration |
| 08-Troubleshooting | Real deployment issues and solutions |

---

## Features

- Dedicated LVM volume for PostgreSQL
- Persistent mounting using UUID
- PostgreSQL backend
- Nginx web server
- PHP-FPM integration
- Systemd service management
- Production-style deployment
- Comprehensive troubleshooting guide

---

## Lessons Learned

This project demonstrates practical experience in deploying a production-style monitoring platform while resolving real infrastructure issues including:

- Filesystem mounting failures
- LVM configuration mistakes
- PostgreSQL permission problems
- Incorrect database ownership
- Missing database schema
- Zabbix startup failures
- Nginx routing issues
- PHP-FPM configuration problems
- Repository accessibility issues
- Service debugging using systemd and log analysis

---

## License

This project is published for educational purposes.