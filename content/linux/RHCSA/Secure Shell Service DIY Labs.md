# Secure Shell Service DIY Labs

### Lab: Establish Key-Based Authentication 

- Create user account user20 on both systems and assign a password. 
```bash
[root@server40 ~]# adduser user20
[root@server40 ~]# passwd user20
Changing password for user user20.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: all authentication tokens updated successfully.
```

- As user20 on server40, generate a private/public key pair without a passphrase using the ssh-keygen command. 
```bash
[user20@server40 ~]# ssh-keygen -N "" -q
Enter file in which to save the key (/root/.ssh/id_rsa): 
```

- Distribute the public key to server30 with the ssh-copy-id command. 
`[user20@server40 ~]# ssh-copy-id server30`
- Log on to server30 as user20 and accept the fingerprints for the server if presented. 
```bash
[user20@server40 ~]# ssh server30
Activate the web console with: systemctl enable --now cockpit.socket

Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
Last login: Fri Jul 19 14:09:22 2024
[user20@server30 ~]# 
```

- On subsequent log in attempts from server40 to server30, user20 should not be prompted for their password. 

### Lab: Test the Effect of PermitRootLogin Directive 

- As user1 with sudo on server30, edit the /etc/ssh/sshd_config file and change the value of the directive PermitRootLogin to "no". 
`[user1@server30 ~]$ sudo vim /etc/ssh/sshd_config` 

- Use the systemctl command to activate the change. 
```bash
[user1@server30 ~]$ systemctl restart sshd
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ====
Authentication is required to restart 'sshd.service'.
Authenticating as: root
Password: 
==== AUTHENTICATION COMPLETE ====
```

- As root on server40, run ssh server40 (or use its IP). You'll get permission denied message. 

(this didn't work, I think it's because I configured passwordless authentication on here)

- Reverse the change on server40 and retry ssh server40. You should be able to log in. 

