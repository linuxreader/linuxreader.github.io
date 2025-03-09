## Reset the root User Password

- Terminate the boot process at an early stage to be placed in a special debug shell in order to reset the *root* password.

1. Reboot or reset *server1*, and interact with GRUB2 by pressing a key before the autoboot times out. Highlight the default kernel entry in the GRUB2 menu and press *e* to enter the edit mode. Scroll down to the line entry that begins with the keyword "linux" and press the End key to go to the end of that line:

2. Modify this kernel string and append "rd.break" to the end of the line. 
![](/images/Pasted%20image%2020240321045933.png)

3. Press Ctrl+x when done to boot to the special shell. The system mounts the root file system read-only on the */sysroot* directory. Make */sysroot* appear as mounted on */* using the *chroot* command:
![](/images/Pasted%20image%2020240321050831.png)

4. Remount the root file system in read/write mode for the *passwd* command to be able to modify the *shadow* file with a new
password:
`mount -o remount,rw /`

5. Enter a new password for *root* by invoking the *passwd* command:
`passwd`

6. Create a hidden file called *.autorelabel* to instruct the operating system to run SELinux relabeling on all files, including the *shadow* file that was updated with the new *root* password, on the next reboot:
`touch .autorelabel`

7. Issue the *exit* command to quit the chroot shell and then the *reboot* command to restart the system and boot it to the default target.
`exit` `reboot`

## Second method

Look into using `init=/bin/bash` for password recovery as a second method. 