## Linux Kernel
- controls everything on the system.  
	- hardware
	- enforces security and access controls
	- runs, schedules, and manages processes and service daemons. 
- comprised of several modules. A 
- new kernel must be installed or an existing kernel must be upgraded when the need arises from an application or functionality standpoint.
- core of the Linux system. It 
- manages
- hardware,
- enforces security,
- regulates access to the system, as well as

handles
- processes, 
- services, and 
- application workloads. It is a 

- collection of software components called *modules* 
	- Modules 
		- device drivers that control hardware devices
			- processor
			- memory
			- storage
			- controller cards
			- peripheral equipment
		- interact with software subsystems
			- storage partitioning
			- file systems
			- networking
			- virtualization

- Some modules are static to the kernel and are integral to system functionality, 
- Some modules are loaded dynamically as needed
- RHEL 8.0 and RHEL 8.2 are shipped with kernel version 4.18.0 (4.18.0-80 and 4.18.0-193 to be specific) for the 64-bit Intel/AMD processor architecture computers with single, multi-core, and multi-processor configurations. 
- `uname -m` shows the architecture of the system.

- Kernel requires a rebuild when a new functionality is added or removed. 
- functionality may be introduced by:
	- installing a new kernel
	- upgrading an existing one
	- installing a new hardware device, or 
	- changing a critical system component. 

- existing functionality that is no longer needed may be removed to make the overall footprint of the kernel smaller for improved performance and reduced memory utilization.

- tunable parameters are set that define a baseline for kernel functionality. 
- Some parameters must be tuned for some applications and database software to be installed smoothly and operate properly.

- You can generate and store several custom kernels with varied configuration and required modules, but 
- only one of them can be active at a time.
- different kernel may be loaded by interacting with GRUB2.

### Kernel Packages

- set of core kernel packages that must be installed on the system at a minimum to
make it work. 
- Additional packages providing supplementary kernel support
are also available.

Core and some add-on kernel packages.

| **Kernel Package**                  | **Description**                                                               |
| ----------------------------------- | ----------------------------------------------------------------------------- |
| kernel                              | Contains no files, but ensures other kernel packages are accurately installed |
| kernel-core                         | Includes a minimal number of modules to provide core functionality            |
| kernel-devel                        | Includes support for building kernel modules                                  |
| kernel-modules                      | Contains modules for common hardware devices                                  |
| kernel-modules-extra                | Contains modules for not-so-common hardware devices                           |
| kernel-headers                      | Includes files to support the interface between the kernel and userspace      |
| kernel-tools-libs                   | Includes the libraries to support the kernel tools                            |
| libraries and programs kernel-tools | Includes tools to manipulate the kernel                                       |


### Kernel Packages

- packages containing the source code for RHEL 8 are also available for those who wish to customize and recompile the code 

List kernel packages installed on the system:
`dnf list installed kernel*`

- Shows six kernel packages that were loaded during the OS installation.

### Analyzing Kernel Version

Check the version of the kernel running on the system to check for compatibility with an application or database:
```bash
uname -r
5.14.0-362.24.1.el9_3.x86_64

```

5 - Major version
14 - Major revision
0 - Kernel patch version
362 - Red Hat version
el9 - Enterprise Linux 9
x86_64 - Processor architecture


### Kernel Directory Structure

Kernel and its support files (noteworthy locations)
- /boot
- /proc
- /usr/lib/modules

### /boot 

- Created at system installation.
- Linux kernel
- GRUB2 configuration
- other kernel and boot support files. 

View the /boot filesystem:
`ls -l /boot`

- four files are for the kernel and
	- vmlinuz - main kernel file
	- initramfs - main kernel's boot image
	- config - configuration
	- System.map - mapping
- two files for kernel rescue version
	- Have the current kernel version appended to their names. 
	- have the string "rescue" embedded within their names

/boot/efi/ and /boot/grub2/ 
- hold bootloader information specific to firmware type used on the system: UEFI or BIOS.

List /boot/Grub2:
```bash
[root@localhost ~]# ls -l /boot/grub2
total 32
-rw-r--r--. 1 root root   64 Feb 25 05:13 device.map
drwxr-xr-x. 2 root root   25 Feb 25 05:13 fonts
-rw-------. 1 root root 7049 Mar 21 04:47 grub.cfg
-rw-------. 1 root root 1024 Mar 21 05:12 grubenv
drwxr-xr-x. 2 root root 8192 Feb 25 05:13 i386-pc
drwxr-xr-x. 2 root root 4096 Feb 25 05:13 locale

```

- grub.cfg 
	- bootable kernel information 
- grub.env 
	- environment information that the kernel uses.

/boot/loader
- storage location for configuration of the running and rescue kernels.
- Configuration is stored in files under the /boot/loader/entries/

