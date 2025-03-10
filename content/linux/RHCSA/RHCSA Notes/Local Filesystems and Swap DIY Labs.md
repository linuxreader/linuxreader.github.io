## Local Filesystems and Swap DIY Labs 

### Lab: Create VFAT, Ext4, and XFS File Systems in Partitions and Mount Persistently 

- Create three 70MB primary partitions on one of the available 250MB disks (lsblk) by invoking the parted utility directly at the command prompt. 
```bash
[root@server2 mapper]# parted /dev/sdc mklabel msdos
Information: You may need to update /etc/fstab.

[root@server2 mapper]# parted /dev/sdc mkpart primary 1 70m
Information: You may need to update /etc/fstab.

root@server2 mapper]# parted /dev/sdb print
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  70.3MB  69.2MB  primary

```

```bash
parted) mkpart primary 71MB 140MB                                    
Warning: The resulting partition is not properly aligned for best performance: 138671s % 2048s != 0s
Ignore/Cancel?                                                            
Ignore/Cancel? ignore                                                     
(parted) mkpart primary 140MB 210MB
Warning: The resulting partition is not properly aligned for best performance: 273438s % 2048s != 0s
Ignore/Cancel? ignore                                                     
(parted) print                                                            
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  70.3MB  69.2MB  primary
 2      71.0MB  140MB   69.0MB  primary
 3      140MB   210MB   70.0MB  primary
```

- Apply label "msdos" if the disk is new.
- Initialize partition 1 with VFAT, partition 2 with Ext4, and partition 3 with XFS file system types. 
```bash
[root@server2 mapper]# sudo mkfs -t vfat /dev/sdc1
mkfs.fat 4.2 (2021-01-31)

[root@server2 mapper]# sudo mkfs -t ext4 /dev/sdc2
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 67380 1k blocks and 16848 inodes
Filesystem UUID: 43b590ff-3330-4b88-aef9-c3a97d8cf51e
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

[root@server2 mapper]# sudo mkfs -t xfs /dev/sdc3
Filesystem should be larger than 300MB.
Log size should be at least 64MB.
Support for filesystems like this one is deprecated and they will not be supported in future releases.
meta-data=/dev/sdb3              isize=512    agcount=4, agsize=4273 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=17089, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

- Create mount points /vfatfs5, /ext4fs5, and /xfsfs5, and mount all three manually. 

```bash
[root@server2 mapper]# mkdir /vfatfs5 /ext4fs5 /xfsfs5

[root@server2 mapper]# mount /dev/sdc1 /vfatfs5
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.

[root@server2 mapper]# mount /dev/sdc2 /ext4fs5
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.

[root@server2 mapper]# mount /dev/sdc3 /xfsfs5
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.

[root@server2 mapper]# mount
/dev/sdb1 on /vfatfs5 type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,errors=remount-ro)
/dev/sdb2 on /ext4fs5 type ext4 (rw,relatime,seclabel)
/dev/sdb3 on /xfsfs5 type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)

```

- Determine the UUIDs for the three file systems, and add them to the fstab file.
```bash
[root@server2 mapper]# blkid /dev/sdc1 /dev/sdc2 /dev/sdc3 >> /etc/fstab

[root@server2 mapper]# vim /etc/fstab

```


- Unmount all three file systems manually, and execute `mount -a` to mount them all.
`umount /dev/sdb1 /dev/sdb2 /dev/sdb3`
- Run `df -h` for verification. 
### Lab: Create XFS File System in LVM VDO Volume and Mount Persistently

- Ensure that VDO software is installed.
`sudo dnf install kmod-kvdo`

- Create a volume vdo5 with a logical size 20GB on a 5GB disk (lsblk) using the `lvcreate command`. 
```bash
[root@server2 ~]# sudo lvcreate -n vdo5 -l 1279 -V 20G --type vdo vgvdo1
WARNING: vdo signature detected on /dev/vgvdo1/vpool0 at offset 0. Wipe it? [y/n]: y
  Wiping vdo signature on /dev/vgvdo1/vpool0.
    The VDO volume can address 2 GB in 1 data slab.
    It can grow to address at most 16 TB of physical storage in 8192 slabs.
    If a larger maximum size might be needed, use bigger slabs.
  Logical volume "vdo5" created.
