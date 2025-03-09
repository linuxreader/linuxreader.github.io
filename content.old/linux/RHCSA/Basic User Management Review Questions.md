## Basic User Management Review Questions

Q. What would the command `useradd -D` do?
A. This command displays the defaults settings used at the time of user creation or modification.

---

Q. What does the `lastlog` command do? 
A. The `lastlog` command provides information about recent user logins. 

---

Q. What would the command `useradd user500` do? 
A. The command provided will add user500 with all predefined default values. 

---

Q. The `id` and `groups` commands are useful for listing a user identification. True or False. 
A. False. Only the `id` command is used for this purpose.

---

Q. Name the four local user authentication files. 
A. The passwd, shadow, group, and gshadow files. 

---

Q. The passwd file contains secondary user group information. True or False? 
A. False. The passwd file contains primary user group information. 

---

Q. What is the first UID assigned to a normal user? 
A. The first UID assigned to a normal user is 1000. 

---

Q. Name the three fundamental user account classes in RHEL. 
A. The three fundamental user account categories are root, normal, and system. 

---

Q. Every user in RHEL gets a private group by default. True or False? 
A. True.

---

Q. What does the “x” in the password field in the passwd file imply? 
A. The “x” in the password field implies that the hashed password is stored in the shadow file. 

---

Q. Name the file that the `who` command consults to display logged-in users. 
A. The who command consults the /run/utmp file to list logged-in users. 

---

Q. What information does the `lastb` command provide? 
A. The lastb command reports the history of unsuccessful user login attempts. 

---

Q. The `who` command may be used to view logged out users. True or False? 
A. False. The who command shows currently logged-in users only. 

---

Q. What would the `userdel` command do if it is run with the `-r` option?
A. The `userdel` command with `-r` will delete the specified user along with their home directory.

---

Q. What is the first GID assigned to a group? 
A. The first GID assigned to a normal user is 1000. 

---

Q. What is the name of the default backup file for shadow? 
A. The name of the default backup file for shadow is shadow-. 

---

Q. Which file does the `last` command consult to display reports? 
A. The last command consults the /var/log/wtmp file to display reports.

---

Q. UID 999 is reserved for normal users. True or False? 
A. False. UID 999 is reserved for system accounts. 

---

A. Which file is assigned to users to deny them login access? 
Q. The /sbin/nologin file is assigned to users to deny them login. 

---

Q. You execute the `lastb` command as a normal user. The output reports an error. What do you need to do to run this command successfully? 
A. You need to run the `lastb` command as root or with the root privilege.




