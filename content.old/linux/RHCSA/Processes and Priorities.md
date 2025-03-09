## Processes and Priorities
Process
- a unit for provisioning system resources. 
- any program, application, or command that runs on the system.
- created in memory when a program, application, or command is initiated.
- organized in a hierarchical fashion.
- Each process has a parent process (a.k.a. a calling process) that spawns it. 
- A single parent process may have one or many child processes
	- passes many of its attributes to them at the time of their creation.
- Each process is assigned an exclusive identification number (Process IDentifier (PID))
	- is used by the kernel to manage and control the process through its lifecycle. 
- When a process completes its lifespan or is terminated, this event is reported back to its parent process, and all the resources provisioned to it (cpu cycles, memory, etc.) are then freed and the PID is removed from the system.
- background system processes are called daemons
	- which sit in the memory and wait for an event to trigger a request to use their services.
- */proc*
	- Where information for each running process is recorded and maintained.
	- Referenced by `ps` and other commands
- 
### Process States
- Five basic process states: 
	- running
		- being executed by the system CPU.
	- sleeping
		- waiting for input from a user or another process. 
	- waiting
		- has received the input it was waiting for and is now ready to run as soon as its turn comes.
	- stopped
		- currently halted and will not run even when its turn comes unless a signal is sent to change its behavior.
	- zombie
		- Dead. 
		- Exists in the process table alongside other process entries
		- takes up no resources.
		- entry is retained until its parent process permits it to die
		- also called a defunct process.
		
![](image-5YWJN0KN%201.jpg)
### `ps` command
- Lists processes specific to the terminal where this command is issued.
- Shows:
	- PID
	- terminal (TTY) the process spawned in
	- cumulative time (TIME) the system CPU has given to the process
	- name of the command or program (CMD) being executed.
	- may be customized to view only desired columns
	- can use ps to list a process by it's ownership or owning group.
- Output with -ef
	- UID
		- UID of process owner
	- PID
		- Process ID
	- PPID
		- Parent Process ID
	- C
		- CPU utilization
	- STIME
		- Start time
	- TTY
		- Controlling terminal
		- ?
			- daemon process
		- console
			- system console
	- TIME
		- Aggregated execution time
	- CMD
		- command or program name
- Flags
	- -e
		- every
	- -f 
		- full format
	- -F
		- Extra full format
	- -l
		- long format
	- -efl
		- Detailed process report  
	- --forest
		- tree like hierarchy
	- -x 
		- include daemon processes
	- -o 
		- user-defined format
		- Make sure there are no white spaces between comma separated values.
	- -C
		- command list
		- list processes that match a specific command name.
	- -U or -u
		- List user supplied as argument.
	- -G or -g
		- List processes owned by a specific group

### `top` command
- Display processes in real time
- q or ctrl+c to quit
- Hotkeys while in top
	- o
		- re-sequence the process list.
	- f
		- add or remove fields
	- F 
		- select the field to sort on
	- h 
		- help
	- 
- summary portion
	- First 5 lines
		- 1
			- system uptime, number of users logged in, and system load averages over the period of 1, 5, and 15 minutes.
		- 2
			- task (or process) information
			- total number of tasks running
			- How many of the total are running, sleeping, stopped, and zombie
		- 3
			- processor usage
			- CPU time in percentage spent in running user and system processes, in idling and waiting, and so on.
		- 4 
			- memory utilization
				- total, free, used, and allocated for buffering and caching
		- 5
			- swap useage
				- total, free, and in use
			- avail Mem
				- estimate of memory available for starting processes without using swap.
- tasks portion
	- details for each process
	- 12 columns
		- 1 and 2
			- Process identifier (PID) and owner (USER) 
		- 3 and 4
			- Process priority (PR) and nice value (NI) 
		- 5 and 6 
			- Depict amounts of virtual memory (VIRT) and non-swapped resident memory (RES) in use 
		- 7 
			- Shows the amount of shareable memory available to the process (SHR) 
		- 8 
			- Represents the process status (S)  
		- 9 and 10 
			- Express the CPU (%CPU) and memory (%MEM) utilization
		- 11 
			- Exhibits the CPU time in hundredths of a second (TIME+) 
		- 12 
			- Identifies the process name (COMMAND)
 
### Listing a Specific Process

### `pidof` and `pgrep` command
- List only the PID of a specific process
- pass a process name as an argument to view its PID
- identical if used without any options

### Listing Processes by User and Group Ownership

- can use `ps` to list a process by it's ownership or owning group.

### Process Niceness and Priority
- A process is spawned at a certain priority,
- priority is established based on the nice value.
- Higher niceness lowers execution priority of a process
- Lower niceness increase priority.
- Child process inherits nice value of it's calling process.
- Can choose a nicenes based on urgency, importance, or system load.
- Normal users can only increase niceness of their processes.
- Root can raise or lower niceness of any process.
- 40 nice values
	- -20 
		- highest and most favorable
	- +19
		- lowest and least favorable
	- 0 
		- default
- Showing nice and priority with ps
	- niceness of 0 corresponds to priority of 80
	- -20 corresponds to priority of 60
- Showing nice and priority with top.
	- niceness of 0 corresponds to priority of 20
	- -20 corresponds to priority of 0

### `nice` command
- Launch a program at a non-default priority.

### `renice` command
- Alter the priority of a running program

### Controlling Processes with Signals
- terminating the process gracefully
- killing it abruptly
- forcing it to re-read its configuration.
- Ordinary users can kill processes that they own, while the root user privilege is needed to kill any process on the system.
- Processes in a waiting state ignore the soft termination signal.

### `kill` command
- Pass a signal to a process
- Requires one or more PIDs

Flags
- -l
	- view a list of signals

Common signals
	- 1 SIGHUP (hangup)
		- causes a process to disconnect itself from a closed terminal that it was tied to
		- instruct a running daemon to re-read its configuration without a restart. 
	- 2 SIGINT 
		- ^c (Ctrl+c) signal issued on the controlling terminal to interrupt the execution of a process. 
	- 9 SIGKILL 
		- Terminates a process abruptly 
	- 15 SIGTERM (default)
		- Soft termination signal to stop a process in an orderly fashion. 
		- Default signal if none is specified with the command. 
	- 18 SIGCONT 
		- Same as using the bg command to resume 
	- 19 SIGSTOP 
		- Same as using Ctrl+z to suspend a job 
	- 20 SIGTSTP 
		- Same as using the fg command

### pkill command
- pass a signal to a process
- requires one or more process names to send a signal to.
