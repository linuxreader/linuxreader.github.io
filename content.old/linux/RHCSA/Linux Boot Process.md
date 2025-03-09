## Linux Boot Process

Multiple phases during the boot process. 
- starts selective services during its transition from one phase into another. 
- presents the administrator an opportunity to interact with a preboot program to boot the system into a non-default target, 
- pass an option to the kernel
- reset the lost or forgotten root user password. 
- launches a number of services during its transition to the default or specified target.
- boot process after the system has been powered up or restarted. 
- lasts until all enabled services are started. 
- login prompt will appear on the screen 
- boot process is automatic, but you 
	- may need to interact with it to take a non-default action, such as 
		- booting an alternative kernel, 
		- booting into a non-default operational state, 
		- repairing the system, 
		- recovering from an unbootable state
boot process on an x86 computer may be split into four major phases:
	(1) the firmware phase
	(2) the bootloader phase
	(3) the kernel phase 
	(4) the initialization phase. 

The system accomplishes these phases
one after the other while performing and attempting to complete the
tasks identified in each phase.

### The Firmware Phase (BIOS and UEFI)

firmware:  
- BIOS (*Basic Input/Output System*) or the UEFI (*Unified Extensible Firmware Interface*) code that is stored in flash memory on the x86-based system board. 
- runs the *Power-On-Self-Test* (POST) to detect, test, and initialize the system hardware components.
- Installs appropriate drivers for the video hardware
- exhibits system messages on the screen.
- scans available storage devices to locate a boot device, 
	- starting with a 512-byte image that contains 
		- 446 bytes of the bootloader program, 
		- 64 bytes for the partition table
		- last two bytes with the boot signature. 
		- referred to as the *Master Boot Record* (MBR) 
		- located on the first sector of the boot disk. 
		- As soon as it discovers a usable boot device, it loads the bootloader into memory and passes control over to it.

BIOS 
- small memory chip in the computer that stores 
	- system date and time, 
	- list and sequence of boot devices,
	- I/O configuration,
	- *etc.*
- configuration is customizable. 
- hardware initialization phase 
	- detecting and diagnosing peripheral devices. 
	- runs the POST on the devices as it finds them, 
	- installs drivers for the graphics card and the attached monitor, 
	- begins exhibiting system messages on the video hardware.  
	- discovers a usable boot device,
	- loads the bootloader program into memory, and passes control over to it. 

UEFI
- new 32/64-bit architecture-independent specification replacing BIOS. 
- delivers enhanced boot and runtime services
- superior features such as speed over the legacy 16-bit BIOS. 
- has its own device drivers
- able to mount and read extended file systems
- includes UEFI-compliant application tools, and
- supports one or more bootloader programs.
- comes with a boot manager that allows you to choose an alternative boot source. 

### Bootloader Phase

- Once the firmware phase is over and a boot device is detected, 
- system loads a piece of software called *bootloader* that is located in the boot sector of the boot device. 
- RHEL uses GRUB2 (*GRand Unified Bootloader*) version 2 as the bootloader program. GRUB2 supports both BIOS and UEFI firmware.

The primary job of the bootloader program is to
- spot the Linux kernel code in the */boot* file system, 
- decompress it
- load it into memory based on the configuration defined in the **boot/grub2/grub.cfg** file
- transfer control over to it to further the boot process. 

UEFI-based systems, 
- GRUB2 looks for the EFI system partition **/boot/efi** instead, and 
- runs the kernel based on the configuration defined in the /boot/efi/EFI/redhat/grub.efi file. 

### Kernel Phase

