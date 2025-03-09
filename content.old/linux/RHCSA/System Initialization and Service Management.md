## System Initialization and Service Management

**systemd** (system daemon)
- System initialization and service management mechanism.
- Units and targets for initialization, service administration, and state changes
- Has fast-tracked system initialization and state transitioning by introducing:
	- Parallel processing of startup scripts
	- Improved handling of service dependencies
	- On-demand activation of services
- Supports snapshotting of system states.
- Used to handle operational states of services
- Boots the system into one of several predefined targets 
- Tracks processes using control groups
- Automatically maintains mount points.
- First process with PID 1 that spawns at boot  
- Last process that terminates at shutdown.
- Spawns several processes during a service startup.
- Places the processes in a private hierarchy composed of *control groups* (or *cgroups* for short) to organize processes for the purposes of monitoring and controlling system resources such as:
	- processor
	- memory
	- network bandwidth
	- disk I/O
- Limit, isolate, and prioritize process usage of resources. 
- Resources distributed among users, databases, and applications based on need and priority

- Initiates distinct services concurrently, taking advantage of multiple CPU cores and other compute resources. 
- Creates sockets for all enabled services that support socket-based activation at the very beginning of the initialization process. 
- It passes them on to service daemon processes as they attempt to start in parallel. 
- This lets systemd handle inter-service order dependencies 
- Allows services to start without any delays.
- Systemd creates sockets first, starts daemons next, and caches any client requests to daemons that have not yet started in the socket buffer. 
- Files the pending client requests when the daemons they were awaiting come online.

Socket
- Communication method that allows a single process to talk to another process on the same or remote system.

During the operational state, systemd:
- maintains the sockets and uses them to reconnect other daemons and services that were interacting with an old instance of a daemon before that daemon was terminated or restarted.
- services that use activation based on D-Bus (*Desktop Bus*) are started when a client application attempts to communicate with them for the first time. 
- Additional methods used by systemd for activation are 
	- device-based
		- starting the service when a specific hardware type such as USB is plugged in
	- path-based
		- starting the service when a particular file or directory alters its state.

D-Bus is another 
- Allows multiple services running in parallel on a system or remote systems to talk to one another 

on-demand activation, 
- systemd defers the startup of services---Bluetooth and printing---until they are actually needed  Together, 

parallelization and on-demand activation 
- save time and compute resources. 
- contribute to expediting the boot process considerably.

benefit of parallelism witnessed at system boot is 
- the file systems are checked that may result in unnecessary delays. 
- With autofs, the file systems are temporarily mounted on their normal mount points
- as soon as the checks on the file systems are finished, systemd remounts them using their standard devices. 
- Parallelism in file system mounts does not affect the root and virtual file systems.

### Units

*Units* 
- systemd objects used for organizing boot and maintenance tasks, such as:
	- hardware initialization
	- socket creation
	- file system mounts
	- service startups. 
- Unit configuration is stored in their respective configuration files
- Config files are:
	- Auto-generated from other configurations
	- Created dynamically from the system state
	- Produced at runtime 
	- User-developed. 
- Units operational states:
	- active
	- inactive
	- in the process of being activated 
	- deactivated 
	- failed. 
	
- Units can be enabled or disabled
	- enabled unit 
		- can be started to an active state
	- disabled unit 
		- cannot be started.

Units have a name and a type, and they are 
- encoded in files with names in the form unitname.type. Some 
- examples:
	- tmp.mount
	- sshd.service
	- syslog.socket
	- umount.target. 
	
There are two types of unit configuration files: 
- System unit files
	- distributed with installed packages and located in the /usr/lib/systemd/system/
- User unit files 
	- user-defined and stored in the /etc/systemd/user/ 

View unit config file directories:
`ls -l /usr/lib/systemd/system`
`ls -l /etc/systemd/user`

`pkg-config` command:
- View systemd unit config directory information:
`pkg-config systemd --variable=systemdsystemunitdir`
`pkg-config systemd --variable=systemduserconfdir`
 

