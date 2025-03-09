# File System Management 

#### File System Administration Commands 
- Some are limited to their operations on the Extended, XFS, or VFAT file system type.  
- Others are general and applicable to all file system types. 

#### Extended File System Management Commands            

`e2label`                             
  - Modifies the label of a file system

`tune2fs`                            
  - Tunes or displays file system attributes

#### XFS Management Commands                             

`xfs_admin  `                         
 - Tunes file system attributes

`xfs_growfs `                         
  - Extends the size of a file system

`xfs_info  `                          
- Exhibits information about a file system

#### General File System Commands        

`blkid  `                             
  - Displays block device attributes including their UUIDs and labels

`df `                                 
  - Reports file system utilization

`du   `                               
- Calculates disk usage of directories and file systems

`fsadm   `                            
- Resizes a file system. This command is automatically invoked when the `lvresize` command is run with the `-r` switch.

`lsblk  `                             
- Lists block devices and file systems and their attributes including their UUIDs and labels

`mkfs  `                              
- Creates a file system. Use the `-t` option and specify ext3, ext4, vfat, or xfs file system type.

`mount    `                           
- Mount a file system for user access. 
- Display currently mounted file systems.

`umount   `                           
- Unmount a file system.

### Mounting and Unmounting File Systems 

- File system must be connected to the directory structure at a desired attachment point, (mount point) 
- A mount point in essence is any empty directory that is created and used for this purpose.

Use the mount command to view information about xfs mounted file systems:
```bash
[root@server2 ~]# mount -t xfs
/dev/mapper/rhel-root on / type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
/dev/sda1 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
```

#### Mount command
- `-t` option 
	- type. 
- Mount a file system to a mount point.
- Performed with the root user privileges.
- Requires the absolute pathnames of the file system block device and the mount point name. 
- Accepts the UUID or label of the file system in lieu of the block device name. 
- Mount all or a specific type of file system. 
- Upon successful mount, the kernel places an entry for the file system in the */proc/self/mounts* file.
- A mount point should be empty when an attempt is made to mount a file system on it, otherwise the content of the mount point will hide.
- The mount point must not be in use or the mount attempt will fail.

**auto (noauto)**                       
- Mounts (does not mount) the file system when the `-a` option is specified

**defaults**                            
- Mounts a file system with all the default values (async, auto, rw, etc.)

**\_netdev**                         
- Used for a file system that requires network connectivity in place before it can be mounted. NFS is an example.

**remount**                             
- Remounts an already mounted file system to enable or disable an option

**ro (rw)**                             
- Mounts a file system read-only read/write)

#### umount Command
- Detach a file system from the directory hierarchy and make it inaccessible to users and applications. 
- Expects the absolute pathname to the block device containing the file system or its mount point name in order to detach it. 
- Unmount all or a specific type of file system. 
- Kernel removes the corresponding file system entry from the */proc/self/mounts* file after it has been successfully disconnected.
### Determining the UUID of a File System 

- Extended and XFS file systems have a 128-bit (32 hexadecimal characters) UUID (Universally Unique IDentifier) assigned to it at the time of its creation.
- UUIDs assigned to vfat file systems are 32-bit (8 hexadecimal characters) in length. 
- Assigning a UUID makes the file system unique among many other file systems that potentially exist on the system. 
- Persistent across system reboots. 
- Used by default in RHEL 9 in the */etc/fstab* file for any file system that is created by the system in a standard partition.

- RHEL attempts to mount all file systems listed in the */etc/fstab* file at reboots. 
- Each file system has an associated device file and UUID, but may or may not have a corresponding label. 
- The system checks for the presence of each file system's device file, UUID, or label, and then attempts to mount it.

Determine the UUID of /boot
```bash
[root@server2 ~]# lsblk | grep boot
├─sda1          8:1    0    1G  0 part /boot
```

```bash
[root@server2 ~]# sudo xfs_admin -u /dev/sda1
UUID = 630568e1-608f-4603-9b97-e27f82c7d4b4

[root@server2 ~]# sudo blkid /dev/sda1
/dev/sda1: UUID="630568e1-608f-4603-9b97-e27f82c7d4b4" TYPE="xfs" PARTUUID="7dcb43e4-01"

[root@server2 ~]# sudo lsblk -f /dev/sda1
NAME FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda1 xfs                630568e1-608f-4603-9b97-e27f82c7d4b4  616.1M    36% /boot
```

