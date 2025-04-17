+++
title = 'Ansible Inventory and Ansible.cfg'
description = 'Setting up and Ansible environment'
+++


## Ansible projects

For small companies, you can use a single Ansible configuration. But for larger ones, it's a good idea to use different project directories. A project directory contains everything you need to work on a single project. Including:
- playbooks
- variable files
- task files
- inventory files
- ansible.cfg

*playbook*
An Ansible script written in YAML that enforce the desired configuration on manage hosts.

## Inventory
A file that Identifies hosts that Ansible has to manage. You can also use this to list and group hosts and specify host variables. Each project should have it's own inventory file. /etc/ansible/hosts can be used for system wide inventory. Which is the default if no inventory file is specified. (That file also has some basic inventory formatting info if you forget) Ansible will also target localhosts if no hosts are found in the inventory file. It's a good idea to store inventory files in large environments in their own project folders. 

localhost is not defined in inventory. It is an implicit hosts that is usable and refers to the Ansible control machine. Using localhost can be a good way to verify the accessibility of services on managed hosts. 

### Listing hosts
List hosts by IP address or hostname. You can list a range of hosts in an inventory file as well such as `web-server[1:10].example.com`
```yaml
ansible1:2222 < specify ssh port if the host is not using the default port 22
ansible2
10.0.10.55
web-server[1:10].example.com
```

### Listing groups
You can list groups and groups of groups. See the groups web and db are included in the group "servers:children"
```yaml
ansible1
ansible2
10.0.10.55
web-server[1:10].example.com

[web]
web-server[1:10].example.com

[db]
db1
db2

[servers:children] <-- servers is the group of groups and children is the parameter that specifies child groups
web
db
```

There are 3 general approaches to using groups:

**Functional groups**
Address a specific group of hosts according to use. Such as web servers or database servers. 

**Regional host groups**
Used when working with region oriented infrastructure. Such as USA, Canada.

**Staging host groups**
Used to address different hosts according to the staging phase that the current environment is in. Such as testing, development, production. 

Undefined host groups are called implicit host groups. These are **all**, **ungrouped**, and **localhost**. Names making the meaning obvious. 

### Inventory commands:

To view the inventory, specify the inventory file such as ~/base/inventory in the command line. You can name the inventory file anything you want. You can also set the default in the ansible.cfg file.

View the current inventory:
`ansible -i inventory <pattern> --list-hosts` 

List inventory hosts in JSON format:
`ansible-inventory -i inventory --list`

Display overview of hosts as a graph:
`ansible-inventory -i inventory --graph`

In our lab example:
```bash
[ansible@control base]$ pwd
/home/ansible/base

[ansible@control base]$ ls
inventory

[ansible@control base]$ cat inventory
ansible1
ansible2

[web]
web1
web2

[ansible@control base]$ ansible-inventory -i inventory --graph
@all:
  |--@ungrouped:
  |  |--ansible1
  |  |--ansible2
  |--@web:
  |  |--web1
  |  |--web2

[ansible@control base]$ ansible-inventory -i inventory --list
{
    "_meta": {
        "hostvars": {}
    },
    "all": {
        "children": [
            "ungrouped",
            "web"
        ]
    },
    "ungrouped": {
        "hosts": [
            "ansible1",
            "ansible2"
        ]
    },
    "web": {
        "hosts": [
            "web1",
            "web2"
        ]
    }
}

[ansible@control base]$ ansible -i inventory all --list-hosts
  hosts (4):
    ansible1
    ansible2
    web1
    web2
    
[ansible@control base]$ ansible -i inventory ungrouped --list-hosts
  hosts (2):
    ansible1
    ansible2

```

### Host variables
In older versions of Ansible you could define variables for hosts. This is no longer used. Example:
```yaml

[groupname:vars]
ansible=ansible_user
```

Variables are now set using host_vars and group_vars directories instead. 

### Dynamic inventory scripts
A script is used to detect inventory hosts so that you do not have to manually enter them. This is good for larger environments. You can find community provided dynamic inventory scripts that come with an .ini file that provides information on how to connect to a resource. 

Inventory scripts must include --list and --host options and output must be JSON formatted. Here is an example from [sandervanvught](https://github.com/sandervanvugt/rhce8-book/blob/master/listing33.py) that generates an inventory script using /etc/hosts:
```python
[ansible@control base]$ cat inventory-helper.py

#!/usr/bin/python

from subprocess import Popen,PIPE
import sys

try:
     import json
except ImportError:
     import simplejson as json



result = {}

result['all'] = {}



pipe = Popen(['getent', 'hosts'], stdout=PIPE, universal_newlines=True)


result['all']['hosts'] = []

for line in pipe.stdout.readlines():
    s = line.split()
    result['all']['hosts']=result['all']['hosts']+s


result['all']['vars'] = {}


if len(sys.argv) == 2 and sys.argv[1] == '--list':
    print(json.dumps(result))

elif len(sys.argv) == 3 and sys.argv[1] == '--host':
    print(json.dumps({}))

else:
    print("Requires an argument, please use --list or --host <host>")
```

When ran on our sample lab:
```bash
[ansible@control base]$sudo python3 ./inventory-helper.py
Requires an argument, please use --list or --host <host>

[ansible@control base]$ sudo python3 ./inventory-helper.py --list
{"all": {"hosts": ["127.0.0.1", "localhost", "localhost.localdomain", "localhost4", "localhost4.localdomain4", "127.0.0.1", "localhost", "localhost.localdomain", "localhost6", "localhost6.localdomain6", "192.168.124.201", "ansible1", "192.168.124.202", "ansible2"], "vars": {}}}

```

To use a dynamic inventory script:
```bash
[ansible@control base]$ chmod u+x inventory-helper.py 
[ansible@control base]$ sudo ansible -i inventory-helper.py all --list-hosts
[WARNING]: A duplicate localhost-like entry was found (localhost). First found localhost was 127.0.0.1
  hosts (11):
    127.0.0.1
    localhost
    localhost.localdomain
    localhost4
    localhost4.localdomain4
    localhost6
    localhost6.localdomain6
    192.168.124.201
    ansible1
    192.168.124.202
    ansible2
```

### Multiple inventory files
Put all inventory files in a directory and specify the directory as the inventory to be used. For dynamic directories you also need to set the execution bit on the inventory file. 


## ansible.cfg
You can store this in a project's directory or a user's home directory, in the case that multiple user's want to have their own Ansible configuration. Or in /etc/ansible if the configuration will be the same for every user and every project. You can also specify these settings in Ansible playbooks. The settings in a playbook take precedence over the .cfg file. 

**ansible.cfg precedence** (Ansible uses the first one it finds and ignores the rest.)
1. ANSIBLE_CONFIG environment variable
2. ansible.cfg in current directory
3. ~/.ansible.cfg
4. /etc/ansible/ansible.cfg

Generate an example config file in the current directory.  All directive are commented out by default:
`[ansible@control base]$ ansible-config init --disabled > ansible.cfg`

Include existing plugin to the file:
`ansible-config init --disabled -t all > ansible.cfg`

This generates an extremely large file. So I'll just show Van Vugt's example in .ini format:
```bash
[defaults] <-- General information
remote_user = ansible <--Required
host_key_checking = false <-- Disable SSH host key validity check
inventory = inventory

[privilege_escalation] <-- Define how ansible user requires admin rights to connect to hosts
become = True <-- Escalation required
become_method = sudo
become_user = root <-- Escalated user
become_ask_pass = False <-- Do not ask for escalation password
```

Privilege escalation parameters can be specified in ansible.cfg, playbooks, and on the command line. 

