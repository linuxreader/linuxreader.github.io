+++
title = 'Ad Hoc Ansible Commands'
description = 'Getting started with some simple Ansible commands'
+++

Building off our lab, we need a playbook that give instructions for getting managed nodes to their desired states. Playbooks are scripts written in YAML. There are some things you need to know when working with playbooks:
- Ad Hoc Commands
- Modules
- Module Documentation
- Ad Hoc commands from bash scripts

## Ad Hoc Commands
 Ad hoc commands are ansible tasks you can run against managed hosts without the need of a playbook or script. These are used for bringing nodes to their desired states, verifying playbook results, and verifying nodes meet any needed criteria/pre-requisites. These must be ran as the ansible user (whatever your remote_user directive is set to under \[defaults\] in ansible.cfg)

Run the user module with the argument name=lisa on all hosts to make sure the user "lisa" exists. If the user doesn't exist, it will be created on the remote system:
`ansible all -m user -a "name=lisa"`

`{command} {host} -m {module} -a {"argument1 argument2 argument3"}`

In our lab:
```bash
[ansible@control base]$ ansible all -m user -a "name=lisa"
web1 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web1: Name or service not known",
    "unreachable": true
}
web2 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web2: Name or service not known",
    "unreachable": true
}
ansible1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "comment": "",
    "create_home": true,
    "group": 1001,
    "home": "/home/lisa",
    "name": "lisa",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 1001
}
ansible2 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "comment": "",
    "create_home": true,
    "group": 1001,
    "home": "/home/lisa",
    "name": "lisa",
    "shell": "/bin/bash",
    "state": "present",
    "system": false,
    "uid": 1001
}
```

This Ad Hoc command created user "Lisa" on ansible1 and ansible2. If we run the command again, we get "SUCCESS" on the first line instead of "CHANGED". Which means the hosts already meet the requirements:
```bash
[ansible@control base]$ ansible all -m user -a "name=lisa"
web2 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web2: Name or service not known",
    "unreachable": true
}
web1 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web1: Name or service not known",
    "unreachable": true
}
ansible2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "append": false,
    "changed": false,
    "comment": "",
    "group": 1001,
    "home": "/home/lisa",
    "move_home": false,
    "name": "lisa",
    "shell": "/bin/bash",
    "state": "present",
    "uid": 1001
}
ansible1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "append": false,
    "changed": false,
    "comment": "",
    "group": 1001,
    "home": "/home/lisa",
    "move_home": false,
    "name": "lisa",
    "shell": "/bin/bash",
    "state": "present",
    "uid": 1001
}

```

*indempotent* 
Regardless of current condition, the host is brought to the desired state. Even if you run the command multiple times. 

Run the command `id lisa` on all managed hosts:
```bash
[ansible@control base]$ ansible all -m command -a "id lisa"
web1 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web1: Name or service not known",
    "unreachable": true
}
web2 | UNREACHABLE! => {
    "changed": false, module should you use to run the rpm -qa | grep httpd command?


    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web2: Name or service not known",
    "unreachable": true
}
ansible1 | CHANGED | rc=0 >>
uid=1001(lisa) gid=1001(lisa) groups=1001(lisa)
ansible2 | CHANGED | rc=0 >>
uid=1001(lisa) gid=1001(lisa) groups=1001(lisa)
```

Here, the command module is used to run a command on the specified hosts. And the output is displayed on screen. TO note, this does not show up in our ansible user's command history on the host:
```bash
[ansible@ansible1 ~]$ history
    1  history
```

Remove the userLlisa from all managed hosts:
```bash
[ansible@control base]$ ansible all -m user -a "name=lisa state=absent"
web2 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web2: Name or service not known",
    "unreachable": true
}
web1 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web1: Name or service not known",
    "unreachable": true
}
ansible1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "force": false,
    "name": "lisa",
    "remove": false,
    "state": "absent"
}
ansible2 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": true,
    "force": false,
    "name": "lisa",
    "remove": false,
    "state": "absent"
}
[ansible@control base]$ ansible all -m command -a "id lisa"
web1 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web1: Name or service not known",
    "unreachable": true
}
web2 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web2: Name or service not known",
    "unreachable": true
}
ansible1 | FAILED | rc=1 >>
id: ‘lisa’: no such usernon-zero return code
ansible2 | FAILED | rc=1 >>
id: ‘lisa’: no such usernon-zero return code
```

