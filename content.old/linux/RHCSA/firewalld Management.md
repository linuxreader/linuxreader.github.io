## firewalld Management 

- listing, querying, adding, changing, and removing zones, services, ports, IP sources, and network connections. Three methods:
	- `firewall-cmd`
	- Web interface for graphical administration. 
	- Edit Zone and service templates manually

### `firewall-cmd` Command 

- Add or remove rules from the runtime configuration, or save any modifications to service configuration for persistence. 
- Supports numerous options for the management of zones, services, ports, connections, and so on
 
### Common options

#### General                             

**\--state**                            
- Displays the running status of firewalld

**\--reload**                           
- Reloads firewall rules from zone files. All runtime changes are lost.

**\--permanent**                        
- Stores a change persistently. The change only becomes active after a service reload or restart.

#### Zones                               

**\--get-default-zone**                 
- Shows the name of the default/active zone

**\--set-default-zone**                 
- Changes the default zone for both runtime and permanent configuration

**\--get-zones**                        
- Prints a list of available zones

**--get-active-zones**                 
- Displays the active zone and the assigned interfaces

**\--list-all**                         
- Lists all settings for a zone

**\--list-all-zones**                   
-  Lists the settings for all available zones

**--zone**                             
- Specifies the name of the zone to work on. Without this option, the default zone is used.

#### Services                            

**\--get-services**                     
- Prints predefined services

**\--list-services**                    
- Lists services for a zone

**\--add-service**                      
- Adds a service to a zone

**\--remove-service**                  
- Removes a service from a zone

**\--query-service**                    
 - Queries for the presence of a service

#### Ports                               

**\--list-ports**                       
- Lists network ports

**\--add-port**                         
- Adds a port or a range of ports to a zone

**\--remove-port**                      
- Removes a port from a zone

**\--query-port**                       
- Queries for the presence of a port

#### Network Connections                 

**\--list-interfaces**                  
- Lists network connections assigned to a zone

**\--add-interface**                    
- Binds a network connection to a zone

**\--change-interface**                 
- Changes the binding of a network connection to a different zone

**\--remove-interface**                 
- Unbinds a network connection from a zone

#### IP Sources                          

**\--list-sources**                     
- Lists IP sources assigned to a zone

**\--add-source**                       
- Adds an IP source to a zone

**\--change-source**                    
- Changes an IP source

**\--remove-source**                    
- Removes an IP source from a zone

---

**\--add and \--remove options**
- \--permanent switch may be specified to ensure the rule is stored in the zone configuration file under the */etc/firewalld/zones* directory for persistence. 

### Querying the Operational Status of firewalld 

Check the running status of the firewalld service using either the `systemctl` or the `firewall-cmd` command. 

```bash
[root@server20 ~]# firewall-cmd --state
running

[root@server20 ~]# systemctl status firewalld -l --no-pager
● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; preset: enabled)
     Active: active (running) since Thu 2024-07-25 13:25:21 MST; 44min ago
       Docs: man:firewalld(1)
   Main PID: 829 (firewalld)
      Tasks: 2 (limit: 11108)
     Memory: 43.9M
        CPU: 599ms
     CGroup: /system.slice/firewalld.service
             └─829 /usr/bin/python3 -s /usr/sbin/firewalld --nofork --nopid

Jul 25 13:25:21 server20 systemd[1]: Starting firewalld - dynamic firewall daemon...
Jul 25 13:25:21 server20 systemd[1]: Started firewalld - dynamic firewall daemon.
```

### Lab: Add Services and Ports, and Manage Zones 

- Determine the current active zone. 
- Add and activate a permanent rule to allow HTTP traffic on port 80
- Add a runtime rule for traffic intended for TCP port 443 (the HTTPS service). 
- Add a permanent rule to the internal zone for TCP port range 5901 to 5910. 
- Confirm the changes and display the contents of the affected zone files. 
- Switch the default zone to the internal zone and activate it.

1\. Determine the name of the current default zone:
```bash
[root@server20 ~]# firewall-cmd --get-default-zone
public
```

2\. Add a permanent rule to allow HTTP traffic on its default port:
```bash
[root@server20 ~]# firewall-cmd --permanent --add-service http
success
```

