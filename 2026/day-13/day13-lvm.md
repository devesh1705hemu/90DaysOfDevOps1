# Day 13 – Linux Volume Management (LVM)

## Objective

Learn how to manage Linux storage flexibly using **LVM (Logical Volume Management)**.

By the end of this task, I learned how to:

* Create a Physical Volume (PV)
* Create a Volume Group (VG)
* Create a Logical Volume (LV)
* Format and mount an LV
* Extend an existing Logical Volume
* Resize the filesystem after extending the LV

---

# LVM Architecture

```text
Physical Disk
     │
     ▼
Physical Volume (PV)
     │
     ▼
Volume Group (VG)
     │
     ▼
Logical Volume (LV)
     │
     ▼
Filesystem
     │
     ▼
Mount Point
```

### Example Used

```text
/dev/nvme1n1
      │
      ▼
Physical Volume
      │
      ▼
devops-vg
      │
      ▼
app-data
      │
      ▼
ext4
      │
      ▼
/mnt/app-data
```

---

# Before You Start

## Switch to Root User

Option 1:

```bash
sudo -i
```

Option 2:

```bash
sudo su
```

Check the current user:

```bash
whoami
```

Expected output:

```text
root
```

---

# If You Don't Have a Spare Disk

A virtual disk can be created using a disk image.

## Create a 1 GB Disk Image

```bash
dd if=/dev/zero of=/tmp/disk1.img bs=1M count=1024
```

### Explanation

| Option              | Meaning               |
| ------------------- | --------------------- |
| `if=/dev/zero`      | Input source of zeros |
| `of=/tmp/disk1.img` | Output disk image     |
| `bs=1M`             | Block size = 1 MB     |
| `count=1024`        | Create 1024 blocks    |

## Attach Image as Loop Device

```bash
losetup -fP /tmp/disk1.img
```

Check the assigned loop device:

```bash
losetup -a
```

Example:

```text
/dev/loop5: ... /tmp/disk1.img
```

In this case, the device is:

```text
/dev/loop5
```

---

# Task 1 – Check Current Storage

## List Block Devices

```bash
lsblk
```

My EC2 instance showed:

```text
nvme0n1      26G  disk
├─nvme0n1p1  24.9G part /
├─nvme0n1p13 1023M part /boot
├─nvme0n1p14 4M part
└─nvme0n1p15 106M part /boot/efi

nvme1n1      10G  disk
```

The additional unused disk is:

```text
/dev/nvme1n1
```

> Important: `/dev/nvme0n1` contains the operating system. Do not use it for `pvcreate`.

---

## Check Physical Volumes

```bash
pvs
```

Detailed information:

```bash
pvdisplay
```

---

## Check Volume Groups

```bash
vgs
```

Detailed information:

```bash
vgdisplay
```

---

## Check Logical Volumes

```bash
lvs
```

Detailed information:

```bash
lvdisplay
```

---

## Check Filesystem Disk Usage

```bash
df -h
```

---

# Task 2 – Create Physical Volume

For my EC2 instance, the spare disk is:

```text
/dev/nvme1n1
```

Create the Physical Volume:

```bash
pvcreate /dev/nvme1n1
```

Expected output:

```text
Physical volume "/dev/nvme1n1" successfully created.
```

Verify:

```bash
pvs
```

Detailed verification:

```bash
pvdisplay
```

### What is a Physical Volume?

A **Physical Volume (PV)** is a physical disk or partition prepared for use by LVM.

Example:

```text
/dev/nvme1n1
      ↓
     PV
```

---

# Task 3 – Create Volume Group

Create a Volume Group named `devops-vg`:

```bash
vgcreate devops-vg /dev/nvme1n1
```

Verify:

```bash
vgs
```

Detailed information:

```bash
vgdisplay devops-vg
```

### What is a Volume Group?

A **Volume Group (VG)** combines one or more Physical Volumes into a storage pool.

```text
/dev/nvme1n1
      ↓
Physical Volume
      ↓
   devops-vg
```

---

# Task 4 – Create Logical Volume

Create a 500 MB Logical Volume:

```bash
lvcreate -L 500M -n app-data devops-vg
```

Check the Logical Volume:

```bash
lvs
```

Detailed information:

```bash
lvdisplay /dev/devops-vg/app-data
```

The Logical Volume path is:

```text
/dev/devops-vg/app-data
```

### What is a Logical Volume?

An **LV** is a virtual block device created from free space inside a Volume Group.

```text
devops-vg
    │
    └── app-data
          500M
```

---

# Task 5 – Format and Mount

## Format the Logical Volume

