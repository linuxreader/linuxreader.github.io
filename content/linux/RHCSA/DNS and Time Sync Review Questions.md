## DNS and Time Sync Review Questions

Q\. Maximum number of nameservers that can be defined in the resolver configuration file?
A\. Three.

---

Q\. What is the purpose of a drift file in Chrony/NTP?
A\. The purpose of a drift file is to keep track of the rate at which the system clock gains or losses time.

---

Q\. The Chrony client is preconfigured when the chrony software package is installed. You just need to start the service to synchronize the clock. True or False?
A\. True.

---

Q\. List any three DNS lookup tools.
A\. `dig`, `host`, `getent`, and `nslookup`.

---

Q\. List two utilities that you can use to change system time.
A\. The `timedatectl` and `date` utilities can be used to modify the system time.

---

Q\. What is a relative distinguished name in a DNS hierarchy?
A\. A relative distinguished name represents individual components of a distinguished name.

---

Q\. What would you add to the Chrony configuration file if you want to use the local system clock as the provider of time?
A\. You will add "server 127.127.1.0" to the Chrony configuration file and restart the service.

---

Q\. BIND is an implementation of Domain Name System. True or False?
A\. True.

---

Q\. Define time source.
A\. A time source is a reference device that provides time to other devices.

---

Q\. Name the three DNS roles that a RHEL system can play.
A\. From a DNS perspective, a RHEL machine can be a primary server, a secondary server, or a client.

---

Q\. Chrony is an implementation of the Network Time Protocol. True or False?
A\. True.

---

Q\. What is the name and location of the DNS resolver file?
A\. The DNS resolver file is called resolv.conf and it is located in the /etc directory.

---

Q\. What stratum level do two peer time sources operate on a network?
A\. Two peers on a network operate at the same stratum level.

---

Q\. What would you run to check the NTP bind status with time servers?
A\. You will run `chronyc sources` to check the binding status.

---

Q\. Define DNS name space.
A\. DNS name space is a hierarchical organization of all the domains on the Internet.

---

Q\. Name the four Chrony/NTP roles that a RHEL system can play.
A\. From a Chrony/NTP standpoint, a RHEL machine can be a primary server, a secondary server, a peer, or a client.

---

Q\. Which file defines the name resolution sources and controls the order in which they are consulted?
A\. The /etc/nsswitch.conf file.

---

Q\. What is the filename and directory location for the Chrony configuration file?
A\. The name of the Chrony configuration file is chrony.conf and it is located in the /etc directory.