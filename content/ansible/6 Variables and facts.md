+++
title = 'Variables and Facts'
description = 'All about Ansible variables and facts'
+++

- Using and working with variables
- Ansible Facts
- Using Vault
- Capture command output using register

Three types of variables:
- Fact
- Variable
- Magic Variable

**Variables** make Ansible really flexible. Especially when used in combination with conditionals. These are defined at the discretion of the user.:
```yaml
---
- name: create a user using a variable
  hosts: ansible1
  vars:
    users: lisa <-- defaults value for this play
  tasks:
    - name: create a user {{ users }} on host {{ ansible_hostname }} <-- ansible fact variable
      user:
        name: "{{ users }}" <-- If value starts with variable, the whole line must have double quotes
``` 

An ansible **fact** variable is a variable that is automatically set based on the managed system. Facts are a default behavior used to discover information to use in conditionals. They are collected when Ansible executes on a remote system. 

There are **systems facts** and **custom facts**. Systems facts are system property values. And custom facts are user-defined variables stored on managed hosts. 

If no variables are defined at the command prompt, it will use the variable set for the play. You can also define the variables with the `-e` flag when running the playbook:
```bash
[ansible@control base]$ ansible-playbook variable-pb.yaml -e users=john

PLAY [create a user using a variable] ************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [ansible1]

TASK [create a user john on host ansible1] *******************************************************************************************************************
changed: [ansible1]

PLAY RECAP ***************************************************************************************************************************************************
ansible1                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

A **magic variable** is a system variable that is automatically set. 

Notice the "Gathering Facts" task. when you run a playbook. This is an implicit tasks ran every time you run a playbook. This task grabs facts from managed hosts and stores them in the variable **ansible_facts**. 

You can use the **debug** module to display variables like so:
```yaml
---
- name: show facts
  hosts: all
  tasks:
  - name: show facts
    debug:
      var: ansible_facts <-- this module does require variables to be enclosed in curly brackets
```

This outputs a gigantic list of facts from our managed nodes. 