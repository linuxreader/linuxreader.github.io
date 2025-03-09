## Appendix A: Sample RHCSA Exam 1

Time Duration:3 hours
Passing Score:70% (210 out of 300)

All settings performed in the virtual machines must survive system reboots, or you will lose marks.
### Setup for Sample Exam 1:

RHEL 9 Server with GUI 
20GB disk for the OS with default partitioning. 
2x300MB disks and a network interface. (.293GiB for virt-manager)
Do not configure the network interface or create a normal user account during installation.

### Tasks:

**Task 01:** 
Assuming the root user password is lost, and your system is running in multi-user target with no current root session open. Reboot the system into an appropriate target level and reset the root user password to root1234. (Exercise 11-2). After completing this task, log in as the root user and perform the remaining tasks presented below.


**Task 02:** 
Using a manual method (create/modify files by hand), 
- configure a network connection on the primary network device with:
- IP address 192.168.0.241/24
- gateway 192.168.0.1
- nameserver 192.168.0.1

Use different IP assignments based on your lab setup.

**Task 03:** 
Using a manual method (modify file by hand), set the system hostname to rhcsa1.example.com and alias rhcsa1. Make sure that the new hostname is reflected in the command prompt.


Task 04: Set the default boot target to multi-user. 


Task 05: Set SELinux to permissive mode. 


**Task 06:** 
Perform a case-insensitive search for all lines in the /usr/share/dict/linux.words file that begin with the pattern "essential". Redirect the output to /var/tmp/pattern.txt file. Make sure that empty lines are omitted. 


**Task 07:** 
Change the primary command prompt for the root user to display the hostname, username, and current working directory information in that order. Update the per-user initialization file for permanence.


**Task 08:** 
Create user accounts called user10, user20, and user30. Set their passwords to Temp1234. Make user10 and user30 accounts to expire on December 31, 2023. 


**Task 09:** 
Create a group called group10 and add user20 and user30 as secondary members. 


**Task 10:** 
Create a user account called user40 with UID 2929. Set the password to user1234. 


**Task 11:** 
Attach the RHEL 9 ISO image to the VM and mount it persistently to /mnt/cdrom. Define access to both repositories and confirm. 


**Task 12:** 
Create a logical volume called lvol1 of size 280MB in vgtest volume group. Mount the ext4 file system persistently to /mnt/mnt1


**Task 13:** 
Change group membership on /mnt/mnt1 to group10. Set read/write/execute permissions on /mnt/mnt1 for group members and revoke all permissions for public.


**Task 14:** 
Create a logical volume called lvswap of size 280MB in vgtest volume group. Initialize the logical volume for swap use. Use the UUID and place an entry for persistence. 


**Task 15:** 
Use the combination of tar and bzip2 commands to create a compressed archive of the /usr/lib directory. Store the archive under /var/tmp as usr.tar.bz2. 


**Task 16:** 
Create a directory hierarchy /dir1/dir2/dir3/dir4 and apply SELinux contexts of /etc on it recursively. 


**Task 17:** 
Enable access to the atd service for user20 and deny for user30. 

**Task 18:** 
Add a custom message "This is RHCSA sample exam on \$(date) by \$LOGNAME" to the /var/log/messages file as the root user. Use regular expression to confirm the message entry to the log file. 


**Task 19:** 
Allow user20 to use sudo without being prompted for their password.

**Task 20:** 
Write a bash shell script to create three user accounts---user555, user666, and user777---with no login shell and passwords matching their usernames. The script should also extract the three usernames from the /etc/passwd file and redirect them into /var/tmp/newusers. 


**Task 21:** 
Launch a container as user20 using the latest version of ubi8 image. Configure the container to auto-start at system reboots without the need for user20 to log in.


**Task 22:** 
Launch a container as user20 using the latest version of ubi9 image with two environment variables SHELL and HOSTNAME. Configure the container to auto-start via systemd without the need for user20 to log in. Connect to the container and verify variable settings. 

Reboot the system and validate the configuration. 