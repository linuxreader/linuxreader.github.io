## Shell Scripts 

- A group of Linux commands along with control structures and optional comments stored in a text file. 
- Can be executed directly at the Linux command prompt. 
- Do not need to be compiled as they are interpreted by the shell line by line.
- managing packages and users, administering partitions and file systems, monitoring file system utilization, trimming log files, archiving and compressing files, finding and removing unnecessary files, starting and stopping database services and applications, and producing reports. 
- Run by the shell one at a time in the order in which they are listed. 
- Each line is executed as if it is typed and run at the command prompt. 
- Control structures are utilized for creating and managing conditional and looping constructs. 
- Comments are also generally included to add information about the script such as the author name, creation date, previous modification dates, purpose, and usage. 
- If the script encounters an error during execution, the error message is printed on the screen.

- Can use the nl command to enumerate the lines for troubleshooting.
- Can store your scripts in the */usr/local/bin* directory, which is included in the PATH of all users by default.

### Script01: Displaying System Information

- Create the first script called *sys_info.sh* in */usr/local/bin/*   
- Use the vim editor with sudo to write the script. 
```bash
#!/bin/bash
echo "Display Basic System Information"
echo "=================================="
echo
echo "The hostname, hardware, and OS information is:"
/usr/bin/hostnamectl
echo
echo "The Following users are currently logged in:"
/usr/bin/who
```

- Within vim, press the ESC key and then type :set nu to view line numbers associated with each line entry.
- Must add execute bit to run the script



### Executing a Script 

```bash
[chmod +x /usr/local/bin/sys_info.sh](<[root@server30 ~]# chmod +x /usr/local/bin/sys_info.sh
[root@server30 ~]# ll /usr/local/bin/sys_info.sh
-rwxr-xr-x. 1 root root 244 Jul 30 09:47 /usr/local/bin/sys_info.sh>)
```

- Any user on the system can now run this script using either its name or the full path.

Let's run the script and see what the output will look like:
```bash
[root@server30 ~]# /usr/local/bin/sys_info.sh
Display Basic System Information
==================================

The hostname, hardware, and OS information is:
 Static hostname: server30
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: eaa6174e108d4a27bd619754…
         Boot ID: 13d8b3c167b24757b3678e4f…
  Virtualization: oracle
Operating System: Red Hat Enterprise Linux…
     CPE OS Name: cpe:/o:redhat:enterprise…
          Kernel: Linux 5.14.0-362.24.1.el…
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox

The Following users are currently logged in:
root     pts/0        2024-07-30 07:22 (172.16.7.95)
```

### Debugging a Script 

Can either append the `-x` option to the "`#!/bin/bash`" at the beginning of the script to look like "`#!/bin/bash -x`", or execute the script as follows:
```bash
[root@server30 ~]# bash -x sys_info.sh
+ echo 'Display Basic System Information'
Display Basic System Information
+ echo ==================================
==================================
+ echo

+ echo 'The hostname, hardware, and OS information is:'
The hostname, hardware, and OS information is:
+ /usr/bin/hostnamectl
 Static hostname: server30
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: eaa6174e108d4a27bd6197548ce77270
         Boot ID: 13d8b3c167b24757b3678e4fd3fe19ee
  Virtualization: oracle
Operating System: Red Hat Enterprise Linux 9.3 (Plow)     
     CPE OS Name: cpe:/o:redhat:enterprise_linux:9::baseos
          Kernel: Linux 5.14.0-362.24.1.el9_3.x86_64
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox
+ echo

+ echo 'The Following users are currently logged in:'
The Following users are currently logged in:
+ /usr/bin/who
root     pts/0        2024-07-30 07:22 (172.16.7.95)
```


- Actual lines from the script prefixed by the + sign and followed by the command execution result. 
- Shows the line number of the problem line in the output if there is any. 
- This way you can identify any issues pertaining to the path, command name, use of special characters, etc., and address it quickly.

Change one of the echo commands in the script to "iecho" and re-run the script in the debug mode to see the error:
```bash
[root@server30 ~]# bash -x sys_info.sh
+ echo 'Display Basic System Information'
Display Basic System Information
+ echo ==================================
==================================
+ iecho
/usr/local/bin/sys_info.sh: line 4: iecho: command not found
+ echo 'The hostname, hardware, and OS information is:'
The hostname, hardware, and OS information is:
+ /usr/bin/hostnamectl
 Static hostname: server30
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: eaa6174e108d4a27bd6197548ce77270
         Boot ID: 13d8b3c167b24757b3678e4fd3fe19ee
  Virtualization: oracle
Operating System: Red Hat Enterprise Linux 9.3 (Plow)     
     CPE OS Name: cpe:/o:redhat:enterprise_linux:9::baseos
          Kernel: Linux 5.14.0-362.24.1.el9_3.x86_64
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox
+ echo

+ echo 'The Following users are currently logged in:'
The Following users are currently logged in:
+ /usr/bin/who
root     pts/0        2024-07-30 07:22 (172.16.7.95)
```

### Script02: Using Local Variables

- Create a script called *use_var.sh* will 
- Define a local variable and display its value on the screen. 
- Re-check the value of the variable after the script execution has completed. 
```bash
[root@server30 ~]# vim /usr/local/bin/use_var.sh

#!/bin/bash

echo "Setting a Local Variable"
echo "========================"
SYSNAME=server30.example.com
echo "The hostname of this system is $SYSNAME"
```

```bash
[root@server30 ~]# chmod +x /usr/local/bin/use_var.sh

[root@server30 ~]# use_var.sh
Setting a Local Variable
========================
The hostname of this system is server30.example.com
```

If you run the echo command to see what is stored in the SYSNAME
variable, you will get nothing:
```bash
[root@server30 ~]# echo $SYSNAME

```

### Script03: Using Pre-Defined Environment Variables 

The following script called pre_env.sh will display the values of SHELL and LOGNAME environment variables:
```bash
[root@server30 ~]# vim /usr/local/bin/pre_env.sh

#!/bin/bash
echo "The location of my shell command is:"
echo $SHELL
echo "I am logged in as $LOGNAME"
```

```bash
[root@server30 ~]# chmod +x /usr/local/bin/pre_env.sh

[root@server30 ~]# pre_env.sh
The location of my shell command is:
/bin/bash
I am logged in as root
```

### Script04: Using Command Substitution

- Can use the command substitution feature of the bash shell and store the output generated by the command
into a variable. 

- Two different ways to use command substitution: Backtics or subshell

```bash
#!/bin/bash
SYSNAME=$(hostname)
KERNVER=`uname -r`
echo "The hostname is $SYSNAME"
echo "The kernel version is $KERNVER"
```

```bash
[root@server30 ~]# vim /usr/local/bin/cmd_out.sh

[root@server30 ~]# chmod +x /usr/local/bin/cmd_out.sh

[root@server30 ~]# cmd_out.sh
The hostname is server30
The kernel version is 5.14.0-362.24.1.el9_3.x86_64
```



