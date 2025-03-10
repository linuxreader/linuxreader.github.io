## Advanced Package Management DIY Labs

1. Configure Access to RHEL 8 Repositories (Make sure the RHEL 8 ISO image is attached to the VM and mounted.)  Create a definition file under */etc/yum.repos.d/*, and define two blocks (one for BaseOS and another for AppStream). 

```bash
  vim /etc/yum.repos.d/local.repo
```

```bash
 [BaseOS]
 name=BaseOS 
 baseurl=file:///mnt/BaseOS 
 gpgcheck=0 

 [AppStrean]
 name=AppStream 
 baseurl=file:///mnt/AppStream 
 gpgcheck=0
```

4. Verify the configuration with dnf repolist. You should see numbers in thousands under the Status column for both repositories. 
```bash
 dnf repolist -v
```

### Lab:  Install and Manage Individual Packages

1. List all installed and available packages separately. 
```bash
 dnf list --available && dnf list --installed
```

2. Show which package contains the /etc/group file. 
```bash
 dnf provides /etc/group
```

3. Install the package *httpd*. 
```bash
 dnf -y install httpd
```

4. Review /var/log/yum.log/ for confirmation. (/var/lib/dnf/history)
```bash
  dnf history
```

5. Perform the following on the httpd package: 
1. Show information 
```bash
 dnf info httpd
```

2. List dependencies
```bash
 dnf repoquery --requires httpd
```

3. Remove it
```bash
 dnf remove httpd
```

### Lab Install and Manage Package Groups

1. List all installed and available package groups separately. 
```bash
 dnf group list available && dnf group list installed
```

1. Install package groups *Security Tools* and *Scientific Support.*

```bash
 dnf group install 'Security Tools'
```

2. Review /var/log/yum.log for confirmation. 
```bash
 dnf history
```

3. Show the packages included in the Scientific Support package group, and delete this group.
```bash
 dnf group info 'Scientific Support' && dnf group  remove 'Scientific Support'
```


### Lab: Install and Manage Modules

4. List all modules. Identify which modules, streams and profiles are installed, default, disabled, and enabled from the output. 
```bash
 dnf module list
```

5. Install the default stream of the development profile for module *php*, and verify. 
```bash
 dnf module install php && dnf module list
```

6. Remove the module. 
```bash
 dnf module remove php
```
### Lab Switch Module Streams and Install Software

7. List *postgresql* module. This will display the streams and profiles, and their status. 
```bash
 dnf module list postgresql
```

8. Reset both streams
```bash
 dnf module reset postgresql
```

9. enable the stream for the older version, and install its client profile.
```bash
 dnf module install postgresql:15
```