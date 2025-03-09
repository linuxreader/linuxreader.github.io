/rpm## Basic Package Management Labs

### Lab: Mount RHEL 9 ISO Persistently
1. Go to the VirtualBox VM Manager and make sure that the RHEL 8 image is attached to RHEL9-VM1 as depicted below:
![](Pasted%20image%2020240713164339.png)

2. Open the /etc/fstab file in the vim editor (or another editor of your choice) and add the following line entry at the end of the file to mount the DVD image (/dev/sr0) in read-only (ro) mode on the /mnt directory.

	```
	/dev/sr0 /mnt iso9660 ro 0 0
	```

Note: sr0 represents the first instance of the optical device and iso9660 is the standard format for optical file systems.

3. Mount the file system as per the configuration defined in the /etc/fstab file using the mount command with the -a (all) option:

	```
	sudo mount -a
	```

4. Verify the mount using the df command:

	```
	df -h | grep mnt
	```

Note: The image and the packages therein can now be accessed via the /mnt directory just like any other local directory on the system.

5. List the two directories—/mnt/BaseOS/Packages and /mnt/AppStream/Packages—that contain all the software packages (directory names are case sensitive):

	```
	ls -l /mnt/BaseOS/Packages | more
	```

### Lab: Query Packages (RPM)
1. query all installed packages:
	`rpm -qa`

2. query whether the perl package is installed:
	`rpm -q perl`

3. list all files in a package:
	`rpm -ql iproute`

4. list only the documentation files in a package:
	`rpm -qd audit`

5. list only the configuration files in a package:
	`rpm -qc cups`

6. identify which package owns the specified file:
	`rpm -qf /etc/passwd`

7. display information about an installed package including version, release, installation status, installation date, size, signatures, description, and so on:
	`rpm -qi setup`

8. list all file and package dependencies for a given package:
	`rpm -qR chrony`

9. query an installable package for metadata information (version, release, architecture, description, size, signatures, etc.):
	`rpm -qip /mnt/BaseOS/Packages/zsh-5.5.1-6.el8.x86_64.rpm`

10. determine what packages require the specified package in order to operate properly:
	`rpm -q --whatrequires lvm2`


### Lab: Installing a Package (RPM)
1. Install zsh-5.5.1-6.el8.x86_64.rpm
`sudo rpm -ivh /mnt/BaseOS/Packages/zsh-5.5.1-6.el8.x86_64.rpm`

### Lab: Upgrading a Package (RPM)
1. Upgrade sushi with the -U option:
`sudo rpm -Uvh /mnt/AppStream/Packages/sushi-3.28.3-1.el8.x86_64.rpm`

### Lab: Freshening a Package
1. Freshen the sushi package:
`sudo rpm -Fvh /mnt/AppStream/Packages/sushi-3.28.3-1.el8.x86_64.rpm`


### Lab: Overwriting a Package
1. Overwrite zsh-5.5.1-6.el8.x86_64
`sudo rpm -ivh --replacepkgs /mnt/BaseOS/Packages/zsh-5.5.1-6.el8.x86_64`

### Lab: Removing a Package
1. Remove sushi
`sudo rpm sushi -ve`
### Lab: Extracting Files from an Installable Package
1. You have lost /etc/crony.conf. Determine what package this file comes from:
`rpm -qf /etc/chrony.conf`

2. Extract all files from the crony package to /tmp and create the directory structure:
```bash
[root@server30 mnt]# cd /tmp

[sudo rpm2cpio /mnt/BaseOS/Packages/chrony-3.3-3.el8.x86_64.rpm | cpio -imd
1066 blocks](<[root@server30 tmp]# rpm2cpio /mnt/BaseOS/Packages/chrony-4.3-1.el9.x86_64.rpm | cpio -imd
1253 blocks>)
```

3. Use find to locate the crony.conf file:
`sudo find . -name chrony.conf`

4. Copy the file to /etc:


### Lab: Validating Package Integrity and Credibility
1. Check the integrity of zsh-5.5.1-6.el8.x86_64.rpm located in /mnt/BaseOS/Packages:
`rpm -K /mnt/BaseOS/Packages/zsh-5.5.1-6.el8.x86_64.rpm --nosignature`
2. Import the GPG key from the proper file and verify the signature for the zsh-5.5.1-6.el8.x86_64.rpm package. 
```
sudo rpmkeys --import /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
sudo rpmkeys -K /mnt/BaseOS/Packages/zsh-5.5.1-6.el8.x86_64.rpm
```

### Lab: Viewing GPG Keys
1. List the imported key: 
`rpm -q gpg-pubkey`
2. View details for the first key:
`rpm -qi gpg-pubkey-fd431d51-4ae0493b`

