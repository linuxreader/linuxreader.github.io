## Advanced User Management Review Questions 

Q. Which command can be used to display the effective (current) username?
A. `whoami `

---

Q. What is the recommended location to store custom sudo rules? 
A. The custom sudo rules should be stored in files under the /etc/sudoers.d directory.

---

Q. What would the command `passwd -l user10` do? 
A. This command will lock user10.

---
 
Q. The `chown` command may be used to modify both ownership and group membership on a file. True or False? 
A. True.

---
 
Q. What would the command `passwd -n 7 -x 15 -w 3` user5 do? 
A. The command will set mindays to 7, maxdays to 15, and warndays to 3 for user5.

---
 
Q. Write the two command names for managing all of the password aging attributes. 
A. The `passwd` and `chage` commands.

---

Q. Which file has the default password aging settings defined? 
A. The /etc/login.defs file has the defaults for password aging defined.

---
 
Q. What would the command `chage -E 2020-10-22` user10 do? 
A. The command provided will set October 22, 2020 as the expiry date for user10. 

---

Q. What would the command `chage -l user5` do?
A. The command provided will display password aging attributes for user5. 

---

Q. Which two commands can be used to lock and unlock a user account?
A. `usermod` and `passwd`

---

Q. When using `sudo`, log files record activities under the root user account. True or False? 
A. False. The activities are registered under the username who invokes the `sudo` command. 

---

Q. What is the difference between running the `su` command with and without the hyphen sign? 
A. With the dash sign the su command processes the specified user’s startup files, and it won’t without this sign. 

---

Q. What is the significance of the `-o` option with the `groupadd` and `groupmod` commands? 
A. The `-o` option lets the commands share a GID between two or more groups. 

---

Q. What would the entry user10 ALL=(ALL) NOPASSWD:ALL in the sudoers file imply? 
A. It will give user10 password-less access to all privileged commands. 

---

Q. The `chgrp` command may be used to modify both ownership and group membership on a file. True or False? 
A. False. 

---
 
Q. What would the command `chage -d 0 user60` do?
A. The command provided will force user60 to change their password at next login.


