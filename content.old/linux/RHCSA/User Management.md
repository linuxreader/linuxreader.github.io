## User Management

### Switching Users

#### su command
Ctrl-d
	- return to previous user
su -
	- switch user with startup scripts
-c 
	- issue a command as a user without switching to them.
- root user can switch into any user account that exists on the system without being prompted for that user’s password.
- switching into the root account to execute privileged actions is not recommended.

### whoami command
- show current user
### logname command
- Identity of the user who originally logged in.
### groupdel command
- removes entries for the specified group from both group and gshadow files.

### Doing as Superuser (substutute user)
- Any normal user that requires privileged access to administrative commands or non-owning files is defined in the sudoers file.
	- File may be edited with a command called `visudo`
	- Creates a copy of the file as sudoers.tmp and applies the changes there. After the visudo session is over, the updated updated file overwrites the original sudoers file and sudoers.tmp is deleted.
	- syntax
		- user1 ALL=(ALL) ALL
		- %dba ALL=(ALL) ALL
			group is prefixed by %
	- Make it so members are not prompted for password
		- user1 ALL=(ALL) NOPASSWD:ALL
		- %dba ALL=(ALL) NOPASSWD:ALL
	- Limit access to a single command
		- user1 ALL=/usr/bin/cat
		- %dba ALL=/usr/bin/cat
- too many entries can clutter sudoers file. Use aliases instead:
	- User_Alias
		- you can define a User_Alias called PKGADM for user1, user100, and user200. These users may or may not belong to the same Linux group.
	- Cmnd_Alias
		- you can define a Cmnd_Alias called PKGCMD containing yum and rpm package management commands
### sudo command
- /etc/sudoers
- /etc/sudoers.d/
	- drop-in directory
/var/log/secure
	- Sudo logs successful authentication and command data to here under the name of the user using the command.
### Owning User and Owning Group
- Every file and directory has an owner.
- Creator assumes ownership by default.
- Every user is a member of one or more groups.
- Owners group is also assigned to file or directory by default.

### chown command
- alter the ownership for files and directories
- Must have root privileges.
- Can also change owning group.
### chgrp command
- alter the owning group for files and directories
- Must have root privileges.
