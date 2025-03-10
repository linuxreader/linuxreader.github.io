## SELinux Booleans 

- on/off switches that SELinux uses to determine whether to permit an action. 
- Activate or deactivate certain rule in the SELinux policy immediately and without the need to recompile or reload the policy. 
- For instance, the *ftpd_anon_write* Boolean can be turned on to enable anonymous users to upload files. 
- This privilege can be revoked by turning this Boolean off. 
- Boolean values are stored in virtual files located in */sys/fs/selinux/booleans/*. 
- The filenames match the Boolean names. 

A sample listing of this directory is provided below:
```bash
[root@server30 ~]# ls -l /sys/fs/selinux/booleans/ | head -7
total 0
-rw-r--r--. 1 root root 0 Jul 23 04:44 abrt_anon_write
-rw-r--r--. 1 root root 0 Jul 23 04:44 abrt_handle_event
-rw-r--r--. 1 root root 0 Jul 23 04:44 abrt_upload_watch_anon_write
-rw-r--r--. 1 root root 0 Jul 23 04:44 antivirus_can_scan_system
-rw-r--r--. 1 root root 0 Jul 23 04:44 antivirus_use_jit
-rw-r--r--. 1 root root 0 Jul 23 04:44 auditadm_exec_content
```

- The manual pages of the Booleans are available through the `selinux-policy-doc` package. 
- Once installed, use the `-K` option with the `man` command to bring the pages up for a specific Boolean. 
- For instance, issue `man -K abrt_anon_write` to view the manual pages for the `abrt_anon_write` Boolean.

- Can be viewed, and flipped temporarily or for permanence.
- New value takes effect right away. 
- Temporary changes are stored as a "1" or "0" in the corresponding Boolean file in the */sys/fs/selinux/booleans/* 
- Permanent changes are saved in the policy database. 
