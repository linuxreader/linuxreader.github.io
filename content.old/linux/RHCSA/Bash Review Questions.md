## Bash Review Questions

Q. You want to change the command prompt for yourself. Where (the bash shell file) would you define it so that you get the custom command prompt whenever you log in? 
A. You can define it in the *~/.bash_profile* file. 

---

Q. What is the other name for the command line completion feature? 
A. The other name for the command line completion feature is tab completion. 

---

Q. What is a common use of the pipe symbol? 
A. A common use of the pipe character is to receive the output of one command and pass it along to the next command as an input. 

---

Q. Which directory location stores most, if not all, privileged commands? 
A. Privileged commands are stored in */usr/sbin* directory. 

---

Q. You have a file called “?” in your home directory along with other one-letter files. What would you run to delete the “?” file only? 
A. `rm \?` to remove the file. 

---

Q. What would the command `ls /cdr /usr > output` do? Assume /cdr does not exist. 
A. The command provided will redirect `ls /usr` output to the specified file and `ls /cdr` (error) to the terminal. 

---

Q. What is the name of the command that allows a user to define shortcuts to lengthy commands? 
A. The `alias` command. 

---

Q. Which file typically defines the history variables for all users? 
A. `/etc/profile` is the file where the history variables are defined for all users. 

---

Q. You have a command running that is tied to your terminal window. You want to get the command prompt back without terminating the running command. Which key combination would you press to return to the command prompt? 
A. The `Ctrl+z` key combination will suspend the running program and give you access to the command prompt. 

---

Q. Name the three quoting mechanisms? 
A. The three quoting mechanisms are backslash, single quotation mark, and double quotation mark. 

---

Q. What would the command `export VAR1=”I passed RHCSA”` do? 
A. The command provided will set an environment variable with the quoted string as its value. 

---

Q. Name the default filename and location where user command history is stored? 
A. The user command history is stored in *~/.bash_history* file. 

---

Q. What is the primary function of the shell in Linux? 
A. The shell acts as an interface between a user and the system. 

---

Q. What would the command `cd ~user20` do if executed as user10? 
A. The command provided will allow user10 to cd into user20’s home directory provided user10 has proper access permissions. 

---

Q. Which command would you use to display all matching lines from a text file? 
A. The `grep` command. 

---

Q. What would the command `ls > ls.out` do?
A. The command provided will redirect the ls command output to the specified file.

