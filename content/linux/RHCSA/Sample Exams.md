+++
title = 'Sample Exams'
description = 'Sample Exams 1-4 in the book'
+++
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
*done

**Task 02:** 
Configure a network connection on the primary network device with:
- IP address 192.168.0.241/24
- gateway 192.168.0.1
- nameserver 192.168.0.1

Use different IP assignments based on your lab setup.
*done

**Task 03:** 
Using a manual method (modify file by hand), set the system hostname to rhcsa1.example.com and alias rhcsa1. Make sure that the new hostname is reflected in the command prompt.
*done

**Task 04:**
Set the default boot target to multi-user. 
*done

**Task 05:**
Set SELinux to permissive mode. 
*done

**Task 06:** 
Perform a case-insensitive search for all lines in the /usr/share/dict/linux.words file that begin with the pattern "essential". Redirect the output to /var/tmp/pattern.txt file. Make sure that empty lines are omitted. 
*done

**Task 07:** 
Change the primary command prompt for the root user to display the hostname, username, and current working directory information in that order. Update the per-user initialization file for permanence.
*done

**Task 08:** 
Create user accounts called user10, user20, and user30. Set their passwords to Temp1234. Make user10 and user30 accounts to expire on December 31, 2023. 
*done

**Task 09:** 
Create a group called group10 and add user20 and user30 as secondary members. 
*done

**Task 10:** 
Create a user account called user40 with UID 2929. Set the password to user1234. 
*done

**Task 11:** 
Attach the RHEL 9 ISO image to the VM and mount it persistently to /mnt/cdrom. Define access to both repositories and confirm. 
*done

**Task 12:** 
Create a logical volume called lvol1 of size 280MB in vgtest volume group. Mount the ext4 file system persistently to /mnt/mnt1
*30 minutes in  done

**Task 13:** 
Change group membership on /mnt/mnt1 to group10. Set read/write/execute permissions on /mnt/mnt1 for group members and revoke all permissions for public.
*done

**Task 14:** 
Create a logical volume called lvswap of size 280MB in vgtest volume group. Initialize the logical volume for swap use. Use the UUID and place an entry for persistence. 
*done

**Task 15:** 
Use the combination of tar and bzip2 commands to create a compressed archive of the /usr/lib directory. Store the archive under /var/tmp as usr.tar.bz2. 
*done

**Task 16:** 
Create a directory hierarchy /dir1/dir2/dir3/dir4 and apply SELinux contexts of /etc on it recursively. 
*done

**Task 17:** 
Enable access to the atd service for user20 and deny for user30. 
*done 

**Task 18:** 
Add a custom message "This is RHCSA sample exam on \$(date) by \$LOGNAME" to the /var/log/messages file as the root user. Use regular expression to confirm the message entry to the log file. 
*done

**Task 19:** 
Allow user20 to use sudo without being prompted for their password.
*done

**Task 20:** 
Write a bash shell script to create three user accounts---user555, user666, and user777---with no login shell and passwords matching their usernames. The script should also extract the three usernames from the /etc/passwd file and redirect them into /var/tmp/newusers. 
*done (1 hour in)


**Task 21:** 
Launch a container as user20 using the latest version of ubi8 image. Configure the container to auto-start at system reboots without the need for user20 to log in.
*done

**Task 22:** 
Launch a container as user20 using the latest version of ubi9 image with two environment variables SHELL and HOSTNAME. Configure the container to auto-start via systemd without the need for user20 to log in. Connect to the container and verify variable settings. 
*done 

Reboot the system and validate the configuration. 
## Appendix B: Sample RHCSA Exam 2
Started at 13:15 on the first timer
### Setup for Sample Exam 2: 
- Build a virtual machine with RHEL 9 Server with GUI  
- Use a 20GB disk for the OS with default partitioning. 
- Add 1x400MB disk and a network interface. (.381GiB in Virt-Manager)
- Do not configure the network interface or create a normal user account during installation.

### Tasks: 
**Task 01:** Using the nmcli command, 
- Configure a network connection on the primary network device:
	- IP address 192.168.0.242/24, 
	- gateway 192.168.0.1, and 
	- nameserver 192.168.0.1. 

Use different IP assignments based on your lab environment.
*done

**Task 02:** Using the hostnamectl command, 
- set the system hostname to rhcsa2.example.com and alias rhcsa2. 
- Make sure that the new hostname is reflected in the command prompt. 
*done

**Task 03:** Create a user account called user70 
- With UID 7000 and 
- Comments "I am user70"
- Set the maximum allowable inactivity for this user to 30 days. 
*done