Create an ext4 filesystem:

```bash
mkfs.ext4 /dev/devops-vg/app-data
```

> Warning: `mkfs` formats the device and can destroy existing data. Only run it on the newly created LV.

---

## Create Mount Directory

```bash
mkdir -p /mnt/app-data
```

---

## Mount the Logical Volume

```bash
mount /dev/devops-vg/app-data /mnt/app-data
```

---

## Verify Mount

```bash
df -h /mnt/app-data
```

Also check:

```bash
lsblk
```

You should see the LV mounted at:

```text
/mnt/app-data
```

---

# Task 6 – Extend the Logical Volume

Initially, the LV is:

```text
500M
```

Extend it by 200 MB:

```bash
lvextend -L +200M /dev/devops-vg/app-data
```

Check the new LV size:

```bash
lvs
```

---

# Resize the Filesystem

For an ext4 filesystem:

```bash
resize2fs /dev/devops-vg/app-data
```

Check the final size:

```bash
df -h /mnt/app-data
```

The filesystem should now be approximately:

```text
700M
```

---

# Important LVM Commands Cheat Sheet

## Physical Volume Commands

### Create PV

```bash
pvcreate /dev/nvme1n1
```

### List PVs

```bash
pvs
```

### Detailed PV information

```bash
pvdisplay
```

### Remove PV

```bash
pvremove /dev/nvme1n1
```

---

## Volume Group Commands

### Create VG

```bash
vgcreate devops-vg /dev/nvme1n1
```

### List VGs

```bash
vgs
```

### Detailed VG information

```bash
vgdisplay
```

### Extend VG with another disk

```bash
vgextend devops-vg /dev/nvme2n1
```

### Remove VG

```bash
vgremove devops-vg
```

---

## Logical Volume Commands

### Create LV

```bash
lvcreate -L 500M -n app-data devops-vg
```

### List LVs

```bash
lvs
```

### Detailed LV information

```bash
lvdisplay
```

### Extend LV

```bash
lvextend -L +200M /dev/devops-vg/app-data
```

### Extend LV to a specific size

```bash
lvextend -L 1G /dev/devops-vg/app-data
```

### Remove LV

```bash
lvremove /dev/devops-vg/app-data
```

---

# Filesystem Commands

## Create ext4 Filesystem

```bash
mkfs.ext4 /dev/devops-vg/app-data
```

## Create Mount Directory

```bash
mkdir -p /mnt/app-data
```

## Mount

```bash
mount /dev/devops-vg/app-data /mnt/app-data
```

## Unmount

```bash
umount /mnt/app-data
```

## Check Disk Usage

```bash
df -h
```

## Resize ext4 Filesystem

```bash
resize2fs /dev/devops-vg/app-data
```

---

# Complete Command Sequence

For my EC2 instance, the complete workflow is:

```bash
# Check disks
lsblk

# Create Physical Volume
sudo pvcreate /dev/nvme1n1

# Check PV
sudo pvs

# Create Volume Group
sudo vgcreate devops-vg /dev/nvme1n1

# Check VG
sudo vgs

# Create Logical Volume
sudo lvcreate -L 500M -n app-data devops-vg

# Check LV
sudo lvs

# Create filesystem
sudo mkfs.ext4 /dev/devops-vg/app-data

# Create mount directory
sudo mkdir -p /mnt/app-data

# Mount LV
sudo mount /dev/devops-vg/app-data /mnt/app-data

# Check mounted filesystem
df -h /mnt/app-data

# Extend LV by 200 MB
sudo lvextend -L +200M /dev/devops-vg/app-data

# Resize filesystem
sudo resize2fs /dev/devops-vg/app-data

# Verify final size
df -h /mnt/app-data
```

---

# Useful Verification Commands

Run these whenever you want to understand the current LVM state:

```bash
lsblk
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

For detailed information:

```bash
pvdisplay
vgdisplay
lvdisplay
```

---


# What I Learned

### 1. LVM provides flexible storage management

LVM allows storage to be organized into PVs, VGs, and LVs instead of managing partitions directly.

### 2. Logical Volumes can be extended

An existing Logical Volume can be increased without recreating it.

```bash
lvextend -L +200M /dev/devops-vg/app-data
```

The filesystem must also be resized when required:

```bash
resize2fs /dev/devops-vg/app-data
```

### 3. LVM separates physical storage from logical storage

```text
Physical Disk
      ↓
      PV
      ↓
      VG
      ↓
      LV
      ↓
Filesystem
      ↓
Mount Point
```

This makes storage management much more flexible for Linux and DevOps environments.

---

