## Process and Task Scheduling Labs

### Lab: ps 
1. ps
```
ps
```

2. Check manual pages:
```
man ps
```

3. Run with "every" and "full format" flags:
```
ps -ef
```

4. Produce an output with the command name in column 1, PID in column 2, PPID in column 3, and owner name in column 4, run it as follows:
```
ps -o comm,pid,ppid,user
```

5. Check how many sshd processes are currently running on the system:
```
ps -C sshd
```

### Lab: top
1. top
```
top
```

2. View manual page:
```
man top
```

### Lab: List a specific process
1. list the PID of the rsyslogd daemon
```
pidof rsyslogd
or
pgrep rsyslogd
```

### Lab: Listing Processes by User and Group Ownership
1. List processes owned by user1:
```
ps -U user1
```

2. List processes owned by group root:
```
ps -G root
```

### Lab: nice
1. View the default nice value:
```
nice
```

2. List priority and niceness for all processes:
```
ps -efl
```

### Lab: Start Processes at Non-Default Priorities (2 terminals)
1. Run the top command at the default priority/niceness in Terminal 1:
```
top
```

2. Check the priority and niceness for the top command in Terminal 2 using the ps command:
```
ps -efl | grep top
```

3. Terminate the top session in Terminal 1 by pressing the letter q and relaunch it at a lower priority with a nice value of +2:
```
nice -n 2 top
```

4. \Check the priority and niceness for the top command in Terminal 2 using the ps command:
```
ps -efl | grep top
```

5. Terminate the top session in Terminal 1 by pressing the letter q and relaunch it at a higher priority with a nice value of -10. Use sudo for root privileges.
```
sudo nice -n -10 top
```

6. Check the priority and niceness for the top command in Terminal 2 using the ps command:
```
ps -efl | grep top
```

7.  Terminate the top session by pressing the letter q.

### Lab: Alter Process Priorities (2 terminals)
1. Run the top command at the default priority/niceness in Terminal 1: 
```
top
```

2. Check the priority and niceness for the top command in Terminal 2 using the ps command:
```
ps -efl | grep top
```

3. While the top session is running in Terminal 1, increase its priority by renicing it to -5. Use the command substitution to get the PID of top. Prepend the renice command by sudo. The output indicates the old (0) and new (-5) priorities for the process. 
```
sudo renice -n -5 $(pidof top)
```

4. Validate the above change with ps. Focus on columns 7 and 8.
```
ps -efl | grep top
```

5. Repeat the above but set the process to run at a lower priority by renicing it to 8: The output indicates the old (-5) and new (8) priorities for the process. 
```
sudo renice -n 8 $(pidof top)
```

6. Validate the above change with ps. Focus on columns 7 and 8.
```
ps -efl | grep top
```

### Lab: Controlling Processes with Signals
1. Pass the soft termination signal to the crond daemon, use either of the following:
```
sudo pkill crond
# or
sudo kill $(pidof crond)
```

2.  Confirm:
```
ps -ef | grep crond
```

3. Forcefully kill crond:
```
sudo pkill -9 crond
# or
sudo pkill -s SIGKILL crond
# or
sudo kill -9 $(pgrep crond)
```

4. Kill all crond processes:
```
sudo killall crond
```

5. View manual pages:
```
man kill
man pkill
man killall
```

### Lab: cron and atd
1. View log files for cron and atd
```
sudo cat /var/log/cron
```


### Lab: at and crond
1. run /home/user1/.bash_profile file for user1 2 hours from now:
```
at -f ~/.bash_profile now + 2 hours
```

2. Consult crontab manual pages:
```
man crontab
```

### Lab: Submit, View, List, and Erase an at Job
1.Run the at command and specify the correct execution time and date for the job. Type the entire command at the first at> prompt and press Enter. Press Ctrl+d at the second at> prompt to complete the job submission and return to the shell prompt. 
```
at 1:30pm 3/31/20
date &> /tmp/date.out
```

The system assigned job ID 5 to it, and the output also pinpoints the job’s execution time. 

2.List the job file created in the /var/spool/at directory: 
```
sudo ls -l /var/spool/at/
```

3.List the spooled job with the at command. You may alternatively use atq to list it. 
```
at -l
# or
atq
```

4.Display the contents of this file with the at command and specify the job ID: 
```
at -c 5
```

5.Remove the spooled job with the at command by specifying its job ID. You may alternatively run atrm 5 to delete it. 
```
at -d 5
```

This should erase the job file from the /var/spool/at directory. You can 

6. confirm the deletion by running atq or at -l.
```
atq
```

### Lab: Add, List, and Erase a Cron Job
assume that all users are currently denied access to cron

1. Edit the /etc/cron.allow file and add user1 to it: 
```
sudo vim /etc/cron.allow
user1
```

2. Switch to user1 Open the crontable and append the following schedule to it. Save the file when done and exit out of the editor. 
```
crontab -e
*/5 10-11 5,20 * * echo "Hello, this is a cron test." > /tmp/hello.out
```

3. Check for the presence of a new file by the name user1 under the /var/spool/cron directory:
```
sudo ls -l /var/spool/cron
```

4. List the contents of the crontable: 
```
crontab -l
```

5. Remove the crontable and confirm the deletion: 
```
crontab -r
crontab -l
```



### Lab: Anacron
1. View the default content of /etc/anacrontab without commented or empty lines:
```
cat /etc/anacrontab | grep -ve ^# -ve ^$
```

2. View anacron man pages:
```
man anacron
```

### Lab 8-1: Nice and Renice a Process 
1. As user1 with sudo on server1, open two terminal sessions. Run the top command in terminal 1. Run the pgrep or ps command in terminal 2 to determine the PID and the nice value of top. 
```
ps -efl | grep top
```

2. Stop top on terminal 1 and relaunch at a lower priority (+8). 
```
nice -n 8 top
```

3. Confirm the new nice value of the process in terminal 2. 
```
ps -efl | grep top
```

4. Issue the renice command in terminal 2 and increase the priority of top to -10:
```
renice -n -10 $(pidof top)
```

5. Confirm: 
```
ps -efl | grep top
```

### Lab 8-2: Configure a User Crontab File 
As user1 on server1, run the tty and date commands to determine the terminal file (assume /dev/pts/1) and current system time. 
```
tty
date
```

Create a cron entry to display “Hello World” on the terminal. Schedule echo “Hello World” > /dev/tty/1 to run 3 minutes from the current system time. 
```
crontab -e
*/3 * * * * echo "Hello World" > /dev/pts/2
```

As root, ensure user1 can schedule cron jobs. 
```
sudo vim /etc/cron.allow
user1
```