- additional system units that are created at runtime and destroyed when they are no longer needed. They are 
- located in  /run/systemd/system/
- runtime unit files take precedence over the system unit files
- user unit files take priority over the runtime files.

Unit configuration files
- direct replacement of the initialization scripts found in /etc/rc.d/init.d/ in older RHEL releases.

11 unit types

| **Unit Type** | **Description**                                                                                            |
| ------------- | ---------------------------------------------------------------------------------------------------------- |
| Automount     | automount capabilities for on-demand mounting of file systems                                              |
| Device        | Exposes kernel devices in systemd and may be used to implement device-based activation                     |
| Mount         | Controls when and how to mount or unmount file systems                                                     |
| Path          | Activates a service when monitored files or directories are accessed                                       |
| Scope         | Manages foreign processes instead of starting them                                                         |
| Service       | Starts, stops, restarts, or reloads service daemons and the processes they are made up of                  |
| Slice         | May be used to group units, which manage system processes in a tree-like structure for resource management |
| Socket        | Encapsulates local inter-process communication or network sockets for use by matching service units        |
| Swap          | Encapsulates swap partitions                                                                               |
| Target        | Defines logical grouping of units                                                                          |
| Timer         | Useful for triggering activation of other units based on timers                                            |

Unit files contain common and specific configuration elements. 
Common elements 
- fall under the [Unit] and [Install] sections
	- description, 
	- documentation location, 
	- dependency information,
	- conflict information, and 
	- other options 
- independent of the type of unit
unit-specific configuration data
- located under the unit type section: 
	- [Service] for the service unit type
	- [Socket] for the socket unit type
	- so forth

Sample unit file for sshd.service from the /usr/lib/systemd/system/:

```bash
david@fedora:~$ cat /usr/lib/systemd/system/sshd.service
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.target
Wants=sshd-keygen.target

# Migration for Fedora 38 change to remove group ownership for standard host keys
# See https://fedoraproject.org/wiki/Changes/SSHKeySignSuidBit
Wants=ssh-host-keys-migration.service

[Service]
Type=notify
EnvironmentFile=-/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```
 

- Units can have dependencies based on a sequence (ordering) or a requirement. 
	- sequence 
		- outlines one or more actions that need to be taken before or after the activation of a unit (the Before and After directives). A 
	- requirement specifies what must already be running (the Requires directive) or not running (the Conflicts directive) in order for the successful launch of a unit. 

Example:
- the *graphical.target* unit file tells us that the system must already be operating in the multi-user mode and not in rescue mode in order for it to boot successfully into the graphical mode. \\

Wants
- May be used instead of Requires in the \[Unit\] or \[Install\] section so that the unit is not forced to fail activation if a required unit fails to start. 

Run **man systemd.unit** for details on systemd unit files.

- There are also other types of dependencies
- systemd generally sets and maintains inter-service dependencies automatically
	- this can be done manually if needed.

Targets

- logical collections of units
- special systemd unit type with the .target file extension.
- share the directory locations with other unit configuration files. 
- used to execute a series of units.
	- true for booting the system to a desired operational run level with all the required services up and running.
- Some targets inherit services from other targets and add their own to them. 
- systemd includes several predefined targets 
  
| Target     | Description                                                                                                                                                                                        |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| halt       | Shuts down and halts the system                                                                                                                                                                    |
| poweroff   | Shuts down and powers off the system                                                                                                                                                               |
| shutdown   | Shuts down the system                                                                                                                                                                              |
| rescue     | Single-user target for running administrative and recovery functions. All local file systems are mounted. Some essential services are started, but networking remains disabled.                    |
| emergency  | Runs an emergency shell. The root file system is mounted in read-only mode; other file systems are not mounted. Networking and other services remain disabled.                                     |
| multi-user | Multi-user target with full network support, but without GUI                                                                                                                                       |
| graphical  | Multi-user target with full network support and GUI                                                                                                                                                |
| reboot     | Shuts down and reboots the system                                                                                                                                                                  |
| default    | A special soft link that points to the default system boot target (multi-user.target or graphical.target)                                                                                          |
| hibernate  | Puts the system into hibernation by saving the running state of the system on the hard disk and powering it off. When powered up, the system restores from its saved state rather than booting up. |