You can also use the `-u` option to specify the user that Ansible will use to run the command. Remember, with no modules specified, ansible uses the `command` module:
`ansible all -a "free -m" -u david`
## Modules

There are more than 3000 Ansible modules available for a variety of different tasks and servers. The more modules you know, the better you will be at Ansible. They are essentially plug ins written in Python that can be used in playbooks or Ad Hoc commands. Make sure to use modules that are the most specific to the task you are trying to accomplish. 

### Important modules

#### Arbitrary Modules
Limit your use of these, it's hard to track what has been changed using these modules. Use the more specific indempotent module for the task instead, if you can. 

*command*
Runs arbitrary commands (not using the shell). Shell stuff such as pipes and redirects will not work with this module. This is the default module if no modules are specified. You can set different default module in ansible.crf by using "module_name = module". A python script is generated on the manage host and executed. 
`ansible all -m command -a "rpm -qa | grep httpd"` (the pipe gets ignored)

Check status of httpd:
`ansible all -m command -a "systemctl status httpd"`

*shell*
Same as above but with the shell. So pipes and redirects will work. A python script is generated on the manage host and executed. 
`ansible all -m shell -a "rpm -qa | grep httpd"` (the pipe is not ignored)

*raw*
Runs arbitrary command on top of SSH without using Python. Good for managed hosts that don't have Python. Or install Python during setup:
`ansible -u root -i inventory ansible3 --ask-pass -m raw -a "yum install python3`

#### Indempotent modules
These are easier to track and guarantee indempotency. 

*copy*
Copy files or lines of text to files
`ansible all -m copy -a 'content="hello world" dest=/etc/motd'

*yum*
Manage packages on RHEL hosts. Can use the package module to install packages on any Linux distro. Use the `yum` module if you need specific yum features. And `package` module if you need to manage software on different distros. 

Install latest version of nmap:
`ansible all -m yum -a "name=nmap state=latest"

List httpd details:
`ansible all -m yum -a "list=httpd"`


*service*
Manage state of systemd and system-V services. Make sure to use enabled=yes and state=started to make sure services are enabled at startup. 
`ansible -m service -a "name=httpd state=started enabled=yes"`

*ping*
Checks if managed hosts are in a manageable state. 
`ansible all -m ping`


### Viewing available modules with `ansible-doc`

As noted before, there are over 3,000 modules that come with Ansible. These are installed on your system when you install Ansible. View all the modules available like so:
`ansible-doc -l`

Filter to get more specific results:
```bash
[ansible@control ~]$ ansible-doc -l | grep package
ansible.builtin.apt                    Manages apt-packages                
ansible.builtin.debconf                Configure a .deb package            
ansible.builtin.dnf                    Manages packages with the `dnf' pack...
ansible.builtin.dpkg_selections        Dpkg package selection selections   
ansible.builtin.package                Generic OS package manager          
ansible.builtin.package_facts          Package information as facts        
ansible.builtin.yum                    Manages packages with the `yum' pack...
```

Finding details on a specific module:
`ansible-doc ping`

Output shows the module name, maintainter information, options available, related modules, module author, examples, and return values. 

Each module is a Python script on your system that you can view if you want to see what is going on under the hood:
`> ANSIBLE.BUILTIN.PING    (/usr/lib/python3.9/site-packages/ansible/modules/ping.py)`

Make sure you read the modules description for details!
```
A trivial test module, this module always returns `pong' on successful contact. It does not make sense in playbooks,
        but it is useful from `/usr/bin/ansible' to verify the ability to login and that a usable Python is configured. This
        is NOT ICMP ping, this is just a trivial test module that requires Python on the remote-node. For Windows targets, use
        the [ansible.windows.win_ping] module instead. For Network targets, use the [ansible.netcommon.net_ping] module
        instead.
```

Note that mandatory option are listed as =option instead of -option.
```
OPTIONS (= is mandatory):

- data
        Data to return for the `ping' return value.
        If this parameter is set to `crash', the module will cause an exception.
        default: pong
        type: str
```

