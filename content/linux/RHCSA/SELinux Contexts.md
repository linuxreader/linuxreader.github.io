## SELinux Contexts
### SELinux Contexts for Users 

- SELinux contexts define security attributes placed on subjects and objects. 
- Each context contains a type and a security level with subject and object information. 

Use the `id` command with the `-Z` option to view the context set on Linux users:
```bash
[root@server30 ~]# id -Z
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

Output:
- Mapped to the SELinux unconfined_u user
- No SELinux restrictions placed on this user.
- All Linux users, including root, run unconfined by default, (full system access)

- Seven confined user identities with restricted access to objects. 
	- Mapped to Linux users via SELinux policy.
	- Helps safeguard the system from potential damage that Linux users might inflict on the system.

Use the `seinfo` query command to list the SELinux users; however, the `setools-console` software package must be installed before doing so.
```bash
[root@server30 ~]# seinfo -u

Users: 8
   guest_u
   root
   staff_u
   sysadm_u
   system_u
   unconfined_u
   user_u
   xguest_u
```

Use the `semanage` command to view the mapping between Linux and SELinux users:
```bash
[root@server30 ~]# semanage login -l

Login Name           SELinux User         MLS/MCS Range        Service

__default__          unconfined_u         s0-s0:c0.c1023       *
root                 unconfined_u         s0-s0:c0.c1023       *
```

**MLS/MCS Range** 
- Associated security level and the context for the Linux user (the \* represents all services). 
- By default, all non-root Linux users are represented as `__default__`, which is mapped to the unconfined_u user in the policy.

### SELinux Contexts for Processes 

Determine the context for processes using the `ps` command with the `-Z` flag: 
```bash
[root@server30 ~]# ps -eZ | head -2
LABEL                               PID TTY          TIME CMD
system_u:system_r:init_t:s0           1 ?        00:00:02 systemd
```

Output:
- The subject *system_u* is a SELinux username (mapped to Linux user root)
- Object is *system_r*
- Domain *init_t* reveals the type of protection applied to the process. 
- Level of security `s0`

- A process that is unprotected will run in the unconfined_t domain.

### SELinux Contexts for Files 

`ls -Z`
- View context for files and directories.

Show the four attributes set on the /etc/passwd file:
```bash
[root@server30 ~]# ls -lZ /etc/passwd
-rw-r--r--. 1 root root system_u:object_r:passwd_file_t:s0 2806 Jul 19 21:54 /etc/passwd
```

- Subject *system_u*
- Object *object_r*
- Type `passwd_file_t` 
- Security level `s0` for the passwd file. 
 
 */etc/selinux/targeted/contexts/files/file_contexts* 
 */etc/selinux/targeted/contexts/files/file_contexts.local*
- Stores contexts for system-installed and user-created files.
- Policy files.
- Can be updated using the `semanage` command.

### Copying, Moving, and Archiving Files with SELinux Contexts 

- All files in RHEL are labeled with an SELINUX security context by default.
- New files inherit the parent directory's context at the time of creation. 

Rules for copy move and archive:

1. If a file is copied to a different directory, the destination file will receive the destination directory's context, unless the `--preserve=context` switch is specified with the `cp` command to retain the source file's original context.

2. If a copy operation overwrites the destination file in the same or different directory, the file being copied will receive the context of the overwritten file, unless the `--preserve=context` switch is specified with the `cp` command to preserve the source file's original context.

3. If a file is moved to the same or different directory, the SELinux context will remain intact, which may differ from the destination directory's context.

4. If a file is archived with the tar command, use the `--selinux` option to preserve the context.

### SELinux Contexts for Ports 

View attributes for network ports with the `semanage` command:
```bash
[root@server30 ~]# semanage port -l | head -7
SELinux Port Type              Proto    Port Number

afs3_callback_port_t           tcp      7001
afs3_callback_port_t           udp      7001
afs_bos_port_t                 udp      7007
afs_fs_port_t                  tcp      2040
afs_fs_port_t                  udp      7000, 7005
```

- By default, SELinux allows services to listen on a restricted set of network ports only.

 
