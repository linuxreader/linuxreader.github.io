## System Logging

- Log files need to be rotated periodically to prevent the file system space from filling up. 
- Configuration files that define the default and custom locations to direct the log messages to and to configure rotation settings. 
system log file
- records custom messages sent to it. 
- systemd includes a service for viewing and managing system logs in addition to the traditional logging service. 
- This service maintains a log of runtime activities for faster retrieval and can be configured to store the information permanently.

*System logging* (*syslog* for short) 
- capture messages generated by:
	- kernel
	- daemons
	- commands
	- user activities
	- applications
	- other events
- Forwards messages to various log files
- For security auditing, service malfunctioning, system troubleshooting, or informational purposes.

*rsyslogd* daemon (*rocket-fast system for log processing*)
- Responsible for system logging
- Multi-threaded
- support for:
	- enhanced filtering
	- encryption-protected message relaying
	- variety of configuration options.
- Reads its configuration file /etc/rsyslog.conf and the configuration files located in /etc/rsyslog.d/ at startup. 
- /var/log
	- Default depository for most system log files 
	- Other services such as audit, Apache, etc. have subdirectories here as well.

rsyslog service
- modular
	- allows the modules listed in its configuration file to be dynamically loaded in the kernel when/as needed. 
	- Each module brings a new functionality to the system upon loading.

*rsyslogd* daemon 
- can be stopped manually using `systemctl stop rsyslog` 
- start, restart, reload, and status options are also available

- A PID is assigned to the daemon at startup
- rsyslogd.pid file is created in the */run* directory to save the PID. 
- PID is stored to prevent multiple instances of this daemon.

### TheSyslog Configuration File
/etc/rsyslog.conf 
- primary syslog configuration file

View /etc/rsyslog.conf:
`cat /etc/rsyslog.conf` 

Output:
Three sections:
- Modules, Global Directives, and Rules. 
	- Modules section 
		- default defines two modules imuxsock and imjournal
		- loaded on demand.
imuxsock module 
- furnishes support for local system logging via the *logger* command
imjournal module
- allows access to the systemd journal.

- Global Directives section 
	- contains three active directives. 
	- Definitions in this section influence the overall functionality of the *rsyslog* service.
		- first directive 
			- Sets the location for the storage of auxiliary files (/var/lib/rsyslog).
		- second directive 
			- instructs the *rsyslog* service to save captured messages using traditional file formatting
		- third directive 
			- directs the service to load additional configuration from files located in the /etc/rsyslogd.d/ directory.

- Rules section
	- Right field is referred to as *action*. 
	selector field 
	- left field of the rules section
	- divided into two period-separated sub-fields called 
		- *facility* (left) 
			- representing one or more system process categories that generate messages
		- *priority* (right)
			- identifying the severity associated with the messages.  
	- semicolon (;) is used as a distinction mark if multiple facility.priority groups are present.
	action field
	- determines the destination to send the messages to.
	- numerous supported facilities:
		- auth
		- authpriv
		- cron
		- daemon
		- kern
		- lpr
		- mail
		- news
		- syslog
		- user 
		- uucp
		- local0 throughv local7
		- asterisk (\*) character represents all of them.
	- supported priorities in the descending criticality order:
		- emerg
		- alert
		- crit
		- error
		- warning
		- notice
		- info
		- debug
		- none
- If a lower priority is selected, the daemon logs all messages of the service at that and higher levels.

After modifying the syslog configuration file,  Inspect it and set the verbosity:
`rsyslogd -N 1`  (-N inspect, 1 level 1)  

- Restart or reload the rsyslog service in order for the changes to take effect.

### Rotating Log Files

Log location is defined in the rsyslog configuration file. 

View the /var/log/ directory:
`ls -l /var/log` 

systemd unit file called logrotate.timer under the /usr/lib/systemd/system directory invokes the logrotate service (/usr/lib/systemd/system/logrotate.service) on a daily basis. Here is what this file contains:
```bash
 [root@localhost cron.daily]# systemctl cat logrotate.timer

 # /usr/lib/systemd/system/logrotate.timer
 [Unit]
 Description=Daily rotation of log files
 Documentation=man:logrotate(8) man:logrotate.conf(5)

 [Timer]
 OnCalendar=daily
 AccuracySec=1h
 Persistent=true

 [Install]
 WantedBy=timers.target
```

