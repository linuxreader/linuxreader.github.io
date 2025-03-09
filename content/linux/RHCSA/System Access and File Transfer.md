## System Access and File Transfer 

### Lab: Access RHEL System from Another RHEL System 

- issue the ssh command as user1 on server10 to log in to server20. 
- Run appropriate commands on server20 for validation.
- Log off and return to the originating system.

1\. Issue the ssh command as user1 on server10:

```bash
[user1@server30 tmp]$ ssh server20
```

2\. Issue the basic Linux commands whoami, hostname, and pwd to confirm
that you are logged in as user1 on server20 and placed in the correct
home directory:
```bash
[user1@server40 ~]$ whoami
user1
[user1@server40 ~]$ hostname
server40
[user1@server40 ~]$ pwd
/home/user1

```

3\. Run the logout or the exit command or simply press the key combination Ctrl+d to log off server20 and return to server10:
```bash
[user1@server40 ~]$ exit
logout
Connection to server40 closed.
```



If you wish to log on as a different user such as user2 (assuming user2
exists on the target server server20), you may run the ssh command in
either of the following ways:

`[user1@server30 tmp]$ ssh -l user2 server40`

`[user1@server30 tmp]$ ssh user2@server40`

### Lab: Generate, Distribute, and Use SSH Keys 

- Generate a passwordless ssh key pair using RSA algorithm for user1 on server10.
- display the private and public file contents. 
- Distribute the public key to server20 and attempt to log on to server20 from server10. 
- Show the log file message for the login attempt.

1\. Log on to server10 as user1.

2\. Generate RSA keys without a password (-N) and without detailed
output (-q). Press Enter when prompted to provide the filename to store
the private key.
```bash
[user1@server30 tmp]$ ssh-keygen -N "" -q
Enter file in which to save the key (/home/user1/.ssh/id_rsa): 
```

View the private key:
`[user1@server30 tmp]$ cat ~/.ssh/id_rsa`

View the public key:
`[user1@server30 tmp]$ cat ~/.ssh/id_rsa.pub`

3\. Copy the public key file to server20 under /home/user1/.ssh
directory. 
```bash
user1@server30 tmp]$ ssh-copy-id server40
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/user1/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
user1@server40's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'server40'"
and check to make sure that only the key(s) you wanted were added.

```

- This command also creates or updates the known_hosts
file on server10 and stores the fingerprints for server20 in it. 

`[user1@server30 tmp]$ cat ~/.ssh/known_hosts`

4\. On server10, run the ssh command as user1 to connect to server20.
You will not be prompted for a password because there was none assigned
to the ssh keys.
```bash
[user1@server30 tmp]$ ssh server40
Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
Last login: Sun Jul 21 01:20:17 2024 from 192.168.0.30
```

View this login attempt in the /var/log/secure file on server20:
`[user1@server40 ~]$ sudo tail /var/log/secure`

### Executing Commands Remotely Using ssh 

- Can use `ssh` command to run programs without remoting in:

Execute the hostname command on server20:
```bash
[user1@server30 tmp]$ ssh server40 hostname
server40
```


Run the `nmcli` command on server20 to show (s) active network connections(c):
```bash
[user1@server30 tmp]$ ssh server40 nmcli c s
NAME    UUID                                  TYPE      DEVICE 
enp0s3  1c391bb6-20a3-4eb4-b717-1e458877dbe4  ethernet  enp0s3 
lo      175f8a4c-1907-4006-b838-eb43438d847b  loopback  lo 
```

### sftp` command 
- Interactive file transfer tool. 

On server10, to connect to server20:
```bash
[user1@server30 tmp]$ sftp server40
Connected to server40.
sftp> 
```

Type ? at the prompt to list available commands along with a short description:
```bash
[user1@server30 tmp]$ sftp server40
Connected to server40.
sftp> ?
Available commands:
bye                                Quit sftp
cd path                            Change remote directory to 'path'
chgrp [-h] grp path                Change group of file 'path' to 'grp'
chmod [-h] mode path               Change permissions of file 'path' to 'mode'
chown [-h] own path                Change owner of file 'path' to 'own'
df [-hi] [path]                    Display statistics for current directory or
                                   filesystem containing 'path'
exit                               Quit sftp
get [-afpR] remote [local]         Download file
help                               Display this help text
lcd path                           Change local directory to 'path'
lls [ls-options [path]]            Display local directory listing
lmkdir path                        Create local directory
ln [-s] oldpath newpath            Link remote file (-s for symlink)
lpwd                               Print local working directory
ls [-1afhlnrSt] [path]             Display remote directory listing
lumask umask                       Set local umask to 'umask'
mkdir path                         Create remote directory
progress                           Toggle display of progress meter
put [-afpR] local [remote]         Upload file
pwd                                Display remote working directory
quit                               Quit sftp
reget [-fpR] remote [local]        Resume download file
rename oldpath newpath             Rename remote file
reput [-fpR] local [remote]        Resume upload file
rm path                            Delete remote file
rmdir path                         Remove remote directory
symlink oldpath newpath            Symlink remote file
version                            Show SFTP version
!command                           Execute 'command' in local shell
!                                  Escape to local shell
?                                  Synonym for help
```

Example:

```bash
sftp> ls
sftp> mkdir /tmp/dir10-20
sftp> cd /tmp/dir10-20
sftp> pwd
Remote working directory: /tmp/dir10-20
sftp> put /etc/group
Uploading /etc/group to /tmp/dir10-20/group
group                                       100% 1118     1.0MB/s   00:00    
sftp> ls -l
-rw-r--r--    1 user1    user1        1118 Jul 21 01:41 group
sftp> cd ..
sftp> pwd
Remote working directory: /tmp
sftp> cd /home/user1
sftp> get /usr/bin/gzip
Fetching /usr/bin/gzip to gzip
gzip                                        100%   90KB  23.0MB/s   00:00    
sftp> 
```

- `lcd`, `lls`, `lpwd`, and `lmkdir` are run on the source server. 
- Other commands are also available. (See man pages)

Type quit at the sftp\> prompt to exit the program when you're done:
```bash
sftp> quit
[user1@server30 tmp]$ 
```

