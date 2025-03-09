## Local User Authentication Files
- Three supported account types: root, normal, service
- root 
	- has full access to all services and administrative functions on the system.
	- created by default during installation.
- Normal
	- user-level privileges
	- cannot perform any administrative functions
	- can run applications and programs that have been authorized.
- Service
	- take care of their respective services, which include apache, ftp, mail, and chrony.
- User account information for local users is stored in four files that are located in the /etc directory.
	- passwd, shadow, group, and gshadow (user authentication files)
	 - updated when a user or group account is created, modified, or deleted.
	 - referenced to check and validate the credentials for a user at the time of their login attempt,
	 - system creates their automatic backups by default as passwd-, shadow-, group-, and gshadow- in the /etc directory.
### /etc/passwd
- vital user login data
- each row hold info for one user 
- 644 permissions by default
- 7 feilds per row
	- login name
		- up to 255 characters
		- _ and - characters are supported
		- not recommended to include special characters and uppercase letters in login names.
	- password
		- "x" in this field points to /etc/shadow for actual password.
		- "\*" identifies disabled account
		- Can also include a hashed password (RHEL uses SHA-512 by default)
	- UID
		- Number between 0 and 4.2 billion
		- UID 0 is reserved for root account
		- UIDs 1-200 are used by Red Hat for core service accounts
		- UIDs 201-999 are reserved for non-core service accounts
		- UIDs 1000 < are for normal user accounts (starts at 1000 by default)
	- GID
		- GID that matches entry in /etc/group (primary group)
		- Group for every user by default that matches UID
	- Comments (GECOS) or (GCOS)
		- general comments about the user
	- Home Directory
		- absolute path to the user home directory.
	- Shell
		- absolute path of the shell file for the user's primary shell after logging in. (default = (/bin/bash))

### /etc/shadow
- no access permissions for any user (even root) (but owned by root)
- secure password control (shadow password)
- user passwords are hashed and stored in a more secure file /etc/shadow,
- limits on user passwords in terms of expiration, warning period, etc. applied on per-user basis
- limits and other settings are defined in /etc/login.defs
- user is initially checked in the passwd file for existence and then in the shadow file for authenticity.
- contains user authentication and password aging information. 
- Each row in the file corresponds to one entry in the passwd file. 
- login names are used as a common key between the shadow and passwd files.
- nine colon-separated fields per line entry.
	- 1 Login name
	- 2 Encrypted password
		- ! at the beginning of this field shows that the user account is locked
		- if field is empty then user has passwordless entry
	- 3 last change
		- Number of days (lastchg) since the UNIX epoch, (UNIX time (January 01, 1970 00:00:00 UTC) when the password was last modified. 
		- Empty field represents the passiveness of password aging features.
		- 0 forces the user to change their password upon next login.
	- 4 minimum 
		- number of days (mindays) that must elapse before the user is allowed to change their password
		- can be altered using the `chage` command with the `-m` option or the `passwd` command with the `-n` option. 
		- 0 or null in this field disables this feature.
	- 5 (Maximum)
		- maximum number of days (maxdays) before the user password expires and must be changed. 
		- may be altered using the `chage` command with the `-M `option or the `passwd` command with the `-x` option. 
		- null value here disables this feature along with other features such as the maximum password age, warning alerts, and the user inactivity period.
	- 6 Field 6 (Warning)
		- number of days (warndays) the user gets warnings for changing their password before it expires.
		- may be altered using the `chage` command with the `-W` option or the `passwd` command with the `-w` option. 
		- 0 or null in this field disables this feature.
	- 7 Password Expiry)
		 - maximum allowable number of days for the user to be able to log in with the expired password. (inactivity period).
		 - may be altered using the `chage` command with the `-I` option or the `passwd` command with the `-i` option. 
		 - empty field disables this feature. 
	- 8 (Account Expiry) 
		- number of days since the UNIX time when the user account will expire and no longer be available. 
		- may be altered using the chage command with the `-E` option. 
		- empty field disables this feature. 
	- 9 (Reserved): Reserved for future use.

