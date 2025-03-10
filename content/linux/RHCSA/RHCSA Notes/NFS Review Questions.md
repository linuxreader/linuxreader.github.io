## NFS Review Questions

\

Q. The name of the AutoFS service daemon is autofs. True or False?
A. False. The name of the AutoFS service daemon is automount.

---

Q\. What would the line entry /dir1 \*(rw) in the /etc/exports file mean?
A\. The line entry would export /dir1 in read/write mode to all systems.

---

Q\. What type of AutoFS map would have the /- /etc/auto.media entry in the auto.master file?
A\. A direct map.

---

Q\. AutoFS requires root privileges to automatically mount a network file system. True or False?
A\. False.

---

Q\. What is the default timeout value for a file system before AutoFS unmounts it automatically?
A\. Five minutes.

---

Q\. Name the three common types of maps that AutoFS support.
A\. The three common AutoFS maps are master, direct, and indirect.

---

Q\. Arrange the tasks in three different correct sequences to export a share using NFS: (a) update /etc/exports, (b) add service to firewall,
(c) run exportfs, (d) install nfs-utils, and (e) start nfs service.
A\. d/e/b/a/c, d/e/a/c/b, or d/e/a/b/c.

---

Q\. What would the entry \* server10:/home/& in an AutoFS indirect map imply?
A\. This indirect map entry would mount individual user home directories from server10.

---

Q\. Which command is used to export a share?
A\. The exportfs command.

---

Q\. An NFS server exports a share and an NFS client mounts it. True or False?
A\. True.

---

Q\. Which command would you use to unexport a share?
A\. The exportfs command with the -u switch.

---

Q\. What is the name of the NFS server configuration file?
A\. The /etc/nfs.conf file.

---

Q\. What is the name of the AutoFS configuration file and where is it
located?
A\. The name of the AutoFS configuration file is autofs.conf and it is
located in the /etc directory.