```

- Initialize the volume with XFS file system type. 
```bash
[root@server2 mapper]# sudo mkfs.xfs /dev/mapper/vgvdo1-vdo5
meta-data=/dev/mapper/vgvdo1-vdo5 isize=512    agcount=4, agsize=1310720 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=5242880, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.
```

- Create mount point /vdofs5, and mount it manually. 
```bash
[root@server2 mapper]# mkdir /vdofs5
[root@server2 mapper]#mount /dev/mapper/vgvdo1-vdo5 /vdofs5)/etc/fstab
[root@server2 mapper]# umount /dev/mapper/vgvdo1-vdo5
```

- Unmount the file system manually and execute `mount -a` to mount it back. 
```bash
[root@server2 mapper]# blkid /dev/mapper/vgvdo1-vdo5 >> /etc/fstab
[root@server2 mapper]# vim /etc/fstab
```

- Run `df -h` to confirm. 

### Lab: Create Ext4 and XFS File Systems in LVM Volumes and Mount Persistently 

- Initialize an available 250MB disk for use in LVM (lsblk). 
```bash
[root@server2 mapper]# parted /dev/sdc mklabel msdos
Warning: The existing disk label on /dev/sdc will be destroyed and all data on
this disk will be lost. Do you want to continue?
Yes/No? y                                                                 
Information: You may need to update /etc/fstab.

[root@server2 mapper]# parted /dev/sdc mkpart primary 1 100%
Information: You may need to update /etc/fstab.
```

- Create volume group vg with PE size 8MB and add the physical volume. 
```bash
[root@server2 ~]# sudo pvcreate /dev/sdc1
  Devices file /dev/sdc is excluded: device is partitioned.
  WARNING: adding device /dev/sdc1 with idname t10.ATA_VBOX_HARDDISK_VB6894bac4-590d5546 which is already used for /dev/sdc.
  Physical volume "/dev/sdc1" successfully created.
  
[root@server2 ~]# vgcreate -s 8 vg /dev/sdc1
  Devices file /dev/sdc is excluded: device is partitioned.
  WARNING: adding device /dev/sdc1 with idname t10.ATA_VBOX_HARDDISK_VB6894bac4-590d5546 which is already used for /dev/sdc.
  Volume group "vg" successfully created

```

- Create two logical volumes lv200 and lv300 of sizes 120MB and 100MB. 
```bash
[root@server2 ~]# lvcreate -n lv200 -L 120 vg
  Devices file /dev/sdc is excluded: device is partitioned.
  Logical volume "lv200" created.
  
[root@server2 ~]# lvcreate -n lv300 -L 100 vg
  Rounding up size to full physical extent 104.00 MiB
  Logical volume "lv300" created.
```

- Use the `vgs`, `pvs`, `lvs`, and `vgdisplay` commands for verification. 
- Initialize the volumes with Ext4 and XFS file system types. 
```bash
[root@server2 ~]# mkfs.ext4 /dev/vg/lv200
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 122880 1k blocks and 30720 inodes
Filesystem UUID: 52eac2ee-b5bd-4025-9e40-356b38d21996
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done 

[root@server2 ~]# mkfs.xfs /dev/vg/lv300
Filesystem should be larger than 300MB.
Log size should be at least 64MB.
Support for filesystems like this one is deprecated and they will not be supported in future releases.
meta-data=/dev/vg/lv300          isize=512    agcount=4, agsize=6656 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=26624, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

- Create mount points /lvmfs5 and /lvmfs6, and mount them manually. 
```bash
[root@server2 ~]# mkdir /lvmfs5 /lvmfs6
[root@server2 ~]# mount /dev/vg/lv200 /lvmfs5
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
[root@server2 ~]# mount /dev/vg/lv300 /lvmfs6
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

- Add the file system information to the fstab file using their device files. 
```bash
[root@server2 ~]# blkid /dev/vg/lv200 >> /etc/fstab
[root@server2 ~]# blkid /dev/vg/lv300 >> /etc/fstab
[root@server2 ~]# vim /etc/fstab
```

- Unmount the file systems manually, and execute mount -a to mount them back. Run `df -h` to confirm. 
```bash
[root@server2 ~]# umount /dev/vg/lv200 /dev/vg/lv300
[root@server2 ~]# mount -a
```

### Lab 14-4: Extend Ext4 and XFS File Systems in LVM Volumes 

- initialize an available 250MB disk for use in LVM (lsblk). 
```bash
[root@server2 ~]# pvcreate /dev/sdb
  Devices file /dev/sdc is excluded: device is partitioned.
