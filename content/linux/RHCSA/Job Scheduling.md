## Job Scheduling
- Run a command at a specified time.
- One time or periodic.
- One time command can be used to run a command at a time with low system usage.
- Periodic examples: 
	- creating a compressed archive
	- trimming log files
	- monitoring the system
	- running a custom script 
	- removing unwanted files from the system.
- `atd` and `crond` manage jobs

### atd
- Run one time jobs.
- atd daemon retries a missed job at the same time next day.
- Does not need a restart with changes

### crond 
- Run periodic scheduled jobs.
- Daemon reads the schedules in files located in the */var/spool/cron* and */etc/cron.d* directories.
	- scans these files in short intervals
	- updates the in-memory schedules to reflect any modifications. 
	- runs a job at its scheduled time only
	- does not entertain any missed jobs.
	- Does not need a restart with changes

### Controlling user access
- all users can schedule jobs
- access to job scheduling can be edited
	- must add users to allowed or deny file in */etc*
		- */etc/at.allow* & */etc/cron.allow*
			- Does not exist by default.
		- */etc/at.deny* & */etc/cron.deny*
			- Exists by default
	- list one username per line
	- root user is always permitted
- Denial message appears if unauthorized user attempts to use at or cron.
	- Only if there is an entry for the calling user in the deny files.
	
	| at.allow / cron.allow             | at.deny / cron.deny               | Impact                                                          |
	| --------------------------------- | --------------------------------- | --------------------------------------------------------------- |
	| Exists, and contains user entries | Existence does not matter         | All users listed in allow files are permitted                   |
	| Exists, but is empty              | Existence does not matter         | No users are permitted                                          |
	| Does not exist                    | Exists, and contains user entries | All users, other than those listed in deny files, are permitted |
	| Does not exist                    | Exists, but is empty              | All users are permitted                                         |
	|            Does not exist                       |      Does not exist                             |            No users are permitted                                                      |
	          

### Scheduler Log File
*/var/log/cron*
	- Logs for both `atd` and `cron`
	Shows
		- time of activity
		- hostname
		- process name and PID
		- owner
		- message for each invocation
		- service start time and delays
	- must have root privileges to view

### `at` command
- schedule a one-time execution of a program in the future. 
- Submitted jobs are spooled in the */var/spool/at/* and executed by the atd daemon at the specified time. 
- file created containing the settings for establishing the user’s shell environment to ensure a successful execution. 
	- also includes the name of the command or program to be run.
- no need to restart the daemon after a job submission.
- assumes the current year and today’s date if the year and date are not mentioned.
- ways to express time:
	- at 1:15am 
		- (executes the task at the next 1:15 a.m.) 
	- at noon 
		- (executes the task at 12:00 p.m.) 
	- at 23:45 
		- (executes the task at 11:45 p.m.) 
	- at midnight 
		- (executes the task at 12:00 a.m.) 
	- at 17:05 tomorrow 
		- (executes the task at 5:05 p.m. on the next day) 
	- at now + 5 hours 
		- (executes the task 5 hours from now. We can specify minutes, days, or weeks in place of hours) 
	- at 3:00 10/15/20 
		- (executes the task at 3:00 a.m. on October 15, 2020)
- Flags
	- -f
		- supply a filename

### Crontab

### `crontab` command
- other method for scheduling tasks for running in the future. 
- Unlike atd, crond executes cron jobs on a regular basis as defined in the */etc/crontab* file. 
- Crontables (another name for crontab files) are located in the */var/spool/cron* directory. 
- Each authorized user with a scheduled job has a file matching their login name in this directory.
	- such as */var/spool/cron/user1*
- */etc/crontab/* & */etc/cron.d/*
	- Other locations for system crontables.
	- Only root can create, modify, or delete them. 
- crond daemon 
	- scans entries in all 3 directories.
	- adds log entry to */var/log/cronfile*
	- no need to start after modifying cron jobs.
- flags
	- -e
		- edit crontables
	- -l
		- list crontables
	- -r
		- remove crontables. 
		- Do not run crontab -r if you do not wish to remove the crontab file. Instead, edit the file with crontab -e and just erase the entry.
	- -u 
		- modify a different user’s crontable
		- provided they are allowed to do so and the other user is listed in the cron.allow file.
		- root user can use the -u flag to alter other users’ crontables even if the affected users are not listed in the allow file.