The command made a copy of the *public.xml* file from */usr/lib/firewalld/zones* directory into the */etc/firewalld/zones* directory, and added the rule for the HTTP service.

3\. Activate the new rule:
```bash
[root@server20 zones]# firewall-cmd --reload
success
```

4\. Confirm the activation of the new rule:
```bash
[root@server20 zones]# firewall-cmd --list-services
cockpit dhcpv6-client http nfs ssh
```

5\. Display the content of the default zone file to confirm the addition of the permanent rule:
```bash
[root@server20 zones]# cat /etc/firewalld/zones/public.xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
  <service name="nfs"/>
  <service name="http"/>
  <forward/>
</zone>

```

6\. Add a runtime rule to allow traffic on TCP port 443 and verify:
```bash
[root@server20 zones]# firewall-cmd --add-port 443/tcp
success

[root@server20 zones]# firewall-cmd --list-ports
443/tcp
```

7\. Add a permanent rule to the internal zone for TCP port range 5901 to 5910:
```bash
[root@server20 zones]# firewall-cmd --add-port 5901-5910/tcp --permanent --zone internal
success
```

8\. Display the content of the internal zone file to confirm the addition of the permanent rule:
```bash
[root@server20 zones]# cat /etc/firewalld/zones/internal.xml
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Internal</short>
  <description>For use on internal networks. You mostly trust the other computers on the networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="mdns"/>
  <service name="samba-client"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
  <port port="5901-5910" protocol="tcp"/>
  <forward/>
</zone>
```

- The firewall-cmd command makes a backup of the affected zone file with a .old extension whenever an update is made to a zone.

9\. Switch the default zone to internal and confirm:
```bash
[root@server20 zones]# firewall-cmd --set-default-zone internal
success
```

```bash
[root@server20 zones]# firewall-cmd --get-default-zone
internal
```

10\. Activate the rules defined in the internal zone and list the port range added earlier:
```bash
[root@server20 zones]# firewall-cmd --list-ports
5901-5910/tcp
```

### Lab: Remove Services and Ports, and Manage Zones 

- Remove the two permanent rules that were added in the last lab. 
- Switch the public zone back as the default zone, and confirm the changes.

1\. Remove the permanent rule for HTTP from the public zone:
```bash
[root@server20 zones]# firewall-cmd --remove-service=http --zone public --permanent
success
```

- Must specify public zone as it is not the current default.

2\. Remove the permanent rule for ports 5901 to 5910 from the internal zone:
```bash
[root@server20 zones]# firewall-cmd --remove-port 5901-5910/tcp --permanent
success
```

3\. Switch the default zone to public and validate:
```bash
[root@server20 zones]# firewall-cmd --set-default-zone=public
success
[root@server20 zones]# firewall-cmd --get-default-zone 
public
```

4\. Activate the public zone rules, and list the current services:
```bash
[root@server20 zones]# firewall-cmd --reload
success
[root@server20 zones]# firewall-cmd --list-services
cockpit dhcpv6-client nfs ssh
```

### Lab: Test the Effect of Firewall Rule 

- Remove the sshd service rule from the runtime configuration on server20 
- Try to access the server from server10 using the ssh command.

1\. Remove the rule for the sshd service on server20:
```bash
[root@server20 zones]# firewall-cmd --remove-service ssh
success
```

2\. Issue the ssh command on server10 to access server20:
```bash
[root@server10 ~]# ssh 192.168.0.37
ssh: connect to host 192.168.0.37 port 22: No route to host
```

3\. Add the rule back for sshd on server20:
```bash
[root@server20 zones]# firewall-cmd --add-service ssh
success
```

4\. Issue the ssh command on server10 to access server20. Enter "yes" if
prompted and the password for user1.
```bash
[root@server10 ~]# ssh 192.168.0.37
The authenticity of host '192.168.0.37 (192.168.0.37)' can't be established.
ED25519 key fingerprint is SHA256:Z8nFu0Jj1ASZeXByiy3aAWHpUhGhUmDCr+Omu/iWTjs.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.0.37' (ED25519) to the list of known hosts.
root@192.168.0.37's password: 
Web console: https://server20:9090/ or https://192.168.0.37:9090/

Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
Last login: Thu Jul 25 13:37:47 2024 from 192.168.0.21
```
