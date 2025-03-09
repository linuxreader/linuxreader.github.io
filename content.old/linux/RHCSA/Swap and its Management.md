## Swap and its Management 

- Move pages of idle data between physical memory and swap. 
- Swap areas act as extensions to the physical memory.
- May be activated or deactivated independent of swap spaces located in other partitions and volumes.
- The system splits the physical memory into small logical chunks called pages and maps their physical locations to virtual locations on the swap to facilitate access by system processors. 
- This physical-to-virtual mapping of pages is stored in a data structure called page table, and it is maintained by the kernel.
- When a program or process is spawned, it requires space in the physical memory to run and be processed. 
- Although many programs can run concurrently, the physical memory cannot hold all of them at once. 
- The kernel monitors the memory usage. 
- As long as the free memory remains above a high threshold, nothing happens.
- When the free memory falls below that threshold, the system starts moving selected idle pages of data from physical memory to the swap space to make room to accommodate other programs. 
- This piece in the process is referred to as **page out**. 
- Since the system CPU performs the process execution in around-robin fashion, when the system needs this paged-out data for execution, the CPU looks for that data in the physical memory and a **pagefault** occurs, resulting in moving the pages back to the physical memory from the swap. 
- This return of data to the physical memory is referred to as **page in**. 
- The entire process of paging data out and in is known as **demand paging**.

- RHEL systems with less physical memory but high memory requirements can become over busy with paging out and in. 
- When this happens, they do not have enough cycles to carry out other useful tasks, resulting in degraded system performance. 
- The excessive amount of paging that affects the system performance is called **thrashing**.
- When thrashing begins, or when the free physical memory falls below a low threshold, the system deactivates idle processes and prevents new processes from being launched. 
- The idle processes are only reactivated, and new processes are only allowed to be started when the system discovers that the available physical memory has climbed above the threshold level and thrashing has ceased.
### Determining Current Swap Usage 

- Size of a swap area should not be less than the amount of physical memory.
- Depending on workload requirements, it may be twice the size or larger. 
- It is also not uncommon to see systems with less swap than the actual amount of physical memory. 
- This is especially witnessed on systems with a huge physical memory size.

### `free` command 
- View memory and swap space utilization.
- view how much physical memory is installed (total), used (used), available (free), used by shared library routines (shared), holding data before it is written to disk (buffers), and used to store frequently accessed data (cached) on the system. The 
- `-h`
	- list the values in human-readable format,
- `-k`
	- for KB, 
- `-m`
	- for MB, 
- `-g` 
	- for GB,
- `-t` 
	- display a line with the "total" at the bottom of the output. 

```bash
[root@server2 mapper]# free -ht
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       783Mi       714Mi       5.0Mi       440Mi       991Mi
Swap:          2.0Gi          0B       2.0Gi
Total:         3.7Gi       783Mi       2.7Gi
```

Try `free -hts 3` and `free -htc 2` to refresh the output every three seconds (-s) and to display the output twice (-c).


- Reads memory and swap information from the */proc/meminfo* file to produce the report. The values are shown in KBs by
default, and they are slightly off from what is shown above with `free`. Here are the relevant fields from this file:

```bash
[root@server2 mapper]# cat /proc/meminfo | grep -E 'Mem|Swap'
MemTotal:        1818080 kB
MemFree:          731724 kB
MemAvailable:    1015336 kB
SwapCached:            0 kB
SwapTotal:       2097148 kB
SwapFree:        2097148 kB
```
### Prioritizing Swap Spaces 

- You may find multiple swap areas configured and activated to meet the workload demand. 
- The default behavior of RHEL is to use the first activated swap area and move on to the next when the first one is exhausted.
- The system allows us to prioritize one area over the other by adding the option "pri" to the swap entries in the fstab file. 
- This flag supports a value between -2 and 32767 with -2 being the default. 
- A higher value of "pri" sets a higher priority for the corresponding swap region. 
- For swap areas with an identical priority, the system alternates between them.
### Swap Administration Commands 

- In order to create and manage swap spaces on the system, the `mkswap`, `swapon`, and `swapoff` commands are available. 
- Use `mkswap` to initialize a partition for use as a swap space. 
- Once the swap area is ready, you can activate or deactivate it from the command line with the help of the other two commands, 
- Can also set it up for automatic activation by placing an entry in the fstab file. 
- The fstab file accepts the swap area's device file, UUID, or label.
### Lab: Create and Activate Swap in Partition and Logical Volume (server 2)