### /etc/group
- plaintext file and contains critical group information. 
- 644 permissions by default and owned by root.
- Each row in the file stores information for one group entry. 
- Every user on the system must be a member of at least one group (User Private Group (UPG)). 
- a group name matches the username it is associated with by default
- four colon-separated fields per line entry.
	- Field 1 (Group Name): 
		- Holds a group name that must begin with a letter. Group names with up to 255 characters, including the 
		- uppercase, underscore (\_) and hyphen (-) characters, are also supported. (not recommended)
	 - Field 2 (Encrypted Password): 
		 - Can be empty or contain an “x” (points to the /etc/gshadow file for the actual password), or a hashed group-level password. 
		 - can set a password on a group for non-members to be able to change their group identity temporarily using the `newgrp `command.
		 - non-members must enter the correct password in order to do so. 
	- Field 3 (GID): 
		- Holds a GID, that is also placed in the GID field of the passwd file. 
		- By default, groups are created with GIDs starting at 1000 and with the same name as the username.  
		- system allows several users to belong to a single group
		- also allows a single user to be a member of multiple groups at the same time. 
	- Field 4 (Group Members): 
		- Lists the membership for the group. (user’s primary group is always defined in the GID field of the passwd file.)



### /etc/gshadow
- no access permissions for any user (even root)
- group passwords are hashed and stored
- group names are used as a common key between the gshadow and group files.
- 000 permissions and owned by root
- four colon-separated fields
	- Field 1 (Group Name): 
		- Consists of a group name as appeared in the group file. 
	- Field 2 (Encrypted Password): 
		- Can contain a hashed password, which may be set with the gpasswd command for non-group members to access the group temporarily using the newgrp command. 
		- single exclamation mark (!) or a null value in this field allows group members password-less access and restricts non-members from switching into this group. 
	- Field 3 (Group Administrators): 
		- Lists usernames of group administrators that are authorized to add or remove members with the gpasswd command. 
	- Field 4 (Members): 
		- comma-separated list of members.
### gpasswd command:
	- add group administrators.
	- add or delete group members.
	- assign or revoke a group-level password.
	- disable the ability of the newgrp command to access a group. 
	- picks up the default values from the /etc/login.defs file.

### useradd and login.defs configuration files
### useradd command
- picks up the default values from the /etc/default/useradd and /etc/login.defs files for any options that are not specified at the command line when executing it.
- login.defs file is also consulted by the usermod, userdel, chage, and passwd commands 
- Both files store several defaults including those that affect the password length and password lifecycle.
/etc/default/useradd Default Directives:
- starting GID (GROUP) (provided the USERGROUPS_ENAB directive in the login.defs file is set to no)
- home directory location (HOME), 
- number of inactivity days between password expiry and permanent account disablement (INACTIVE), 
- account expiry date (EXPIRE), 
- login shell (SHELL), 
- skeleton directory location to copy user initialization files from (SKEL)
- whether to create mail spool directory (CREATE_MAIL_SPOOL)

### /etc/login.defs default directives:
MAIL_DIR 
- mail directory location 

PASS_MAX_DAYS, PASS_MIN_DAYS, PASS_MIN_LEN, and PASS_WARN_AGE 
- password aging attributes. 

UID_MIN, UID_MAX, GID_MIN, and GID_MAX 
- ranges of UIDs and GIDs to be allocated to new users and groups 

SYS_UID_MIN, SYS_UID_MAX, SYS_GID_MIN, and SYS_GID_MAX 
- ranges of UIDs and GIDs to be allocated to new service users and groups 

CREATE_HOME 
- whether to create a home directory 

UMASK 
- permissions to be set on the user home directory at creation based on this umask value 

USERGROUPS_ENAB 
- whether to delete a user’s group (at the time of user deletion) if it contains no more members 

ENCRYPT_METHOD 
- encryption method for user passwords