- *kernel* is the central program of the operating system, providing access to hardware and system services. 
- After getting control from the bootloader, the kernel: 
- extracts the *initial RAM disk* (initrd) file system image found in the */boot* file system into memory, 
- decompresses it
- mounts it as read-only on */sysroot* to serve as the temporary root file system
- loads necessary modules from the initrd image to allow access to the physical disks and the partitions and file systems therein. 
- loads any required drivers to support the boot process. 
- Later, it unmounts the initrd image and mounts the actual physical root file system on */* in read/write mode.

- At this point, the necessary foundation has been built for the boot process to carry on and to start loading the enabled services.
- kernel executes the *systemd* process with PID 1 and passes the control over to it.

### Initialization Phase

- fourth and the last phase in the boot process.
- Systemd: 
- takes control from the kernel and continues the boot process.
- is the default system initialization scheme used in RHEL 9.
- starts all enabled userspace system and network services
- Brings the system up to the preset boot target.
- A boot target is an operational level that is achieved after a series of services have been started to get to that state. 

- system boot process is considered complete when all enabled services are operational for the boot target and users are able to log in to the system 

### GRUB2 Bootloader

- After the firmware phase has concluded, the 
- Bootloader presents a menu with a list of bootable kernels available on the system
- Waits for a predefined amount of time before it times out and boots the default kernel. 
- You may want to interact with GRUB2 before the autoboot times out to boot with a non-default kernel, boot to a different target, or customize the kernel boot string.
- Press a key before the timeout expires to interrupt the autoboot process and interact with GRUB2. 
- autoboot countdown default value is 5 seconds.

### Interacting with GRUB2

- GRUB2 main menu shows a list of bootable kernels at the top. 
- Edit a selected kernel menu entry by pressing an *e* or go to the grub\> command prompt by pressing a *c*.

edit mode, 
- GRUB2 loads the configuration for the selected kernel entry from the /boot/grub2/grub.cfg file in an editor
- enables you to make a desired modification before booting the system. 
- you can boot the system into a less capable operating target by adding "rescue", "emergency", or "3" to the end of the line that begins with the keyword "linux", 
- Press `Ctrl+x` when done to boot. 
- one-time temporary change and it won't touch the *grub.cfg* file.
- press `ESC` to discard the changes and return to the main menu.
- `grub>` command prompt appears when you press `Ctrl+c` while in the edit window 
- or a `c` from the main menu. 
- command mode: execute debugging, recovery, etc. 
- view available commands by pressing the TAB key.

### GRUB2 Commands

#### Understanding GRUB2 Configuration Files

**/boot/grub2/grub.cfg**
- Referenced at boot time. 
- Generated automatically when a new kernel is installed or upgraded
- not advisable to modify it directly, as your changes will be overwritten. The 

**/etc/default/grub**
- primary source file that is used to regenerate grub.cfg.
- Defines the directives that govern how GRUB2 should behave at boot time. 
- Any changes made to the *grub* file will only take effect after the *grub2-mkconfig* utility has been executed
- Defines the directives that control the behavior of GRUB2 at boot time. 
- Any changes in this file must be followed by the execution of the `grub2-mkconfig` command in order to be reflected in grub.cfg.

Default settings:
```bash
[root@localhost default]# nl /etc/default/grub
     1	GRUB_TIMEOUT=5
     2	GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
     3	GRUB_DEFAULT=saved
     4	GRUB_DISABLE_SUBMENU=true
     5	GRUB_TERMINAL_OUTPUT="console"
     6	GRUB_CMDLINE_LINUX="crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/rhel-swap rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet"
     7	GRUB_DISABLE_RECOVERY="true"
     8	GRUB_ENABLE_BLSCFG=true
```


| **Directive**         | **Description**                                                                            |
| --------------------- | ------------------------------------------------------------------------------------------ |
| GRUB_TIMEOUT          | Wait time, in seconds, before booting off the default kernel. Default is 5.                |
| GRUB_DISTRIBUTOR      | Name of the Linux distribution                                                             |
| GRUB_DEFAULT          | Boots the selected option from the previous system boot                                    |
| GRUB_DISABLE_SUBMENU  | Enables/disables the appearance of GRUB2 submenu                                           |
| GRUB_TERMINAL_OUTPUT  | Sets the default terminal                                                                  |
| GRUB_CMDLINE_LINUX    | Specifies the command line options to pass to the kernel at boot time                      |
| GRUB_DISABLE_RECOVERY | Lists/hides system recovery entries in the GRUB2 menu                                      |
| GRUB_ENABLE_BLSCFG    | Defines whether to use the new bootloader specification to manage bootloader configuration |

