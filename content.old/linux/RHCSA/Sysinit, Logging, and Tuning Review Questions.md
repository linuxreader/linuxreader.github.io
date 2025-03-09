## Sysinit, Logging, and Tuning Review Questions

Q. The *systemd* command may be used to rebuild a new kernel. True or False?
A. False.

---

Q. Which command is used to manage system services?
A. The *systemctl* command.

---

Q. Which configuration file must be modified to ensure journal log entries are stored persistently?
A. /etc/systemd/journald.conf

---

Q. What is the PID of the *systemd* process?
A. The PID of the *systemd* process is 1.

---

Q. What is a target in *systemd*?
A. A target is a collection of units.

---

Q. You need to append a text string "Hello world" to the system log file. What would be the command to achieve this?
A. `logger -i "Hello world"`

---

Q. What is the recommended location to store custom log configuration files?
A.  /etc/rsyslog.d/ 

---

Q. What would the command `systemctl list dependencies crond` do?
A. Display all dependent units associated with the specified service.

---

Q. Name the two directory paths where *systemd* unit files are stored.
A. /etc/systemd/system/ and /usr/lib/systemd/system/

---

Q. What would you run to identify the recommended tuning profile for the system?
A. `tuned-adm recommend` 

---

Q. What would the command `systemctl get-default` do?
A. Reveal the current default boot target.

---

Q. *systemd* starts multiple services concurrently during system boot. True or False?
A. True.

---

Q. What is the name and location of the boot log file?
A. /var/log/boot.log

---

Q. Which *systemctl* subcommand is executed after a unit
configuration file has been modified to apply the changes?
A. The `daemon-reload` subcommand.

---

Q. Which other logging service complements the *rsyslog*
service?
A. The *systemd-journald* service.

---

Q. A RHEL system is booted up. You want to view all messages that were generated during the boot process. Which log file would you look at?
A. The /var/log/boot.log file.

---

Q. What would the command `systemctl restart rsyslog` do?
A. The command provided will restart the *rsyslog* service.

---

Q. What are the two common *systemd* targets production RHEL servers are typically configured to run at?
A. The two common *systemd* boot targets are multi-user and graphical.

---

Q. By default, log files are rotated automatically every week. True or False?
A. True.