### Lab: Verifying Package Attributes
1. Run a check on the at program:
`sudo rpm -V at`

2. Change permissions of one of the files and run the check again:
```bash
ls -l /etc/sysconfig/atd
sudo chmod -v 770 /etc/sysconfig/atd
sudo rpm -V at
```

3. Run the check directly on the file:
`sudo rpm -Vf /etc/sysconfig/atd`

4. Reset the value and check the file again:
```bash
sudo chmod -v 644 /etc/sysconfig/atd
sudo rpm -V at
```



### Lab: Perform Package Management Using rpm
1. Run the `ls` command on the /mnt/AppStream/Packages directory to confirm that the rmt package is available:
```bash
[root@server30 tmp]# ls -l /mnt/BaseOS/Packages/rmt*
-r--r--r--. 1 root root 49582 Nov 20  2021 /mnt/BaseOS/Packages/rmt-1.6-6.el9.x86_64.rpm
```

2. Run the rpm command and verify the integrity and credibility of the package:
```bash
[root@server30 tmp]# rpmkeys -K /mnt/BaseOS/Packages/rmt-1.6-6.el9.x86_64.rpm
/mnt/BaseOS/Packages/rmt-1.6-6.el9.x86_64.rpm: digests signatures OK
```

3. Install the Package:
```bash
[root@server30 tmp]# rpmkeys -K /mnt/BaseOS/Packages/rmt-1.6-6.el9.x86_64.rpm
/mnt/BaseOS/Packages/rmt-1.6-6.el9.x86_64.rpm: digests signatures OK
[root@server30 tmp]# rpm -ivh /mnt/BaseOS/Packages/rmt-1.6-6.el9.x86_64.rpm
Verifying...                         ################################# [100%])
Preparing...                         ################################# [100%])
Updating / installing...
   1:rmt-2:1.6-6.el9                 ################################# [100%])
```

4. Show basic information about the package:
```bash
[root@server30 tmp]# rpm -qi rmt
Name        : rmt
Epoch       : 2
Version     : 1.6
Release     : 6.el9
Architecture: x86_64
Install Date: Sat 13 Jul 2024 09:02:08 PM MST
Group       : Unspecified
Size        : 88810
License     : CDDL
Signature   : RSA/SHA256, Sat 20 Nov 2021 08:46:44 AM MST, Key ID 199e2f91fd431d51
Source RPM  : star-1.6-6.el9.src.rpm
Build Date  : Tue 10 Aug 2021 03:13:47 PM MST
Build Host  : x86-vm-55.build.eng.bos.redhat.com
Packager    : Red Hat, Inc. <http://bugzilla.redhat.com/bugzilla>
Vendor      : Red Hat, Inc.
URL         : http://freecode.com/projects/star
Summary     : Provides certain programs with access to remote tape devices
Description :
The rmt utility provides remote access to tape devices for programs
like dump (a filesystem backup program), restore (a program for
restoring files from a backup), and tar (an archiving program).
```

5. Show all the files the package contains:
```bash
[root@server30 tmp]# rpm -ql rmt
/etc/default/rmt
/etc/rmt
/usr/lib/.build-id
/usr/lib/.build-id/c2
/usr/lib/.build-id/c2/6a51ea96fc4b4367afe7d44d16f1405c3c7ec9
/usr/sbin/rmt
/usr/share/doc/star
/usr/share/doc/star/CDDL.Schily.txt
/usr/share/doc/star/COPYING
/usr/share/man/man1/rmt.1.gz
```

6. List the documentation files the package has:
```bash
[root@server30 tmp]# rpm -qd rmt
/usr/share/doc/star/CDDL.Schily.txt
/usr/share/doc/star/COPYING
/usr/share/man/man1/rmt.1.gz
```

7. Verify the attributes of each file in the package. Use verbose mode.
```bash
[root@server30 tmp]# rpm -vV rmt
.........  c /etc/default/rmt
.........    /etc/rmt
.........  a /usr/lib/.build-id
.........  a /usr/lib/.build-id/c2
.........  a /usr/lib/.build-id/c2/6a51ea96fc4b4367afe7d44d16f1405c3c7ec9
.........    /usr/sbin/rmt
.........    /usr/share/doc/star
.........  d /usr/share/doc/star/CDDL.Schily.txt
.........  d /usr/share/doc/star/COPYING
.........  d /usr/share/man/man1/rmt.1.gz
```

8. Remove the package:
```bash
[root@server30 tmp]# rpm -ve rmt
Preparing packages...
rmt-2:1.6-6.el9.x86_64
```