For extended file systems, you can use the `tune2fs`, `blkid`, or `lsblk` commands to determine the UUID.

A UUID is also assigned to a file system that is created in a VDO or LVM volume; however, it need not be used in the fstab file, as the device
files associated with the logical volumes are always unique and persistent.

### Labeling a File System 

- A unique label may be used instead of a UUID to keep the file system association with its device file exclusive and persistent across system reboots. 
- A label is limited to a maximum of 12 characters on the XFS file system 
- 16 characters on the Extended file system. 
- By default, no labels are assigned to a file system at the time of its creation.

The */boot* file system is located in the */dev/sda1* partition and its type is XFS. You can use the `xfs_admin` or the `lsblk` command as follows to
determine its label:
```bash
[root@server2 ~]# sudo xfs_admin -l /dev/sda1
label = ""

[root@server2 ~]# sudo lsblk -f /dev/sda1
NAME FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda1 xfs                630568e1-608f-4603-9b97-e27f82c7d4b4  616.1M    36% /boot
```

- Not needed on a file system if you intend to use its UUID or if it is created in a logical volume
- You can still apply one using the `xfs_admin` command with the `-L` option. 
- Labeling an XFS file system requires that the target file system be unmounted.

unmount /boot, set the label "bootfs" on its device file, and remount it:
```bash
[root@server2 ~]# sudo umount /boot
[root@server2 ~]# sudo xfs_admin -L bootfs /dev/sda1
writing all SBs
new label = "bootfs"

```

Confirm the new label by executing `sudo xfs_admin -l /dev/sda1` or `sudo lsblk -f /dev/sda1`.

For extended file systems, you can use the `e2label` command to apply a label and the `tune2fs`, `blkid`, and `lsblk` commands to view and verify.