- Default settings are good enough for normal system operation.

#### /boot/grub2/grub.cfg - /boot/efi/EFI/redhat/grub.cfg

- Main GRUB2 configuration file that supplies boot-time configuration information. 
- located in the /boot/grub2/ on BIOS-based systems 
- /boot/efi/EFI/redhat/ on UEFI-based systems. 
- can be recreated manually with the `grub2-mkconfig` utility
- automatically regenerated when a new kernel is installed or upgraded. 
- file will lose any previous manual changes made to it.

`grub2-mkconfig` command
- Uses the settings defined in helper scripts located in the /etc/grub.d directory. 

```bash
[root@localhost default]# ls -l /etc/grub.d
total 104
-rwxr-xr-x. 1 root root  9346 Jan  9 09:51 00_header
-rwxr-xr-x. 1 root root  1046 Aug 29  2023 00_tuned
-rwxr-xr-x. 1 root root   236 Jan  9 09:51 01_users
-rwxr-xr-x. 1 root root   835 Jan  9 09:51 08_fallback_counting
-rwxr-xr-x. 1 root root 19665 Jan  9 09:51 10_linux
-rwxr-xr-x. 1 root root   833 Jan  9 09:51 10_reset_boot_success
-rwxr-xr-x. 1 root root   892 Jan  9 09:51 12_menu_auto_hide
-rwxr-xr-x. 1 root root   410 Jan  9 09:51 14_menu_show_once
-rwxr-xr-x. 1 root root 13613 Jan  9 09:51 20_linux_xen
-rwxr-xr-x. 1 root root  2562 Jan  9 09:51 20_ppc_terminfo
-rwxr-xr-x. 1 root root 10869 Jan  9 09:51 30_os-prober
-rwxr-xr-x. 1 root root  1122 Jan  9 09:51 30_uefi-firmware
-rwxr-xr-x. 1 root root   218 Jan  9 09:51 40_custom
-rwxr-xr-x. 1 root root   219 Jan  9 09:51 41_custom
-rw-r--r--. 1 root root   483 Jan  9 09:51 README

```

00_header
- sets the GRUB2 environment
10_linux
- searches for all installed kernels on the same disk partition 
30_os-prober
- searches for the presence of other operating systems
40_custom and 41_custom are to 
- introduce any customization. 
- like add custom entries to the boot menu.

grub.cfg file 
- Sources /boot/grub2/grubenv for kernel options and other settings. 
```bash
[root@localhost grub2]# cat grubenv
# GRUB Environment Block
# WARNING: Do not edit this file by tools other than grub-editenv!!!
saved_entry=8215ac7e45d34823b4dce2e258c3cc47-5.14.0-362.24.1.el9_3.x86_64
menu_auto_hide=1
boot_success=0
boot_indeterminate=0
############################################################################
############################################################################
```


If a new kernel is installed:
- the existing kernel entries remain intact.
- All bootable kernels are listed in the GRUB2 menu
- any of the kernel entries can be selected to boot.

### Lab: Change Default System Boot Timeout

- change the default system boot timeout value to 8 seconds persistently, and validate.

1. Edit the /etc/default/grub file and change the setting as follows:
`GRUB_TIMEOUT=8

2. Execute the *grub2-mkconfig* command to reproduce *grub.cfg*:
`grub2-mkconfig -o /boot/grub2/grub.cfg`

3.Restart the system with *sudo reboot* and confirm the new
timeout value when GRUB2 menu appears.