**Task 04:** Create a user account called user50 
- With a non-interactive shell. 
*done

**Task 05:** Attach the RHEL 9 ISO image to the VM and mount it persistently
to /mnt/dvdrom. 
- Define access to both repositories and confirm.
*done 13:15 minutes in

**Task 06:** Create a logical volume called lv1
- Size equal to 10 LEs in vg1 volume group (create vg1 with PE size 8MB in a partition on the 400MB disk). 
- Initialize the logical volume with XFS type and mount it on /mnt/lvfs1. 
- Create a file called lv1file1 in the mount point. 
- Set the file system to automatically mount at each system reboot. 
*done

**Task 07:** Add a group called group20
- Change group membership on /mnt/lvfs1 to group20. 
- Set read/write/execute permissions on /mnt/lvfs1 for the owner, group members, and others. 
*done

**Task 08:** Extend the file system in the logical volume lv1 by 64MB
without unmounting it and without losing any data. 
- Confirm the new size for the logical volume and the file system. 
*done

**Task 09:** Create a swap partition of size 85MB on the 400MB disk. Use its
UUID and ensure it is activated after every system reboot. 
*done

**Task 10:** Create a disk partition of size 100MB on the 400MB disk 
- Format it with Ext4 file system structures. 
- Assign label stdlabel to the file system. 
- Mount the file system on /mnt/stdfs1 persistently using the label. 
- Create file stdfile1 in the mount point. 
*done 43 minutes in


**Task 11:** Use the tar and gzip command combination
- create a compressed archive of the /etc directory. 
- Store the archive under /var/tmp using a filename of your choice. 
*done


**Task 12:** Create a directory /direct01
- apply SELinux contexts for /root to it. 
*done

**Task 13:** Set up a cron job for user70
- To search for files by the name "core" in the /var directory and 
- copy them to the directory /var/tmp/coredir1. 
- This job should run every Monday at 1:20 a.m.
*done

**Task 14:** Search for all files in the entire directory structure 
- That have been modified in the past 30 days and save the file listing in the /var/tmp/modfiles.txt file. 
*done

**Task 15:** Modify the bootloader program and set the default autoboot
timer value to 2 seconds.
*boot

**Task 16:** Determine the recommended tuning profile for the system and
apply it. 
*done

**Task 17:**
Configure Chrony to synchronize system time with the hardware
clock. Remove all other NTP sources. 
*done

**Task 18:** Install package group called "Development Tools" 
- capture its information in /var/tmp/systemtools.out file. 
*done 1hour 13 in

**Task 19:** Lock user account user70. 
- Use regular expressions to capture the line that shows the lock and store the output in file /var/tmp/user70.lock. 
*bash

**Task 20:** Write a bash shell script 
- so that it prints RHCSA when RHCE is passed as an argument, and vice versa. 
- If no argument is provided, the script should print a usage message and quit with exit value 5. 
*done 1 hour 43 in

**Task 21:** Launch a rootful container and configure it to auto-start via
systemd. 
*done

**Task 22:** Launch a rootless container 
- As user80 with /data01 mapped to /data01 
- using the latest version of the ubi9 image. 
- Configure a systemd service to auto-start the container on system reboots without the need for user80 to log in. 
- Create files under the shared mount point and validate data persistence.
*done
1 hour 54 minutes

## Appendix C: Sample RHCSA Exam 3


### Setup for Sample Exam 3: 

Two virtual machines with RHEL 9 Server with GUI 
- Use a 20GB disk for the OS with default partitioning. 
- Add 1x5GB disk to VM1 and a network interface to both virtual machines. 
- Do not configure the network interfaces or create a normal user account during installation.

### Tasks:

Task 01: On VM1, set the system hostname to rhcsa3.example.com and alias
rhcsa3 using the hostnamectl command. 
- Make sure that the new hostname is reflected in the command prompt. 

Task 02: On rhcsa3, configure a network connection on the primary
network device:
- IP address 192.168.0.243/24, gateway 192.168.0.1, and nameserver 192.168.0.1 using the nmcli command 


Task 03: On VM2, set the system hostname to rhcsa4.example.com 
- alias rhcsa4 using a manual method (modify file by hand). 
- Make sure that the new hostname is reflected in the command prompt. 


Task 04: On rhcsa4, configure a network connection on the primary
network device 
- IP address 192.168.0.244/24, gateway 192.168.0.1, and nameserver 192.168.0.1 using a manual method (create/modify files by hand). 


