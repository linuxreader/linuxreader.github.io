## Basic User Management Labs
### Lab: who

```
who
```

### Lab: what
```
w
```

### Lab: last

1. List all user login, logout, and system reboot occurrences:
```
last
```

2. List system reboot info only:
```
last reboot
```

### Lab: lastb
```
lastb
```

### Lab: lastlog
```
lastlog
```

### Lab: id
1. View info about currently active user:
```
id
```

2. View info about another user:
```
id user1
```

### Lab: groups
1. View current user's groups:
```
groups
```

2. View groups of another user:
```
groups user1
```

### Lab: user authentication files
1. list of the four files and their backups from the /etc directory:
```bash
ls -l /etc/passwd* /etc/group* /etc/shadow* /etc/gshadow*
```

2. View first and last 3 lines of the passwd file
```bash
head -3 /etc/passwd ; tail -3 /etc/passwd
```

3. verify the permissions and ownership on the passwd file:
```bash
ls -l /etc/passwd
```

4. View first and last 3 lines of the shadow file:
```bash
head -3 /etc/shadow ; tail -3 /etc/shadow
```

5. verify the permissions and ownership on the shadow file:
```bash
ls -l /etc/shadow
```

6. View first and last 3 lines of the group file:
```bash
head -3 /etc/group ; tail -3 /etc/group
```

7. Verify the permissions and ownership on the group file:
```bash
ls -l /etc/group
```

8. View first and last 3 lines of the gshadow file:
```bash
head -3 /etc/gshadow ; tail -3 /etc/gshadow
```

9. Verify the permissions and ownership on the gshadow file:
```bash
ls -l /etc/gshadow
```

### Lab: useradd and login.defs
1. use the cat or less command to view the useradd file content or display the settings with the useradd command:
```bash
useradd -D
```
2. grep on the/etc/login.defs with uncommented and non-empty lines:
```
grep -v ^# /etc/login.defs | grep -v ^$
```

### Lab: Create a User Account with Default Attributes (root)
1. Create user2 with all the default directives:
```bash
useradd user2
```

2. Assign this user a password and enter it twice when prompted:
```bash
passwd user2
```

3. grep for user2: on the authentication files to examine what the useradd command has added:
```bash
cd /etc ; grep user2: passwd shadow group gshadow
```

4. Test this new account by logging in as user2 and then run the id and groups commands to verify the UID, GID, and group membership information:
```bash
su - user2
id
groups
```

### Lab: Create a User Account with Custom Values

1. Create user3 with UID 1010 (-u), home directory /usr/user3a (-d), and shell /bin/sh (-s): 
```bash
useradd -u 1010 -d /usr/user3a -s /bin/sh user3
```
2. Assign user1234 as password (passwords assigned in the following way is not recommended; however, it is okay in a lab environment):
```bash
echo user1234 | passwd --stdin user3
```

3. grep for user3: on the four authentication files to see what was added for this user:
```bash
cd /etc ; grep user3: passwd shadow group gshadow
```

4. Test this account by switching to or logging in as user3 and entering user1234 as the password. Run the id and groups commands for further verification.
```bash
su - user3 
id
groups
```

### Lab: Modify and Delete a User Account
1. Modify the login name for user2 to user2new (-l), UID to 2000 (-u), home directory to /home/user2new (-m and -d), and login shell to /sbin/nologin (-s). 
```bash
usermod -l user2new -m -d /home/user2new -s /sbin/nologin -u 2000 user2
```

2. Obtain the information for user2new from the passwd file for confirmation:
```bash
grep user2new /etc/passwd
```

3. Remove user2new along with their home and mail spool directories (-r):
```bash
userdel -r user2new
```

4. Confirm the user deletion:
```bash
grep user2new /etc/passwd
```

### Lab: Create a User Account with No-Login Access (root)
1. Look at the current nologin users:
```bash
grep nologin /etc/passwd
```

2. Create user4 with non-interactive shell file /sbin/nologin:
```bash
useradd -s /sbin/nologin user4
```

3. Assign user1234 as password:
```bash
echo user1234 | passwd --stdin user4
```

4. grep for user4 on the passwd file and verify the shell field containing the nologin shell:
```bash
grep user4 /etc/passwd
```

5. Test this account by attempting to log in or switch:
```bash
su - user4
```

### Lab: Check User Login Attempts (root)
1. execute the last, lastb, and lastlog commands, and observe the outputs. 
```bash
last
lastb
lastlog
```

2. List the timestamps when the system was last rebooted (last). 
```bash
last | grep reboot
```

### Lab 5-2: Verify User and Group Identity (user1)
1. run the who and w commands one at a time, and compare the outputs. 
```bash
who
w
```

2. Execute the `id` and `groups` commands, and compare the outcomes. Examine the extra information that the id command shows, but not the groups command. 
```bash
id
groups
```

### Lab 5-3: Create Users (root)
1. create user account user4100 with UID 4100 and home directory under /usr. 
```bash
useradd -m -d /usr/user4100 -u 4100 user4100 
```

2. Create another user account user4200 with default attributes.
```bash
useradd user4200
```

3. Assign both users a password. 
```bash
passwd user4100
passwd user4200
```

4. View the contents of the passwd, shadow, group, and gshadow files, and observe what has been added for the two new users. 
```bash
cat /etc/passwd
cat /etc/shadow
cat /etc/group
cat /etc/gshadow
```
### Lab: Create User with Non-Interactive Shell (root)
1. Create user account user4300 with the disability of logging in. 
```bash
useradd -s /sbin/nologin user4300
```

3. Assign this user a password. 
```bash
passwd user4300
```

5. Try to log on with this user and see is displayed on the screen. 
```bash
su - user4300
```

7. View the content of the passwd file, and see what is there that prevents this user from logging in. 
```bash
cat /etc/passwd
```
