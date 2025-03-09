## Advanced Package Management DIY Labs

1. Configure Access to RHEL 8 Repositories (Make sure the RHEL 8 ISO image is attached to the VM and mounted.)  Create a definition file under */etc/yum.repos.d/*, and define two blocks (one for BaseOS and another for AppStream). 

`vim /etc/yum.repos.d/local.repo`

```
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
`dnf repolist`

### Lab:  Install and Manage Individual Packages

1. List all installed and available packages separately. 
`dnf repoquery && dnf list installed`

2. Show which package contains the /etc/group file. 
`dnf provides /etc/group`

3. Install the package *httpd*. 
`dnf -y install httpd`

4. Review /var/log/yum.log/ for confirmation. (/var/lib/dnf/history)
`dnf history`

5. Perform the following on the httpd package: 
1. Show information 
`dnf info httpd`

2. List dependencies
`dnf repoquery --requires httpd`

3. Remove it
`dnf remove httpd`

### Lab Install and Manage Package Groups

1. List all installed and available package groups separately. 
`dnf group list available && dnf group list installed`

2. Install package groups *Security Tools* and *Scientific Support. 
`dnf group install 'Security Tools'`

3. Review /var/log/yum.log for confirmation. 
`dnf history`

4. Show the packages included in the Scientific Support package group, and delete this group.
`dnf group info 'Scientific Support' && dnf group remove 'Scientific Support'`


### Lab: Install and Manage Modules

1. List all modules. Identify which modules, streams and profiles are installed, default, disabled, and enabled from the output. 
`dnf module list`

2. Install the default stream of the development profile for module *php*, and verify. 
`dnf module install php && dnf module list`

3. Remove the module. 
`dnf module remove php`
### Lab Switch Module Streams and Install Software

1. List *postgresql* module. This will display the streams and profiles, and their status. 
`dnf module list postgresql`

2. Reset both streams
`dnf module reset postgresql`

3. enable the stream for the older version, and install its client profile.
`dnf module install postgresql:15
 `