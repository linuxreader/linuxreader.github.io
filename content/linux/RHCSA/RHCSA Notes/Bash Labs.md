## Bash Labs

### Lab: View environment variables
```
env
printenv
```



### Lab: Viewing Environment variable values
1. View the value for the PATH variable
```
echo $PATH
```

2. View the value for the HOME variable
```
echo $HOME
```

3. View the value for the SHELL variable
```
echo $SHELL
```

4. View the value for the TERM variable
```
echo $TERM
```

5. View the value for the PPID variable
```
echo $PPID
```

6. View the value for the PS1 variable
```
echo $PS1
```

7. View the value for the USER variable
```
echo $USER
```

### Lab: Setting and Unsetting Variables
1. Define a local variable called VR1:
```
VR1=RHEL9
```

2. View the value of VR1:
```
echo $VR1
```

3. Type bash at the command prompt to enter a sub-shell and then run echo $VR1 to check whether the variable is visible in the sub-shell.
```
echo $VR1
```

4. Exit out of the subshell:
```
exit
```

5. Turn VR1 into an environment variable:
```
export VR1
```

6. Type bash at the command prompt to enter a sub-shell and then run echo $VR1 to check whether the variable is visible in the sub-shell.
```
echo $VR1
```

7. Undefine this variable and erase it from the shell environment:
```
unset VR1
```

8. Define a local variable that contains a value with one or more white spaces:
```
VR2="I love RHEL 9"
```

9. Define and make the variable an environment variable at the same time:
```
export VR3="I love RHEL 9"
```

10. View local and environment variables:
```
set
```



### Lab: Modify Primary Command Prompt
1. Change the value of the variable PS1 to reflect the desired information:
```
export PS1="< $LOGNAME on $HOSTNAME in \$PWD > " 
```

2. Edit the .bash_profile file for user1 and define the value exactly as it was run in Step 1.
```
vim .bash_profile
```

3. Test by logging off as user1 and logging back in. The new command prompt will be displayed.

### Lab: Redirecting Standard Input
1. Have the cat command read the */etc/redhat-release* file and display its content on the standard output (terminal screen):
```
cat < /etc/redhat-release
```

### Lab: Redirecting Standard Output
1. Direct the ls command output to a file called ls.out:
```
ls > ls.out
```

2. Do the same thing but using file descriptors:
```
ls 1> ls.out
```

3. Activate the noclobber feature then try the redirect feature again:
```
set -o noclobber
ls > ls.out
```

4. Deactivate noclobber
```bash
set +o noclobber
```

5. Direct the ls command to append the output to the ls.out file instead of overwriting it:
```bash
ls >> ls.out
or
ls 1>> ls.out
```


### Lab: Redirecting Standard Error
1. Direct the find command issued as a normal user to search for all occurrences of files by the name core in the entire directory tree and sends any error messages produced to /dev/null
```bash
find / -name core -print 2> /dev/null
```

2. Redirect both standard output and error:
```bash
ls /usr /cdr &> outerr.out
or
ls /usr /cdr 1> outerr.out 2>&1
# means to redirect file descriptor 1 to file outerr.out as well as to file descriptor 2.
```

3. Same as above but append to file:
```bash
ls /usr/cdr &>> outerr.out
```

### Lab: Show History Variables
1. View HISTFILE Variable:
```bash
echo $HISTFILE
```

2. View HISTSIZE variable:
```bash
echo $HISTSIZE
```

3. View HISTFILESIZE variable:
```bash
echo $HISTFILESIZE
```

### Lab: History command
1. Rune history without any options:
```bash
history
```

2. Display 10 entries:
```bash
history 10
```

3. Run the 15th command in history:
```bash
!15
```

4. re-execute the most recent occurrence of a command that started with a particular letter or series of letters (ch for example):
```bash
!ch
```

5. Issue the most recent command that contained “grep”:
```bash
!?grep?
```

6. Remove entry 24 from history:
```bash
history -d 24
```

7. Repeat the last command executed:
```bash
!!
```

### Lab: Tilde expansion
1. Display user's home directory:
```bash
echo ~
```

2. Display the current directory:
```bash
echo ~+
```


3. Display the previous working directory:
```bash
echo ~-
```

4. Display user1's home directory:
```bash
echo ~user1
```

5. cd into the home directory of user1 and confirm:
```bash
cd ~user1
pwd
```

6. cd into a subdirectory:
```bash
cd ~/Documents/
```

7. View directory information of the root user's desktop:
```bash
ls -ld ~root/Desktop
```

### Lab: Command Aliasing
1. shows all aliases that are currently set for user1:
```bash
su - user1
alias
```

2. Run alias as root:
```bash
alias
```

3. Create an alias “search” to abbreviate the find command with several switches and arguments. Enclose the entire command within single quotation marks (‘’) to ensure white spaces are taken care of. Do not leave any spaces before and after the equal sign (=).
```bash
alias search='find / -name core -exec ls -l {} \;'
```

4. Search with the new alias:
```bash
search
```

5. Create and alias by the same name as rm command but adding the -i flag:
```bash
alias rm='rm -i'
```

6. Run rm without using the alias:
```bash
rm file1
```

