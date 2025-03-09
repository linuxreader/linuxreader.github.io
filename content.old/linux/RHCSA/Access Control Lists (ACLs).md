## Access Control Lists (ACLs)

- Setting a default ACL on a directory allows content sharing among user's without having to modify access on each new file and subdirectory.
- Extra permissions that can be set on files and directories.
- Define permissions for named user and named groups.
- Configured the same way on both files and directories.
- Named Users
	- May or may not be a part of the same group.
- 2 different groups of ACLs. Default ACLs and Access ACLs.
	- Access ACLs
		- Set on individual files and directories
	- Default ACLs
		- Applied on directories
		- files and subdirectories inherit the ACL
		- Execute bit must be set on the directory for public.
		- Files receive the shared directory's default ACLs as their access ACLs - what the mask limits.
		- Subdirectories receive both default ACLs and access ACLs as they are.

- A "+" at the end of ls -l listing indicates ACL is set
	- -rw-rw-r--+

### ACL Commands
`getfacl`
- Display ACL settings
	- Displays:
	- name of file
	- owner
	- owning group
	- Permissions
		- colon characters save space for named user/group (or UID/GID) when extended Permissions are set.
		- Example: user:1000:r--
			- the named user with UID 1000, who is neither the file owner nor a member of the owning group, is allowed read-only access to this file.
		- Example: group:dba:rw- 
			- give the named group (dba) read and write access to the file.
`setfacl`
	- set, modify, substitute, or delete ACL settings
	- If you want to give read and write permissions to a specific user (user1) and change the mask to read-only at the same time, the setfacl command will allocate the permissions as mentioned; however, the effective permissions for the named user will only be read-only.

u:UID:perms
- named user must exist in /etc/passwd
- if no user specified, permissions are given to the owner of the file/directory

g:GID:perms
- Named group must exist in /etc/group
- If no group specified, permissions are given to the owning group of the file/directory

o:perms
- Neither owner or owning group

m:perms
- Maximum permissions for named user or named group

Switches

| Switch | Description                     |
| ------ | ------------------------------- |
| -b     | Remove all Access ACLs          |
| -d     | Applies to default ACLs         |
| -k     | Removes all default ACLs        |
| -m     | Sets or modifies ACLs           |
| -n     | Prevent auto mask recalculation |
| -R     | Apply Recursively to directory  |
| -x     | Remove Access ACL               |
| -c     | Display output without header   |

### Mask Value

- Determine maximum allowable permissions for named user or named group
- Mask value displayed on separate line in getfacl output
- Mask is recalculated every time an ACL is modified unless value is manually entered.
- Overrides the set ACL value.
