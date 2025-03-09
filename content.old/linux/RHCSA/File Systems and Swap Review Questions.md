## File Systems and Swap Review Questions 

Q. What type of information does the `blkid` command display?
A. Attributes for block devices.

---

Q\. What would the command `xfs_admin -L bootfs /dev/sda1` do?
A. Apply the specified label to the XFS file system in /dev/sda1.

---

Q. The `lsblk` command cannot be used to view file system UUIDs. True or False?
A. False.

---

Q. Which two file systems are created in a default RHEL 9 installation?
A. / and /boot.

---

Q. What would the command `lvresize -r -L +30 /dev/vg02/lvol2` do?
A. Expand the logical volume lvol2 in volume group vg02 along with the file system it contains by 30MB.

---

Q. XFS is the default file system type in RHEL 9. True or False?
A. True.

---

Q. What is the process of paging out and paging in known as?
A. Demand paging.

---

Q. What would the command `mkswap /dev/sdc2` do?
A. The command provided will create swap structures in the /dev/vdc2 partition.

---

Q. What would happen if you tried to mount a file system on a directory that already contains files in it?
A. The files in the directory will hide.

---

Q. A UUID is always assigned to a file system at its creation time. True or False?
A. True.

---

Q. Arrange the activities to create and activate a swap space while ensuring persistence: 
(a) `swapon`
(b) update fstab
(c) `mkswap`
(d) reboot
A. c/a/b or c/b/a.

---

Q. The difference between the primary and backup superblocks is that the primary superblock includes pointers to the data blocks where the actual file contents are stored whereas the backup superblocks don't. True or False?
A. False.

---

Q. What would the command `mkfs.ext4 /dev/vgtest/lvoltest` do?
A. The command provided will format /dev/vgtest/lvoltest logical volume with Ext4 file system type.

---

Q. Arrange the tasks in correct sequence: 
- umount file system
- mount file system
- create file system
- remove file system
A. Create, mount, unmount, and remove.

---


Q. Which of these statements is wrong with respect to file systems:
(a) optimize each file system independently
(b) keep dissimilar data in separate file systems
(c) grow and shrink a file system independent of other file systems
(d) file systems cannot be expanded independent of other file systems.
A. d is incorrect.

---

Q. Which command can be used to create a label for an XFS file system?
A. The `xfs_admin` command can be used to create a label for an XFS file system.

---


Q. Which virtual file contains information about the current swap?
A. The */proc/swaps* file contains information about the current swap.

---

Q. The */etc/fstab* file can be used to activate swap spaces automatically at system reboots. True or False?
A. True.

---

Q. What is the default file system type used for optical media?
A. The default file system type for optical devices is ISO9660.

---

Q. The `xfs_repair` command must be run on a mounted file system. True or False?
A. False.

---

Q. Provide two commands that can be used to activate and deactivate swap spaces manually.
A. The `swapon` and `swapoff` commands.

---

Q. Provide the fstab file entry for an Ext4 file system located in device /dev/mapper/vg20-lv1 and mounted with default options on the
/ora1 directory.
A. /dev/mapper/vg20-lv1 /ora1 ext4 defaults 0 0

---

Q. What is the name of the virtual file that holds currently mounted file system information?
A. */proc/self/mounts*

---

Q. Both Ext3 and Ext4 file system types support journaling. True or False?
A. True.

---

Q. What would the `mount` command do with the `-a` switch?
A. The command provided will mount all file systems listed in the */etc/fstab* file but are not currently mounted.

---

Q. What would the command `df -t xfs` do?
A. The command provided will display all mounted file systems of type XFS.

---

Q. Which command can be used to determine the total and used physical memory and swap in the system?
A. The `free` command.

---

Q. Name three commands that can be employed to view the UUID of an XFS file system?
A. You can use the `xfs_admin`, `lsblk`, and `blkid` commands to view the UUID of an XFS file system.