7. Remove the two aliases we just created:
```bash
unalias search rm
```

### Lab: Wildcards and Metacharacters
1. List all files in the */etc* directory that begin with letters “ma” and followed by any characters:
```bash
ls /etc/ma*
```

2. List all hidden files and directories in /home/user1:
```bash
ls -d .*
```

3. List all files in the */var/log* directory that end in “.log”:
```bash
ls /var/log/*.log
```

4. List all files and directories under */var/log* with exactly four characters in their names:
```bash
ls -d /var/log/????
```

5. Include all files and directories that begin with either of the two characters and followed by any number of characters.
```bash
ls /usr/bin/[yw]*
```

6. Match all directory names that begin with any letter between “m” and “o” in the */etc/systemd/system* directory:
```bash
ls -d /etc/systemd/system/[m-o]*
```

6. Inverse results of the previous:
```bash
ls -d /etc/systemd/system/[!m-o]*
```

### Lab: Piping
1. Pipe the output to the less command in order to view the directory listing one screenful at a time:
```bash
ls -l /etc | less
```

2. Run the last command and pipe the output to nl to number each line:
```bash
last | nl
```

3. Send the output of ls to grep for the lines that do not contain the pattern “root”.  Piped again for a case-insensitive selection of all lines that exclude the pattern “dec”. Number the output, and show the last four lines on the display:
```bash
ls -l /proc | grep -v root | grep -iv dec | nl | tail -4
```

### Lab: Quoting Mechanisms
1. Remove a file called * without deleting everything in the directory:
```bash
rm \*
```

2. Display $LOGNAME without expanding the LOGNAME variable:
```bash
echo '$LOGNAME'
```

3. Run the following with double quotes and without:
```bash
echo "$SHELL"
echo "\$PWD"
echo "'\'"
```

### Lab: grep and regex
1. Search for the pattern “operator” in the */etc/passwd* file:
```bash
grep operator /etc/passwd
```

2. Search for the space-separated pattern “aliases and functions” in the *$HOME/.bashrc* file:
```bash
grep 'aliases and functions' .bashrc
```

3. Search for the pattern “nologin” in the passwd file and exclude (-v) the lines in the output that contain this pattern. Add the -n switch to show the line numbers associated with the matched lines.
```bash
grep -nv nologin /etc/passwd
```

4. Find any duplicate entries for the root user in the passwd file. Prepend the caret sign (^) to the pattern “root” to mark the beginning of a line.
```bash
grep ^root /etc/passwd
```

5. Identify all users in the passwd file with bash as their primary shell.
```bash
grep bash$ /etc/passwd
```

6. Show the entire login.defs file but exclude all the empty lines:
```bash
grep -v ^$ /etc/login.defs
```

7. Perform a case-insensitive search (-i) for all the lines in the /etc/bashrc file that match the pattern “path.”
```bash
grep -i path /etc/bashrc
```

8. Print all the lines from the */etc/lvm/lvm.conf* file that contain an exact match for a word (-w) “acce” followed by exactly two characters:
```bash
grep -w acce.. /etc/lvm/lvm.conf
```

9. Print all the lines from the `ls` command output that include either (-E) the pattern “cron” or “ly”.
```bash
ls -l /etc | grep -E 'cron|ly'
```

10. Show all the lines from the */etc/ssh/sshd_config* file but exclude (-v) the empty lines and commented lines. Use the -e flag multiple times instead of | for either or.
```bash
sudo grep -ve ^$ -ve ^# /etc/ssh/sshd_config
```

11. Learn more about regex:
```bash
man 7 regex
```

12. Consult the grep man pages:
```bash
man grep
```

### Lab: Managing jobs
1. Issue the jobs command with the -l switch to view all the jobs running in the background:
```bash
jobs -l
```

2. bring job ID 1 to the foreground and start running it:
```
fg %1
```

3. Suspend job 1 with ctrl+z and then let it run in the background:
```
bg %1
```

4. Terminate job ID 1, supply its PID (31726) to the kill command:
```
kill 31726
```



### Lab: Shell Startup Files
1. View the first 10 lines of /etc/bashrc:
```bash
head /etc/bashrc
```

2. View the first 10 lines of /etc/profile:
```bash
head /etc/profile
```

3. View the directory /etc/profile.d
```bash
ls -l /etc/profile.d/
```

4. View .bashrc
```bash
cat ~/.bashrc
```

5. View .bash_profile
```bash
cat ~/.bash_profile
```

### Lab:  Customize the Command Prompt (user1)
1. Permanently customize the primary shell prompt to display “<user1@server1 in /etc >:” when this user switches into the /etc directory. The prompt should always reflect the current directory directory path. 
```bash
vim ~/.bash_profile
export PS1='$USERNAME $PWD'
```

### Lab 7-2: Redirect the Standard Input, Output, and Error (user1)
1. Run the `ls` command on */etc*, */dvd*, and */var*. Have the output printed on the screen and the errors forwarded to file */tmp/ioerror*. 
```bash
ls /etc /dvd /var 2> /tmp/ioerror
```

2. Check the file after the command execution and analyze the results:
```bash
cat /tmp/ioerror
```
