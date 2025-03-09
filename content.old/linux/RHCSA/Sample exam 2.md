## Appendix B: Sample RHCSA Exam 2

### Setup for Sample Exam 2: 
- Build a virtual machine with RHEL 9 Server with GUI  
- Use a 20GB disk for the OS with default partitioning. 
- Add 1x400MB disk and a network interface. 
- Do not configure the network interface or create a normal user account during installation.

### Tasks: 
Task 01: Using the nmcli command, 
- Configure a network connection on the primary network device:
	- IP address 192.168.0.242/24, 
	- gateway 192.168.0.1, and 
	- nameserver 192.168.0.1. 

Use different IP assignments based on your lab environment.


Task 02: Using the hostnamectl command, 
- set the system hostname to rhcsa2.example.com and alias rhcsa2. 
- Make sure that the new hostname is reflected in the command prompt. 

Task 03: Create a user account called user70 
- With UID 7000 and 
- Comments "I am user70". 
- Set the maximum allowable inactivity for this user to 30 days. 


Task 04: Create a user account called user50 
- With a non-interactive shell. 


Task 05: Attach the RHEL 9 ISO image to the VM and mount it persistently
to /mnt/dvdrom. 
- Define access to both repositories and confirm.


Task 06: Create a logical volume called lv1
- Size equal to 10 LEs in vg1 volume group (create vg1 with PE size 8MB in a partition on the 400MB disk). 
- Initialize the logical volume with XFS type and mount it on /mnt/lvfs1. 
- Create a file called lv1file1 in the mount point. 
- Set the file system to automatically mount at each system reboot. 


Task 07: Add a group called group20
- Change group membership on /mnt/lvfs1 to group20. 
- Set read/write/execute permissions on /mnt/lvfs1 for the owner, group members, and others. 

Task 08: Extend the file system in the logical volume lv1 by 64MB
without unmounting it and without losing any data. 
- Confirm the new size for the logical volume and the file system. 


Task 09: Create a swap partition of size 85MB on the 400MB disk. Use its
UUID and ensure it is activated after every system reboot. 


Task 10: Create a disk partition of size 100MB on the 400MB disk 
- Format it with Ext4 file system structures. 
- Assign label stdlabel to the file system. 
- Mount the file system on /mnt/stdfs1 persistently using the label. 
- Create file stdfile1 in the mount point. 



Task 11: Use the tar and gzip command combination
- create a compressed archive of the /etc directory. 
- Store the archive under /var/tmp using a filename of your choice. 



Task 12: Create a directory /direct01
- apply SELinux contexts for /root to it. 


Task 13: Set up a cron job for user70
- To search for files by the name "core" in the /var directory and 
- copy them to the directory /var/tmp/coredir1. 
- This job should run every Monday at 1:20 a.m.


Task 14: Search for all files in the entire directory structure 
- That have been modified in the past 30 days and save the file listing in the /var/tmp/modfiles.txt file. 


Task 15: Modify the bootloader program and set the default autoboot
timer value to 2 seconds.


Task 16: Determine the recommended tuning profile for the system and
apply it. 


Task 17: Configure Chrony to synchronize system time with the hardware
clock. Remove all other NTP sources. 


Task 18: Install package group called "Development Tools" 
- capture its information in /var/tmp/systemtools.out file. 


Task 19: Lock user account user70. 
- Use regular expressions to capture the line that shows the lock and store the output in file /var/tmp/user70.lock. 


Task 20: Write a bash shell script 
- so that it prints RHCSA when RHCE is passed as an argument, and vice versa. 
- If no argument is provided, the script should print a usage message and quit with exit value 5. 


Task 21: Launch a rootful container and configure it to auto-start via
systemd. 


Task 22: Launch a rootless container 
- As user80 with /data01 mapped to /data01 
- using the latest version of the ubi9 image. 
- Configure a systemd service to auto-start the container on system reboots without the need for user80 to log in. 
- Create files under the shared mount point and validate data persistence.
