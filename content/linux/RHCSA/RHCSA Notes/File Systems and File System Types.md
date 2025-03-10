## File Systems and File System Types 

**File systems** 
- Can be optimized, resized, mounted, and unmounted independently.
- Must be connected to the directory hierarchy in order to be accessed by users and applications. 
- Mounting may be accomplished automatically at system boot or manually as required.
- Can be mounted or unmounted using their unique identifiers, labels, or device files. 
- Each file system is created in a discrete partition, VDO volume, or logical volume. 
- A typical production RHEL system usually has numerous file systems. 
- During OS installation, only two file systems--- / and /boot ---are created in the default disk layout, but you can design a custom disk layout and construct separate containers to store dissimilar information. 
- Typical additional file systems that may be created during an installation are /home, /opt, /tmp, /usr, and /var. 
- / and /boot---are required for installation and booting.

Storing disparate data in distinct file systems versus storing all data in a single file system offers the following advantages:

- Make any file system accessible (mount) or inaccessible (unmount) to users independent of other file systems. This hides or reveals information contained in that file system.
- Perform file system repair activities on individual file systems
- Keep dissimilar data in separate file systems
- Optimize or tune each file system independently
- Grow or shrink a file system independent of other file systems

3 types of file systems: 
- disk-based, network-based, and memory-based.

**Disk-based**
- Typically created on physical drives using SATA, USB, Fibre Channel, and other technologies. 
- store information persistently
 
**Network-based** 
- Essentially disk-based file systems shared over the network for remote access. 
- store information persistently

**Memory-based** 
- Virtual
- Created at system startup and destroyed when the system goes down.
- data saved in virtual file systems does not survive across system reboots.

 Ext3
- Disk based                  
- The third generation of the extended filesystem. 
- Metadata journaling for faster recovery
- Superior reliability
- Creation of up to 32,000 subdirectories
- supports larger file systems and bigger files than its predecessor 

Ext4
- Disk based
- Successor to Ext3. 
  - Supports all features of Ext3 in addition to:
    - Larger file system size
    - Bigger file size
    - Unlimited number of subdirectories
    - Metadata and quota journaling 
    - Extended user attributes

XFS                     
- Disk based
- Highly scalable and high-performing 64-bit file system.
- Supports:
	- Metadata journaling for faster crash recovery
	- Online defragmentation, expansion, quota journaling, and extended user attributes 
- default file system type in RHEL 9.

VFAT                    
- Disk based                
- Used for post-Windows 95 file system formats on hard disks, USB drives, and floppy disks.

ISO9660                 
- Disk based                
- Used for optical file systems such as CD and DVD.

NFS - (Network File System.)          
- Network based              
- Shared directory or file system for remote access by other Linux systems.

AutoFS  (Auto File System)               
- Network based
- NFS file system set to mount and unmount automatically on remote client systems.

### Extended File Systems

- First generation is obsolete and is no longer supported
- Second, third, and fourth generations are currently available and supported. 
- Fourth generation is the latest in the series and is superior in features and enhancements to its predecessors.
- Structure is built on a partition or logical volume at the time of file system creation. 
- Structure is divided into two sets: 
	- **first set** holds the file system's **metadata** and it is very tiny. 
		- Superblock
			- keeps vital file system structural information:
				- type
				- size
				- status of the file system
				- number of data blocks it contains
				- automatically replicated and maintained at various known locations throughout the file system. 
				- primary superblock
					- superblock at the beginning of the file system 
				- backup superblocks. 
					- I used to supplant the corrupted or lost primary superblock to bring the file system back to its normal state.
					- Copy of the primary
		- Inode table
			- maintains a list of index node (inode) numbers. 
			- Each file is assigned an **inode number** at the time of its creation, and the inode number
				- holds the file's attributes such as:
					- type
					- permissions
					- ownership
					- owning group
					- size
					- last access/modification time
					- holds and keeps track of the pointers to the actual data blocks where the file contents are located.
	- **second set** stores the actual data, and it occupies almost the entire partition or the logical volume (VDO and LVM) space.\

**journaling** 
- Supported by Ext3 and Ext4
- Recover swiftly after a system crash.
- keep track of recent changes in their metadata in a journal (or log). 
- Each metadata update is written in its entirety to the journal after completion. 
- The system peruses the journal of each extended file system following the reboot after a crash to determine if there are any errors
- Lets the system recover the file system rapidly using the latest metadata information stored in its journal.

- Ext3 that supports file systems up to 16TiB and files up to 2TiB, 
- Ext4 supports very large file systems up to 1EiB (ExbiByte) and files up to 16TiB (TebiByte).
	- Uses a series of contiguous physical blocks on the hard disk called extents, resulting in improved read and write performance with reduced fragmentation. 
	- Supports extended user attributes, metadata and quota journaling, etc.

### XFS File System

- High-performing 64-bit extent-based journaling file system type. 
- Allows the creation of file systems and files up to 8EiB (ExbiByte).
- Does not run file system checks at system boot
- Relies on you to use the `xfs_repair` utility to manually fix any issues.
- Sets the extended user attributes and certain mount options by default on new file systems. 
- Enables defragmentation on mounted and active file systems to keep as much data in contiguous blocks as possible for faster access. 
- Inability to shrink.
- Uses journaling for metadata operations, guaranteeing the consistency of the file system against abnormal or forced unmounting. 
- Journal information is read and any pending metadata transactions are replayed when the XFS file system is remounted.
- Speedy input/output performance.
- Can be snapshot in a mounted, active state.

### VFAT File System 

- Extension to the legacy FAT file system (FAT16)
- Supports 255 characters in filenames including spaces and periods
- Does not differentiate between lowercase and uppercase letters. 
- Primarily used on removable media, such as floppy and USB flash drives, for exchanging data between Linux and Windows.

### ISO9660 File System

- For removable optical disc media such as CD/DVD drives
