# `firewalld` Zones

## `firewalld`
- The host-based firewall solution employed in RHEL uses a kernel module called **netfilter** together with a filtering and packet classification framework called **nftables** for policing the traffic movement. 
- It also supports other advanced features such as **Network Address Translation (NAT)** and **port forwarding**. 
- This firewall solution inspects, modifies, drops, or routes incoming, outgoing, and forwarded network packets based on defined rulesets.
- Default host-based firewall management service in RHEL
- Ability to add, modify, or delete firewall rules immediately without disrupting current network connections or restarting the service process.
- Also allows to save rules persistently so that they are activated automatically at system reboots.
- Lets you perform management operations at the command line using the `firewall-cmd` command, graphically using the web console, or manually by editing rules files. 
- Stores the default rules in files located in the /usr/lib/firewalld directory, and those that contain custom rules in the /etc/firewalld directory. 
- The default rules files may be copied to the custom rules directory and modified.

### firewalld Zones 

- Easier and transparent traffic management. 
- Define policies based on the trust level of network connections and source IP addresses. 
- A network connection can be part of only one zone at a time; 
- A zone can have multiple network connections assigned to it. 
- Zone configuration may include services, ports, and protocols that may be open or closed.
- May include rules for advanced configuration items such as masquerading, port forwarding, NAT'ing, ICMP filters, and rich language. 
- Rules for each zone are defined and manipulated independent of other zones.

Match source ip to zone that matches address > match based on zone the interface is in > matches default zone
- firewalld inspects each incoming packet to determine the source IP address and applies the rules of the zone that has a match for the address. 
- In the event no zone configuration matches the address, it associates the packet with the zone that has the network connection defined, and applies the rules of that zone. 
- If neither works, firewalld associates the packet with the default zone, and enforces the rules of the default zone on the packet.

- Several predefined zone files that may be selected or customized. 
- These files include templates for traffic that must be blocked or dropped, and for traffic that is:
	- public-facing
	- internal
	- external
	- home
	- public
	- trusted
	- work-related. 
- public zone is the default zone, and it is activated by default when the firewalld service is started.

Predefined zones sorted based on the trust level from trusted to
untrusted:

**trusted**                             
  - Allow all incoming traffic

**internal**                            
  - Reject all incoming traffic except for what is allowed. Intended for use on internal networks.

**home**                                
  - Reject all incoming traffic except for what is allowed. Intended for use in homes.

**work**                                
  - Reject all incoming traffic except for what is allowed. Intended for use at workplaces.

**dmz**                                 
  - Reject all incoming traffic except for what is allowed. Intended for use in publicly accessible demilitarized zones.

**external**                           
  - Reject all incoming traffic except for what is allowed. 
  - Outgoing IPv4 traffic forwarded through this zone is masqueraded to look like it originated from the IPv4 address of an outgoing network interface. 
  - Intended for use on external networks with masquerading enabled.

**public**                              
  - Reject all incoming traffic except for what is allowed. 
  - Default zone for any newly added network interfaces. 
  - Intended for us in public places.

**block**                               
  - Reject all incoming traffic with icmp-host-prohibited message returned. 
  - Intended for use in secure places.

**drop**                                
  - Drop all incoming traffic without responding with ICMP errors. 
  - Intended for use in highly secure places.

- For all the predefined zones, outgoing traffic is allowed by default.

### Zone Configuration Files 

- firewalld stores zone rules in XML format at two locations	
	- **system-defined rules** in the */usr/lib/firewalld/zones* directory
		- can be used as templates for adding new rules, or applied instantly to any available network connection
		- automatically copied to the /etc/firewalld/zones directory if it is modified with a management tool
	- **user-defined rules** in the */etc/firewalld/zones* directory

- can copy the required zone file to the */etc/firewalld/zones* directory manually, and make the necessary changes. 
- The firewalld service reads the files saved in this location, and applies the rules defined in them.

View the system Zones:
```bash
[root@server30 ~]# ll /usr/lib/firewalld/zones
total 40
-rw-r--r--. 1 root root 312 Nov  6  2023 block.xml
-rw-r--r--. 1 root root 306 Nov  6  2023 dmz.xml
-rw-r--r--. 1 root root 304 Nov  6  2023 drop.xml
-rw-r--r--. 1 root root 317 Nov  6  2023 external.xml
-rw-r--r--. 1 root root 410 Nov  6  2023 home.xml
-rw-r--r--. 1 root root 425 Nov  6  2023 internal.xml
-rw-r--r--. 1 root root 729 Feb 21 23:44 nm-shared.xml
-rw-r--r--. 1 root root 356 Nov  6  2023 public.xml
-rw-r--r--. 1 root root 175 Nov  6  2023 trusted.xml
-rw-r--r--. 1 root root 352 Nov  6  2023 work.xml
```


View the public zone:
```bash
[root@server30 ~]# cat /usr/lib/firewalld/zones/public.xml 
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
  <forward/>
</zone>
```

- See the manual pages for firewalld.zone for details on zone configuration files.
