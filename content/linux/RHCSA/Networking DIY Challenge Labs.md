## Networking DIY Challenge Labs


### Lab 15-1: Add New Interface and Configure Connection Profile with nmcli

- Add a third network interface to rhel9server40 in VirtualBox.
- As user1 with sudo on server40, run `ip a` and verify the addition of the new interface. 
- Use the `nmcli` command and assign IP 192.168.0.40/24 and gateway 192.168.0.1
```bash
[root@server40 ~]# nmcli c a type Ethernet ifname enp0s8 con-name enp0s8 ip4 192.168.0.40/24 gw4 192.168.0.1
```

- Deactivate and reactivate this connection manually.
```bash
[root@server40 ~]# nmcli c d enp0s8
Connection 'enp0s8' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)

[root@server40 ~]# nmcli c s
NAME    UUID                                  TYPE      DEVICE 
enp0s3  6e75a5e4-869b-3ed1-bdc4-c55d2d268285  ethernet  enp0s3 
lo      66809437-d3fa-4104-9777-7c3364b943a9  loopback  lo     
enp0s8  9a32e279-84c2-4bba-b5c5-82a04f40a7df  ethernet  --     
[root@server40 ~]# nmcli c u enp0s8
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)

[root@server40 ~]# nmcli c s
NAME    UUID                                  TYPE      DEVICE 
enp0s3  6e75a5e4-869b-3ed1-bdc4-c55d2d268285  ethernet  enp0s3 
enp0s8  9a32e279-84c2-4bba-b5c5-82a04f40a7df  ethernet  enp0s8 
lo      66809437-d3fa-4104-9777-7c3364b943a9  loopback  lo   
```

- Add entry server40 to server30's hosts table.
```bash
[root@server30 ~]# vim /etc/hosts
[root@server30 ~]# ping server40
PING server40.example.com (192.168.0.40) 56(84) bytes of data.
64 bytes from server40.example.com (192.168.0.40): icmp_seq=1 ttl=64 time=3.20 ms
64 bytes from server40.example.com (192.168.0.40): icmp_seq=2 ttl=64 time=0.628 ms
64 bytes from server40.example.com (192.168.0.40): icmp_seq=3 ttl=64 time=0.717 ms
^C
--- server40.example.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2009ms
rtt min/avg/max/mdev = 0.628/1.516/3.204/1.193 ms
```

### Lab: Add New Interface and Configure Connection Profile Manually (server30)

Add a third network interface to RHEL9server30 in VirtualBox.
 
run `ip a` and verify the addition of the new interface. 

Use the `nmcli` command and assign IP 192.168.0.30/24 and gateway 192.168.0.1
```bash
nmcli c a type Ethernet ifname enp0s8 con-name enp0s8 ip4 192.168.0.30/24 gw4 192.168.0.1
```


Deactivate and reactivate this connection manually. Add entry server30 to the hosts table of server 40 
```bash
[root@server30 system-connections]# nmcli c d enp0s8
Connection 'enp0s8' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)

[root@server30 system-connections]# nmcli c u enp0s8
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)

/etc/hosts

192.168.0.30 server30.example.com server30

```

ping tests to server30 from server 40 
```bash
[root@server40 ~]# ping server30
PING server30.example.com (192.168.0.30) 56(84) bytes of data.
64 bytes from server30.example.com (192.168.0.30): icmp_seq=1 ttl=64 time=1.59 ms
64 bytes from server30.example.com (192.168.0.30): icmp_seq=2 ttl=64 time=0.474 ms
^C
--- server30.example.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.474/1.032/1.590/0.558 ms
```


Or create the profile manually and restart network manager:
```bash
[connection]
id=enp0s8
type=ethernet
interface-name=enp0s8
uuid=92db4c65-2f13-4952-b81f-2779b1d24a49

[ethernet]

[ipv4]
method=manual
address1=10.1.13.3/24,10.1.13.1

[ipv6]
addr-gen-mode=default
method=auto

[proxy]
```