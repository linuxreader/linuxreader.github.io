+++
title = 'Using Ad Hoc commands in scripts'
description = 'Using Ad Hoc commands in scripts'
+++
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