## Listing Logged-In Users

A list of the users who have successfully signed on to the system with valid credentials can be printed using `who` and `w`

### who command
- references the /run/utmp file and displays the information.
- displays login name of user
- shows terminal session device filename
- pts stands for pseudo terminal session
- shows data and time of user login
- Shows if terminal session is graphical(:0), remote(IP address), or textual on the console

### what command (w)
- Shows length of time the user has been idle
- CPU time used by all processes including any existing background jobs attached to this terminal (JCPU),
- CPU time used by the current process (PCPU),
- current activity (WHAT).
- current system time
- system up duration
- number of users logged in
- cpu averages over last 1, 5, and 15 minutes
- load average (CPU load): 0.00 and 1.00 correspond to no load and full load, and a number greater than 1.00 signifies excess load (over 100%).

### last command
- Reports the history of successful user login attempts and system 
- Consults the wtmp file located in the /var/log directory.
- wtmp keeps a record of login/logout activities
	- login time
	- duration a user stayed logged in
	- tty
- Output
	- Login name
	- Terminal name
	- Terminal name or IP from where connection was established
	- Day, Month, date, and time when the connection was established
	- Log out time or (still logged in)
	- Duration of session
	- Action name (system reboots section)
	- Activity name (system reboots section)
	- Linux kernel version (system reboots section)
	- Day, Month, date, and time when the reboot command was issued (system reboots section)
	- System restart time (system reboots section)
	- Duration the system remained down or (still running) (system reboots section)
	- log filename (wtmp) (last line)

### lastb command
- reports failed login attempts
- Consults /var/log/btmp
	- record of failed login attempts
	- login name
	- time
	- tty
- Must be root to run this command
- Columns
	- name of user
	- protocol used
	- terminal name or ip address
	- Day, Month, Date, and time of the attempt
	- Duration the attempt was tried
	- Duration the attempt last for
	- log filename (btmp) (last line)

### lastlog command
- most recent login evidence info for every user account that exists on the system
- Consults /var/log/lastlog
	- record of most recent user attempts
	- login name
	- time
	- port (or tty)
	- Columns:
		- Login name of user
		- Terminal name assigned upon Logging in
		- Terminal name or Ip address from where the session was initiated
		- Timestamp for the latest login or "Never logged in"
	- service accounts are used by their respective services, and they are not meant for logging.

### id (identifier) Command
- displays the calling user’s:
	- UID (User IDentifier)
	- username
	- GID (Group IDentifier)
	- group name
	- all secondary groups the user is a member of
	- SELinux security context

### groups Command:
- lists all groups the calling user is a member of:
- first group listed is the primary group for the user who executed this command
- other groups are secondary (or supplementary).
- can also view group membership information for a different user.
