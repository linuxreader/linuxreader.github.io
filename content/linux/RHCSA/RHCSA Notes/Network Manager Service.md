## Network Manager Service

Default service in RHEL for network:
- interface and connection configuration.
- Administration.
- Monitoring.

**NetworkManager daemon**
- Responsible for keeping interfaces and connection up and active.
- Includes:
	- `nmcli`
	- `nmtui `(text-based)
	- `nm-connection-editor` (GUI)
- Does not manage loopback interfaces.
### Interface Connection Profiles

- Configuration file on each interface that defines IP assignments and other relevant parameters for it. 
- The networking subsystem reads this file and applies the settings at the time the connection is activated. 
- Connection configuration files (or **connection profiles**) are stored in a central location under the */etc/NetworkManager/system-connections* directory. 
-  The filenames are identified by the interface connection names with *nmconnection* as the extension. 
- Some instances of connection profiles are: *enp0s3.nmconnection*, *ens160.nmconnection*, and *em1.nmconnection*.

	On server10 and server20, the device name for the first interface is enp0s3 with connection name enp0s3 and relevant connection information stored in the enp0s3.nmconnection file. 

This connection was established at the time of RHEL installation. The current content of the file from server10 are presented below:
```bash
[root@server200 system-connections]# cat /etc/NetworkManager/system-connections/enp0s3.nmconnection 
[connection]
id=enp0s3
uuid=45d6a8ea-6bd7-38e0-8219-8c7a1b90afde
type=ethernet
autoconnect-priority=-999
interface-name=enp0s3
timestamp=1710367323

[ethernet]

[ipv4]
method=auto

[ipv6]
addr-gen-mode=eui64
method=auto

[proxy]

```

- Each section defines a set of networking properties for the connection. 

Directives

**id**                                  
- Any description given to this connection. The default matches the interface name.

**uuid**                                
- The UUID associated with this connection

**type**                                
- Specifies the type of this connection

**autoconnect-priority**                
- If the connection is set to autoconnect, connections with higher priority will be preferred. A higher number means higher priority. The range is between -999 and 999 with 0 being the default.

**interface_name**                      
- Specifies the device name for the network interface

**timestamp**                           
- The time, in seconds since the Unix Epoch that the connection was last activated successfully. This field is automatically populated each time the connection is activated.

**address1/method**                     
- Specifies the static IP for the connection if the method property is set to manual. /24 represents  the subnet mask.

**addr-gen-mode/method**                
- Generates an IPv6 address based on the hardware address of the interface.

View additional directives:
```bash
man nm-settings
```

Naming rules for devices are governed by udevd service based on:
- Device location
- Topology
- setting in firmware
- virtualization layer