Now you can replace the `UUID=\"22d05484-6ae1-4ef8-a37d-abab674a5e35`\" for /boot in the fstab file with `LABEL=bootfs`, and unmount and remount /boot as demonstrated above for confirmation.
```bash
[root@server2 ~]# mount /boot
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

A label may also be applied to a file system created in a logical volume; however, it is not recommended for use in the fstab file, as the
device files for logical volumes are always unique and remain persistent across system reboots.

### Automatically Mounting a File System at Reboots 

#### */etc/fstab*
- File systems defined in the */etc/fstab* file are mounted automatically at reboots. 
- Must contain proper and complete information for each listed file system. 
- An incomplete or inaccurate entry might leave the system in an undesirable or unbootable state. 
- Only need to specify one of the four attributes
	- Block device name
	- UUID
	- label
	- mount point
- The `mount` command obtains the rest of the information from this file. 
- Only need to specify one of these attributes with the `umount` command to detach it from the directory hierarchy.
- Contains entries for file systems that are created at the time of installation. 

```bash
[root@server2 ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Sun Feb 25 12:11:47 2024
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root   /                       xfs     defaults        0 0
LABEL=bootfs /boot                   xfs     defaults        0 0
/dev/mapper/rhel-swap   none                    swap    defaults        0 0
```

EXAM TIP: Any missing or invalid entry in this file may render the system unbootable. You will have to boot the system in emergency mode to fix this file. Ensure that you understand each field in the file for both file system and swap entries.

The format of this file is such that each row is broken out into six columns to identify the required attributes for each file system to be successfully mounted. Here is what the columns contain:

Column 1:
- physical or virtual device path where the file system is resident, or its associated UUID or label. 
- can be entries for network file systems here as well.

Column 2: 
- Identifies the mount point for the file system. 
- swap partitions, use either "none" or "swap".

Column 3: 
- Type of file system such as Ext3, Ext4, XFS, VFAT, or ISO9660. 
- For swap, the type "swap" is used.  
- may use "auto" instead to leave it up to the mount command to determine the type of the file system.

Column 4: 
- Identifies one or more comma-separated options to be used when mounting the file system. 
- Consult the manual pages of the `mount` command or the fstab file for additional options and details.

Column 5: 
- Used by the dump utility to ascertain the file systems that need to be dumped. 
- Value of 0 (or the absence of this column) disables this check. 
- This field is applicable only on Extended file systems; 
- XFS does not use it.

Column 6: 
- Sequence number in which to run the `e2fsck` (file system check and repair utility for Extended file system types) utility on the file system at system boot. 
- By default, 0 is used for memory-based, remote, and removable file systems, 1 for /, and 2 for /boot and other physical file systems. 0 can also be used for /, /boot, and other physical file systems you don't want to be checked or repaired. 
- Applicable only on Extended file systems; 
- XFS does not use it.

- 0 in columns 5 and 6 for XFS, virtual, remote, and removable file system types has no meaning. You do not need to add them for these file system types.

## Lab: Create and Mount Ext4, VFAT, and XFS File Systems in Partitions (server2)

- Create 2 x 100MB partitions on the /dev/sdb disk, 
- initialize them separately with the Ext4 and VFAT file system types, 
- define them for persistence using their UUIDs, 
- create mount points called /ext4fs1 and /vfatfs1, 
- attach them to the directorystructure
- verify their availability and usage
- you will use the disk /dev/sdc and repeat the above procedure to establish an XFS file system in it and mount it on /xfsfs1.

1\. Apply the label "msdos" to the sdb disk using the parted command:
```bash
[root@server20 ~]# sudo parted /dev/sdb mklabel msdos
Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be
lost. Do you want to continue?
Yes/No? y                                                                 
Information: You may need to update /etc/fstab.

```

2\. Create 2 x 100MB primary partitions on sdb with the parted command:
```bash
[root@server20 ~]# sudo parted /dev/sdb mkpart primary 1 101m
Information: You may need to update /etc/fstab.

[root@server20 ~]# sudo parted /dev/sdb mkpart primary 102 201m
Information: You may need to update /etc/fstab.

```

3\. Initialize the first partition (sdb1) with Ext4 file system type using the mkfs command:
```bash
[root@server20 ~]# sudo mkfs -t ext4 /dev/sdb1
mke2fs 1.46.5 (30-Dec-2021)
/dev/sdb1 contains a LVM2_member file system
Proceed anyway? (y,N) y
Creating filesystem with 97280 1k blocks and 24288 inodes
Filesystem UUID: 73db0582-7183-42aa-951d-2f48b7712597
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done 
```

4\. Initialize the second partition (sdb2) with VFAT file system type using the mkfs command:
```bash
[root@server20 ~]# sudo mkfs -t vfat /dev/sdb2
mkfs.fat 4.2 (2021-01-31)
```

5\. Initialize the whole disk (sdc) with the XFS file system type using the mkfs.xfs command. Add the -f flag to force the removal of any old partitioning or labeling information from the disk.
```bash
[root@server20 ~]# sudo mkfs.xfs /dev/sdc -f 
Filesystem should be larger than 300MB.
Log size should be at least 64MB.
Support for filesystems like this one is deprecated and they will not be supported in future releases.
meta-data=/dev/sdc               isize=512    agcount=4, agsize=16000 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=64000, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

6\. Determine the UUIDs for all three file systems using the lsblk command:
```bash
[root@server2 ~]# lsblk -f /dev/sdb /dev/sdc
NAME   FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sdb                                                                           
├─sdb1 ext4   1.0         0bdd22d0-db53-40bb-8cc7-36efc9184196                
└─sdb2 vfat   FAT16       FB3A-6572                                           
sdc    xfs                91884326-9686-4569-96fa-9adb02c1f6f4>)
```

7\. Open the /etc/fstab file, go to the end of the file, and append entries for the file systems for persistence using their UUIDs:
```bash
UUID=0bdd22d0-db53-40bb-8cc7-36efc9184196 /ext4fs1 ext4 defaults 0 0                
UUID=FB3A-6572 /vfatfs1 vfat defaults 0 0                                          
UUID=91884326-9686-4569-96fa-9adb02c1f6f4 /xfsfs1 xfs defaults 0 0
```

8\. Create mount points /ext4fs1, /vfatfs1, and /xfsfs1 for the three
file systems using the mkdir command:
`[root@server2 ~]# sudo mkdir /ext4fs1 /vfatfs1 /xfsfs1`

9\. Mount the new file systems using the mount command. This command will fail if there are any invalid or missing information in the file.
```bash
[root@server2 ~]# sudo mount -a
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

10\. View the mount and availability status as well as the types of all three file systems using the df command:
```bash
[root@server2 ~]# df -hT
Filesystem            Type      Size  Used Avail Use% Mounted on
devtmpfs              devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs                 tmpfs     888M     0  888M   0% /dev/shm
tmpfs                 tmpfs     356M  5.1M  351M   2% /run
/dev/mapper/rhel-root xfs        17G  2.0G   15G  12% /
/dev/sda1             xfs       960M  344M  617M  36% /boot
tmpfs                 tmpfs     178M     0  178M   0% /run/user/0
/dev/sdb1             ext4       84M   14K   77M   1% /ext4fs1
/dev/sdb2             vfat       95M     0   95M   0% /vfatfs1
/dev/sdc              xfs       245M   15M  231M   6% /xfsfs1
```

## Lab: Create and Mount Ext4 and XFS File Systems in LVM Logical Volumes (server2)


- Create a volume group called *vgfs* comprised of a 172MB physical volume created in a partition on the */dev/sdd* disk.
- The PE size for the volume group should be set at 16MB. 
- Create two logical volumes called *ext4vol* and *xfsvol* of sizes 80MB each and initialize them with the Ext4 and XFS file system types.
- Ensure that both file systems are persistently defined using their logical volume device filenames. 
- Create mount points called /ext4fs2 and /xfsfs2, 
- Mount the file systems.
- Verify their availability and usage.

1\. Create a 172MB partition on the sdd disk using the parted command:
```bash
[root@server2 ~]# sudo parted /dev/sdd mkpart pri 1 172m
Information: You may need to update /etc/fstab.
```

2\. Initialize the sdd1 partition for use in LVM using the pvcreate command:
```bash
[root@server2 ~]# sudo pvcreate /dev/sdd1
  Device /dev/sdb2 has updated name (devices file /dev/sdd2)
  Device /dev/sdb1 has no PVID (devices file brKVLFEG3AoBzhWoso0Sa1gLYHgNZ4vL)
  Physical volume "/dev/sdd1" successfully created.
```

3\. Create the volume group vgfs with a PE size of 16MB using the physical volume sdd1:
```bash
[root@server2 ~]# sudo vgcreate -s 16 vgfs /dev/sdd1
  Volume group "vgfs" successfully created

```

The PE size is not easy to alter after a volume group creation, so ensure it is defined as required at creation.

4\. Create two logical volumes ext4vol and xfsvol of size 80MB each in vgfs using the `lvcreate` command:
```bash
[root@server2 ~]# sudo lvcreate -n ext4vol -L 80 vgfs
  Logical volume "ext4vol" created.
  
[root@server2 ~]# sudo lvcreate  -n xfsvol -L 80 vgfs
  Logical volume "xfsvol" created.
```

5\. Format the ext4vol logical volume with the Ext4 file system type using the mkfs.ext4 command:
```bash
[root@server2 ~]# sudo mkfs.ext4 /dev/vgfs/ext4vol
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 81920 1k blocks and 20480 inodes
Filesystem UUID: 4ed1fef7-2164-485b-8035-7f627cd59419
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```

You can also use `sudo mkfs -t ext4 /dev/vgfs/ext4vol`.

6\. Format the xfsvol logical volume with the XFS file system type using the mkfs.xfs command:
```bash
[root@server2 ~]# sudo mkfs.xfs /dev/vgfs/xfsvol
Filesystem should be larger than 300MB.
Log size should be at least 64MB.
Support for filesystems like this one is deprecated and they will not be supported in future releases.
meta-data=/dev/vgfs/xfsvol       isize=512    agcount=4, agsize=5120 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=20480, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

You may also use `sudo mkfs -t xfs /dev/vgfs/xfsvol` instead.

7\. Open the /etc/fstab file, go to the end of the file, and append entries for the file systems for persistence using their device files:
```bash
/dev/vgfs/ext4vol /ext4fs2 ext4 defaults 0 0
/dev/vgfs/xfsvol /xfsfs2 xfs defaults 0 0
```

8\. Create mount points /ext4fs2 and /xfsfs2 using the mkdir command:
`[root@server2 ~]# sudo mkdir /ext4fs2 /xfsfs2`

9\. Mount the new file systems using the mount command. This command will fail if there is any invalid or missing information in the file.
```bash
[root@server2 ~]# sudo mount -a
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

10\. View the mount and availability status as well as the types of the
new LVM file systems using the lsblk and df commands:
```bash
[root@server2 ~]# lsblk /dev/sdd
NAME             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdd                8:48   0  250M  0 disk 
└─sdd1             8:49   0  163M  0 part 
  ├─vgfs-ext4vol 253:2    0   80M  0 lvm  /ext4fs2
  └─vgfs-xfsvol  253:3    0   80M  0 lvm  /xfsfs2
[root@server2 ~]# df -hT | grep fs2
/dev/mapper/vgfs-ext4vol ext4       70M   14K   64M   1% /ext4fs2
/dev/mapper/vgfs-xfsvol  xfs        75M  4.8M   70M   7% /xfsfs2
```

### Lab: Resize Ext4 and XFS File Systems in LVM Logical Volumes (server 2)

- Grow the size of the *vgfs* volume group that was created in the last lab by adding the whole *sde* disk to it. 
- Extend the *ext4vol* logical volume along with the file system it contains by 40MB using two separate commands. 
- Extend the *xfsvol* logical volume along with the file system it contains by 40MB using a single command. 
- Verify the new extensions.

1\. Initialize the sde disk and add it to the vgfs volume group:

sde had a gpt partition table with no partitions ran the following to reset it:
```bash
[root@server2 ~]# dd if=/dev/zero of=/dev/sde bs=1M count=2 conv=fsync
2+0 records in
2+0 records out
2097152 bytes (2.1 MB, 2.0 MiB) copied, 0.0102036 s, 206 MB/s
[root@server2 ~]# sudo partprobe /dev/sde
[root@server2 ~]# sudo pvcreate /dev/sde
  Physical volume "/dev/sde" successfully created.
```

```bash
[root@server2 ~]# sudo pvcreate /dev/sde
  Physical volume "/dev/sde" successfully created.
[root@server2 ~]# sudo vgextend vgfs /dev/sde
  Volume group "vgfs" successfully extended
```

2\. Confirm the new size of vgfs using the `vgs` and `vgdisplay` commands:
```bash
[root@server2 ~]# sudo vgs
  VG   #PV #LV #SN Attr   VSize   VFree  
  rhel   1   2   0 wz--n- <19.00g      0 
  vgfs   2   2   0 wz--n- 400.00m 240.00m

```

```bash
[root@server2 ~]# vgdisplay vgfs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 PVID qeP9dCevNnTy422I8p18NxDKQ2WyDodU last seen on /dev/sdf1 not found.
  --- Volume group ---
  VG Name               vgfs
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               400.00 MiB
  PE Size               16.00 MiB
  Total PE              25
  Alloc PE / Size       10 / 160.00 MiB
  Free  PE / Size       15 / 240.00 MiB
  VG UUID               amDADJ-I4dH-jQUF-RFcE-58iL-jItl-5ti6LS
```

There are now two physical volumes in the volume group and the total size increased to 400MiB.

3\. Grow the logical volume ext4vol and the file system it holds by 40MB using the `lvextend` and `fsadm` command pair. Make sure to use an uppercase L to specify the size. The default unit is MiB. The plus sign (+) signifies an addition to the current size.
```bash
[root@server2 ~]# sudo lvextend -L +40 /dev/vgfs/ext4vol
  Rounding size to boundary between physical extents: 48.00 MiB.
  Size of logical volume vgfs/ext4vol changed from 80.00 MiB (5 extents) to 128.00 MiB (8 extents).
  Logical volume vgfs/ext4vol successfully resized.
  
[root@server2 ~]# sudo fsadm resize /dev/vgfs/ext4vol
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/mapper/vgfs-ext4vol is mounted on /ext4fs2; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/mapper/vgfs-ext4vol is now 131072 (1k) blocks long.
```

The resize subcommand instructs the `fsadm` command to grow the file system to the full length of the specified logical volume.

4\. Grow the logical volume xfsvol and the file system (-r) it holds by (+) 40MB using the lvresize command:
```bash
[root@server2 ~]# sudo lvresize -r -L +40 /dev/vgfs/xfsvol
  Rounding size to boundary between physical extents: 48.00 MiB.
  Size of logical volume vgfs/xfsvol changed from 80.00 MiB (5 extents) to 128.00 MiB (8 extents).
  File system xfs found on vgfs/xfsvol mounted at /xfsfs2.
  Extending file system xfs to 128.00 MiB (134217728 bytes) on vgfs/xfsvol...
xfs_growfs /dev/vgfs/xfsvol
meta-data=/dev/mapper/vgfs-xfsvol isize=512    agcount=4, agsize=5120 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=20480, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 20480 to 32768
xfs_growfs done
  Extended file system xfs on vgfs/xfsvol.
  Logical volume vgfs/xfsvol successfully resized.
```

5\. Verify the new extensions to both logical volumes using the `lvs` command. You may also issue the `lvdisplay` or `vgdisplay` command instead.
```bash
[root@server2 ~]# sudo lvs | grep vol
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 PVID qeP9dCevNnTy422I8p18NxDKQ2WyDodU last seen on /dev/sdf1 not found.
  ext4vol vgfs -wi-ao---- 128.00m                                                    
  xfsvol  vgfs -wi-ao---- 128.00m   
```

6\. Check the new sizes and the current mount status for both file systems using the `df` and `lsblk` commands:

```bash
[root@server2 ~]# df -hT | grep -E 'ext4vol|xfsvol'
/dev/mapper/vgfs-xfsvol  xfs       123M  5.4M  118M   5% /xfsfs2
/dev/mapper/vgfs-ext4vol ext4      115M   14K  107M   1% /ext4fs2
```

```bash
[root@server2 ~]# lsblk /dev/sdd /dev/sde
NAME             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdd                8:48   0  250M  0 disk 
└─sdd1             8:49   0  163M  0 part 
  ├─vgfs-ext4vol 253:2    0  128M  0 lvm  /ext4fs2
  └─vgfs-xfsvol  253:3    0  128M  0 lvm  /xfsfs2
sde                8:64   0  250M  0 disk 
├─vgfs-ext4vol   253:2    0  128M  0 lvm  /ext4fs2
└─vgfs-xfsvol    253:3    0  128M  0 lvm  /xfsfs2
```

## Lab: Create and Mount XFS File System in LVM VDO Volume 

- Create an LVM VDO volume called *lvvdo* of virtual size 20GB on the 5GB sdf disk in a volume group called *vgvdo1*.
- Initialize the volume with the XFS file system type.
- Define it for persistence using its device files.
- Create a mount point called */xfsvdo1*, attach it to the directory structure.
- verify its availability and usage.\

1\. Initialize the sdf disk using the `pvcreate` command:
```bash
[root@server2 ~]# sudo pvcreate /dev/sdf
  WARNING: adding device /dev/sdf with idname t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 which is already used for missing device.
  Physical volume "/dev/sdf" successfully created.
```

2\. Create vgvdo1 volume group using the `vgcreate` command:
```bash
[root@server2 ~]# sudo vgcreate vgvdo1 /dev/sdf
  WARNING: adding device /dev/sdf with idname t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 which is already used for missing device.
  Volume group "vgvdo1" successfully created
```

3\. Display basic information about the volume group:
```bash
root@server2 ~]# sudo vgdisplay vgvdo1
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 PVID qeP9dCevNnTy422I8p18NxDKQ2WyDodU last seen on /dev/sdf1 not found.
  --- Volume group ---
  VG Name               vgvdo1
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <5.00 GiB
  PE Size               4.00 MiB
  Total PE              1279
  Alloc PE / Size       0 / 0   
  Free  PE / Size       1279 / <5.00 GiB
  VG UUID               b9u8Ng-m3BF-Jz2b-sBu8-gEG1-bBGQ-sBgrt0

