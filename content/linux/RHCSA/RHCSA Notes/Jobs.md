## Jobs
- job
	- Process that is started in the background and controlled by the terminal where it is spawned.
	- Assigned a PID (process identifier) by the kernel and, additionally, a job ID by the shell.
	- Does not hold the terminal window where it is initiated.
	- Can run multiple job simultaneously.
	- Can be brought to foreground, returned to the background, suspended, or stopped.
- job control
	- management of multiple jobs within a shell environment

commands and control sequences for administering the jobs.
- `jobs` 
	- Shell built-in command
	- display jobs.
- `bg `
	- Shell built-in command 
	- Move a job to the background or restart a job in the background that was suspended with `Ctrl+z`.
- `fg `
	- Shell built-in command 
	- Move a job to the foreground 
- `Ctrl+z `
	- Suspends a foreground job and allows the terminal window to be used for other purposes

### jobs command
output
	plus sign (+) 
		- indicates the current background job
	minus sign (-) 
		- signifies the previous job.
	Stopped
		- currently suspended
		- can be signaled to continue their execution with bg or fg
