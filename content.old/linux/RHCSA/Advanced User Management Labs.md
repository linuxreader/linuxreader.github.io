## Advanced User Management Labs

### Lab: Set and Confirm Password Aging with chage (root)
1. Set password aging parameters for user100 to mindays (-m) 7, maxdays (-M) 28, and warndays (-W) 5:
```bash
chage -m 7 -M 28 -W 5 user100
```

2. Confirm
```bash
chage -l user100
```

3. Set the account expiry to January 31, 2020
```bash
chage -E 2020-01-31 user100
```

4. Verify the new account expiry setting
```bash
chage -l user100
```

### Lab: Set and Confirm Password Aging with passwd (root)

1. Set password aging attributes for user200 to mindays 10, maxdays 90, and warndays 14:
```bash
passwd -n 10 -x 90 -w 14 user200
```

2. Confirm:
```bash
passwd -S user200
```

3. Set the number of inactivity days to 5:
```bash
passwd -i 5 user200
```

4. Confirm:
```bash
passwd -S user200
```

5. Ensure that the user is forced to change their password at next login:
```bash
passwd -e user200
```

6. Confirm:
```bash
passwd -S user200
```

### Lab: Lock and Unlock a User Account with usermod and passwd (root)
1. Obtain the current password information for user200 from the shadow file:
```bash
grep user200 /etc/shadow
```

2. Lock the account for user200:
```bash
usermod -L user200 
```

3. Confirm:
```bash
grep user200 /etc/shadow
```

4. Unlock the account with either of the following:
```bash
usermod -U user200
or
passwd -u user200
```

5. confirm
```bash
grep user200 /etc/shadow
```


### Lab: Create a Group and Add Members (root)
1. Create the group linuxadm with GID 5000:
```bash
groupadd -g 5000 linuxadm
```

2. Create a group called dba with the same GID as that of group linuxadm:
```bash
groupadd -o -g 5000 dba
```

3. Confirm:
```bash
grep linuxadm /etc/group
grep dba /etc/group
```

4. Add user1 as a secondary member of group dba using the `usermod `command. The existing membership for the user must remain intact.
```bash
usermod -aG dba user1
```

5. Verify the updated group membership information for user1 by extracting the relevant entry from the group file, and running the id and groups command for user1:
```bash
grep dba /etc/group
id user1
groups user1
```


### Lab: Modify and Delete a Group Account (root)
1. Alter the name of linuxadm to sysadm:
```bash
groupmod -n sysadm linuxadm
```

2. Change the GID of sysadm to 6000:
```bash
groupmod -g 6000 sysadm
```

3. Confirm:
```bash
grep sysadm /etc/group
grep linuxadm /etc/group
```

4. Delete sysadm group and confirm:
```bash
groupdel sysadm
grep sysadm /etc/group
```

### Lab: To switch from user1 (assuming you are logged in as user1) into root without executing the startup scripts
1. 
```bash
su
```

2. switch to user100
```bash
su - user100
```

3. See what whoami and logname reports now:
```bash
whoami
logname
```

4. use su as follows and execute this privileged command to obtain desired results:
```bash
su -c 'firewall-cmd --list-services'
```
### Lab: Add user1 to sudo file but only for the cat command.
1. Open up /etc/sudoers and add the following:
```bash
user1 ALL=/usr/bin/cat
```

2. run cat as user1 with and without sudo:
```bash
cat /etc/sudoers
sudo cat /etc/sudoers
```

### Lab: Add user and command aliases to the sudoer file.

1. Add the following to the bottom of the sudoers file:
```bash
Cmnd_Alias PKGCMD = /usr/bin/yum, /usr/bin/rpm
User_Alias PKGADM = user1, user100, user200 
PKGADM ALL=PKGCMD
```

2. Run rpm or yum with sudo as one of the users.
```bash
sudo yum 
```

### Lab: Take a look at examples in the sudoers file.
```bash
cat /etc/sudoers
```

### Lab: Viewing owner and group information
1. Create a file file1 as user1 in their home directory and exhibit the file’s long listing:
```bash
touch file1
ls -l file1
```

2. View the corresponding UID and GID instead, you can specify the -n option with the command:
```bash
ls -ln file1
```
### Lab: Modify File Owner and Owning Group
1. Change into the /tmp directory and create file10 and dir10:
```bash
cd /tmp
touch file10
mkdir dir10
```

2. Check and validate that both attributes are set to user1:
```bash
ls -l file10
ls -ld dir10
```

3. Set the ownership of file10 to user100 and confirm:
```bash
sudo chown user100 file10
ls -l file10
```

4. Alter the owning group to dba and verify:
```bash
sudo chgrp dba file10
ls -l file10
```

5. Change the ownership to user200 and owning group to user100 and confirm:
```bash
sudo chown user200:user100 file10
```

6. Modify the ownership to user200 and owning group to dba recursively on dir10 and validate:
```bash
sudo  chown -R user200:dba dir10
ls -ld dir10
```

### Lab: Create User and Configure Password Aging (root)
1. Create group lnxgrp with GID 6000. 
```bash
groupadd lnxgrp -g 6000
```

2. Create user user5000 with UID 5000 and GID 6000. Assign this user a password.
```bash
useradd -u 5000 -g 6000 user5000
```

3. Establish password aging attributes so that this user cannot change their password within 4 days after setting it and with a password validity of 30 days. This user should start getting warning messages for changing password 10 days prior to account lock down. 
```bash
chage -m 4 -M 30 -W 10 user5000
```

4. This user account needs to expire on the 20th of December, 2021. 
```bash
chage -E 2021-12-20 user5000
```

### Lab 6-2: Lock and Unlock User (root)
1. Lock the user account for user5000 using the passwd command, and 
```bash
passwd -l user5000
```

2. confirm by examining the change in the /etc/shadow file. 
```bash
cat /etc/shadow
```

3. Try to log in with user5000 and observe what happens. 
```bash
su - user1
su - user5000
```

4. Use the usermod command and unlock
