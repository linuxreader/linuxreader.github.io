## Networking Review Questions


Q. Which service is responsible for maintaining consistent device naming?
A. The udevd service handles consistent naming of network devices.

---

Q. List three key differences between TCP and UDP protocols.
A. TCP is connection-oriented, reliable, and point-to-point; UDP is connectionless, unreliable, and multi-point.

---

Q. What is the significance of the **NAME** and **DEVICE** directives in a connection profile?
A. The **NAME** directive sets the name for the network connection and the **DEVICE** directive defines the network device the connection is associated
with.

---

Q. Which class of IP addresses has the least number of node addresses?
A. The C class supports the least number of node addresses.

---

Q. Which command can you use to display the hardware address of a network device?
A. The `ip` command.

---

Q. Define protocol.
A. A set of rules that govern the exchange of information between two network entities.

---

Q. Which directory stores the network connection profiles?
A. */etc/NetworkManager/system-connections/*

---

Q. True or False. A network device is a physical or virtual network port and a network connection is a configuration file attached to it.
A. True.

---

Q. IPv4 is a 32-bit software address. How many bits does an IPv6 address have?
A. 128.

---

Q. Which file defines the port and protocol mapping?
A. The */etc/protocols* file.

---

Q. What would the command `hostnamectl set-hostname host20` do?
A. The command provided will update the /etc/hostname file with the specified hostname and restart the systemd-hostnamed daemon for the
change to take effect.

---

Q. Name the file that stores the hostname of the system.
A. The /etc/hostname file.

---

Q. What would the command `nmcli cs` do?
A. The command provided will display the status information for all network connections.

---

Q. The /etc/hosts file maintains hostname to hardware address mappings. True or False?
A. False. This file maintains hostname to IP address mapping.

---

Q. Which file contains service, port, and protocol mappings?
A. The /etc/services file.

---

Q. What would the `ip addr` command produce?
A. This command provided will display information about network connections including IP assignments and hardware address.

---

Q. Which file would you consult to identify the port number and protocol associated with a network service?
A. The /etc/services file.

---

Q. Name four commands that can be used to display the hostname.
A. The `hostname`, `uname`, `hostnamectl`, and `nmcli` commands can be used to view the system hostname.

---

Q. List any two benefits of subnetting.
A. Better manageability and less traffic.