### Syntax of User Crontables
- */etc/crontab*
	- Specifies the syntax that each user cron job must comply with in order for crond to interpret and execute it successfully.
- Each entry for a user crontable has 6 lines
	- 1-5
		- schedule
	- 6
		- login name of executing user
	- rest for command or program to be executed. 
example crontable line
	- 20 1,12 1-15 feb * ls> /tmp/ls.out
- Field Content Description 
- 1 
	- Minute of the hour 
	- Valid values are 0 (the exact hour) to 59. This field can have one specific value as in field 1, multiple comma-separated values as in field 2, a range of values as in field 3, a mix of fields 2 and 3 (1-5,6-19), or an * representing every minute of the hour as in field 5. 
- 2 
	- Hour of the day 
	- Valid values are 0 (midnight) to 23. Same usage applies as described for field 1. 
- 3 
	- Day of the month 
	- Valid values are 1 to 31. Same usage applies as described for field 1. 
- 4 
	- Month of the year 
	- Valid values are 1 to 12 or jan to dec. Same usage applies as described for field 1. 
- 5 
	- Day of the week 
	- Valid values are 0 to 7 or sun to sat, with 0 and 7 representing Sunday, 1 representing Monday, and so on. Same usage applies as described for field 1. 
- 6 
	- Command or program to execute 
	- Specifies the full path name of the command or program to be executed, along with any options or arguments that it requires.

*/etc/crontab* contents:
![](Pasted%20image%2020240815080652.png)

- Step values may be used with \* and ranges in the crontables using the forward slash character (/). 
- Step values allow the number of skips for a given value. 
- Example:
	- /2 in the minute field 
		- every second minute
	- /3 in the minute field
		- every third minute, 
	- 0-59/4 in the minute field
		- every 4th minute

Make sure you understand and memorize the order of the fields defined in crontables.

### Anacron
- service that runs after every system reboot
- checks for any cron and at jobs that were scheduled for execution during the time the system was down and were missed as a result.
- useful on laptop, desktop, and similar purpose systems with extended periods of frequent downtimes and are not intended for 24/7 operations.
- Scans the */etc/cron.hourly/0anacron* file for three factors to learn whether to run missed jobs.
- May be run manually at the command line.
	- Run `anacron` to run all jobs in */etc/anacrontab* that were missed.
- */var/spool/anacron*
	- Where anacron stores job execution dates
- 3 factors must be true for anacron to execute scripts in */etc/cron.daily*, */etc/cron.weekly*, and */etc/cron.monthly*
	- 1. Presence of the */var/spool/anacron/cron.daily* file.
	- 2. Elapsed time of 24 hours since it was last run.
	- 3. System is plugged in to an AC source.
- settings defined in */etc/anacrontab*
	- 5 variables defined by default:
		- **SHELL** and **PATH** 
			- Set the shell and path to be used for executing the programs. 
		- **MAILTO** 
			- Defines the login name or an email of the user who is to be sent any output and error messages. 
		- **RANDOM_DELAY** 
			- Expresses the maximum arbitrary delay in minutes added to the base delay of the jobs as defined in column 2 of the last three lines. 
		- **START_HOURS_RANGE** 
			- States the hour duration within which the missed jobs could be run.
	- Bottom 3 lines define the schedule and the programs to be executed:
		- Column 1: 
			- Period in days (or @daily, @weekly, @monthly, or @yearly)
			- How often to run the specified job.
		- Column 2: 
			- How many minutes to wait after system boot to execute the job. 
		- Column 3: 
			- Unique job identifier 
		- Columns 4 to 6: 
			- Command to be used to execute the scripts located under the */etc/cron.daily*, */etc/cron.weekly*, and */etc/cron.monthly* directories. 
			- By default, the `run-parts` command is invoked for execution at the default niceness.
	- For each job:
		- Examines whether the job was already run during the specified period (column 1). 
		- Executes it after waiting for the number of minutes (column 2) plus the **RANDOM_DELAY** value if it wasn’t. 
		- When all missed jobs have been carried out and there is none pending, Anacron exits.
