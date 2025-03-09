## Chapter 08 Process and task scheduling

\

### Linux Processes and Job Scheduling {.style3}

\

::: {style="border-style: double; border-color: rgb(0, 0, 0); border-top-width: 1px; width: 0.375;"}
:::

\

#### This chapter describes the following major topics:

\

Identify and display system and user executed processes

\

View process states and priorities

\

Examine and change process niceness and priority

\

Understand signals and their use in controlling processes

\

Review job scheduling

\

Control who can schedule jobs

\

Schedule and manage jobs (tasks) using at

\

Comprehend crontab and understand the syntax of crontables

\

Schedule and manage jobs using cron

\

#### RHCSA Objectives:

\

19\. Identify CPU/memory intensive processes, adjust process priority
with renice, and kill processes

20\. Adjust process scheduling

38\. Schedule tasks using at and cron

::: {style="page-break-before: always;"}
:::

\

Aprocess is any program, command, or application running on the system.
Every process has a unique numeric identifier, and it is managed by the
kernel through its entire lifespan. It may be viewed, listed, and
monitored, and can be launched at a non-default priority based on the
requirement and available computing resources. A process may also be
re-prioritized while it is running. A process is in one of several
states at any given time during its lifecycle. A process may be tied to
the terminal window where it is initiated, or it may run on the system
as a service.

\

There are plenty of signals that may be passed to a process to
accomplish various actions. These actions include hard killing a
process, soft terminating it, and forcing it to restart with the same
identifier.

\

::: {style="text-align: center;"}
![](image-BR20BNCA%201.jpg){height="100%"}
:::

Job scheduling allows a user to schedule a command for a one-time or
recurring execution in the future. A job is submitted and managed by
authorized users only. All executed jobs are logged.

::: {style="page-break-before: always;"}
:::