### Systemd Targets


Target unit files 
- contain all information under the \[Unit\] section
	- description
	- documentation location
	- dependency and conflict information. 

Show the graphical target file (/usr/lib/systemd/system/graphical.target):
```bash
root@localhost ~]# cat /usr/lib/systemd/system/graphical.target
[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes
``` 

Requires, Wants, Conflicts, and After suggests that the system must have already accomplished the rescue.service, rescue.target, multi-user.target, and display-manager.service levels in order to be declared running in the graphical target. 

Run `man systemd.target`for details

### systemctl Command

- performs administrative functions and supports plentiful subcommands and flags. 

| **Subcommand**              | **Description**                                                                                                   |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| daemon-reload               | Re-reads and reloads all unit configuration files and recreates the entire user dependency tree.                  |
| enable (disable)            | Activates (deactivates) a unit for autostart at system boot                                                       |
| get-default (set-default)   | Shows (sets) the default boot target                                                                              |
| get-property (set-property) | Returns (sets) the value of a property                                                                            |
| is-active                   | Checks whether a unit is running                                                                                  |
| is-enabled                  | Displays whether a unit is set to autostart at system boot                                                        |
| is-failed                   | Checks whether a unit is in the failed state                                                                      |
| isolate                     | Changes the running state of a system                                                                             |
| kill                        | Terminates all processes for a unit                                                                               |
| list-dependencies           | Lists dependency tree for a unit                                                                                  |
| list-sockets                | Lists units of type socket                                                                                        |
| list-unit-files             | Lists installed unit files                                                                                        |
| list-units                  | Lists known units. This is the default behavior when systemctl is executed without any arguments.                 |
| mask (unmask)               | Prohibits (permits) auto and manual activation of a unit to avoid potential conflict                              |
| reload                      | Forces a running unit to re-read its configuration file. This action does not change the PID of the running unit. |
| restart                     | Stops a running unit and restarts it                                                                              |
| show                        | Shows unit properties                                                                                             |
| start (stop)                | Starts (stops) a unit                                                                                             |
| status                      | Presents the unit status information                                                                              |

### Listing and Viewing Units

List all units that are currently loaded in memory along with their status and description:
`systemctl` 

Output:
UNIT column 
- shows the name of the unit and its location in the tree
LOAD column 
- reflects whether the unit configuration file was properly loaded (loaded, not found, bad setting, error, and masked)
ACTIVE column 
- returns the high-level activation state ( active, reloading, inactive, failed, activating, and deactivating)
SUB column 
- depicts the low-level unit activation state (reports unit-specific information)
DESCRIPTION column 
- illustrates the  unit's content and functionality.

- systemctl only lists active units by default

`--all`
- include the inactive units:

List all active and inactive units of type (-t) socket:
`systemctl -t socket --all`
 

List all units of type socket currently loaded in memory and the service they activate, sorted by the listening address:
`systemctl list-sockets` 

List all unit files (column 1) installed on the system and their current state (column 2): 
`systemctl list-unit-files` 

List all units that failed (--failed) to start at the last system boot:
`systemctl --failed`
 
List the hierarchy of all dependencies (required and wanted units) for the current default target:
`systemctl list-dependencies`

List the hierarchy of all dependencies (required and wanted units) for a specific unit such as *atd.service*:
`systemctl list-dependencies atd.service` 


### Managing Service Units

systemctl subcommands to manage service units, including 
- starting
- stopping
- restarting
- checking status

Check the current operational status and other details for the *atd* service:
`systemctl status atd`
 
Output:
service description 
- read from /usr/lib/systemd/system/atd.service
load status, which 
- reveals the current load status of the unit configuration file in memory. 
- Other possibilities for "Loaded" include 
	- "error" (if there was a problem loading the file)
	- \"not-found\" (if no file associated with this unit was found)
	- \"bad-setting\" (if a key setting was missing)
	- \"masked\" (if the unit configuration file is masked)