Task 05: Run "ping -c2 rhcsa4" on rhcsa3. 
- Run "ping -c2 rhcsa3" on rhcsa4. 
- You should see 0% loss in both outputs. 


Task 06: On rhcsa3 and rhcsa4, attach the RHEL 9 ISO image to the VM and
mount it persistently to /mnt/sr0. 
- Define access to both repositories and confirm. 


Task 07: On rhcsa3, add HTTP port 8300/TCP to the SELinux policy
database persistently. 

Task 08: On rhcsa3, create LVM VDO volume 
- Called vdo1 on the 5GB disk
- logical size 20GB and mounted with Ext4 structures on /mnt/vdo1.


Task 09: Configure NFS service on rhcsa3
- share /rh_share3 with rhcsa4. 
- Configure AutoFS direct map on rhcsa4 to mount /rh_share3 on /mnt/rh_share4. 
- User user80 (create on both systems) should be able to create files under the share on the NFS server as well as under the mount point on the NFS client.

Task 09: Configure NFS service on rhcsa3
- share /rh_share3 with rhcsa4. 
- Configure AutoFS direct map on rhcsa4 to mount /rh_share3 on /mnt/rh_share4. 
- User user80 (create on both systems) should be able to create files under the share on the NFS server as well as under the mount point on the NFS client.

Task 10: Configure NFS service on rhcsa4 and 
- share the home directory for user60 (create user60 on both systems) with rhcsa3.
- Configure AutoFS indirect map on rhcsa3 to automatically mount the home directory under /nfsdir when user60 logs on to rhcsa3. 


Task 11: On rhcsa3, create a group called group30 with GID 3000
- add user60 and user80 to this group. 
- Create a directory called /sdata, enable setgid bit on it.
- Add write permission bit for group members.
- Set ownership and owning group to root and group30. 
- Create a file called file1 under /sdata as user60 and modify the file as user80 successfully.


Task 12: On rhcsa3, create directory /var/dir1 
- with full permissions for everyone. 
- Disallow non-owners to remove files. 
- Test by creating file /var/dir1/stkfile1 as user60 and removing it as user80. 


Task 13: On rhcsa3, search for all manual pages for the description containing the keyword "password".
- redirect the output to file /var/tmp/man.out. 


Task 14: On rhcsa3, create file lnfile1 under /var/tmp 
- Create one hard link /var/tmp/lnfile2 and one soft link /boot/file1. 
- Edit lnfile1 using one link at a time and confirm.


Task 15: On rhcsa3, install software group called "Legacy UNIX
Compatibility". 

Task 16: On rhcsa3, add the http service to "external" firewalld zone persistently. 

Task 17: On rhcsa3, set SELinux type shadow_t on a new file testfile1 in
/usr
- ensure that the context is not affected by a SELinux
relabeling. 

Task 18: Configure passwordless ssh access for user60 from rhcsa3 to
rhcsa4. (Exercise 18-2).


Task 19: Write a bash shell script
- Checks for the existence of files (not directories) under the /usr/bin directory
- That begin with the letters "ac" and display their statistics (the stat command). 

Task 20: On rhcsa3, write a containerfile to include the ls and pwd
commands in a custom ubi8 image. 
- Launch a named rootless container as user60 using this image. 
- Confirm command execution. 

Task 21: On rhcsa3, launch a named rootless container as user60 
- with host port 10000 mapped to container port 80. 
- Employ the latest version of the ubi8 image. 
- Configure a systemd service to autostart the container without the need for user60 to log in. 
- Validate port mapping using an appropriate podman subcommand. 


Task 22: On rhcsa3, launch another named rootless container (use a
unique name for the container) as user60 with /host_data01 mapped to
/container_data01, one variable ENVIRON=Exam, and host port 1050 mapped
to container port 1050. Use the latest version of the ubi9 image.
Configure a separate systemd service to auto-start the container without
the need for user60 to log in. Create a file under the shared directory
and validate data persistence. Verify port mapping and variable settings
using appropriate podman subcommands. 

## Appendix D: Sample RHCSA Exam 4 
(Using server1 and server2)
### Setup for Sample Exam 4: 

Build two virtual machines with RHEL 9 Server with GUI (Exercises 1-1
and 1-2). Use a 20GB disk for the OS with default partitioning. Add
1x5GB disk to VM2 and a network interface to both virtual machines. Do
not configure the network interfaces or create a normal user account
during installation.

### Tasks:

Task 01: On VM1, set the system hostname to rhcsa5.example.com and alias
rhcsa5 using the hostnamectl command. Make sure that the new hostname is
reflected in the command prompt. (Exercises 15-1 and 15-5).
*done

