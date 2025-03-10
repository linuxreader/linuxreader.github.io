## Linux Groups and their Management
- /etc/group
	- group info
- /etc/login.defs
	- default policies
- /etc/gshadow
	- group administrator information and group-level passwords
- group management tools
	- `groupadd`, `groupmod`, and `groupdel`
	- create, alter, and erase groups

### groupadd command
- adds entries to the group and gshadow files for each group added to the system
- picks up default values from /etc/login.defs
- Switches
	- -g (--gid) 
		- Specifies the GID to be assigned to the group 
	- -o (--non-unique) 
		- Creates a group with a matching GID of an existing group. When two groups have an identical GID, members of both groups get identical rights on each other’s files. This should only be done in specific situations. 
	- -r 
		- Creates a system group with a GID below 1000 
	- groupname 
		- Specifies a group name

### groupmod command
- syntax of this command is very similar to the groupadd with most options identical.
- Additional flags
	- -n
		- change name of existing group
