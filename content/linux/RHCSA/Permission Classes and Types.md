## Permission Classes and Types

**Permission classes**
- user (u)
- group (g)
- other (o) (public)
- all (a) <- all combined

**Permission types**
- r,w,x
- works differently on files and directories
- hyphen (-) represents no permissions set

### ls results permissions groupings 
- - rwx rw- r--
	- user (owner), group, and other (public)
### ls results first character meaning
- regular file
d directory
l symbolic link
c character device file
b block device file
p named pipe
s socket
	

### Modifying Access Permission Bits

### `chmod`  command 
- Modify permissions using symbolic or octal notation.
- Used by root or the file owner.

Flags
chmod -v ::: Verbose.

### Symbolic notation 
- Letters (ugo/rwx) and symbols (+, -, =) used to add, revoke, or assign permission bits.

### Octal Notation 
Three-digit numbering system ranging from 0 to 7.
	0 ---
	1 --x
	2 -w-
	3 -wx
	4 r--
	5 r-x
	6 rw-
	7 rwx

### Default Permissions
- Calculated based on the umask (user mask) value subtracted from the initial permissions value.

#### umask 
- Three-digit value (octal or symbolic) that refers to read, write, and execute permissions for owner, group, and public.
- Default umask value is 0022 for the root user and 0002 normal users.
- The left-most 0 has no significance.
- If umask is set to 000 files will get max of 666
- If the initial permissions are 666 and the umask is 002 then the default permissions are 664. (666-002)
- Any new files or directories created after changing the umask will have the new default permissions set. 
- umask settings are lost when you log off. Add it to the appropriate startup file to make it permanent.

Defaults
- files 666 rw-rw-rw-
- directories 777 rwxrwxrwx

### umask command

Options
- -S symbolic form 

### Special Permission Bits
---
- 3 types of special permission bits for executable files or directories for non root users
	- setuid
	- setgid
	- sticky
- setuid 
	- set on exe's to provide non-owners the ability to run them with the privileges of the owning user
	- may be set on directories and files but will have no effect.
	- example: the su command
	- shows an 's' in ls -l listing at the end of owners permissions
	- If the file already has the “x” bit set for the user, the long listing will show a lowercase “s”, otherwise it will list it with an uppercase “S”.
- setgid 
	- set on exe's to provide non-group members the ability to run them with the privileges of the owning group.
	- May be set on shared directories
		- allow files and subdirectories created underneath to automatically inherit the directory’s owning group.
		- saves group members who are sharing the directory contents from changing the group ID for every new file and subdirectory that they add.
	- write command has this set by default so a member of the tty group can run it. If the file already has the “x” bit set for the group, the long listing will show a lowercase “s”, otherwise it will list it with an uppercase “S”.
- Sticky bit 
	- may be set on public directories for inhibiting file deletion by non-owners
	- may be set on directories and files but will have no effect.
	- Set on /tmp and /var/tmp by default
	- Letter "t" in other permission feild	
	- If the directory already has the “x” bit set for public, the long listing will show a lowercase “t”, otherwise it will list it with an uppercase “T”.