The logrotate service runs rotations as per the schedule and other parameters defined in the /etc/logrotate.conf and additional log configuration files located in the /etc/logrotate.d directory. 

### /etc/cron.daily/logrotate script
- invokes the *logrotate* command on a daily basis. 
- runs a rotation as per the schedule defined in /etc/logrotate.conf and the 
- configuration files for various services are located in /etc/logrotate.d/ The 
- configuration files may be modified to alter the schedule or include additional tasks on log files such as:
	- removing
	- compressing
	- emailing 
	
```bash
 grep -v ^$ /etc/logrotate.conf
 # see "man logrotate" for details
 # global options do not affect preceding include directives
 # rotate log files weekly
 weekly
 # keep 4 weeks worth of backlogs
 rotate 4
 # create new (empty) log files after rotating old ones
 create
 # use date as a suffix of the rotated file
 dateext
 # uncomment this if you want your log files compressed
 #compress
 # packages drop log rotation information into this directory
 include /etc/logrotate.d
 # system-specific logs may be also be configured here.
```

content:
- default log rotation frequency (weekly).
- period of time (4 weeks) to retain the rotated logs before deleting them. 

 - Each time a log file is rotated:
	 - Empty replacement file is created with the date as a suffix to its name
	- *rsyslog* service is restarted
- script presents the option of compressing the rotated files using the *gzip* utility. 
* logrotate command checks for the presence of additional log configuration files in /etc/logrotate.d/ and includes them as necessary.
* directives defined in /etc/logrotate.conf file have a global effect on all log files
* can define custom settings for a specific log file in /etc/logrotate.conf/ or create a separate file in /etc/logrotate.d/
* settings defined in user-defined files overrides the global settings.

The /etc/logrotate.d/ directory includes additional configuration files for other service logs:
```bash
 ls -l /etc/logrotate.d/
```
 

Show the file content for btmp (records of failed user login attempts) that is used to control the rotation behavior for  /var/log/btmp:
```bash
 cat /etc/logrotate.d/btmp
	``` 

- rotation is once a month. 
- replacement file created will get read/write permission bits for the owner (*root*)
- owning group will be set to *utmp*
- rsyslog service will maintain one rotated copy of the *btmp* log file.

### The Boot Log File
Logs generated during the system startup:
- Display the service startup sequence.
- Status showing whether the service was started successfully. 
- May help in any post-boot troubleshooting if required. 
- /var/log/boot.log

View /var/log/boot.log:
```
 sudo head /var/log/boot.log
 ```

output:
- OK or FAILED 
	- indicates if the service was started successfully or not.

### The System Log File
/var/log/messages
- default location for storing most system activities, as defined in the *rsyslog.conf* file
- saves log information in plain text format 
- may be viewed with any file display utility (*cat*, *more*, *pg*, *less*, *head*, or *tail*.) 
- may be observed in real time using the *tail* command with the -f switch. The *messages* file 
- captures:
	- the date and time of the activity, 
	- hostname of the system, 
	- name and PID of the service
	- short description of the event being logged.

View /var/log messages:
```bash
 tail /var/log/messages
 ```
### Logging Custom Messages

The Modules section in the *rsyslog.conf* file 
- Provides the support via the imuxsock module to record custom messages to the *messages* file using the *logger* command.  

### logger command

Add a note indicating the calling user has rebooted the system:
```
 logger -i "System rebooted by $USER"
```
 
observe the message recorded along with the timestamp, hostname, and PID:
```bash
 tail -l /var/log/messages
 ``` 

-p option
- specify a priority level either as a numerical value or in the facility.priority format. 
- default priority 
	- user.notice. 

View logger man pages:
`man logger`

### The systemd Journal

- Systemd-based logging service for the collection and storage of logging data. 
- Implemented via the *systemd-journald* daemon. 
- Gather, store, and display logging events from a variety of sources such as:
	- the kernel
	- *rsyslog* and other services
	- initial RAM disk
	- alerts generated during the early boot stage. 
