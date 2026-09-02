# Postgresql Storage with LVM

## Objective 
Configure a separate disk using LVM and use it as the storage location for Postgresql data.

## Stprage architecture
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
/var/lib/mongodb
```