```

4\. Create a VDO volume called lvvdo1 using the `lvcreate` command. Use the `-l` option to specify the number of logical extents (1279) to be allocated and the `-V` option for the amount of virtual space (20GB).
```bash
[root@server2 ~]# sudo lvcreate -n lvvdo -l 1279 -V 20G --type vdo vgvdo1
WARNING: vdo signature detected on /dev/vgvdo1/vpool0 at offset 0. Wipe it? [y/n]: y
  Wiping vdo signature on /dev/vgvdo1/vpool0.
    The VDO volume can address 2 GB in 1 data slab.
    It can grow to address at most 16 TB of physical storage in 8192 slabs.
    If a larger maximum size might be needed, use bigger slabs.
  Logical volume "lvvdo" created.
```

5\. Display detailed information about the volume group including the logical volume and the physical volume:
```bash
[root@server2 ~]# sudo vgdisplay -v vgvdo1
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 PVID qeP9dCevNnTy422I8p18NxDKQ2WyDodU last seen on /dev/sdf1 not found.
  --- Volume group ---
  VG Name               vgvdo1
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <5.00 GiB
  PE Size               4.00 MiB
  Total PE              1279
  Alloc PE / Size       1279 / <5.00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               b9u8Ng-m3BF-Jz2b-sBu8-gEG1-bBGQ-sBgrt0
   
  --- Logical volume ---
  LV Path                /dev/vgvdo1/vpool0
  LV Name                vpool0
  VG Name                vgvdo1
  LV UUID                nTPKtv-3yTW-J7Cy-HVP1-Aujs-cXZ6-gdS2fI
  LV Write Access        read/write
  LV Creation host, time server2, 2024-07-01 12:57:56 -0700
  LV VDO Pool data       vpool0_vdata
  LV VDO Pool usage      60.00%
  LV VDO Pool saving     100.00%
  LV VDO Operating mode  normal
  LV VDO Index state     online
  LV VDO Compression st  online
  LV VDO Used size       <3.00 GiB
  LV Status              NOT available
  LV Size                <5.00 GiB
  Current LE             1279
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
   
  --- Logical volume ---
  LV Path                /dev/vgvdo1/lvvdo
  LV Name                lvvdo
  VG Name                vgvdo1
  LV UUID                Z09BdK-ETJk-Gi53-m8Cg-mnTd-RYug-Z9nV0L
  LV Write Access        read/write
  LV Creation host, time server2, 2024-07-01 12:58:02 -0700
  LV VDO Pool name       vpool0
  LV Status              available
  # open                 0
  LV Size                20.00 GiB
  Current LE             5120
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:6
   
  --- Physical volumes ---
  PV Name               /dev/sdf     
  PV UUID               WKc956-Xp66-L8v9-VA6S-KWM5-5e3X-kx1v0V
  PV Status             allocatable
  Total PE / Free PE    1279 / 0
```

6\. Display the new VDO volume creation using the lsblk command:
```bash
[root@server2 ~]# sudo lsblk /dev/sdf
NAME                    MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
sdf                       8:80   0   5G  0 disk 
└─vgvdo1-vpool0_vdata   253:4    0   5G  0 lvm  
  └─vgvdo1-vpool0-vpool 253:5    0  20G  0 lvm  
    └─vgvdo1-lvvdo      253:6    0  20G  0 lvm  
```

The output shows the virtual volume size (20GB) and the underlying disk size (5GB).

7\. Initialize the VDO volume with the XFS file system type using the `mkfs.xfs` command. The VDO volume device file is
/dev/mapper/vgvdo1-lvvdo as indicated in the above output. Add the `-f` flag to force the removal of any old partitioning or labeling information from the disk.
```bash
[root@server2 mapper]# sudo mkfs.xfs /dev/mapper/vgvdo1-lvvdo
meta-data=/dev/mapper/vgvdo1-lvvdo isize=512    agcount=4, agsize=1310720 blks
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

(lab said vgvdo1-lvvdo1 but it didn't exist for me.)

8\. Open the /etc/fstab file, go to the end of the file, and append the following entry for the file system for persistent mounts using its device file:
```bash
/dev/mapper/vgvdo1-lvvdo /xfsvdo1 xfs defaults 0 0 
```

9\. Create the mount point /xfsvdo1 using the mkdir command:
```bash
[root@server2 mapper]# sudo mkdir /xfsvdo1
```

10\. Mount the new file system using the mount command. This command will fail if there are any invalid or missing information in the file.
```bash
[root@server2 mapper]# sudo mount -a
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
```

The mount command with the `-a` flag is a validation test for the fstab file. It should always be executed after updating this file and before rebooting the server to avoid landing the system in an unbootable state.

11\. View the mount and availability status as well as the type of the VDO file system using the `lsblk` and `df` commands:
```bash
[root@server2 mapper]# lsblk /dev/sdf
NAME                    MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
sdf                       8:80   0   5G  0 disk 
└─vgvdo1-vpool0_vdata   253:4    0   5G  0 lvm  
  └─vgvdo1-vpool0-vpool 253:5    0  20G  0 lvm  
    └─vgvdo1-lvvdo      253:6    0  20G  0 lvm  /xfsvdo1

[root@server2 mapper]# df -hT /xfsvdo1
Filesystem               Type  Size  Used Avail Use% Mounted on
/dev/mapper/vgvdo1-lvvdo xfs    20G  175M   20G   1% /xfsvdo1
```
