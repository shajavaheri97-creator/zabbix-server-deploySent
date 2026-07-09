# LVM Configuration

## Overview

This document describes the complete storage configuration process for the PostgreSQL database using **Logical Volume Manager (LVM)**.

Instead of storing the database files on the operating system partition, a dedicated disk is configured as an LVM volume and mounted to PostgreSQL's data directory.

Using LVM provides flexibility, scalability, and easier storage management for production environments.

---

# Objectives

The objectives of this configuration are:

* Separate database storage from the operating system.
* Create a dedicated Logical Volume for PostgreSQL.
* Format the Logical Volume with the ext4 filesystem.
* Mount the filesystem permanently using UUID.
* Verify storage persistence after reboot.

---

# Initial Storage Layout

Before configuration:

```text
/dev/sda
├── /boot
└── /

/dev/sdb
└── Unallocated Disk
```

After configuration:

```text
/dev/sda
├── /boot
└── /

/dev/sdb
└── Physical Volume
    └── Volume Group (vg_data)
        └── Logical Volume (lv_pgsql)
            └── ext4
                └── /var/lib/postgresql
```

---

# Step 1 - Verify Available Disks

Before creating the LVM structure, identify the available disks.

Command:

```bash
lsblk -f
```

Why?

This command displays:

* Existing disks
* Mounted filesystems
* Available block devices
* Filesystem types
* UUID information

Verification:

Ensure that the new disk is available and not mounted.

---

# Step 2 - Create Physical Volume

Create a Physical Volume on the dedicated disk.

Command:

```bash
sudo pvcreate /dev/sdb
```

Why?

A Physical Volume (PV) is the first building block of LVM.

Verification:

```bash
pvs
```

Expected output:

* Physical Volume name
* Volume size
* Free space

---

# Step 3 - Create Volume Group

Create a Volume Group using the new Physical Volume.

Command:

```bash
sudo vgcreate vg_data /dev/sdb
```

Why?

The Volume Group combines one or more Physical Volumes into a storage pool.

Verification:

```bash
vgs
```

Expected output should include:

* Volume Group name
* Total size
* Free space

---

# Step 4 - Create Logical Volume

Create the Logical Volume that will store PostgreSQL data.

Command:

```bash
sudo lvcreate -L 14G -n lv_pgsql vg_data
```

Why?

The Logical Volume behaves like a virtual disk.

Verification:

```bash
lvs
```

Expected output:

* LV Name
* VG Name
* LV Size

---

# Step 5 - Create Filesystem

Format the Logical Volume.

Command:

```bash
sudo mkfs.ext4 /dev/vg_data/lv_pgsql
```

Why?

A Logical Volume cannot store files until a filesystem is created.

Verification:

```bash
lsblk -f
```

The Logical Volume should display:

Filesystem: ext4

---

# Step 6 - Create Mount Point

Create PostgreSQL's storage directory.

Command:

```bash
sudo mkdir -p /var/lib/postgresql
```

Why?

Linux requires an existing directory before mounting a filesystem.

Verification:

```bash
ls -ld /var/lib/postgresql
```

---

# Step 7 - Mount the Logical Volume

Mount the filesystem.

Command:

```bash
sudo mount /dev/vg_data/lv_pgsql /var/lib/postgresql
```

Verification:

```bash
mount | grep postgresql
```

or

```bash
df -h
```

---

# Step 8 - Configure Persistent Mount

Using device names such as:

```
/dev/sdb
```

is not recommended because device names may change after reboot.

Instead, retrieve the filesystem UUID.

Command:

```bash
blkid
```

Example:

```text
UUID="347586ee-129b-4ae4-8d18-157e39497904"
```

Edit the fstab file.

```bash
sudo nano /etc/fstab
```

Add:

```text
UUID=347586ee-129b-4ae4-8d18-157e39497904   /var/lib/postgresql   ext4   defaults   0   2
```

---

# Step 9 - Validate fstab

Never reboot before testing the fstab configuration.

Command:

```bash
sudo mount -a
```

If no errors are returned, the configuration is valid.

Verification:

```bash
df -h
```

```bash
mount | grep postgresql
```

---

# Step 10 - Verify Storage Layout

Display the final storage layout.

```bash
lsblk -f
```

Expected structure:

```text
sdb
└── vg_data-lv_pgsql
      ext4
      /var/lib/postgresql
```

---

# Best Practices

* Use UUID instead of device names.
* Keep PostgreSQL on a dedicated Logical Volume.
* Always verify fstab before rebooting.
* Create the filesystem before mounting.
* Store database files separately from the operating system.

---

# Real Deployment Issues

This deployment encountered several real-world storage issues.

## Issue 1 — Wrong Filesystem Type

### Symptom

```text
wrong fs type, bad superblock
```

### Root Cause

The Logical Volume had not been formatted or the wrong block device was mounted.

### Resolution

Create the ext4 filesystem on the Logical Volume:

```bash
mkfs.ext4 /dev/vg_data/lv_pgsql
```

---

## Issue 2 — Wrong UUID in /etc/fstab

### Symptom

```text
can't find UUID
```

### Root Cause

The UUID of the Physical Volume (LVM2_member) was mistakenly used instead of the UUID of the ext4 filesystem.

### Resolution

Retrieve the correct filesystem UUID using:

```bash
blkid
```

Update `/etc/fstab` with the ext4 UUID.

---

## Issue 3 — Mounting the Wrong Device

### Symptom

```text
wrong fs type
```

### Root Cause

The Physical Disk (`/dev/sdb`) was mounted instead of the Logical Volume (`/dev/vg_data/lv_pgsql`).

### Resolution

Mount the Logical Volume rather than the underlying disk.

---

## Issue 4 — Missing Filesystem

### Symptom

The mount command failed even though the Logical Volume existed.

### Root Cause

The Logical Volume had been created but no filesystem had been initialized.

### Resolution

Create an ext4 filesystem using `mkfs.ext4`.

---

## Issue 5 — Invalid fstab Entry

### Symptom

```text
mount -a
```

returned errors.

### Root Cause

Incorrect filesystem type, invalid UUID, or incorrect mount path.

### Resolution

Review the `/etc/fstab` entry, verify the UUID using `blkid`, and validate the configuration with `mount -a`.

---

## Issue 6 — systemd Cache Warning

### Symptom

```text
your fstab has been modified,
but systemd still uses the old version
```

### Root Cause

systemd had not reloaded its mount configuration after changes to `/etc/fstab`.

### Resolution

Reload the systemd manager configuration:

```bash
sudo systemctl daemon-reload
```

---

# Key Takeaways

* LVM provides flexible and scalable storage management.
* Always format a Logical Volume before mounting it.
* Never use the Physical Volume UUID in `/etc/fstab`.
* Always use the filesystem UUID for persistent mounts.
* Validate `/etc/fstab` with `mount -a` before rebooting.
* Separating database storage from the operating system is considered a production best practice.
