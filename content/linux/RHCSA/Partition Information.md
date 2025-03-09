## Partition Information


Partition information is stored on the disk in a small region, which is read by the operating system at boot time. 
This region is referred to as the Master Boot Record (MBR) on the BIOS-based systems, and GUID Partition Table (GPT) on the UEFI-based systems. 
At system boot, the BIOS/UEFI scans all storage devices, detects the presence of MBR/GPT areas, identifies the boot disks, loads the bootloader program in memory from the default boot disk, executes the boot code to read the partition table and identify the /boot partition, loads the kernel in memory, and passes control over to it. 
Though MBR and GPT store disk partition information and the boot code.


### Master Boot Record (MBR) 


resides on the first sector of the boot disk. 
was the preferred choice for saving partition table information on x86-based
computers.
with the arrival of bigger and larger hard drives, a new firmware specification (UEFI) was introduced. 
still widely used, but its use is diminishing in favor of UEFI.


allows the creation of three types of partition on a single disk.
primary, extended, and logical--- 
only primary and logical can be used for data storage
extended is a mere enclosure for holding the logical partitions and it is not meant for data storage. 
supports the creation of up to four primary partitions numbered 1 through 4 at a time. 
In case additional partitions are required, one of the primary partitions must be deleted and replaced with an extended
partition to be able to add logical partitions (up to 11) within that extended partition. 
Numbering for logical partitions begins at 5. 
supports a maximum of 14 usable partitions (3 primary and 11 logical) on a single disk.

Cannot address storage space beyond 2TB ue to its 32-bit nature and its 512-byte disk sector size. 
non-redundant; the record it contains is not replicated, resulting in an unbootable system in the event of corruption. 
If your disk is smaller than 2TB and you don't intend to build more than 14 usable partitions, you can use MBR
without issues. 

### GUID Partition Table (GPT) 


ability to construct up to 128 partitions (no concept of extended or logical partitions)
utilize disks larger than 2TB
use 4KB sector size
store a copy of the partition information before the end of the disk for redundancy


allows a BIOS-based system to boot from a GPT disk using the bootloader program stored in a protective MBR at the first disk sector
UEFI firmware also supports the secure boot feature, which only allows signed binaries to boot
