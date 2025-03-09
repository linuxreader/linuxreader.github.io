## firewalld Services 

- For easier activation and deactivation of specific rules.
- preconfigured firewall rules delineated for various services and stored in different files. \
- The rules consist of necessary settings, such as the port number, protocol, and possibly helper modules, to support the loading of the service. 
- Can be added to a zone. 
- By default, firewalld blocks all traffic unless a service or port is explicitly opened.

### Service Configuration Files 

- firewalld stores service rules in XML format at two locations: 
- **system-defined** rules in the */usr/lib/firewalld/services* directory
	- Can be used as templates for adding new service rules, or activated instantly. A system service configuration
file is automatically copied to the /etc/firewalld/services directory if
it is modified with a management tool.
- **user-defined** rules in the */etc/firewalld/services* directory. 
	-  You can copy the required service file to the /etc/firewalld/services directory manually, and make the necessary changes. 
- Service reads the files saved in this location, and applies the rules defined in them. 
 
A listing of the system service files is presented below:
```bash
root@server30 ~]# ll /usr/lib/firewalld/services
total 884
-rw-r--r--. 1 root root 352 Nov  6  2023 afp.xml
-rw-r--r--. 1 root root 399 Nov  6  2023 amanda-client.xml
-rw-r--r--. 1 root root 427 Nov  6  2023 amanda-k5-client.xml
-rw-r--r--. 1 root root 283 Nov  6  2023 amqps.xml
-rw-r--r--. 1 root root 273 Nov  6  2023 amqp.xml
-rw-r--r--. 1 root root 285 Nov  6  2023 apcupsd.xml
-rw-r--r--. 1 root root 301 Nov  6  2023 audit.xml
-rw-r--r--. 1 root root 436 Nov  6  2023 ausweisapp2.xml
-rw-r--r--. 1 root root 320 Nov  6  2023 bacula-client.xml
-rw-r--r--. 1 root root 346 Nov  6  2023 bacula.xml
-rw-r--r--. 1 root root 390 Nov  6  2023 bareos-director.xml
...
...
```

Shows the content of the ssh service file:
```bash
[root@server30 ~]# cat /usr/lib/firewalld/services/ssh.xml
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>SSH</short>
  <description>Secure Shell (SSH) is a protocol for logging into and executing commands on remote machines. It provides secure encrypted communications. If you plan on accessing your machine remotely via SSH over a firewalled interface, enable this option. You need the openssh-server package installed for this option to be useful.</description>
  <port protocol="tcp" port="22"/>
</service>
```

- Has a name and description
- Defines the port and protocol for the service. 
- See the manual pages for firewalld.service for details on service configuration files.