journals
- stored in the binary format files
- located in  /run/log/journal/ (remember run is not a persistent directory)
- structured and indexed for faster and easier searches
- May be viewed and managed using the *journalctl* command. 
- Can enable persistent storage for the logs if desired.

- RHEL runs both *rsyslogd* and *systemd-journald* concurrently. 
- data gathered by *systemd-journald* may be forwarded to *rsyslogd* for further processing and persistent storage in text format.

/etc/systemd/journald.conf
- main config file for journald 
- contains numerous default settings that affect the overall functionality of the service. 

### Retrieving and Viewing Messages

### journalctl command 

- retrieve messages from the journal for viewing in a variety of ways using different options. 

run `journalctl` without any options to see all the messages generated since the last system reboot:
`journalctl` 

- format of the messages is similar to that of the events logged to /var/log/messages 
- Each line begins with a timestamp followed by the system hostname, process name with or without a PID, and the actual message.

Display verbose output for each entry:
```bash
 journalctl -o verbose
``` 

View all events since the last system reboot:
```bash
 journalctl -b
``` 

-0 (default, since the last system reboot), 
-1 (the previous system reboot), 
-2 (two reboots before)
1 & 2 only work if there are logs persistently stored.

View only kernel-generated alerts since the last system reboot:
```bash
 journalctl -kb0
``` 

Limit the output to view 3 entries only:
```bash
 journalctl -n3
``` 

To show all alerts generated by a particular service, such as *crond*:
```bash
 journalctl /usr/sbin/crond
 ``` 

Retrieve all messages logged for a certain process, such as the PID associated with the *chronyd* service:
```bash
 journalctl _PID=$(pgrep chronyd)
 ``` 

Reveal all messages for a particular system unit, such as sshd.service:
```bash
 journalctl _SYSTEMD_UNIT=sshd.service
``` 

View all error messages logged between a date range, such as October 10, 2019 and October 16, 2019:
```bash
 journalctl --since 2019-10-16 --until 2019-10-16 -p err
``` 

Get all warning messages that have appeared today and display them in reverse chronological order:
```bash
 journalctl --since today -p warning -r 
```
 
- Can specify the time range in hh:mm:ss format, or yesterday, today, or tomorrow as well.

follow option
```bash
 journalctl -f
```

```bash
 man journalctl
``` 

```
 man systemd-journald
``` 

### Preserving Journal Information

- enable a separate storage location for the journal to save all its messages there persistently.
- default is under /var/log/journal/ 

The *systemd-journald* service supports four options with the Storage directive to control how the logging data is handled. 

| Option     | Description                                                                                                                                                                                                                 |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| volatile   | Stores data in memory only                                                                                                                                                                                                  |
| persistent | Stores data permanently under /var/log/journal and falls back to memory-only option if this directory does not exist or has a permission or other issue. The service creates /var/log/journal in case of its non-existence. |
| auto       | Similar to "persistent" but does not create /var/log/journal if it does not exist. This is the default option.                                                                                                              |
| none       | Disables both volatile and persistent storage options. Not recommended.                                                                                                                                                     |


### Journal Data Storage Options

create the /var/log/journal/ manually and use preferred "auto" option.
- faster query responses from in-memory storage 
- access to historical log data from on-disk storage.

### Lab: Configure Persistent Storage for Journal Information

Run the necessary steps to enable and confirm persistent storage for the journals.

1. Create a subdirectory called *journal* under the /var/log/ directory and confirm:
```bash
  sudo mkdir /var/log/journal
 ``` 

2. Restart the *systemd-journald* service and confirm:
```bash
 systemctl restart systemd-journald && systemctl status systemd- journald
```

3. List the new directory and observe a subdirectory matching the machine ID of the system as defined in the /etc/machine-id file is created:
```bash
 ll /var/log/journal && cat /etc/machine-id
``` 

- This log file is rotated automatically once a month based on the settings in the *journald.conf* file. 

Check the manual pages of journal.conf
```bash
 man journald.conf
```
