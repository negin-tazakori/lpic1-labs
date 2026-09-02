# Postgresql Storage with LVM

## Objective 
Configure a dedicated storage for Postgresql using LVM.

## Storage architecture
``` test
Disk
  ↓
Partition
  ↓
Physical Volume (PV)
  ↓
Volume Group (VG)
  ↓
Logical Volume (LV)
  ↓
ext4
  ↓
/var/lib/postgresql
```

## 1. Create Partition
A separate disk was added to the virtual machine.

The disk was partitioned using fdisk:
```bash
sudo fdisk /dev/sdb
```
inside fdisk:
```text
g
n
1
Enter
Enter
l
t
w
```

- `g` → new GPT partition table
- `n` → new partition
- `1` → partition number
- `Enter` → default first sector
- `Enter` → default last sector (whole disk)
- `l` → list of partition type
- `t` → change the partition type to Linux LVM (44)
- `w` → write the changes and exit

Check the new partition with lsblk:
```bash
lsblk
```

## 2. Create LVM
Create the physical volume:
```bash
sudo pvcreate /dev/sdb1
```
Output:
```text
Physical volume "/dev/sdb1" successfully created.
```
Verify physical volume:
```bash
sudo pvs
```
Output:


Create the volume group:
```bash
sudo vgcreate vg-postgresql /dev/sdb1
```
Output:
```text
Volume group "vg-postgresql" successfully created.
```

Verify volume group:
```bash
sudo vgs
```
Output:





