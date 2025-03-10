## Disk Partitions 

- Be careful when adding a new partition to elude data corruption with overlapping an extant partition or wasting storage by leaving unused space between adjacent partitions. 
- Disk allocated at the time of installation is recognized as sda (s for SATA, SAS, or SCSI device) disk a,  first partition identified as sda1 and the second partition as sda2. 
- Any subsequent disks added to the system will be known as sdb, sdc, sdd, and so on, and will use 1, 2, 3, etc. for partition numbering.

Use `lsblk` to list disk and partition information. 

```bash
[root@server1 ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda             8:0    0   10G  0 disk 
├─sda1          8:1    0    1G  0 part /boot
└─sda2          8:2    0    9G  0 part 
  ├─rhel-root 253:0    0    8G  0 lvm  /
  └─rhel-swap 253:1    0    1G  0 lvm  [SWAP]
sr0            11:0    1  9.8G  0 rom  /mnt
```

sr0 represents the ISO image mounted as an optical medium:

```bash
[root@server1 ~]# sudo fdisk -l
Disk /dev/sda: 10 GiB, 10737418240 bytes, 20971520 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xfc8b3804

Device     Boot   Start      End  Sectors Size Id Type
/dev/sda1  *       2048  2099199  2097152   1G 83 Linux
/dev/sda2       2099200 20971519 18872320   9G 8e Linux LVM


Disk /dev/mapper/rhel-root: 8 GiB, 8585740288 bytes, 16769024 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/rhel-swap: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

identifiers 83 and 8e are hexadecimal values for the partition types

### Storage Management Tools 

`parted`, `gdisk`, and `LVM`
Partitions created with a combination of most of these tools and toolsets can coexist on the same disk.

`parted`
understands both MBR and GPT formats. 

`gdisk`
- support the GPT format only
- may be used as a replacement of parted. 

`LVM`
- feature-rich logical volume management solution that gives flexibility in storage management.