```bash
[root@localhost ~]# ls -l /boot/loader/entries/
total 12
-rw-r--r--. 1 root root 484 Feb 25 05:13 8215ac7e45d34823b4dce2e258c3cc47-0-rescue.conf
-rw-r--r--. 1 root root 460 Mar 16 06:17 8215ac7e45d34823b4dce2e258c3cc47-5.14.0-362.18.1.el9_3.x86_64.conf
-rw-r--r--. 1 root root 459 Mar 16 06:17 8215ac7e45d34823b4dce2e258c3cc47-5.14.0-362.24.1.el9_3.x86_64.conf
```

- The files are named using the machine id of the system as stored in /etc/machine-id/ and the kernel version they are for. 

content of the kernel file:
```bash
[root@localhost entries]# cat /boot/loader/entries/8215ac7e45d34823b4dce2e258c3cc47-5.14.0-362.18.1.el9_3.x86_64.conf
title Red Hat Enterprise Linux (5.14.0-362.18.1.el9_3.x86_64) 9.3 (Plow)
version 5.14.0-362.18.1.el9_3.x86_64
linux /vmlinuz-5.14.0-362.18.1.el9_3.x86_64
initrd /initramfs-5.14.0-362.18.1.el9_3.x86_64.img $tuned_initrd
options root=/dev/mapper/rhel-root ro crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/rhel-swap rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet  $tuned_params
grub_users $grub_users
grub_arg --unrestricted
grub_class rhel

```

-  "title" is displayed on the bootloader screen
- "kernelopts" and "tuned_params" supply values to the booting kernel to control its behavior.

### /proc

- Virtual, memory-based file system
- contents are created and updated in memory at system boot and during runtime
- destroyed at system shutdown
- current state of the kernel, which includes 
	- hardware configuration
	- status information 
		- processor, 
		- memory, s
		- storage, 
		- file systems, 
		- swap, 
		- processes
		- network interfaces
		- connections, 
		- routing, *
		- etc.* 
- Data kept in tens of thousands of zero-byte files organized in a hierarchy.

List /proc:
`ls -l /proc`

- numerical subdirectories contain information about a specific process
	- process ID matches the subdirectory name.
- other files and subdirectories contain information, such as 
	- memory segments for processes and 
	- configuration data for system components. You 
	- can view the configuration in vim

Show selections from the cpuinfo and meminfo files that hold
processor and memory information:
`cat/proc/cpuinfo && cat /proc/meminfo`

- data used by top, ps, uname, free, uptime and w, to display information.

### /us/lib/modules/

- holds information about kernel modules. 
- subdirectories are specific to the kernels installed on the system. 
 
Long listing of /usr/lib/modules/ shows two installed kernels:
```bash
[root@localhost entries]# ls -l /usr/lib/modules
total 8
drwxr-xr-x. 7 root root 4096 Mar 16 06:18 5.14.0-362.18.1.el9_3.x86_64
drwxr-xr-x. 8 root root 4096 Mar 16 06:18 5.14.0-362.24.1.el9_3.x86_64
```

View /usr/lib/modules/5.14.0-362.18.1.el9_3.x86_64/:
`ls -l /usr/lib/modules/5.14.0-362.18.1.el9_3.x86_64`

- Subdirectories hold module-specific information for the kernel version.

/lib/modules/4.18.0-80.el8.x86_64/kernel/drivers/
- stores modules for a variety of hardware and software components in various subdirectories:
`ls -l /usr/lib/modules/5.14.0-362.18.1.el9_3.x86_64/kernel/drivers`

- Additional modules may be installed on the system to support more
components.
### Installing the Kernel

- requires extra care 
- could leave your system in an unbootable or undesirable state. 
- have the bootable medium handy prior to starting the kernel install process. 
- By default, the dnf command adds a new kernel to the system, leaving the existing kernel(s) intact. It does not replace or overwrite existing kernel files.

- Always install a new version of the kernel instead of upgrading it.
- The upgrade process removes any existing kernel and replaces it with a new one. 
- In case of a post-installation issue, you will not be able to revert to the old working kernel.

- Newer version of the kernel is typically required:
	- if an application needs to be deployed on the system that requires a different kernel to operate. 
	- When deficiencies or bugs are identified in the existing kernel, it can hamper the kernel's smooth operation.
- new kernel 
	- addresses existing issues 
	- adds bug fixes 
	- security updates 
	- new features 
	- improved support for hardware devices.

- dnf is the preferred tool to install a kernel
- it resolves and installs any required dependencies automatically.
- rpm may be used but you must install any dependencies manually.

- Kernel packages for RHEL are available to subscribers on Red Hat's Customer Portal. 