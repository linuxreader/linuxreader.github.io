# Virtual Data Optimizer (VDO)

- Used for storage optimization
- Device driver layer that sits between the Linux kernel and the physical storage devices.
- Conserve disk space, improve data throughput, and save on storage cost.
- Employs **thin provisioning**, **de-duplication**, and **compression** technologies to help realize the goals.

### How VDO Conserves Storage

**Stage 1**
- Makes use of thin provisioning to identify and eliminate empty (zero-byte) data blocks. (zero-block elimination)
- Removes randomization of data blocks by moving in-use data blocks to contiguous locations on the storage device.

![](image-XYSCLXMC.jpg)

**Stage 2**
- If it detects that new data is an identical copy of some existing data, it makes an internal note of it but does not actually write the redundant data to the disk. (de-duplication) 
- Implemented with the inclusion of a kernel module called **UDS** (Universal De-duplication Service). 

**Stage 3**
- Calls upon another kernel module called **kvdo**, which compresses the residual data blocks and consolidates them on a lower number of blocks. 
- Results in a further drop in storage space utilization.
- Runs in the background and processes inbound data through the three stages on VDO-enabled volumes. 
- Not a CPU or memory-intensive process

### VDO Integration with LVM
- LVM utilities have been enhanced to include options to support VDO volumes.

### VDO Components

- Utilizes the concepts of pool and volume. 
**pool**
- logical volume that is created inside an LVM volume group using a deduplicated storage space. 
**volume** 
- Just like a regular LVM logical volume, but it is provisioned in a pool. 
- Needs to be formatted with file system structures before it can be used.

### `vdo` and `kmod-kvdo` Commands
- Create, mount, and manage LVM VDO volumes
- Installed on the system by default.

`vdo`
- Installs the tools necessary to support the creation and management of VDO volumes

`kmod-kvdo`
- Implements fine-grained storage virtualization, thin provisioning, and compression.
Not installed by default?


### Exercise 13-12: Create an LVM VDO Volume

- Initialize the 5GB disk (sdf) for use in LVM VDO. 
- Create a volume group called vgvdo and add the physical volume to it.
- List and display the volume group and the physical volume. 
- Create a VDO volume called lvvdo with a virtual size of 20GB.


1\. Initialize the sdf disk using the pvcreate command:
```bash
[root@server2 ~]# sudo pvcreate /dev/sdf
  Physical volume "/dev/sdf" successfully created.
```

2\. Create vgvdo volume group using the vgcreate command:
```bash
[root@server2 ~]# sudo vgcreate vgvdo /dev/sdf
  Volume group "vgvdo" successfully created
```

3\. Display basic information about the volume group:
```bash
[root@server2 ~]# sudo vgdisplay vgvdo
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  --- Volume group ---
  VG Name               vgvdo
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
  VG UUID               tED1vC-Ylec-fpeR-KM8F-8FzP-eaQ4-AsFrgc
```

4\. Create a VDO volume called lvvdo using the `lvcreate` command. Use the -l option to specify the number of logical extents (1279) to be allocated and the -V option for the amount of virtual space.
```bash
[root@server2 ~]# sudo dnf install kmod-kvdo
[root@server2 ~]# sudo lvcreate --type vdo -l 1279 -n lvvdo -V 20G vgvdo
    The VDO volume can address 2 GB in 1 data slab.
    It can grow to address at most 16 TB of physical storage in 8192 slabs.
    If a larger maximum size might be needed, use bigger slabs.
  Logical volume "lvvdo" created.
```

5\. Display detailed information about the volume group including the logical volume and the physical volume:
```bash
[root@server2 ~]# sudo vgdisplay -v vgvdo
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  --- Volume group ---
  VG Name               vgvdo
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
  VG UUID               tED1vC-Ylec-fpeR-KM8F-8FzP-eaQ4-AsFrgc
   
  --- Logical volume ---
  LV Path                /dev/vgvdo/vpool0
  LV Name                vpool0
  VG Name                vgvdo
  LV UUID                yGAsK2-MruI-QGy2-Q1IF-CDDC-XPNT-qkjJ9t
  LV Write Access        read/write
  LV Creation host, time server2, 2024-06-16 09:35:46 -0700
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
  LV Path                /dev/vgvdo/lvvdo
  LV Name                lvvdo
  VG Name                vgvdo
  LV UUID                nnGTW5-tVFa-T3Cy-9nHj-sozF-2KpP-rVfnSq
  LV Write Access        read/write
  LV Creation host, time server2, 2024-06-16 09:35:47 -0700
  LV VDO Pool name       vpool0
  LV Status              available
  # open                 0
  LV Size                20.00 GiB
  Current LE             5120
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:4
   
  --- Physical volumes ---
  PV Name               /dev/sdf     
  PV UUID               0oAXHG-C4ub-Myou-5vZf-QxIX-KVT3-ipMZCp
  PV Status             allocatable
  Total PE / Free PE    1279 / 0
```


The output reflects the creation of two logical volumes: a pool called /dev/vgvdo/vpool0 and a volume called /dev/vgvdo/lvvdo.

### Exercise 13-13: Remove a Volume Group and Uninitialize Physical Volume(Server2)


- remove the vgvdo volume group along with the VDO volumes
- uninitialize the physical volume /dev/sdf. 
- confirm the deletion.

1\. Remove the volume group along with the VDO volumes using the vgremove command:
```bash
[root@server2 ~]# sudo vgremove vgvdo -f
  Logical volume "lvvdo" successfully removed.
  Volume group "vgvdo" successfully removed
```

Remember to proceed with caution whenever you perform erase operations.

2\. Execute `sudo vgs` and `sudo lvs` commands for confirmation.
```bash
[root@server2 ~]# sudo vgs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  VG   #PV #LV #SN Attr   VSize   VFree
  rhel   1   2   0 wz--n- <19.00g    0 
  
[root@server2 ~]# sudo lvs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  LV   VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root rhel -wi-ao---- <17.00g                                                    
  swap rhel -wi-ao----   2.00g  
  ```
  
3\. Remove the LVM structures from sdf using the `pvremove` command:
```bash
[root@server2 ~]# sudo pvremove /dev/sdf
  Labels on physical volume "/dev/sdf" successfully wiped.
```

4\. Confirm the removal by running `sudo pvs`.
```bash
[root@server2 ~]# sudo pvs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd1 not found.
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VB428913dd-446a194f PVID none last seen on /dev/sdd2 not found.
  PV         VG   Fmt  Attr PSize   PFree
  /dev/sda2  rhel lvm2 a--  <19.00g    0 
```
 
The disk is now back to its raw state and can be repurposed.

5\. Verify that the sdf disk used in the previous exercises has returned to its original raw state using the `lsblk` command:
```bash
[root@server2 ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda             8:0    0   20G  0 disk 
├─sda1          8:1    0    1G  0 part /boot
└─sda2          8:2    0   19G  0 part 
  ├─rhel-root 253:0    0   17G  0 lvm  /
  └─rhel-swap 253:1    0    2G  0 lvm  [SWAP]
sdb             8:16   0  250M  0 disk 
sdc             8:32   0  250M  0 disk 
sdd             8:48   0  250M  0 disk 
sde             8:64   0  250M  0 disk 
sdf             8:80   0    5G  0 disk 
sr0            11:0    1  9.8G  0 rom 
```

This brings the exercise to an end.
