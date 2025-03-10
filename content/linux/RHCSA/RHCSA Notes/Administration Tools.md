## Administration Tools

`ip `
- Display monitor and manage network interfaces, routing, connections, traffic, etc. 

`ifup`
- Brings up an interface

`ifdown`
- Brings down an interface

`nmcli`                               
- Creates, updates, deletes, activates, and deactivates a connection profile.

### `nmcli` command

- Create, view, modify, remove, activate, and deactivate network connections.
- Control and report network device status.
- Supports abbreviation of commands.

Operates on 7 different object categories.
1. general
2. networking
3. connection (c)(con)
4. device (d)(dev)
5. radio
6. monitor
7. agent

```bash
[root@server200 system-connections]# nmcli --help
Usage: nmcli [OPTIONS] OBJECT { COMMAND | help }

OPTIONS
  -a, --ask                                ask for missing parameters
  -c, --colors auto|yes|no                 whether to use colors in output
  -e, --escape yes|no                      escape columns separators in values
  -f, --fields <field,...>|all|common      specify fields to output
  -g, --get-values <field,...>|all|common  shortcut for -m tabular -t -f
  -h, --help                               print this help
  -m, --mode tabular|multiline             output mode
  -o, --overview                           overview mode
  -p, --pretty                             pretty output
  -s, --show-secrets                       allow displaying passwords
  -t, --terse                              terse output
  -v, --version                            show program version
  -w, --wait <seconds>                     set timeout waiting for finishing operations

OBJECT
  g[eneral]       NetworkManager\'s general status and operations
  n[etworking]    overall networking control
  r[adio]         NetworkManager radio switches
  c[onnection]    NetworkManager\'s connections
  d[evice]        devices managed by NetworkManager
  a[gent]         NetworkManager secret agent or polkit agent
  m[onitor]       monitor NetworkManager changes
```

### 3. connection
- Activates, deactivates, and administers network connections.

Options:
- `show` (list connections)
- `up`/`down` (Brings connection up or down)
- `add(a)` (adds a connection)
- `edit` (edit connection or add a new one)
- `modify` (modify properties of a connection)
- `delete(d)` (delete a connection)
- `reload` (re-read all connection profiles)
- `load` (re-read a connection profile)

### 4. Device

Options:
- `status` (Displays device status)
- `show` (Displays info about device(s)

Show all connections, inactive or active:
```bash
nmcli c s
```

Deactivate the connection enp0s8:
```bash
sudo nmcli c down enp0s8
```

Note:
```
The connection profile gets detached from the device, disabling the connection.
```

Activate the connection enp0s8:
```bash
$ sudo nmcli c up enp0s8
# connection profile re-attaches to the device.
```

Display the status of all network devices:
```bash
nmcli d s
```

### Lab: Add Network Devices to server10 and one to server20 using VirtualBox

1. Shut down your servers (follow each step for both servers)
```bash
sudo shutdown now
```
2. Add network interface in Virtualbox then power on the VMs
```
Select machine > settings > Network > Adapter 2 > Enable Network Adapter > Internal Network > ok
```
3. Verify the new interfaces:
```bash
ip a
```

### Lab: Configure New Network Connection Using nmcli (server20)

1. Verify the interface that was added from virtualbox:
```bash
nmcli d status | grep enp
```

2. Add connection profile and attach it to the interface:
```bash
sudo nmcli c a type Ethernet ifname enp0s8 con-name enp0s8 ip4 172.10.10.120/24 gw4 172.10.10.1
```

3. Confirm connection status
```
nmcli d status | grep enp
```

4. Verify ip address
```
ip a
```

5. Check the content of the connection profile
```
cat /etc/NetworkManager/system-connections/enp0s8.nmconnection
```
