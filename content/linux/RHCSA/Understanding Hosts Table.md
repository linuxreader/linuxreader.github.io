## Understanding Hosts Table

See [DNS and Time Synchronization](DNS%20and%20Time%20Synchronization.md)

*/etc/hosts* file
- Table used to maintain hostname to IP mapping for systems on the local network, allowing us to access a system by simply employing its hostname.

Each row in the file contains an IP address in column 1 followed by the official (or canonical) hostname in column 2, and one or more optional aliases thereafter. 

EXAM TIP: In the presence of an active DNS with all hostnames resolvable, there is no need to worry about updating the hosts file.

As expressed above, the use of the hosts file is common on small networks, and it should be updated on each individual system to reflect any changes for best inter-system connectivity experience.

### Lab: Update Hosts Table and Test Connectivity.

1. Add both server10 and server20's interfaces to both server's */etc/host* files:
```bash
192.168.0.110  server10.example.com  server10 <-- This is an alias
192.168.0.120  server20.example.com  server20   
172.10.10.110  server10s8.example.com   server10s8
172.10.10.120  server20s8.example.com   server20s8
```

2. Send 2 packets from server10 to server20's IP address:
```
ping -c2 192.168.0.120
```

3. Send 2 pings from server10 to server20's hostname:
```
ping -c2 server20
```
