## The Linux Firewall Review Questions 

Q\. A firewall can be configured between two networks but not between two hosts. True or False?
A\. False. A firewall can also be configured between two host computers.

---

Q\. After changing the default firewalld zone to internal, what would you run to verify?
A\. You run `firewall-cmd --get-default-zone` for validation.

---

Q\. What is the process of data packet formation called?
A\. The process of data packet formation is called encapsulation.

---

Q\. What would the command `firewall-cmd --remove-port=5000/tcp` do?
A\. The command provided will remove the runtime firewall rule for TCP port 5000.

---

Q\. What is the primary command line tool for managing firewalld called?
A\. The primary command line tool for managing firewalld is called `firewall-cmd`.

---

Q\. What would the command `firewall-cmd --permanent --add-service=nfs --zone=external` do?
A\. The command provided will add the nfs service to external firewalld zone persistently.

---

Q\. If you have a set of firewall rules defined for a service stored under both */etc/firewalld* and */usr/lib/firewalld* directories, which of the two sets will take precedence?
A\. The ruleset located in the */etc/firewalld* directory will have precedence.

---

Q\. What is the kernel module in RHEL 9 that implements the host-level protection called?
A\. The kernel module that implements the host-level protection is called **netfilter**.

---

Q\. **firewalld** is the firewall management solution in RHEL 9. True or False?
A\. True.

---

Q\. Which directory stores the configuration file for modified firewalld zones?
A\. The modified firewalld zone files are stored under */etc/firewalld/zones* directory.

---

Q\. Name the default firewalld zone.
A\. The default firewalld zone is the public zone.

---

Q\. What is the purpose of firewalld service configuration files?
A\. firewalld service configuration files store service-specific port, protocol, and other details, which makes it easy to activate and deactivate them.