Task 02: On rhcsa5, configure a network connection on the primary
network device with IP address 192.168.0.245/24, gateway 192.168.0.1,
and nameserver 192.168.0.1 using the nmcli command. Use different IP
assignments based on your lab environment. (Exercise 15-4).
*done

Task 03: On VM2, set the system hostname to rhcsa6.example.com and alias rhcsa6 using a manual method (modify file by hand). Make sure that the
new hostname is reflected in the command prompt. (Exercises 15-1 and
15-5).
*done

Task 04: On rhcsa6, configure a network connection on the primary
network device with IP address 192.168.0.246/24, gateway 192.168.0.1,
and nameserver 192.168.0.1 using a manual method (create/modify files by
hand). Use different IP assignments based on your lab environment.
(Exercise 15-3).
*done

Task 05: Run "ping -c2 rhcsa6" on rhcsa5. Run "ping -c2 rhcsa5" on
rhcsa6. You should see 0% loss in both outputs. (Exercise 15-5).
*done

Task 06: On rhcsa5 and rhcsa6, attach the RHEL 9 ISO image to the VM and
mount it persistently to /mnt/sr0. Define access to both repositories
and confirm. (Exercise 9-1).
*done
*30 minutes in*

Task 07: Export /share5 on rhcsa5 and mount it to /share6 persistently
on rhcsa6. (Exercises 16-1 and 16-2). *done (used notes) 1.5 hours in

Task 08: Use NFS to export home directories for all users (u1, u2, and u3) on rhcsa6 so that tregistry.access.redhat.com/ubi9heir home directories become available under /home1 when they log on to rhcsa5. Create u1, u2, and u3. *done used notes 2 hours in


Task 09: On rhcsa5, add HTTP port 8400/UDP to the public firewall zone persistently. 
*done


Task 10: Configure passwordless ssh access for u1 from rhcsa5 to rhcsa6. Copy the directory /etc/sysconfig from rhcsa5 to rhcsa6 under /var/tmp/remote securely.  *done (2.5 hour in used notes) 


Task 11: On rhcsa6, create LVM VDO volume vdo2 on the 5GB disk with logical size 20GB and mounted persistently with XFS structures on /mnt/vdo2. 
*done 3 hours (used notes)

Task 12: On rhcsa6, flip the value of the Boolean nfs_export_all_rw persistently. 
*done

Task 13: On rhcsa5 and rhcsa6, set the tuning profile to powersave.
*done

Task 14: On rhcsa5, create file lnfile1 under /var/tmp and create three hard links called hard1, hard2, and hard3 for it. Identify the inode number associated with all four files. Edit any of the files and observe the metadata for all the files for confirmation. 
*done (on rhcsa6, oops)

Task 15: On rhcsa5, members (user100 and user200) of group100 should be able to collaborate on files under /shared but cannot delete each other's files. 
*4 hours in (done used notes)

Task 16: On rhcsa6, list all files that are part of the "setup" package, and use regular expressions and I/O redirection to send the output lines containing "hosts" to /var/tmp/setup.pkg. 
*done

Task 17: On rhcsa5, check the current version of the Linux kernel. Download and install the latest version of the kernel from Red Hat website. Ensure that the existing kernel and its configuration remain intact. Reboot the system and confirm the newregistry.access.redhat.com/ubi9 version is loaded. *done with dnf instead of downloading from site (4.5hours in)


Task 18: On rhcsa5, configure journald to store messages permanently under /var/log/journal and fall back to memory-only option if /var/log/journal directory does not exist or has permission/access issues. 
*done

Task 19: Write a bash shell script that defines an environment variable called ENV1=book1 and creates a user account that matches the value of the variable. 
*done

Task 20: On rhcsa5, launch a named rootful container with host port 443 mapped to container port 443. Employ the latest version of the ubi9 image. Configure a systemd service to auto-start the container at system reboots. Validate port mapping using an appropriate podman subcommand.
*done

Task 21: On rhcsa5, launch a named rootless container as user100 with /data01 mapped to /data01 and two variables KERN=\$(uname -r) and SHELL defined. Use the latest version of the ubi8 image. Configure a systemd service to auto-start the container at system reboots without the need
for user100 to log in. Create a file under the shared mount point and validate data persistence. 
*done (5 hours in)

Task 22: On rhcsa5, write a containerfile to include the PATH
environment variable output in a custom ubi9 image. Launch a named rootless container as user100 using this image. Confirm command execution. 
*done (5hours 10 minutes)



