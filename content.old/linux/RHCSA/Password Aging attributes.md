## Password Aging attributes
- Can be done for an individual user or applied to all users.
- Can prevent users from logging in to the system by locking their access for a period of time or permanently.
- Must be performed by a user with elevated privileges of the root user.
- Normal users may be allowed access to privileged commands by defining them appropriately in a configuration file.
- Each file that exists on the system regardless of its type has an owning user and an owning group.  
- every file that a user creates is in the ownership of that user. 
- ownership may be changed and given to another user by a super user.

### Password Aging and management
- settning restrictios on password expiry, account disablement, locking and unlocking users, and password change frequency.
- Can choose to inactivate it completely for an individual user.
- Stored in the /etc/shadow file (fields 4 to 8) and its default policies in the /etc/login.defs configuration file.
- aging management tools—`chage` and `passwd`—
- `usermod` command can be used to implement two aging attributes (user expiry and password expiry) and lock and unlock user accounts.

### chage command
- Set or alter password aging parameters on a user account.
- Changes various fields in the shadow file
- Switches
	- -d (--lastday) 
		- Specifies an explicit date in the YYYY-MM-DD format, or the number of days since the UNIX time when the password was last modified. With -d 0, the user is forced to change the password at next login. It corresponds to field 3 in the shadow file. 
	- -E (--expiredate) 
		- Sets an explicit date in the YYYY-MM-DD format, or the number of days since the UNIX time on which the user account is deactivated. This feature can be disabled with -E -1. It corresponds to the eighth field in the shadow file. 
	- -I (--inactive) 
		- Defines the number of days of inactivity after the password expiry and before the account is locked. The user may be able to log in during this period with their expired password. This feature can be disabled with -I -1. It corresponds to field 7 in the shadow file. 
	- -l 
		- Lists password aging attributes set on a user account. 
	- -m (--mindays) 
		- Indicates the minimum number of days that must elapse before the password can be changed. A value of 0 allows the user to change their password at any time. It corresponds to field 4 in the shadow file. 
	- -M (--maxdays) 
		- Denotes the maximum number of days of password validity before the user password expires and it must be changed. This feature can be disabled with -M -1. It corresponds to field 5 in the shadow file. 
	- -W (--warndays) 
		- Designates the number of days for which the user gets alerts to change their password before it expires. It corresponds to field 6 in the shadow file.

### passwd command
- set or modify a user’s password
- modify the password aging attributes and 
- lock or unlock account
- Switches
	- -d (--delete) 
		- Deletes a user password 
		- does not expire the user account. 
	- -e (--expire) 
		- Forces a user to change their password upon next logon. 
		- sets date to prior to Unix time
	- -i (--inactive) 
		- Defines the number of days of inactivity after the password expiry and before the account is locked. (field 7 in shadow file)
	- -l (--lock) 
		- Locks a user account. 
	- -n (--minimum) 
		- Specifies the number of days that must elapse before the password can be changed. (field 4 in shadow file)
	- -S (--status) 
		- Displays the status information for a user. 
	- -u (--unlock) 
		- Unlocks a locked user account.
	- -w (--warning) 
		- Designates the number of days for which the user gets alerts to change their password before it actually expires. (field 6 in shadow file)
	- -x (--maximum) 
		- Denotes the maximum number of days of password validity before the user password expires and it must be changed. (field 5 in shadow file)
### usermod command
- Modify a user’s attribute
- Lock or unlock their account
- Switches
	- -L (--lock) 
		- Locks a user account by placing a single exclamation mark (!) at the beginning of the password field and before the hashed password string. 
	- -U (--unlock) 
		- Unlocks a user’s account by removing the exclamation mark (!) from the beginning of the password field. 

