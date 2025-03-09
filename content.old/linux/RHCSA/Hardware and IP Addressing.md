# Hardware and IP Addressing
### Ethernet Address

- 48-bit address that is used to identify the correct destination node for data packets transmitted from the source node. 
- The data packets include hardware addresses for the source and the destination node. 
- Also referred to as the hardware, physical, link layer, or MAC address.

List all network interfaces with their ethernet addresses:
```bash
ip addr | grep ether
```


### Subnetting
- Network address space is divided into several smaller and more manageable logical subnetworks (subnets). 
- Benefits:
	- Reduced network traffic
	- Improved network performance
	- de-centralized and easier administration
	- uses the node bits only
- Results in the reduction of usable addresses.
- All nodes in a given subnet have the same subnet mask.
- Each subnet acts as an isolated network and requires a router to talk to other subnets.
- The first and the last IP address in a subnet are reserved. The first address points to the subnet itself, and the last address is the broadcast address. 
### IPv4
View current ipv4 address:
```bash
ip addr
```

### Classful Network Addressing

See [Classful ipv4](Classful%20ipv4.md)

### IPv6 Address

See [ipv6](ipv6.md)

The `ip addr` command also shows IPv6 addresses for the interfaces:
```bash
[root@server200 ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b9:4e:ef brd ff:ff:ff:ff:ff:ff
    inet 172.16.1.155/20 brd 172.16.15.255 scope global dynamic noprefixroute enp0s3
       valid_lft 79061sec preferred_lft 79061sec
    inet6 fe80::a00:27ff:feb9:4eef/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

Tools:
- `ping6`
- `traceroute6`
- `tracepath6`
