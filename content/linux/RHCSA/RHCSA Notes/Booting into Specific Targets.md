## Booting into Specific Targets

RHEL 
- boots into graphical target state by default if the Server with GUI software selection is made during installation. 
- can also be directed to boot into non-default but less capable operating targets from the GRUB2 menu. 
- offers emergency and rescue boot targets. 
	- special target levels can be launched from the GRUB2 interface by 
		- selecting a kernel
		- pressing *e* to enter the edit mode
		- appending the desired target name to the line that begins with the keyword "linux".
		- Press ctrl+x to boot into the supplied target
		- Enter root password
		- `reboot` when you are done

-  You must know how to boot a RHEL 9 system into a specific target from the GRUB2 menu to modify the fstab file or reset an unknown root user password.

Append "emergency" to the kernel line entry:

![](Pasted%20image%2020240321045343.png)


Other options:
- "rescue" 
- "1"
- "s"
- "single"
