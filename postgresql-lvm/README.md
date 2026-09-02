# PostgreSQL Storage with LVM

## Objective 
Configure a dedicated storage for PostgreSQL using LVM.

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
1 GB additional disk was added to the virtual machine for this task.

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
### Create the physical volume:
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
```bash
  PV         VG Fmt  Attr PSize    PFree
  /dev/sdb1     lvm2 ---  1020.00m 1020.00m 
```

### Create the volume group:
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
```bash
  VG            #PV #LV #SN Attr   VSize    VFree
  vg-postgresql   1   0   0 wz--n- 1020.00m 1020.00m 
```

### Create the logical volume:
```bash
sudo lvcreate -l 100%FREE -n lv-postgresql vg-postgresql
```
Output:
```text
Logical volume "lv-postgresql" created.
```
Verify logical volume:
```bash
sudo lvs
```
Output:
```bash
  LV            VG            Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv-postgresql vg-postgresql -wi-ao---- 1020.00m
```
## 3. Create Filesystem:
Create an ext4 filesystem on the logical volume:
```bash
sudo mkfs.ext4 /dev/vg-postgresql/lv-postgresql
```
## 4. Mount the Filesystem
Create the directory that will be used as the PostgreSQL data storage location:
```bash
sudo mkdir -p /var/lib/postgresql
```
### Mount the logical volume to the PostgreSQL data directory:
```bash
sudo mount /dev/vg-postgresql/lv-postgresql /var/lib/postgresql
```
Use `df` to verify that the filesystem is mounted correctly:
```bash
df -h
```
Output:
```text
Filesystem                                 Size  Used Avail Use% Mounted on
/dev/mapper/vg--postgresql-lv--postgresql  986M   63M  856M   7% /var/lib/postgresql
```
## 5. Configure /etc/fstab:
Get the UUID of filesystem:
```bash
sudo blkid /dev/vg-postgresql/lv-postgresql
```
### Add the UUID of filesystem to /etc/fstab for automatic mounting after reboot:
```bash
UUID=<filesystem-UUID>  /var/lib/postgresql  ext4  defaults  0  2
```
## 6. Install PostgreSQL
After preparing the storage, install PostgreSQL:
```bash
sudo apt install postgresql
```
It will use /var/lib/postgresql as it's data storage location.
## NOTE: Existing PostgreSQL installation: 
If PostgreSQL is already installed and contains data, stop it and create a backup of /var/lib/postgresql before mounting the new filesystem.
After mounting, restore the backup to /var/lib/postgresql.







