# PostgreSQL Configuration

## Overview

This document describes the PostgreSQL configuration required for deploying Zabbix Server.

The database backend is the most critical component of a Zabbix installation. During this deployment, several real-world issues were encountered, including incorrect database ownership, schema import mistakes, permission problems, and startup failures.

This document explains not only the installation process but also the troubleshooting steps that were required to achieve a successful deployment.

---

# Architecture

The deployment uses the following architecture:

```text
Zabbix Server
      │
      │
      ▼
PostgreSQL Database
      │
      ▼
Database: zabbix
Owner: zabbix
```

---

# Verify PostgreSQL Installation

Ensure PostgreSQL is installed and running.

```bash
systemctl status postgresql
```

Expected result:

* Active (running)

Verify PostgreSQL version:

```bash
psql --version
```

---

# Create Database User

Create a dedicated database user for Zabbix.

```bash
sudo -u postgres createuser --pwprompt zabbix
```

Why?

Running Zabbix using the PostgreSQL superuser is not recommended.

A dedicated database role follows the principle of least privilege.

---

# Create Database

```bash
sudo -u postgres createdb \
-o zabbix \
-E UTF8 \
zabbix
```

Explanation:

* Owner → zabbix
* Encoding → UTF8

Verify:

```bash
sudo -u postgres psql -l
```

Expected output:

```
zabbix
```

---

# Import Zabbix Schema

Load the initial database schema.

```bash
zcat /usr/share/zabbix/sql-scripts/postgresql/server.sql.gz | \
sudo -u zabbix psql zabbix
```

This command creates:

* Tables
* Views
* Triggers
* Indexes
* Default Data

---

# Configure Zabbix Server

Edit:

```text
/etc/zabbix/zabbix_server.conf
```

Update:

```text
DBHost=localhost

DBName=zabbix

DBUser=zabbix

DBPassword=<your_password>
```

Restart PostgreSQL.

```bash
sudo systemctl restart postgresql
```

Restart Zabbix.

```bash
sudo systemctl restart zabbix-server
```

---

# Verification

Check database connectivity.

```bash
sudo -u zabbix psql zabbix
```

Verify service status.

```bash
systemctl status zabbix-server
```

View logs.

```bash
tail -100 /var/log/zabbix/zabbix_server.log
```

---

# Real Deployment Issues

The following issues were encountered during the deployment.

---

## Issue 1 — Database Created with the Wrong Name

### Symptom

The installation process completed, but the Zabbix server could not connect to the database.

### Root Cause

A database named **zabbix2** was created instead of **zabbix**.

### Resolution

The incorrect database was removed.

A new database named **zabbix** was created with the correct owner.

---

## Issue 2 — Multiple Database Roles

### Symptom

Several PostgreSQL roles existed:

```
postgres

zabbix

zabbix2
```

This caused confusion during the installation.

### Resolution

Unused roles and databases were removed.

The deployment continued using only:

* Database: zabbix
* User: zabbix

---

## Issue 3 — Schema Imported as postgres

### Symptom

The Zabbix server failed immediately after startup.

Log:

```
permission denied for table users
```

```
cannot use database "zabbix":
database is not a Zabbix database
```

### Root Cause

The database schema had been imported using the PostgreSQL superuser.

As a result:

* every table
* every sequence
* every index

was owned by **postgres** instead of **zabbix**.

---

### Investigation

Table ownership was verified.

```sql
SELECT tablename,
tableowner
FROM pg_tables
WHERE schemaname='public';
```

Result:

```
users

Owner:

postgres
```

This immediately explained why the Zabbix service could not access its own tables.

---

### Resolution

The database was completely removed.

A new database was created.

The schema was imported again using the correct user:

```bash
zcat server.sql.gz | sudo -u zabbix psql zabbix
```

After recreating the database, every table became owned by **zabbix**.

---

## Issue 4 — REASSIGN OWNED Failed

Attempt:

```sql
REASSIGN OWNED BY postgres TO zabbix;
```

Result:

```
cannot reassign ownership
```

Reason:

System objects owned by PostgreSQL cannot be reassigned.

Resolution:

Instead of trying to repair the existing database, the entire database was recreated.

This was significantly faster and cleaner.

---

## Issue 5 — Zabbix Service Failed

Service:

```
systemctl status zabbix-server
```

Returned:

```
Failed with result 'protocol'
```

Root Cause

The database initialization had failed because of incorrect table ownership.

Once the database was recreated correctly, the service started successfully.

---

## Issue 6 — PID File Error

Systemd reported:

```
Can't open PID file
```

Although this looked like a service issue, it was only a consequence of the server exiting immediately after failing to initialize the database.

The actual problem was the database configuration.

---

# Lessons Learned

Several important lessons were learned during this deployment.

• Always create the database before importing the schema.

• Always import the schema using the database owner.

• Never import the schema as postgres.

• Verify table ownership whenever database permission errors appear.

• Database recreation is often faster than attempting to repair an incorrectly initialized database.

• Read the Zabbix server log before changing configuration files.

• Many apparent systemd errors are actually caused by PostgreSQL initialization failures.

---

# Verification Checklist

Before continuing with the web frontend installation, verify the following:

✅ PostgreSQL service is running.

✅ Database "zabbix" exists.

✅ Database owner is "zabbix".

✅ All tables are owned by "zabbix".

✅ Zabbix Server starts successfully.

✅ No database permission errors appear in the log.

---

# Key Takeaways

Database ownership is one of the most common causes of failed Zabbix deployments.

Although the symptoms may appear to be related to systemd or the Zabbix service itself, the root cause is often an incorrectly initialized PostgreSQL database.

Creating the database with the correct owner and importing the schema using that same owner eliminates an entire class of permission-related issues and results in a clean, maintainable deployment.
