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




