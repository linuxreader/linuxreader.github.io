## Domain Transitioning 

- SELinux allows a process running in one domain to enter another domain to execute an application that is restricted to run in that domain only.
- A rule must exist in the policy to support such transition. 
- **entrypoint** 
	- Permission setting
	- Control processes that can transition into another domain. 

Example: 
What happens when a Linux user attempts to change their password using the `/usr/bin/passwd` command.

The `passwd` command is labeled with the passwd_exec_t type:
```bash
[root@server30 ~]# ls -lZ /usr/bin/passwd
-rwsr-xr-x. 1 root root system_u:object_r:passwd_exec_t:s0 32648 Aug 10  2021 /usr/bin/passwd
```

The `passwd` command requires access to the `/etc/shadow` file in order to modify a user password. The shadow file has a different type set on it
(*shadow_t*):
```bash
**[root@server30 ~]# ls -lZ /etc/shadow
----------. 1 root root system_u:object_r:shadow_t:s0 2756 Jul 19 21:54 /etc/shadow
```

- The SELinux policy has rules that specifically allow processes running in domain *passwd_t* to read and modify the files with type *shadow_t*, and allow them **entrypoint** permission into domain *passwd_exec_t*. 
- This rule enables the user's shell process executing the `passwd` command to switch into the *passwd_t* domain and update the shadow file.

Open two terminal windows. In window 1, issue the passwd command as
user1 and wait at the prompt:
```bash
[user1@server30 root]$ passwd
Changing password for user user1.
Current password: 
```

In window 2, run the ps command:
```bash
[root@server30 ~]# ps -eZ | grep passwd
unconfined_u:unconfined_r:passwd_t:s0-s0:c0.c1023 13001 pts/1 00:00:00 passwd
```

- The `passwd` command (process) transitioned into the *passwd_t* domain to change the user password. 

 
