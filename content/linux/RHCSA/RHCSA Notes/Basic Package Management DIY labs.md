## Lab 9-1: Install and Verify Packages

As user1 with sudo on server3, 
- make sure the RHEL 9 ISO image is attached to the VM and mounted. 
- Use the rpm command and install the zsh package by specifying its full path. 
```bash
[root@server30 Packages]# rpm -ivh /mnt/BaseOS/Packages/zsh-5.8-9.el9.x86_64.rpm 
Verifying...                         ################################# [100%])
Preparing...                         ################################# [100%])
	package zsh-5.8-9.el9.x86_64 is already installed

```

- Run the rpm command again and perform the following on the zsh package: 
- (1) show information
```bash
[root@server30 Packages]# rpm -qi zsh
Name        : zsh
Version     : 5.8
Release     : 9.el9
Architecture: x86_64
Install Date: Sat 13 Jul 2024 06:49:40 PM MST
Group       : Unspecified
Size        : 8018363
License     : MIT
Signature   : RSA/SHA256, Thu 24 Feb 2022 08:59:15 AM MST, Key ID 199e2f91fd431d51
Source RPM  : zsh-5.8-9.el9.src.rpm
Build Date  : Wed 23 Feb 2022 07:10:14 AM MST
Build Host  : x86-vm-56.build.eng.bos.redhat.com
Packager    : Red Hat, Inc. <http://bugzilla.redhat.com/bugzilla>
Vendor      : Red Hat, Inc.
URL         : http://zsh.sourceforge.net/
Summary     : Powerful interactive shell
Description :
The zsh shell is a command interpreter usable as an interactive login
shell and as a shell script command processor.  Zsh resembles the ksh
shell (the Korn shell), but includes many enhancements.  Zsh supports
command line editing, built-in spelling correction, programmable
command completion, shell functions (with autoloading), a history
mechanism, and more.
```

- (2) validate integrity
```bash
[root@server30 Packages]# rpm -K zsh-5.8-9.el9.x86_64.rpm
zsh-5.8-9.el9.x86_64.rpm: digests signatures OK
```

- (3) display attributes
`[root@server30 Packages]# rpm -V zsh`

## Lab 9-2: Query and Erase Packages

As user1 with sudo on server3, 
- make sure the RHEL 9 ISO image is attached to the VM and mounted. 
- Use the rpm command to perform the following: 
- (1) check whether the setup package is installed
```bash
[root@server30 Packages]# rpm -q setup
setup-2.13.7-10.el9.noarch
```

- (2) display the list of configuration files in the setup package
```bash
[root@server30 Packages]# rpm -qc setup
/etc/aliases
/etc/bashrc
/etc/csh.cshrc
/etc/csh.login
/etc/environment
/etc/ethertypes
/etc/exports
/etc/filesystems
/etc/fstab
/etc/group
/etc/gshadow
/etc/host.conf
/etc/hosts
/etc/inputrc
/etc/motd
/etc/networks
/etc/passwd
/etc/printcap
/etc/profile
/etc/profile.d/csh.local
/etc/profile.d/sh.local
/etc/protocols
/etc/services
/etc/shadow
/etc/shells
/etc/subgid
/etc/subuid
/run/motd
/usr/lib/motd
```


- (3) show information for the zlib-devel package on the ISO image
```bash
[root@server30 Packages]# rpm -qi ./zlib-devel-1.2.11-40.el9.x86_64.rpm
Name        : zlib-devel
Version     : 1.2.11
Release     : 40.el9
Architecture: x86_64
Install Date: (not installed)
Group       : Unspecified
Size        : 141092
License     : zlib and Boost
Signature   : RSA/SHA256, Tue 09 May 2023 05:31:02 AM MST, Key ID 199e2f91fd431d51
Source RPM  : zlib-1.2.11-40.el9.src.rpm
Build Date  : Tue 09 May 2023 03:51:20 AM MST
Build Host  : x86-64-03.build.eng.rdu2.redhat.com
Packager    : Red Hat, Inc. <http://bugzilla.redhat.com/bugzilla>
Vendor      : Red Hat, Inc.
URL         : https://www.zlib.net/
Summary     : Header files and libraries for Zlib development
Description :
The zlib-devel package contains the header files and libraries needed
to develop programs that use the zlib compression and decompression
library.
```


- (4) reinstall the zsh package (--reinstall -vh),
```bash
[root@server30 Packages]# rpm -hv --reinstall ./zsh-5.8-9.el9.x86_64.rpm
Verifying...                         ################################# [100%])
Preparing...                         ################################# [100%])
Updating / installing...
   1:zsh-5.8-9.el9                   ################################# [ 50%])
Cleaning up / removing...
   2:zsh-5.8-9.el9                   ################################# [100%])
```


- (5) remove the zsh package. 
`[root@server30 Packages]# rpm -e zsh`


