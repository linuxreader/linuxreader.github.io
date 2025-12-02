+++
title = 'Ad Hoc Ansible Commands'
description = 'Ad Hoc Ansible Commands '
[_build]
  render = "always"
  list = "never"
+++

 Ad hoc commands are ansible tasks you can run against managed hosts without the need of a playbook or script. These are used for bringing nodes to their desired states, verifying playbook results, and verifying nodes meet any needed criteria/pre-requisites. These must be ran as the ansible user (whatever your remote_user directive is set to under \[defaults\] in ansible.cfg)

Run the user module with the argument name=lisa on all hosts to make sure the user "lisa" exists. If the user doesn't exist, it will be created on the remote system:
`ansible all -m user -a "name=lisa"`

`{command} {host} -m {module} -a {"argument1 argument2 argument3"}`

In our lab:
```bash
ansible all -m user -a "name=lisa"
```

This Ad Hoc command created user "Lisa" on ansible1 and ansible2. If we run the command again, we get "SUCCESS" on the first line instead of "CHANGED". Which means the hosts already meet the requirements:
```bash
[ansible@control base]$ ansible all -m user -a "name=lisa"
```

*indempotent* 
Regardless of current condition, the host is brought to the desired state. Even if you run the command multiple times. 

Run the command `id lisa` on all managed hosts:
```bash
[ansible@control base]$ ansible all -m command -a "id lisa"
```

Here, the command module is used to run a command on the specified hosts. And the output is displayed on screen. To note, this does not show up in our ansible user's command history on the host:
```bash
[ansible@ansible1 ~]$ history
```

Remove the user lisa from all managed hosts:
```bash
[ansible@control base]$ ansible all -m user -a "name=lisa state=absent"
```

You can also use the `-u` option to specify the Ansible user that Ansible will use to run the command. Remember, with no modules specified, ansible uses the `command` module:
`ansible all -a "free -m" -u david`
`