- Create one swap area in a new 40MB partition called *sdb3* using the `mkswap` command. 
- Create another swap area in a 140MB logical volume called swapvol in vgfs. 
- Add their entries to the /etc/fstab file for persistence. 
- Use the UUID and priority 1 for the partition swap and the device file and priority 2 for the logical volume swap. 
- Activate them and use appropriate tools to validate the activation.

EXAM TIP: Use the `lsblk` command to determine available disk space.

1\. Use `parted print` on the sdb disk and the `vgs` command on the vgfs volume group to determine available space for a new 40MB partition and a 144MB logical volume:
```bash
[root@server2 mapper]# sudo parted /dev/sdb print
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sdb: 262MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End    Size    Type     File system  Flags
 1      1049kB  101MB  99.6MB  primary  ext4
 2      102MB   201MB  99.6MB  primary  fat16

[root@server2 mapper]# sudo vgs vgfs
  Devices file sys_wwid t10.ATA_VBOX_HARDDISK_VBa5e3cbf7-10921e08 PVID qeP9dCevNnTy422I8p18NxDKQ2WyDodU last seen on /dev/sdf1 not found.
  VG   #PV #LV #SN Attr   VSize   VFree  
  vgfs   2   2   0 wz--n- 400.00m 144.00m
```

The outputs show 49MB (250MB minus 201MB) free space on the sdb disk and 144MB free space in the volume group.

2\. Create a partition called sdb3 of size 40MB using the parted command:
```bash
[root@server2 mapper]# sudo parted /dev/sdb mkpart primary 202 242
Information: You may need to update /etc/fstab.
```

3\. Create logical volume swapvol of size 144MB in vgs using the `lvcreate` command:
```bash
[root@server2 mapper]# sudo lvcreate -L 144 -n swapvol vgfs               
  Logical volume "swapvol" created.
```

4\. Construct swap structures in sdb3 and swapvol using the mkswap command:
```bash
[root@server2 mapper]# sudo mkswap /dev/sdb3
Setting up swapspace version 1, size = 38 MiB (39841792 bytes)
no label, UUID=a796e0df-b1c3-4c30-bdde-dd522bba4fff

[root@server2 mapper]# sudo mkswap /dev/vgfs/swapvol
Setting up swapspace version 1, size = 144 MiB (150990848 bytes)
no label, UUID=88196e73-feaf-4137-8743-f9340296aeec
```

5\. Edit the fstab file and add entries for both swap areas for auto-activation on reboots. Obtain the UUID for partition swap with
`lsblk -f /dev/sdb3` and use the device file for logical volume. Specify their priorities.
```bash
UUID=a796e0df-b1c3-4c30-bdde-dd522bba4fff swap swap pri=1 0 0
/dev/vgfs/swapvol swap swap pri=2 0 0   
```

EXAM TIP: You will not be given any credit for this work if you forget to add entries to the fstab file.

6\. Determine the current amount of swap space on the system using the swapon command:
```bash
[root@server2]# sudo swapon
NAME      TYPE      SIZE USED PRIO
/dev/dm-1 partition   2G   0B   -2
```

There is one 2GB swap area on the system and it is configured at the default priority of -2.

7\. Activate the new swap regions using the swapon command:
```bash
[root@server2]# sudo swapon -a
```

8\. Confirm the activation using the swapon command or by viewing the /proc/swaps file:
```bash
[root@server2 mapper]# sudo swapon
NAME      TYPE      SIZE USED PRIO
/dev/dm-1 partition   2G   0B   -2
/dev/sdb3 partition  38M   0B    1
/dev/dm-7 partition 144M   0B    2
```

```bash
[root@server2 mapper]# cat /proc/swaps
Filename				Type		Size		Used		Priority
/dev/dm-1                               partition	2097148		0		-2
/dev/sdb3                               partition	38908		0		1
/dev/dm-7                               partition	147452		0		2
#dm is device mapper
```

9\. Issue the free command to view the reflection of swap numbers on the Swap and Total lines:
```bash
[root@server2 mapper]# free -ht
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       793Mi       706Mi       5.0Mi       438Mi       981Mi
Swap:          2.2Gi          0B       2.2Gi
Total:         3.9Gi       793Mi       2.9Gi
```

