## Storage Review Questions


Q. Where is the partition table information stored on BIOS-based systems?
A. The partition table information is stored on the Master Boot Record.

---

Q\. What is the name of the technology that VDO employs to remove randomization of data blocks?
A\. VDO employs thin provisioning technology to remove data block randomization.

---

Q\. What would sdd3 represent?
A\. sdd3 represents the third partition on the fourth disk.

---

Q\. Provide the command to add physical volumes /dev/sdd1 and /dev/sdc to vg20 volume group.
A\. `vgextend vg20 /dev/sdd1 /dev/sdc`

---

Q\. What are the two commands that you can use to add logical extents to a logical volume?
A\. The `lvextend` and `lvresize` commands.

---

Q\. Provide the command to create a volume group called vg20 on /dev/sdd disk with physical extent size 64MB.
A\. `vgcreate -s 64 vg20 /dev/sdd`

---

Q\. Thin provisioning technology allows us to create logical volumes of sizes larger than the actual physical storage size. True or False?
A\. True.

---

Q\. What is the maximum supported number of usable partitions on a GPT disk?
A\. 128.

---

Q\. Which kernel module is responsible for data block compression in VDO volumes?
A\. The kvdo module is responsible for compressing data blocks in VDO volumes.

---

Q\. What are the three techniques VDO volumes employ for storage conservation?
A\. VDO volumes use thin provisioning, de-duplication, and compression techniques for storage conservation.

---

Q\. What would the command `parted /dev/sdc mkpart pri 1 200m` do?
A\. The command provided will create a primary partition of size 200MB on the sdc disk starting at the beginning of the disk.

---

Q\. What is the maximum number of usable partitions that can be created on an MBR disk?
A\. 14.

---

Q\. The gdisk utility can be used to store partition information in MBR format. True or False?
A\. False. The gdisk tool is only for GPT type tables.

---

Q\. What would the command `parted /dev/sdd mklabel msdos` do?
A\. The command provided will apply msdos label to the sdd disk.

---

Q\. Which file in the /proc file system stores the in-memory partitioning information?
A\. The partitions file.

---

Q\. De-duplication is the process of zero-block elimination. True or False?
A\. False. De-duplication is the process of removing blocks of identical data.

---

Q\. The parted utility may be used to create LVM logical volumes. True or False?
A\. False.

---

Q\. What are the two commands that you can use to reduce the number of logical extents from a logical volume?
A\. The `lvreduce` and `lvresize` commands.

---

Q\. A single disk can be used by both parted and LVM solutions at the same time. True or False?
A\. True. A single disk can be shared between parted-created partitions and LVM.

---

Q\. Provide the command to erase /dev/sdd1 physical volume from vg20 volume group.
A\. `vgreduce vg20 /dev/sdd1`

---

Q\. Provide the command to remove vg20 along with logical and physical volumes it contains.
A\. `vgremove -f vg20`

---

Q\. What is the default size of a physical extent in LVM?
A\. The default PE size is 4MB.

---

Q\. What is the default name for the first logical volume in a volume group?
A\. lvol0 is the default name for the first logical volume created in a volume group.

---

Q\. What is one difference between the `pvs` and `pvdisplay` commands?
A\. The `pvs` command lists basic information about physical volumes whereas the `pvdisplay` command shows the details.

---

Q\. When can a disk or partition be referred to as a physical volume?
A\. After the `pvcreate` command has been executed on it successfully.

---

Q\. Provide the command to remove webvol logical volume from vg20 volume group.
A\. `lvremove /dev/vg20/webvol`

---

Q\. It is necessary to create file system structures in a logical volume before it can be used to store files in it. True or False?
A\. True.

---

Q\. Physical and logical extents are typically of the same size. True or False?
A\. True.

---

Q\. What is the purpose of the `pvremove` command?
A\. The `pvremove` command is used to remove LVM information from a physical volume.

---

Q\. What would the command `pvcreate /dev/sdd` do?
A\. The command provided will prepare the /dev/sdd disk for use in a volume group.

---

Q\. A disk or partition can be added to a volume group without being initialized. True or False?
A\. False. A disk or partition must be initialized before it can be added to a volume group.

---

Q\. Provide the command to create a logical volume called webvol of size equal to 100 logical extents in vg20 volume group.
A\. `lvcreate -l 100 -n webvol vg20`

---

Q\. A volume group can be created without any physical volume in it. True or False?
A\. False.

---

Q\. A partition can be used as an LVM object. True or False?
A\. True.

---

Q\. Which command would you use to view the details of a volume group and its objects?
A\. The `vgdisplay` command with the `-v` option.