[]{#chapter0252.html}

### Processes and Priorities {.style3}

\

A process is a unit for provisioning system resources. It is any
program, application, or command that runs on the system. A process is
created in memory when a program, application, or command is initiated.
Processes are organized in a hierarchical fashion. Each process has a
parent process (a.k.a. a calling process) that spawns it. A single
parent process may have one or many child processes and passes many of
its attributes to them at the time of their creation. Each process is
assigned an exclusive identification number known as the Process
IDentifier (PID), which is used by the kernel to manage and control the
process through its lifecycle. When a process completes its lifespan or
is terminated, this event is reported back to its parent process, and
all the resources provisioned to it (cpu cycles, memory, etc.) are then
freed and the PID is removed from the system.

\

Plenty of processes are spawned at system boot, many of which sit in the
memory and wait for an event to trigger a request to use their services.
These background system processes are called daemons and are critical to
system operation.

::: {style="page-break-before: always;"}
:::

[]{#chapter0253.html}

### Process States {.style3}

\

A process changes its operating state multiple times during its
lifecycle. Many factors, such as load on the processor, availability of
free memory, priority of the process, and response from other
applications, affect how often a process jumps from one operating state
to another. It may be in a non-running condition for a while or waiting
for other process to feed it information so that it can continue to run.

\

There are five basic process states: running, sleeping, waiting,
stopped, and zombie. Each process is in one state at any given time. See
Figure 8-1.

\

::: {style="text-align: center;"}
![](image-5YWJN0KN%201.jpg){height="100%"}
:::

Figure 8-1 Process State Transition

\

Running: The process is being executed by the system CPU.

Sleeping: The process is waiting for input from a user or another
process.

Waiting: The process has received the input it was waiting for and is
now ready to run as soon as its turn comes.

Stopped: The process is currently halted and will not run even when its
turn comes unless a signal is sent to change its behavior. (Signals are
explained later in this chapter.)

Zombie: The process is dead. A zombie process exists in the process
table alongside other process entries, but it takes up no resources. Its
entry is retained until its parent process permits it to die. A zombie
process is also called a defunct process.

::: {style="page-break-before: always;"}
:::

[]{#chapter0254.html}

### Viewing and Monitoring Processes with ps {.style3}

\

A system may have hundreds or thousands of processes running
concurrently depending on the purpose of the system. These processes may
be viewed and monitored using various native tools such as ps (process
status) and top (table of processes). The ps command offers plentiful
switches that influence its output, whereas top is used for real-time
viewing and monitoring of processes and system resources.

\

Without any options or arguments, ps lists processes specific to the
terminal where this command is issued:

\

<div>

![](image-2T0S653J%201.jpg){height="100%"}

</div>

The above output returns the elementary information about processes in
four columns. These processes are tied to the current terminal window.
It exhibits the PID, the terminal (TTY) the process spawned in, the
cumulative time (TIME) the system CPU has given to the process, and the
name of the actual command or program (CMD) being executed.

\

Some common options that can be used with the ps command to generate
detailed reports include -e (every), -f (full-format), -F (extra
full-format), and -l (long format). A combination of -e, -F, and -l (ps
-eFl) produces a very thorough process report, however, that much detail
may not be needed in most situations. Other common options such as
\--forest and -x will report the output in tree-like hierarchy and
include the daemon processes as well. Check the manual pages of the
command for additional options and their usage.

\

Here are a few sample lines from the beginning and end of the output
when ps is executed with -e and -f flags on the system.

\

<div>

![](image-QQABWO1C%201.jpg){height="100%"}

</div>

This output is disseminated across eight columns showing details about
every process running on the system. Table 8-1 describes the content
type of each column.

\

  ----------------------------------- -----------------------------------
  Column                              Description

  UID                                 User ID or name of the process
                                      owner

  PID                                 Process ID of the process

  PPID                                Process ID of the parent process

  C                                   CPU utilization for the process

  STIME                               Process start date or time

  TTY                                 The controlling terminal the
                                      process was started on. "Console"
                                      represents the system console and
                                      "?" represents a daemon process.

  TIME                                Aggregated execution time for a
                                      process

  CMD                                 The command or program name
  ----------------------------------- -----------------------------------

::: {style="page-break-before: always;"}
:::

\

Table 8-1 ps Command Output Description

\

The ps output above pinpoints several daemon processes running in the
background. These processes are not associated with any terminal, which
is why there is a ? in the TTY column. Notice the PID and PPID numbers.
The smaller the number, the earlier it is started. The process with PID
0 is started first at system boot, followed by the process with PID 1,
and so on. Each PID has an associated PPID in column 3. The owner of
each process is exposed in the UID column along with the name of the
command or program under CMD.

\

Information for each running process is recorded and maintained in the
/proc file system, which ps and many other commands reference to acquire
desired data for viewing.

\

The ps command output may be customized to view only the desired
columns. For instance, if you want to produce an output with the command
name in column 1, PID in column 2, PPID in column 3, and owner name in
column 4, run it as follows:

\

<div>

![](image-P86YLM59%201.jpg){height="100%"}

</div>

Make sure the -o option is specified for a user-defined format and there
is no white space before or after the column names. You can add or
remove columns and switch their positions as needed.

\

Another switch to look at with the ps command is -C (command list). This
option is used to list only those processes that match the specified
command name. For example, run it to check how many sshd processes are
currently running on the system:

\

<div>

![](image-CQ26C6T5%201.jpg){height="100%"}

</div>

The output exhibits multiple background sshd processes.

::: {style="page-break-before: always;"}
:::

[]{#chapter0255.html}

### Viewing and Monitoring Processes with top {.style3}

\

The other popular tool for viewing process information is the top
command. This command displays statistics in real time and continuously,
and may be helpful in identifying possible performance issues on the
system. A sample of a running top session is shown below:

\

<div>

![](image-9U9PV626%201.jpg){height="100%"}

</div>

Press q or Ctrl+c to quit.

\

The top output may be divided into two major portions: the summary
portion and the tasks portion. The summary area spreads over the first
five lines of the output, and it shows the information as follows:

\

Line 1: Indicates the system uptime, number of users logged in, and
system load averages over the period of 1, 5, and 15 minutes. See the
description for the uptime command output in

[Chapter 02 "Initial Interaction with the System".](#chapter0049.html)

Line 2: Displays the task (or process) information, which includes the
total number of tasks running on the system and how many of them are in
running, sleeping, stopped, and zombie states.

Line 3: Shows the processor usage that includes the CPU time in
percentage spent in running user and system processes, in idling and
waiting, and so on.

Line 4: Depicts memory utilization that includes the total amount of
memory allocated to the system, and how much of it is free, in use, and
allocated for use in buffering and caching.

Line 5: Exhibits swap (virtual memory) usage that includes the total
amount of swap allocated to the system, and how much of it is free and
in use. The "avail Mem" shows an estimate of the amount of memory
available for starting new processes without using the swap.

\

The second major portion in the top command output showcases the details
for each process in 12 columns as described below:

\

Columns 1 and 2: Pinpoint the process identifier (PID) and owner (USER)

Columns 3 and 4: Display the process priority (PR) and nice value (NI)

Columns 5 and 6: Depict amounts of virtual memory (VIRT) and non-swapped
resident memory (RES) in use

Column 7: Shows the amount of shareable memory available to the process
(SHR)

Column 8: Represents the process status (S)

Columns 9 and 10: Express the CPU (%CPU) and memory (%MEM) utilization

Column 11: Exhibits the CPU time in hundredths of a second (TIME+)

Column 12: Identifies the process name (COMMAND)

\

While in top, you can press "o" to re-sequence the process list, "f" to
add or remove fields, "F" to select the field to sort on, and "h" to
obtain help. top is highly customizable. See the command's manual pages
for details.

::: {style="page-break-before: always;"}
:::

[]{#chapter0256.html}

### Listing a Specific Process {.style3}

\

Though the tools discussed so far provide a lot of information about
processes including their PIDs, Linux also offers the pidof and pgrep
commands to list only the PID of a specific process. These commands have
a few switches available to modify their behavior; however, their most
elementary use is to pass a process name as an argument to view its PID.
For instance, to list the PID of the rsyslogd daemon, use either of the
following:

\

<div>

![](image-472TCNXI%201.jpg){height="100%"}

</div>

Both commands produce an identical result if used without an option.

::: {style="page-break-before: always;"}
:::

[]{#chapter0257.html}

### Listing Processes by User and Group Ownership {.style3}

\

A process can be listed by its ownership or owning group. You can use
the ps command for this purpose. For example, to list all processes
owned by user1, specify the -U (or -u) option with the command and then
the username:

\

<div>

![](image-WLH9Y8KU%201.jpg){height="100%"}

</div>

The command lists the PID, TTY, TIME, and CMD name for all the processes
owned by user1. You can specify the -G (or -g) option instead and the
name of an owning group to print processes associated with that group
only:

\

<div>

![](image-NJVHX90K%201.jpg){height="100%"}

</div>

The above output reveals all the running processes with root as their
owning group.

::: {style="page-break-before: always;"}
:::

[]{#chapter0258.html}

### Understanding Process Niceness and Priority {.style3}

\

Linux is a multitasking operating system. It runs numerous processes on
a single processor core by giving each process a slice of time. The
process scheduler on the system performs rapid switching of processes,
giving the notion of concurrent execution of multiple processes.

\

A process is spawned at a certain priority, which is established at
initiation based on a numeric value called niceness (a.k.a. a nice
value). There are 40 niceness values, with -20 being the highest or the
most favorable to the process, and +19 being the lowest or the least
favorable to the process. Most system-started processes run at the
default niceness of 0. A higher niceness lowers the execution priority
of a process, and a lower niceness increases it. In other words, a
process running at a higher priority gets more CPU attention. A child
process inherits the niceness of its calling process in its priority
calculation. Though programs are normally run at the default niceness,
you can choose to initiate them at a different niceness to adjust their
priority based on urgency, importance, or system load. As a normal user,
you can only make your processes nicer, but the root user can raise or
lower the niceness of any process.

\

RHEL provides the nice command to launch a program at a non-default
priority and the renice command to alter the priority of a running
program.

\

The default niceness can be viewed with the nice command as follows:

\

<div>

![](image-DZFM9QSL%201.jpg){height="100%"}

</div>

The ps command with the -l option along with the -ef options can be used
to list priority (PRI, column 7) and niceness (NI, column 8) for all
processes:

\

<div>

![](image-5QPXBUA4%201.jpg){height="100%"}

</div>

The output indicates that the first two processes are running at the
default niceness of 0 and the next three at the highest niceness of -20.
These values are used by the process scheduler to adjust the execution
time of the processes on the CPU. The ps command maintains an internal
mapping between niceness levels and priorities. A niceness of 0 (NI
column) corresponds to priority 80 (PR column), and a niceness of -20
(NI column) maps to priority 60 (PR column).

\

In contrast to the niceness-priority mapping that the ps command uses,
the top command displays it differently. For a 0-80 ps mapping, the top
session will report it 0-20. Likewise, ps' (-20)-60 will be the same as
the top's (-20)-0.

::: {style="page-break-before: always;"}
:::

[]{#chapter0259.html}

### Exercise 8-1: Start Processes at Non-Default Priorities {.style3}

\

This exercise should be done on server1 as user1 with sudo where
required.

\

You will need two terminal sessions to perform this exercise. Let's call
them Terminal 1 and Terminal 2.

\

In this exercise, you will launch the top program three times and
observe how the ps command reports the priority and niceness values. You
will execute the program the first time at the default priority, the
second time at a lower priority, and the third time at a higher
priority. You will control which priority to run the program at by
specifying a niceness value with the nice command. You will verify the
new niceness and priority after each execution of the top session.

\

1\. Run the top command at the default priority/niceness in Terminal 1:

\

<div>

![](image-NX19F2O8%201.jpg){height="100%"}

</div>

2\. Check the priority and niceness for the top command in Terminal 2
using the ps command:

\

<div>

![](image-CWI4DBM5%201.jpg){height="100%"}

</div>

The command reports the values as 80 and 0, which are the defaults.

\

3\. Terminate the top session in Terminal 1 by pressing the letter q and
relaunch it at a lower priority with a nice value of +2:

\

<div>

![](image-C97PWI8J%201.jpg){height="100%"}

</div>

4\. Check the priority and niceness for the top command in Terminal 2
using the ps command:

\

<div>

![](image-6P3O8KHV%201.jpg){height="100%"}

</div>

The command reports the new values as 82 and 2.

\

5\. Terminate the top session in Terminal 1 by pressing the letter q and
relaunch it at a higher priority with a nice value of -10. Use sudo for
root privileges.

\

<div>

![](image-BA4277WB%201.jpg){height="100%"}

</div>

6\. Check the priority and niceness for the top command in Terminal 2
using the ps command:

\

<div>

![](image-J4877DNH%201.jpg){height="100%"}

</div>

As you can see, the process is running at a higher priority (70) with a
nice value of -10. Terminate the top session by pressing the letter q.

\

This concludes the exercise.

::: {style="page-break-before: always;"}
:::

[]{#chapter0260.html}

### Exercise 8-2: Alter Process Priorities {.style3}

\

This exercise should be done on server1 as user1 with sudo where
required.

\

You will need two terminal sessions to perform this exercise. Let's call
them Terminal 1 and Terminal 2.

\

In this exercise, you will launch the top program at the default
priority and alter its priority without restarting it. You will set a
new priority by specifying a niceness value with the renice command. You
will verify the new niceness and priority after each execution of the
top session.

\

1\. Run the top command at the default priority/niceness in Terminal 1:

\

<div>

![](image-KOWFPB1H%201.jpg){height="100%"}

</div>

2\. Check the priority and niceness for the top command in Terminal 2
using the ps command:

\

<div>

![](image-RPCJG0EU%201.jpg){height="100%"}

</div>

The command reports the values as 80 and 0, which are the defaults.

\

3\. While the top session is running in Terminal 1, increase its
priority by renicing it to -5 in Terminal 2. Use the command
substitution to get the PID of top. Prepend the renice command by sudo.

\

<div>

![](image-TI4GZLZS%201.jpg){height="100%"}

</div>

The output indicates the old (0) and new (-5) priorities for the
process.

\

4\. Validate the above change with ps. Focus on columns 7 and 8.

\

<div>

![](image-ZXQM8G2A%201.jpg){height="100%"}

</div>

As you can see, the process is now running at a higher priority (75)
with a nice value of -5.

\

5\. Repeat the above but set the process to run at a lower priority by
renicing it to 8:

\

<div>

![](image-2O1CQAIK%201.jpg){height="100%"}

</div>

The output indicates the old (-5) and new (8) priorities for the
process.

\

6\. Validate the above change with ps. Focus on columns 7 and 8.

\

<div>

![](image-TN53XVDV%201.jpg){height="100%"}

</div>

The process is reported to now running at a lower priority (88) with
niceness 8.

\

This concludes the exercise.

::: {style="page-break-before: always;"}
:::

[]{#chapter0261.html}

### Controlling Processes with Signals {.style3}

\

A system may have hundreds or thousands of processes running on it.
Sometimes it becomes necessary to alert a process of an event. This is
done by sending a control signal to the process. Processes may use
signals to alert each other as well. The receiving process halts its
execution as soon as it gets the signal and takes an appropriate action
as per the instructions enclosed in the signal. The instructions may
include terminating the process gracefully, killing it abruptly, or
forcing it to re-read its configuration.

\

There are plentiful signals available for use, but only a few are
common. Each signal is associated with a unique numeric identifier, a
name, and an action. A list of available signals can be viewed with the
kill command using the -l option:

\

<div>

![](image-AWPNXQBB%201.jpg){height="100%"}

</div>

The output returns 64 signals available for process-to-process and
user-to-process communication. Table 8-2 describes the control signals
that are most often used.

\

  ----------------------- ----------------------- -----------------------
  Signal Number           Signal Name             Action

  1                       SIGHUP                  Hang up signal causes a
                                                  process to disconnect
                                                  itself from a closed
                                                  terminal that it was
                                                  tied to. Also used to
                                                  instruct a running
                                                  daemon to re-read its
                                                  configuration without a
                                                  restart.

  2                       SIGINT                  The \^c (Ctrl+c) signal
                                                  issued on the
                                                  controlling terminal to
                                                  interrupt the execution
                                                  of a process.

  9                       SIGKILL                 Terminates a process
                                                  abruptly

  15                      SIGTERM                 Sends a soft
                                                  termination signal to
                                                  stop a process in an
                                                  orderly fashion. This
                                                  is the default signal
                                                  if none is specified
                                                  with the command.

  18                      SIGCONT                 Same as using the bg
                                                  command to resume

  19                      SIGSTOP                 Same as using Ctrl+z to
                                                  suspend a job

  20                      SIGTSTP                 Same as using the fg
                                                  command
  ----------------------- ----------------------- -----------------------

::: {style="page-break-before: always;"}
:::

\

Table 8-2 Control Signals

\

The commands used to pass a signal to a process are kill and pkill.
These commands are usually used to terminate a process. Ordinary users
can kill processes that they own, while the root user privilege is
needed to kill any process on the system.

\

The kill command requires one or more PIDs, and the pkill command
requires one or more process names to send a signal to. You can specify
a non-default signal name or number with either utility.

\

Let's look at a few examples to understand the usage of these tools.

\

To pass the soft termination signal to the crond daemon, use either of
the following:

\

<div>

![](image-36BT3VMP%201.jpg){height="100%"}

</div>

The pidof command in the above example is used to discover the PID of
the crond process using command substitution and it is then passed to
the kill command for termination. You may also use the pgrep command to
determine the PID of a process, as demonstrated in the next example. Use
ps -ef \| grep crond to confirm the termination.

\

Using the pkill or kill command without specifying a signal name or
number sends the default signal of 15 to the process. This signal may or
not terminate the process. Some processes ignore the soft termination
signal as they might be in a waiting state. These processes may be ended
forcefully using signal 9 in any of the following ways:

\

<div>

![](image-Z7GDROPY%201.jpg){height="100%"}

</div>

You may run the killall command to terminate all processes that match a
criterion. Here is how you can use this command to kill all crond
processes (assuming there are many of them running):

\

<div>

![](image-DBURS3XF%201.jpg){height="100%"}

</div>

There are plenty of options available with the kill, killall, pkill,
pgrep, and pidof commands. Consult respective manual pages for more
details.

::: {style="page-break-before: always;"}
:::

[]{#chapter0262.html}

### Job Scheduling {.style3}

\

Job scheduling allows a user to submit a command for execution at a
specified time in the future. The execution of the command could be one
time or periodic based on a pre-determined time schedule. A one-time
execution may be scheduled for an activity that needs to be performed at
a time of low system usage. One example of such an activity would be the
execution of a lengthy shell program. In contrast, a recurring activity
could include creating a compressed archive, trimming log files,
monitoring the system, running a custom script, or removing unwanted
files from the system.

\

Job scheduling and execution is taken care of by two service daemons:
atd and crond. While atd manages the jobs scheduled to run one time in
the future, crond is responsible for running jobs repetitively at
pre-specified times. At startup, this daemon reads the schedules in
files located in the /var/spool/cron and /etc/cron.d directories, and
loads them in the memory for on-time execution. It scans these files at
short intervals and updates the in-memory schedules to reflect any
modifications. This daemon runs a job at its scheduled time only and
does not entertain any missed jobs. In contrast, the atd daemon retries
a missed job at the same time next day. For any additions or changes,
neither daemon needs a restart.

::: {style="page-break-before: always;"}
:::

[]{#chapter0263.html}

### Controlling User Access {.style3}

\

By default, all users are allowed to schedule jobs using the at and cron
services. However, this access may be controlled and restricted to
specific users only. This can be done by listing users in the allow or
deny file located in the /etc directory for either service. These files
are named at.allow and at.deny for the at service and cron.allow and
cron.deny for the cron service.

\

The syntax for the four files is identical. You only need to list
usernames that are to be allowed or denied access to these scheduling
tools. Each file takes one username per line. The root user is always
permitted; it is affected neither by the existence or non-existence of
these files nor by the inclusion or exclusion of its entry in these
files.

\

[Table 8-3 shows various combinations and their impact on user
access.](#chapter0263.html)

\

  ----------------------- ----------------------- -----------------------
  at.allow / cron.allow   at.deny / cron.deny     Impact

  Exists, and contains    Existence does not      All users listed in
  user entries            matter                  allow files are
                                                  permitted

  Exists, but is empty    Existence does not      No users are permitted
                          matter                  

  Does not exist          Exists, and contains    All users, other than
                          user entries            those listed in deny
                                                  files, are permitted

  Does not exist          Exists, but is empty    All users are permitted

  Does not exist          Does not exist          No users are permitted
  ----------------------- ----------------------- -----------------------

::: {style="page-break-before: always;"}
:::

\

Table 8-3 User Access Restrictions to Scheduling Tools

\

By default, the deny files exist and are empty, and the allow files are
non-existent. This opens up full access to using both tools for all
users.

\

::: {style="border-style: solid; border-color: rgb(0, 0, 0); border-top-width: 1px; width: 0.0625;"}
:::

\

EXAM TIP: One username is entered per line entry in an appropriate allow
or deny file.

\

::: {style="border-style: solid; border-color: rgb(0, 0, 0); border-top-width: 1px; width: 0.0625;"}
:::

\

The following message appears if an unauthorized user attempts to
execute at:

\

<div>

![](image-VWFIN965%201.jpg){height="100%"}

</div>

And the following warning is displayed for unauthorized access attempt
to the cron service:

\

<div>

![](image-NIP1DRXO%201.jpg){height="100%"}

</div>

To generate the denial messages, you need to place entries for user1 in
the deny files.

::: {style="page-break-before: always;"}
:::

[]{#chapter0264.html}

### Scheduler Log File {.style3}

\

All activities for atd and crond services are logged to the
/var/log/cron file. Information such as the time of activity, hostname,
process name and PID, owner, and a message for each invocation is
captured. The file also keeps track of other events for the crond
service such as the service start time and any delays. A few sample
entries from the log file are shown below:

\

<div>

![](image-ME7X1OWZ%201.jpg){height="100%"}

</div>

The truncated output shows some past entries from the file. You need the
root user privilege to be able to read the file content.

::: {style="page-break-before: always;"}
:::

[]{#chapter0265.html}

### Using at {.style3}

\

The at command is used to schedule a one-time execution of a program in
the future. All submitted jobs are spooled in the /var/spool/at
directory and executed by the atd daemon at the specified time. Each
submitted job will have a file created containing the settings for
establishing the user's shell environment to ensure a successful
execution. This file also includes the name of the command or program to
be run. There is no need to restart the daemon after a job submission.

\

There are multiple ways and formats for expressing the time with at.
Some examples are:

\

  ----------------------------------- -----------------------------------
  at 1:15am                           (executes the task at the next 1:15
                                      a.m.)

  at noon                             (executes the task at 12:00 p.m.)

  at 23:45                            (executes the task at 11:45 p.m.)

  at midnight                         (executes the task at 12:00 a.m.)

  at 17:05 tomorrow                   (executes the task at 5:05 p.m. on
                                      the next day)

  at now + 5 hours                    (executes the task 5 hours from
                                      now. We can specify minutes, days,
                                      or weeks in place of hours)

  at 3:00 10/15/23                    (executes the task at 3:00 a.m. on
                                      October 15, 2023)
  ----------------------------------- -----------------------------------

::: {style="page-break-before: always;"}
:::

\

at assumes the current year and today's date if the year and date are
not mentioned.

\

You may supply a filename with the at command using the -f option. The
command will execute that file at the specified time. For instance, the
following will run /home/user1/.bash_profile file for user1 2 hours from
now:

\

<div>

![](image-EZ7PXBXJ%201.jpg){height="100%"}

</div>

The above will be executed as scheduled and will have an entry placed
for it in the log file.

::: {style="page-break-before: always;"}
:::

[]{#chapter0266.html}

### Exercise 8-3: Submit, View, List, and Erase an at Job {.style3}

\

This exercise should be done on server1 as user1.

\

In this exercise, you will submit an at job as user1 to run the date
command at 11:30 p.m. on March 31, 2023, and have the output and any
error messages generated redirected to the /tmp/date.out file. You will
list the submitted job, exhibit its contents for verification, and then
remove the job.

\

1\. Run the at command and specify the correct execution time and date
for the job. Type the entire command at the first at\> prompt and press
Enter. Press Ctrl+d at the second at\> prompt to complete the job
submission and return to the shell prompt.

\

<div>

![](image-UPDGEA0O%201.jpg){height="100%"}

</div>

The system assigned job ID 2 to it, and the output also pinpoints the
job's execution time.

\

2\. List the job file created in the /var/spool/at directory:

\

<div>

![](image-5C170TNI%201.jpg){height="100%"}

</div>

3\. List the spooled job with the at command. You may alternatively use
atq to list it.

\

<div>

![](image-LFXE8NVO%201.jpg){height="100%"}

</div>

4\. Display the content of this file with the at command and specify the
job ID:

\

<div>

![](image-NJLJSZ5H%201.jpg){height="100%"}

</div>

5\. Remove the spooled job with the at command by specifying its job ID.
You may alternatively run atrm 2 to delete it.

\

<div>

![](image-9PRXKDXI%201.jpg){height="100%"}

</div>

This should erase the job file from the /var/spool/at directory. You can
confirm the deletion by running atq or at -l.

::: {style="page-break-before: always;"}
:::

[]{#chapter0267.html}

### Using crontab {.style3}

\

Using the crontab command is the other method for scheduling tasks for
running in the future. Unlike atd, crond executes cron jobs on a regular
basis if they comply with the format defined in the /etc/crontab file.
Crontables (another name for crontab files) are located in the
/var/spool/cron directory. Each authorized user with a scheduled job has
a file matching their login name in this directory.

\

For example, the crontable for user1 would be /var/spool/cron/user1. The
other two locations where system crontables can be stored are the
/etc/crontab file and the /etc/cron.d directory; however, only the root
user is allowed to create, modify, and delete them. The crond daemon
scans entries in the files at the three locations to determine job
execution schedules. The daemon runs the commands or programs at the
specified time and adds a log entry to the /var/log/cron file for each
invocation. There is no need to restart the daemon after submitting a
new or modifying an existing cron job.

\

The crontab command is used to edit (-e), list (-l), and remove (-r)
crontables. The -u option is also available for users who wish to modify
a different user's crontable, provided they are allowed to do so and the
other user is listed in the cron.allow file. The root user can also use
the -u flag to alter other users' crontables even if the affected users
are not listed in the allow file. By default, crontab files are opened
in the vim editor when the crontab command is issued to edit them.

::: {style="page-break-before: always;"}
:::

[]{#chapter0268.html}

### Syntax of User Crontables {.style3}

\

The /etc/crontab file specifies the syntax that each user cron job must
comply with in order for crond to interpret and execute it successfully.
Based on this structure, each line in a user crontable with an entry for
a scheduled job is comprised of six fields. Fields 1 to 5 are for the
schedule, field 6 may contain the login name of the executing user, and
the rest for the command or program to be executed. See Figure 8-2 for
the syntax.

\

::: {style="text-align: center;"}
![](image-GWSG7KXY%201.jpg){height="100%"}
:::

Figure 8-2 Syntax of Crontables

\

A description of each field is provided in Table 8-4.

\

  ----------------------- ----------------------- -----------------------
  Field                   Field Content           Description

  1                       Minute of the hour      Valid values are 0 (the
                                                  exact hour) to 59. This
                                                  field can have one
                                                  specific value as in
                                                  field 1, multiple
                                                  comma-separated values
                                                  as in field 2, a range
                                                  of values as in field
                                                  3, a mix of fields 2
                                                  and 3 (1-5,6-19), or an
                                                  \* representing every
                                                  minute of the hour as
                                                  in field 5.

  2                       Hour of the day         Valid values are 0
                                                  (midnight) to 23. Same
                                                  usage applies as
                                                  described for field 1.

  3                       Day of the month        Valid values are 1 to
                                                  31. Same usage applies
                                                  as described for field
                                                  1.

  4                       Month of the year       Valid values are 1 to
                                                  12 or jan to dec. Same
                                                  usage applies as
                                                  described for field 1.

  5                       Day of the week         Valid values are 0 to 7
                                                  or sun to sat, with 0
                                                  and 7 representing
                                                  Sunday, 1 representing
                                                  Monday, and so on. Same
                                                  usage applies as
                                                  described for field 1.

  6                       Command or program to   Specifies the full path
                          execute                 name of the command or
                                                  program to be executed,
                                                  along with any options
                                                  or arguments that it
                                                  requires.
  ----------------------- ----------------------- -----------------------

::: {style="page-break-before: always;"}
:::

\

Table 8-4 Crontable Syntax Explained

\

Furthermore, step values may be used with \* and ranges in the
crontables using the forward slash character (/). Step values allow the
number of skips for a given value. For example, \*/2 in the minute field
would mean every second minute, \*/3 would mean every third minute,
0-59/4 would mean every fourth minute, and so on. Step values are also
supported in the same format in fields 2 to 5.

\

::: {style="border-style: solid; border-color: rgb(0, 0, 0); border-top-width: 1px; width: 0.0625;"}
:::

\

EXAM TIP: Make sure you understand and memorize the order of the fields
defined in crontables.

\

::: {style="border-style: solid; border-color: rgb(0, 0, 0); border-top-width: 1px; width: 0.0625;"}
:::

\

Consult the manual pages of the crontab configuration file (man 5
crontab) for more details on the syntax.

::: {style="page-break-before: always;"}
:::

[]{#chapter0269.html}

### Exercise 8-4: Add, List, and Erase a Cron Job {.style3}

\

This exercise should be done on server1 as user1 with sudo where
required.

\

For this exercise, assume that all users are currently denied access to
cron.

\

In this exercise, you will submit a cron job as user1 to echo "Hello,
this is a cron test.". You will schedule this command to execute at
every fifth minute past the hour between 10:00 a.m. and 11:00 a.m. on
the fifth and twentieth of every month. You will have the output
redirected to the /tmp/hello.out file, list the cron entry, and then
remove it.

\

1\. Edit the /etc/cron.allow file and add user1 to it:

\

<div>

![](image-XR18P20C%201.jpg){height="100%"}

</div>

2\. Open the crontable and append the following schedule to it. Save the
file when done and exit out of the editor.

\

<div>

![](image-USG2I636%201.jpg){height="100%"}

</div>

3\. Check for the presence of a new file by the name user1 under the
/var/spool/cron directory:

\

<div>

![](image-XCVAY7OR%201.jpg){height="100%"}

</div>

4\. List the contents of the crontable:

\

<div>

![](image-NCXRMCNC%201.jpg){height="100%"}

</div>

5\. Remove the crontable and confirm the deletion:

\

<div>

![](image-Y48V5AOY%201.jpg){height="100%"}

</div>

Do not run crontab -r if you do not wish to remove the crontab file.
Instead, edit the file with crontab -e and just erase the entry.

::: {style="page-break-before: always;"}
:::

[]{#chapter0270.html}

### Chapter Summary {.style3}

\

This chapter discussed two major topics: process management and job
scheduling.

\

It is vital for users, developers, and administrators alike to have a
strong grasp on running processes, resources they are consuming, process
owners, process execution priorities, etc. They should learn how to list
processes in a variety of ways. We looked at the five states a process
is in at any given time during its lifecycle. We examined the concepts
of niceness and reniceness for increasing or decreasing a process's
priority. We analyzed some of the many available signals and looked at
how they could be passed to running processes to perform an action on
them.

\

The second and the last topic talked about submitting and managing tasks
to run in the future one time or on a recurring basis. We learned about
the service daemons that handle the task execution and the control files
where we list users who can or cannot submit jobs. We looked at the log
file that stores information for all executed jobs. We reviewed the
syntax of the crontable and examined a variety of date/time formats for
use with both at and cron job submission. We performed two exercises to
gain a grasp on their usage.

::: {style="page-break-before: always;"}
:::

[]{#chapter0271.html}

### Review Questions {.style3}

\

1\. The default location to send application error messages is the
system log file. True or False?

2\. What are the five process states?

3\. Signal 9 is used for a hard termination of a process. True or False?

4\. You must restart the crond service after modifying the /etc/crontab
file. True or False?

5\. What are the background service processes normally referred to in
Linux?

6\. What is the default nice value?

7\. The parent process gets the nice value of its child process. True or
False?

8\. When would the cron daemon execute a job that is submitted as \*/10
\* 2-6 6 \* /home/user1/script1.sh?

9\. What is the other command besides ps to view running processes?

10\. Every process that runs on the system has a unique identifier
called UID. True or False?

11\. Why would you use the renice command?

12\. What are the two commands to list the PID of a specific process?

13\. By default the \*.allow files exist. True or False?

14\. Where do the scheduling daemons store log information of executed
jobs?

15\. Which user does not have to be explicitly defined in either
\*.allow or \*.deny file to run the at and cron jobs?

16\. When would the at command execute a job that is submitted as at
01:00 12/12/2020?

17\. What are the two commands that you can use to terminate a process?

18\. What is the directory location where user crontab files are stored?

19\. What would the nice command display without any options or
arguments?

20\. Which command can be used to edit crontables?

::: {style="page-break-before: always;"}
:::

[]{#chapter0272.html}

### Answers to Review Questions {.style3}

\

1\. False. The default location is the user screen where the program is
initiated.

2\. The five process states are running, sleeping, waiting, stopped, and
zombie.

3\. True.

4\. False. The crond daemon does not need a restart after a crontable is
modified.

5\. The background service processes are referred to as daemons.

6\. The default nice value is zero.

7\. False. The child process inherits its parent's niceness.

8\. The cron daemon will run the script every tenth minute past the hour
on the 2nd, 3rd, 4th, 5th, and 6th day of every sixth month.

9\. The top command.

10\. False. It is called the PID.

11\. The renice command is used to change the niceness of a running
process.

12\. The pidof and pgrep commands.

13\. False. By default, the \*.deny files exist.

14\. The scheduling daemons store log information of executed jobs in
the /var/log/cron file.

15\. The root user.

16\. The at command will run it at 1am on December 12, 2020.

17\. The kill and pkill commands.

18\. The user crontab files are stored in the /var/spool/cron directory.

19\. The nice command displays the default nice value when executed
without any options.

20\. You can use the crontab command with the -e option to edit
crontables.

::: {style="page-break-before: always;"}
:::

[]{#chapter0273.html}

### Do-It-Yourself Challenge Labs

\

The following labs are useful to strengthen most of the concepts and
topics learned in this chapter. It is expected that you perform the labs
without external help. A step-by-step guide is not supplied, as the
knowledge and skill required to implement the labs have already been
disseminated in the chapter; however, hints to the relevant major
topic(s) are included.

\

Use the lab environment built specifically for end-of-chapter labs. See
sub-section "Lab Environment for End-of-Chapter Labs" in Chapter 01
"Local Installation" for details.

::: {style="page-break-before: always;"}
:::

[]{#chapter0274.html}

### Lab 8-1: Nice and Renice a Process {.style3}

\

As user1 with sudo on server3, open two terminal sessions. Run the top
command in terminal 1. Run the pgrep or ps command in terminal 2 to
determine the PID and the nice value of top. Stop top on terminal 1 and
relaunch at a lower priority (+8). Confirm the new nice value of the
process in terminal 2. Issue the renice command in terminal 2 and
increase the priority of top to -10, and validate. (Hint: Processes and
Priorities).

::: {style="page-break-before: always;"}
:::

[]{#chapter0275.html}

### Lab 8-2: Configure a User Crontab File {.style3}

\

As user1 on server3, run the tty and date commands to determine the
terminal file (assume /dev/pts/1) and current system time. Create a cron
entry to display "Hello World" on the terminal. Schedule echo "Hello
World" \> /dev/tty/1 to run 3 minutes from the current system time. As
root, ensure user1 can schedule cron jobs. (Hint: Job Scheduling).

::: {style="page-break-before: always;"}
:::

[]{#chapter0276.html}
