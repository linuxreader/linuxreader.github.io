## Shell Scripting DIY Labs

### Lab: Write Script to Create Logical Volumes 

- Present 2x1GB virtual disks to server40 in VirtualBox Manager. 
- As user1 with sudo on server40, write a single bash script to create 2x400MB partitions on each disk using parted and then bring both partitions into LVM control with the `pvcreate` command. 
```bash
 vim /usr/local/bin/lvscript.sh
```

- Create a volume group called `vgscript` and add both physical volumes to it. 
- Create three logical volumes each of size 200MB and name them lvscript1, lvscript2, and lvscript3.
```bash
#!/bin/bash
for DEVICE in "/dev/sd"{b..c}
do
        echo "Creating partition 1 with the size of 400MB on $DEVICE"
        parted $DEVICE mklabel msdos
        parted $DEVICE mkpart primary 1 401
        pvcreate $DEVICE[1]

        echo "Creating partition 2 with the size of 400MB on $DEVICE"
        parted $DEVICE mkpart primary 402 802
        pvcreate $DEVICE[2]
        vgcreate vgscript $DEVICE[1] $DEVICE[2]
done
for LV in "lvscript"{1..3}
do
        echo "Creating logical volume $LV in volume group vgscript with the size of 200MB"
        lvcreate vgscript -L 200MB -n $LV
done
```

### Lab: Write Script to Create File Systems 

- Write another bash script to create xfs, ext4, and vfat file system structures in the logical volumes, respectively. 
- Create mount points /mnt/xfs, /mnt/ext4, and /mnt/vfat, and mount the file systems. 
- Include the df command with -h in the script to list the mounted file systems.
```bash
vim /usr/local/bin/fsscript.sh
```

```bash
[root@server40 ~]# chmod +x /usr/local/bin/fsscript.sh
```

```bash
#!/bin/bash
for DEVICE in lvscript{1..3}
do
if [ "$DEVICE" = lvscript1 ]
then
        echo "Creating xfs filesystem on logical volume lvscript1"
        echo
        mkfs.xfs /dev/vgscript/lvscript1
        echo "Creating /mnt/xfs"
        mkdir /mnt/xfs
        echo "Mounting filesystem"
        mount /dev/vgscript/lvscript1 /mnt/xfs
elif [ "$DEVICE" = lvscript2 ]
then    
        echo "Creating ext4 filesystem on logical volume lvscript2"
        echo
        mkfs.ext4 /dev/vgscript/lvscript2
        echo "Creating /mnt/ext4"
        mkdir /mnt/ext4
        echo "Mounting filesystem"
        mount /dev/vgscript/lvscript2 /mnt/ext4

elif [ "$DEVICE" = lvscript3 ]
then    
        echo "Creating vfat filesystem on logical volume lvscript3"
        echo
        mkfs.vfat /dev/vgscript/lvscript3
        echo "Creating /mnt/vfat"
        mkdir /mnt/vfat
        echo "Mounting filesystem"
        mount /dev/vgscript/lvscript3 /mnt/vfat
        echo
        echo
        echo "Done!"
                df -h
else
        echo

fi
done
```

```bash
[root@server40 ~]# fsscript.sh
Creating xfs filesystem on logical volume lvscript1

Filesystem should be larger than 300MB.
Log size should be at least 64MB.
Support for filesystems like this one is deprecated and they will not be supported in future releases.
meta-data=/dev/vgscript/lvscript1 isize=512    agcount=4, agsize=12800 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=51200, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=1368, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Creating /mnt/xfs
Mounting filesystem
Creating ext4 filesystem on logical volume lvscript2

mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 204800 1k blocks and 51200 inodes
Filesystem UUID: b16383bf-7b65-4a00-bb6d-c297733f60b3
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done 

Creating /mnt/ext4
Mounting filesystem
Creating vfat filesystem on logical volume lvscript3

mkfs.fat 4.2 (2021-01-31)
Creating /mnt/vfat
Mounting filesystem


Done!
```
### Lab 21-3: Write Script to Configure New Network Connection Profile 

- Present a new network interface to server40 in VirtualBox Manager. 
- As user1 with sudo on server40, write a single bash script to run the nmcli command to configure custom IP assignments (choose your own settings) on the new network device. 
- Make a copy of the /etc/hosts file as part of this script. 
- Choose a hostname of your choice and add a mapping to the /etc/hosts file without overwriting existing file content.
```bash
[root@server40 ~]# vim /usr/local/bin/network.sh
```

```bash
#!/bin/bash
cp /etc/hosts /etc/hosts.bak &&
nmcli c a type Ethernet con-name enp0s9 ifname enp0s9 ip4 10.32.32.2/24 gw4 10.32.32.1
echo "10.32.33.14 frog.example.com frog" >> /etc/hosts
```

```bash
[root@server40 ~]# chmod +x /usr/local/bin/network.sh
[root@server40 ~]# network.sh
Connection 'enp0s9' (5a342243-e77b-452e-88e2-8838d3ecea6d) successfully added.
```

```bash
[root@server40 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.32.33.14 frog.example.com frog
```

```bash
[root@server40 ~]# ip a
enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:1d:f4:c1 brd ff:ff:ff:ff:ff:ff
    inet 10.32.32.2/24 brd 10.32.32.255 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::2c5d:31cc:1d79:6b43/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

```bash
[root@server40 ~]# nmcli d s
DEVICE  TYPE      STATE                   CONNECTION 
enp0s3  ethernet  connected               enp0s3     
enp0s8  ethernet  connected               enp0s8     
enp0s9  ethernet  connected               enp0s9     
lo      loopback  connected (externally)  lo  
```
