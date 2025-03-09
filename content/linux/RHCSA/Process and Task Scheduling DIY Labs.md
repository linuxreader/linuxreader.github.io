### Lab 8-1: Nice and Renice a Process {.style3}

\

As user1 with sudo on server3, open two terminal sessions. Run the top
command in terminal 1. Run the pgrep or ps command in terminal 2 to
determine the PID and the nice value of top. Stop top on terminal 1 and
relaunch at a lower priority (+8). Confirm the new nice value of the
process in terminal 2. Issue the renice command in terminal 2 and
increase the priority of top to -10, and validate. (Hint: Processes and
Priorities).

### Lab 8-2: Configure a User Crontab File {.style3}

\

As user1 on server3, run the tty and date commands to determine the
terminal file (assume /dev/pts/1) and current system time. Create a cron
entry to display "Hello World" on the terminal. Schedule echo "Hello
World" \> /dev/tty/1 to run 3 minutes from the current system time. As
root, ensure user1 can schedule cron jobs. (Hint: Job Scheduling).