And don't forget to check the "SEE ALSO" section to see if there could be a module that better suits your needs:
```
SEE ALSO:
      * Module ansible.netcommon.net_ping
      * Module ansible.windows.win_ping
```

Here are some examples from the raw module doc:
```
EXAMPLES:

- name: Bootstrap a host without python2 installed
  ansible.builtin.raw: dnf install -y python2 python2-dnf libselinux-python

- name: Run a command that uses non-posix shell-isms (in this example /bin/sh doesn't handle redirection and wildcards together but bash does)
  ansible.builtin.raw: cat < /tmp/*txt
  args:
    executable: /bin/bash

- name: Safely use templated variables. Always use quote filter to avoid injection issues.
  ansible.builtin.raw: "{{ package_mgr|quote }} {{ pkg_flags|quote }} install {{ python|quote }}"

- name: List user accounts on a Windows system
  ansible.builtin.raw: Get-WmiObject -Class Win32_UserAccount
```

The examples show the playbook code for common use cases for running the module. Use the -s flag to show the playbook snippet only:
```bash
[ansible@control ~]$ ansible-doc -s service
- name: Manage services
  service:
      arguments:             # Additional arguments provided on the command line. While using remote hosts with systemd this setting will be ignored.
      enabled:               # Whether the service should start on boot. *At least one of state and enabled are required.*
      name:                  # (required) Name of the service.
      pattern:               # If the service does not respond to the status command, name a substring to look for as would be found in the output of the `ps'
                             # command as a stand-in for a status result. If the string is found, the service will be assumed to
                             # be started. While using remote hosts with systemd this setting will be ignored.
      runlevel:              # For OpenRC init scripts (e.g. Gentoo) only. The runlevel that this service belongs to. While using remote hosts with systemd
                             # this setting will be ignored.
      sleep:                 # If the service is being `restarted' then sleep this many seconds between the stop and start command. This helps to work around
                             # badly-behaving init scripts that exit immediately after signaling a process to stop. Not all
                             # service managers support sleep, i.e when using systemd this setting will be ignored.
      state:                 # `started'/`stopped' are idempotent actions that will not run commands unless necessary. `restarted' will always bounce the
                             # service. `reloaded' will always reload. *At least one of state and enabled are required.* Note
                             # that reloaded will start the service if it is not already started, even if your chosen init
                             # system wouldn't normally.
      use:                   # The service module actually uses system specific modules, normally through auto detection, this setting can force a specific
                             # module. Normally it uses the value of the 'ansible_service_mgr' fact and falls back to the old
                             # 'service' module when none matching is found.
```

The official Ansible documentation will also be available during the RHCE exam:
https://docs.ansible.com/

The docs will also show you how to install additional module collections. To install the posix collection:
`[ansible@control base]$ ansible-galaxy collection install ansible.posix`
## Ad hoc commands in Scripts

Follow normal bash scripting guidelines to run ansible commands in a script:
```bash
[ansible@control base]$ vim httpd-ansible.sh
```

Let's set up a script that installs and starts/enables httpd, creates a user called "anna", and copies the ansible control node's /etc/hosts file to /tmp/ on the managed nodes:

```
#!/bin/bash

ansible all -m yum -a "name=httpd state=latest"
ansible all -m service -a "name=httpd state=started enabled=yes"
ansible all -m user -a "name=anna"
ansible all -m copy -a "src=/etc/hosts dest=/tmp/hosts"

```

```bash
[ansible@control base]$ chmod +x httpd-ansible.sh
[ansible@control base]$ ./httpd-ansible.sh 
web2 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web2: Name or service not known",
    "unreachable": true
}
web1 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname web1: Name or service not known",
    "unreachable": true
}
ansible1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
ansible2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
... <-- Results truncated
```

And from the ansible1 node we can verify:
```bash
[ansible@ansible1 ~]$ cat /etc/passwd | grep anna
anna:x:1001:1001::/home/anna:/bin/bash
```

```bash
[ansible@ansible1 ~]$ cat /tmp/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.124.201 ansible1
192.168.124.202 ansible2
```

View a file from a managed node:
`ansible ansible1 -a "cat /somfile.txt"`