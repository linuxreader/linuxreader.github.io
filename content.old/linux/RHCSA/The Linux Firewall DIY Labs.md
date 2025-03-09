# The Linux Firewall DIY Labs

### Lab: Add Service to Firewall 

- Add and activate a permanent rule for HTTPs traffic to the default zone. 
```bash
[root@server20 ~]# firewall-cmd --add-service https --permanent
success
[root@server20 ~]# firewall-cmd --reload
success

```
- Confirm the change by viewing the zone's XML file and running the `firewall-cmd` command. 
```bash
[root@server20 ~]# cat /etc/firewalld/zones/public.xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
  <service name="nfs"/>
  <service name="https"/>
  <forward/>
</zone>

[root@server20 ~]# firewall-cmd --list-services
cockpit dhcpv6-client https nfs ssh

```

### Lab: Add Port Range to Firewall 

- Add and activate a permanent rule for the UDP port range 8000 to 8005 to the trusted zone. 
```bash
[root@server20 ~]# firewall-cmd --add-port 8000-8005/udp --zone trusted --permanent
success

[root@server20 ~]# firewall-cmd --reload
success
```

- Confirm the change by viewing the zone's XML file and running the `firewall-cmd` command.
```bash
[root@server20 ~]# firewall-cmd --list-ports --zone trusted
8000-8005/udp

[root@server20 ~]# cat /etc/firewalld/zones/trusted.xml 
<?xml version="1.0" encoding="utf-8"?>
<zone target="ACCEPT">
  <short>Trusted</short>
  <description>All network connections are accepted.</description>
  <port port="8000-8005" protocol="udp"/>
  <forward/>
</zone>
```