WARNING: dos signature detected on /dev/sdb at offset 510. Wipe it? [y/n]: y
  Wiping dos signature on /dev/sdb.
  WARNING: adding device /dev/sdb with idname t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f which is already used for missing device.
  Physical volume "/dev/sdb" successfully created.
```

- Add the new physical volume to volume group vg200.
```bash
[root@server2 ~]# vgextend vg /dev/sdb
  Devices file /dev/sdc is excluded: device is partitioned.
  WARNING: adding device /dev/sdb with idname t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f which is already used for missing device.
  Volume group "vg" successfully extended
```

- Expand logical volumes lv200 and lv300 along with the underlying file systems to 200MB and 250MB. 
```bash
[root@server2 ~]# lvextend -L 200m /dev/vg/lv200
  Size of logical volume vg/lv200 changed from 120.00 MiB (15 extents) to 200.00 MiB (25 extents).
  Logical volume vg/lv200 successfully resized.
[root@server2 ~]# lvextend -L 250m /dev/vg/lv200
  Rounding size to boundary between physical extents: 256.00 MiB.
  Size of logical volume vg/lv200 changed from 200.00 MiB (25 extents) to 256.00 MiB (32 extents).
  Logical volume vg/lv200 successfully resized.
```

- Use the `vgs`, `pvs`, `lvs`, `vgdisplay`, and `df` commands for verification. 

### Lab 14-5: Create Swap in Partition and LVM Volume and Activate Persistently 

- Create two 100MB partitions on an available 250MB disk (lsblk) by invoking the parted utility directly at the command prompt. 
- Apply label "msdos" if the disk is new. 
```bash
[root@localhost ~]# parted /dev/sdd mklabel msdos
Information: You may need to update /etc/fstab.

[root@localhost ~]# parted /dev/sdd mkpart primary 1 100MB
Information: You may need to update /etc/fstab.

[root@localhost ~]# parted /dev/sdd mkpart primary 101 201
Information: You may need to update /etc/fstab.

```

- Initialize one of the partitions with swap structures. 
```bash
[root@localhost ~]# sudo mkswap /dev/sdd1
Setting up swapspace version 1, size = 94 MiB (98562048 bytes)
no label, UUID=40eea6c2-b80c-4b25-ad76-611071db52d5
```

- Apply label swappart to the swap partition, and add it to the fstab file. 
```bash
[root@localhost ~]# swaplabel -L swappart /dev/sdd1
[root@localhost ~]# blkid /dev/sdd1 >> /etc/fstab
[root@localhost ~]# vim /etc/fstab
UUID="40eea6c2-b80c-4b25-ad76-611071db52d5" swap swap pri=1 0 0
```

- Execute `swapon -a` to activate it. 
- Run `swapon -s` to confirm activation.

- Initialize the other partition for use in LVM. 
```bash
[root@localhost ~]# pvcreate /dev/sdd2
  Physical volume "/dev/sdd2" successfully created.

```

- Expand volume group vg (Lab 14-3) by adding this physical volume to it. 
```bash
[root@localhost ~]# vgextend vg /dev/sdd2
  Volume group "vg200" successfully extended
```

- Create logical volume swapvol of size 180MB. 
```bash
[root@localhost ~]# lvcreate -L 180 -n swapvol vg
  Logical volume "swapvol" created.
```

- Use the `vgs`, `pvs`, `lvs`, and `vgdisplay` commands for verification. 
- Initialize the logical volume for swap. 
```bash
[root@localhost vg200]# mkswap /dev/vg/swapvol
Setting up swapspace version 1, size = 180 MiB (188739584 bytes)
no label, UUID=a4b939d0-4b53-4e73-bee5-4c402aff6f9b

```

- Add an entry to the fstab file for the new swap area using its device file. 
```bash
[root@localhost vg200]# vim /etc/fstab
/dev/vg200/swapvol swap swap pri=2 0 0
```

- Execute `swapon -a` to activate it. 
- Run `swapon -s` to confirm activation.
