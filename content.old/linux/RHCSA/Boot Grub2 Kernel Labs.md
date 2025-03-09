## Boot Grub2 Kernel Labs

### Lab: Enable Verbose System Boot

- Remove "quiet" from the end of the value of the variable GRUB_CMDLINE_LINUX in the /etc/default/grub file
- Run *grub2-mkconfig* to apply the update. 
- Reboot the system and observe that the system now displays verbose information during the boot process. 

### Lab: Reset root User Password

-  Reset the *root* user password by booting the system into emergency mode with SELinux disabled. 
- Try to log in with root and enter the new password after the reboot. 

### Lab: Install New Kernel

- Check the current version of the kernel using the *uname* or *rpm* command. 
- Download a higher version from the Red Hat Customer Portal or [rpmfind.net](http://rpmfind.net) and install it. 
- Reboot the system and ensure the new kernel is listed on the bootloader menu. 

### Lab: Download and Install a New Kernel

- download the latest available kernel packages from the Red Hat Customer Portal \ 
- install them using the dnf command.
- ensure that the existing kernel and its configuration remain intact.

* As an alternative (preferred) to downloading kernel packages individually and then installing them, you can follow the instructions provided in "Containers" chapter to register *server1* with RHSM and run **sudo dnf install kernel** to install the latest kernel and all the dependencies collectively.

1. Check the version of the running kernel:
`uname -r`

2. List the kernel packages currently installed:
`rpm -qa | grep kernel`

3. Sign in to the [Red Hat Customer Portal](http://access.redhat.com)and click downloads.

4. Click "Red Hat Enterprise Linux 8" under "By Category":

5. Click Packages and enter "kernel" in the Search bar to narrow the list of available packages:

6. Click "Download Latest" against the packages kernel, kernel-core, kernel-headers, kernel-modules, kernel-tools, and kernel-tools-libs to download them.

7. Once downloaded, move the packages to the */tmp* directory using the *mv* command.

8. List the packages after moving them:

9. Install all the six packages at once using the *dnf* command:
`dnf install /tmp/kernel* -y`

10. Confirm the installation alongside the previous version:
`sudo dnf list installed kenel*`


11. The /boot/grub2/grubenv/ file now has the directive "saved_entry" set to the new kernel, which implies that this new kernel will boot up on the next system restart:
`sudo cat /boot/grub2/grubenv`

12. Reboot the system. You will see the new kernel entry in the GRUB2 boot list at the top. The system will autoboot this new default kernel.

13. Run the *uname* command once the system has been booted up to
confirm the loading of the new kernel:
`uname -r`

14. View the contents of the *version* and cmdline files under /proc to verify the active kernel:
`cat /proc/version

Or just `dnf install kernel`