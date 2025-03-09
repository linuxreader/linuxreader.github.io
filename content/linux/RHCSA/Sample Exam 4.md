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


