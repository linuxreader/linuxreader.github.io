## User Account Management
### useradd Command
- add a new user to the system
- adds entries to the four user authentication files for each account added to the system
- creates a home directory for the user and copies the default user startup files from the skeleton directory /etc/skel into the user’s home directory
- used to update the default settings that are used at the time of new user creation for unspecified settings
- Options
	- -b (--base-dir) 
		- Defines the absolute path to the base directory for placing user home directories. The default is /home. 
	- -c (--comment) 
		- Describes useful information about the user. 
	- -d (--home-dir) 
		- Defines the absolute path to the user home directory. 
	- -D (--defaults) 
		- Displays the default settings from the /etc/default/useradd file and modifies them. 
	- -e (--expiredate) 
		- Specifies a date on which a user account is automatically disabled. The format for the date specification is YYYY-MM-DD. 
	- -f (--inactive) 
		- Denotes maximum days of inactivity between password expiry and permanent account disablement. 
	- -g (--gid) 
		- Specifies the primary GID. Without this option, a group account matching the username is created with the GID matching the UID. 
	- -G (--groups) 
		- Specifies the membership to supplementary groups.
	- -k (--skel) 
		- location of the skeleton directory (default is /etc/skel) (stores default user startup files)
			- These files are copied to the user’s home directory at the time of account creation. 
			- Three hidden bash shell files: (default)
				- .bash_profile, .bashrc, and .bash_logout
				- You can customize these files or add your own to be used for accounts created thereafter.
	- -m (--create-home) 
		- Creates a home directory if it does not already exist.
	- -o (--non-unique) 
		- Creates a user account sharing the UID of an existing user. 
		- When two users share a UID, both get identical rights on each other’s files. 
		- Should only be done in specific situations. 
	- -r (--system) 
		- Creates a service account with a UID below 1000 and a never-expiring password.
	- -s (--shell) 
		- Defines the absolute path to the shell file. The default is /bin/bash. 
	- -u (--uid) 
		- Indicates a unique UID. Without this option, the next available UID from the /etc/passwd file is used. 
	- login 
		- Specifies a login name to be assigned to the user account.
### usermod Command
- modify the attributes of an existing user
- similar syntax to useradd and most switches identical.
- Options unique to usermod:
	- -a (--append) 
		- Adds a user to one or more supplementary groups 
	- -l (--login) 
		- Specifies a new login name
	- -m (--move-home) 
		- Creates a home directory and moves the content over from the old location
	- -G
		- Add a list of groups a user is a member of.
### userdel Command
- to remove a user from the system
### passwd Command
- set or modify a user’s password

### No-Login (Non-Interactive) User Account
### nologin command
- /sbin/nologin
- special purpose program that can be employed for user accounts that do not require login access to the system. 
- located in the /usr/sbin (or /sbin) directory
- user is refused with the message, “This account is currently not available.” 
- If a custom message is required, you can create a file called nologin.txt in the /etc directory and add the desired text to it.
- If a no-login user is able to log in with their credentials, there is a problem. Use the grep command against the /etc/passwd file to ensure ‘/sbin/nologin’ is there in the shell field for that user.
- examples of user accounts that do not require login access are the service accounts such as ftp, apache, and sshd.