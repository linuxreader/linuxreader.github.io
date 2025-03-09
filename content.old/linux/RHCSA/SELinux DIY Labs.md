## SELinux DIY Labs

### Lab: Disable and Enable the SELinux Operating Mode 

- Check and make a note of the current SELinux operating mode. 
```bash
[root@server30 ~]# getenforce
Enforcing
```

- Modify the configuration file and set the mode to disabled. 
```bash
[root@server30 ~]# vim /etc/selinux/config 
SELINUX=permissive
```
- Reboot the system to apply the change. 
```bash
[root@server30 ~]# reboot
```

- Run `sudo getenforce` to confirm the change when the system is up. 
```bash
[root@server30 ~]# getenforce
Permissive
```

- Restore the directive's value to enforcing in the configuration file, and reboot to apply the new mode. 
```bash
[root@server30 ~]# vim /etc/selinux/config

SELINUX=enforcing

[root@server30 ~]# reboot

```

- Run `sudo getenforce` to confirm the mode when the system is up.
```bash
[root@server30 ~]# getenforce
Enforcing
```

### Lab: Modify Context on Files 

- Create directory hierarchy */tmp/d1/d2*.
```bash
[root@server30 ~]# mkdir /tmp/d1
[root@server30 ~]# mkdir /tmp/d1/d2
```

- Check the contexts on */tmp/d1* and */tmp/d1/d2*. 
```bash
[root@server30 d1]# ls -ldZ /tmp/d1
drwxr-xr-x. 3 root root unconfined_u:object_r:user_tmp_t:s0 16 Jul 29 13:17 /tmp/d1
[root@server30 d1]# ls -ldZ /tmp/d1/d2
drwxr-xr-x. 2 root root unconfined_u:object_r:user_tmp_t:s0 6 Jul 29 13:17 /tmp/d1/d2
```

- Change the SELinux type on */tmp/d1* to `etc_t` recursively with the `chcon` command and confirm. 
```bash
[root@server30 tmp]# chcon -Rv -t etc_t /tmp/d1
changing security context of '/tmp/d1/d2'
changing security context of '/tmp/d1'

[root@server30 tmp]# ls -ldZ /tmp/d1
drwxr-xr-x. 3 root root unconfined_u:object_r:etc_t:s0 16 Jul 29 13:17 /tmp/d1

[root@server30 tmp]# ls -ldZ /tmp/d1/d2
drwxr-xr-x. 2 root root unconfined_u:object_r:etc_t:s0 6 Jul 29 13:17 /tmp/d1/d2

```

- Add */tmp/d1* to the policy database with the `semanage` command to ensure the new context is persistent on the directory hierarchy.  
```bash
[root@server30 tmp]# semanage fcontext -a -t etc_t /tmp/d1

[root@server30 tmp]# reboot

[root@server30 ~]# ls -ldZ /tmp/d1
drwxr-xr-x. 3 root root unconfined_u:object_r:etc_t:s0 16 Jul 29 13:17 /tmp/d1

[root@server30 ~]# ls -ldZ /tmp/d1/d2
drwxr-xr-x. 2 root root unconfined_u:object_r:etc_t:s0 6 Jul 29 13:17 /tmp/d1/d2
```

### Lab: Add Network Port to Policy Database

- Add network port 9005 to the SELinux policy database for the secure HTTP service using the semanage command.
```bash
[root@server30 ~]# semanage port -l | grep ^http_port
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000

[root@server30 ~]# semanage port -at http_port_t -p tcp 9005
```

- Verify the addition.  
```bash
[root@server30 ~]# semanage port -l | grep ^http_port
http_port_t                    tcp      9005, 80, 81, 443, 488, 8008, 8009, 8443, 9000
```

### Lab: Copy Files with and without Context 

- Create file *sef1* under */tmp*. 
```bash
[root@server30 ~]# touch /tmp/sef1
```

- Copy the file to the */usr/local* directory. 
```bash
[root@server30 ~]# cp /tmp/sef1 /usr/local
```

- Check and compare the contexts on both source and destination files. 
```bash
[root@server30 ~]# ls -lZ /tmp/sef1
-rw-r--r--. 1 root root unconfined_u:object_r:user_tmp_t:s0 0 Jul 29 13:33 /tmp/sef1

[root@server30 ~]# ls -lZ /usr/local/sef1
-rw-r--r--. 1 root root unconfined_u:object_r:usr_t:s0 0 Jul 29 13:33 /usr/local/sef1

```

- Create another file *sef2* under */tmp* and copy it to the */var/local* directory using the `--preserve=context` option with the `cp` command. 
```bash
[root@server30 ~]# touch /tmp/sef2
[root@server30 ~]# cp --preserve=context /tmp/sef2 /var/local/
```

- Check and compare the contexts on both source and destination files. 
```bash
[root@server30 ~]# ls -lZ /tmp/sef2 /var/local/sef2
-rw-r--r--. 1 root root unconfined_u:object_r:user_tmp_t:s0 0 Jul 29 13:35 /tmp/sef2
-rw-r--r--. 1 root root unconfined_u:object_r:user_tmp_t:s0 0 Jul 29 13:36 /var/local/sef2
```

### Lab: Flip SELinux Booleans 

- Check the current value of Boolean `ssh_use_tcpd` using the `getsebool` and `sestatus` commands. 
```bash
[root@server30 ~]# getsebool -a | grep ssh_use_tcpd
ssh_use_tcpd --> off
```

- Use the `setsebool` command and toggle the value of the directive. 
```bash
[root@server30 ~]# setsebool ssh_use_tcpd 1
```

- Confirm the new value with the `getsebool`, `semanage`, or `sestatus` command. 
 ```bash
 [root@server30 ~]# getsebool -a | grep ssh_use_tcpd
ssh_use_tcpd --> on
[root@server30 ~]# sestatus -b | grep ssh_use_tcpd
ssh_use_tcpd                                on
[root@server30 ~]# semanage boolean -l | grep ssh_use_tcpd 
ssh_use_tcpd                   (on   ,  off)  Allow ssh to use tcpd
```