- (enable or disable) for autostart at system boot.
Active
- current activation status
- time the service was started
- Possible states:
	- **Active (running)**: The service is running with one or more processes
	- **Active (exited):** Completed a one-time configuration
	- **Active (waiting)**: Running but waiting for an event
	- **Inactive:** Not running
	- **Activating:** In the process of being activated
	- **Deactivating**: In the process of being deactivated
	- **Failed:** If the service crashed or could not be started
Also includes Main PID of the service process and more.


Disable the *atd* service from autostarting at the next system reboot:
`sudo systemctl disable atd` 

Re-enable *atd* to autostart at the next system reboot:
`systemctl enable atd` 

Check whether *atd* is set to autostart at the next system reboot:
`systemctl is-enabled atd` 

Check whether the *atd* service is running:
`systemctl is-active atd` 

Stop and restart *atd*, run either of the following:
`systemctl stop atd ; systemctl start atd`
`systemctl restart atd` 

Show the details of the *atd* service:
`systemctl show atd`
 
Prohibit *atd* from being enabled or disabled:
`systemctl mask atd` 

Try disabling or enabling *atd* and observe the effect of the previous command:
`systemctl disable atd` 

Reverse the effect of the *mask* subcommand and try disable and enable operations:
`systemctl unmask atd && systemctl disable atd && systemctl enable atd` 

### Managing Target Units

*systemctl* can also manage target units.
- view or change the default boot target
- switch from one running target into another

View what units of type (-t) target are currently loaded and active:
`systemctl -t target` 

output:
- target unit's name
- load state
- high-level and low-level activation states
- short description. 

--all option to the above
- see all loaded targets in either active or inactive state.

**Viewing and Setting Default Boot Target**

- view the current default boot target and to set it. 
	- *get-default* and *set-default* subcommands

Check the current default boot target: 

* You may have to modify the default boot target persistently for the exam.

Change the current default boot target from graphical.target to multi-user.target:
`systemctl set-default multi-user` 

- removes the existing symlink (*default.target*) pointing to the old boot target and replaces it with the new target file path.

revert the default boot target to graphical:
`systemctl set-default graphical` 

### Switching into Specific Targets

- Use  `systemctl` to transition the running system from one target state into another. 
- graphical, multi-user, reboot, shutdown---are the most common
- rescue and emergency targets are for troubleshooting and system recovery purposes, 
- poweroff and halt are similar to shutdown
- hibernate is suitable for mobile devices.

Switch into multi-user using the *isolate* subcommand:
`systemctl isolate multi-user` 

- This will stop the graphical service on the system and display the text-based console login screen. 

Type in a username such as *user1* and enter the password to log in: 

Log in and return to the graphical target:
`systemctl isolate graphical` 

Shut down the system and power it off, use the following or simply
run the *poweroff* command:
`systemctl poweroff`
`poweroff`
 
Shut down and reboot the system:
`systemctl reboot`
`reboot`

`halt`, `poweroff`, and `reboot` are symbolic links to the *systemctl* command:
```bash
[root@localhost ~]# ls -l /usr/sbin/halt /usr/sbin/poweroff /usr/sbin/reboot
lrwxrwxrwx. 1 root root 16 Aug 22  2023 /usr/sbin/halt -> ../bin/systemctl
lrwxrwxrwx. 1 root root 16 Aug 22  2023 /usr/sbin/poweroff -> ../bin/systemctl
lrwxrwxrwx. 1 root root 16 Aug 22  2023 /usr/sbin/reboot -> ../bin/systemctl
``` 

`shutdown` command options:
-H now
- Halt
-P now
- poweroff
-r now
- reboot
- broadcasts a warning message to all logged-in users
- blocks new user login attempts
- waits for the specified amount of time for users to log off
- stops the services
- shut the system down to the specified target state.
