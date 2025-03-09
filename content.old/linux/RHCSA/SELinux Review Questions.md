## SELinux Review Questions 

Q\. What would the command `semanage login -a -s user_u user10` do?
A\. This command provided will map Linux user user10 with SELinux user user_u.

---

Q\. Name the two commands that can be used to modify a Boolean value.
A\. The `semanage` and `setsebool` commands.

---

Q\. Which option is used with the `ps` command to view the security contexts for processes?
A\. The `-Z` option.

---

Q\. What would the command `restorecon -F /etc/sysconfig` do?
A\. The command provided will restore the default SELinux contexts on the specified directory.

---

Q\. What is the name of the default SELinux policy used in RHEL 9?
A\. The default SELinux policy used in RHEL 9 is targeted.

---

Q\. SELinux is an implementation of discretionary access control. True or False?
A\. False. SELinux is an implementation of mandatory access control.

---

Q\. Where are SELinux denial messages logged in the absence of the
auditd daemon?
A\. The SELinux denial messages are logged to the */var/log/messages* file. 

---

Q\. Name the four parts of a process context.
A\. The four parts of a process context are user, role, type/domain, and sensitivity level.

---

Q\. What would the command `sestatus -b | grep nfs_export_all_rw` do?
A\. The command provided will display the current value of the specified Boolean.

---

Q\. Name the directory that stores SELinux Boolean files.
A\. The `/sys/fs/selinux/booleans` directory.

---

Q\. What are the two commands to display and modify the SELinux mode?
A\. The `getenforce` and `setenforce` commands.

---

Q\. Name the two SELinux subjects.
A\. **User** and **process** are two SELinux subjects.

---

Q\. What one task must be done to change the SELinux mode from enforcing to disabled?
A\. The system must be rebooted.

---

Q\. Which option with the `cp` command must be specified to preserve SELinux contexts?
A\. The `--preserve=context` option.

---

Q\. What is the purpose of the command `sestatus`?
A\. The `sestatus` command displays SELinux status information.

---

Q\. What is the name and location of the SELinux configuration file?
A\. The SELinux configuration filename is config and it is located in the */etc/selinux* directory.

---

Q\. What would the command `semanage fcontext -Cl` do?
A\. The command provided will show recent changes made to the SELinux policy database.

---

Q\. Which command can be used to ensure modified contexts will survive file system relabeling?
A\. The `semanage` command.

---

Q\. With SELinux running in enforcing mode and one of the services on the system is compromised, all other services will be affected. True or False?
A\. False.

---

Q\. Name the command that starts the SELinux Troubleshooter program.
A\. The command name is `sealert`.