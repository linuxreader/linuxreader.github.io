## Package Groups
### package group
- Group of packages that serve a common purpose. 
- Can query, install, and delete as a single unit rather than dealing with packages individually.
- Two types of package groups: _environment groups_ and _package groups_. 

**environment groups** available in RHEL 9:
- server, server with GUI, minimal install, workstation, virtualization host, and custom operating system. 
- Listed on the software selection window during RHEL 9 installation. 

**Package groups** include:
- container management, smart card support, security tools, system tools, network servers, etc.

## Individual packages, package groups, and modules:

### Individual Package Management

List, install, query, and remove packages.

#### Listing Available and Installed Packages

 - dnf lists available packages as well as installed packages.
### Lab: list all packages available for installation from all enabled repos,
```bash
 sudo dnf repoquery
```
### Lab: list of packages that are available only from a specific repo:

```bash
 sudo dnf repoquery --repo "BaseOS"
```
### Lab: grep for an expression to narrow down your search. 
For example, to find whether the BaseOS repo includes the *zsh* package.

```bash
 sudo dnf repoquery --repo BaseOS | grep zsh
```
### Lab: list all installed packages on the system:
```bash
 sudo dnf list installed
```

Three columns: 
	- package name
	- package version
	- repo it was installed from. 
	- \@anaconda means the package was installed at the time of RHEL installation.

List all installed packages and all packages available for installation from all enabled repositories:
```bash
 sudo dnf list
```

- @ sign identifies the package as installed.

List all packages available from all enabled repositories that should be able to update:
```bash
 sudo dnf list updates
```

List whether a package (*bc*, for instance) is installed or available for installation from any enabled repository:
```bash
 sudo dnf list bc
```

List all installed packages whose names begin with the string "gnome" followed by any number of characters:
```bash
 sudo dnf list installed ^gnome*
```

List recently added packages:
```bash
 sudo dnf list recent
```

Refer to the *repoquery* and *list* subsections of the *dnf* command manual pages for more options and examples.

### Installing and Updating Packages

Installing a package:
- creates the necessary directory structure
- installs the required files
- runs any post-installation steps. 
- If already installed, *dnf* command updates it to the latest available version.

Attempt to install a package called *ypbind*, proceed to update if it detects the presence of an older version:
```bash
 sudo dnf install ypbind
```

Install or update a package called *dcraw* located locally at /mnt/AppStream/Packages/
```bash
 sudo dnf localinstall /mnt/AppStream/Packages/dcraw*
```

Update an installed package (*autofs*, for example) to the latest available version. Dnf will fail if the specified package is not already installed:
```bash
 sudo dnf update autofs
```

Update all installed packages to the latest available versions:
```bash
 sudo dnf -y update
```

Refer to the *install* and *update* subsections of the *dnf* command manual pages for more options and examples.

### Exhibiting Package Information
Show:
- release
- size
- whether it is installed or available for installation
- repo name it was installed or is available from
- short and long descriptions
- license
- so on

dnf info subcommand

View information about a package called *autofs*:
```bash
 dnf info autofs
```

- Determines whether the specified package is installed or
not.

Refer to the *info* subsection of the *dnf* command manual pages.

### Removing Packages

Removing a package:
- uninstalls it and removes all associated files and directory structure. 
- erases any dependencies as part of the deletion process. 

Remove a package called *ypbind*:
```bash
 sudo dnf remove ypbind
```

Output 
- Resolved dependencies
- List of the packages that it would remove. 
- Disk space that their removal would free up. 
- After confirmation, it erased the identified packages and verified their removal. 
- List of the removed packages

Refer to the *remove* subsection of the *dnf* command manual pages for more options and examples available for removing packages.

### Lab: Manipulate Individual Packages

Perform management operations on a package called `cifs-utils`. Determine if this package is already installed and if it is available for installation. Display its information before installing it. Install the package and exhibit its information. Erase the package along with
its dependencies and confirm the removal.

1. Check whether the *cifs-utils* package is already installed:
```bash
 dnf list installed | grep cifs-utils
```

2. Determine if the *cifs-utils* package is available for installation:
```bash
 dnf repoquery cifs-utils
```

3. Display detailed information about the package:
```bash
 dnf info cifs-utils
```

4. Install the package:
```bash
 dnf install -y cifs-utils
```

5. Display the package information again:
```bash
 dnf info cifs-utils
```

6. Remove the package:
```bash
 dnf remove -y cifs-utils
```

7. Confirm the removal:
```bash
 dnf list installed | grep cif
```

### Determining Provider and Searching Package Metadata

- You can determine what package a specific file belongs to or which package comprises a certain string.

Search for packages that contain a specific file such as /etc/passwd/, use the *provides* or the *whatprovides* subcommand with dnf:
```bash
 dnf provides /etc/passwd
```

- Indicates file is part of a package called *setup*, installed during RHEL installation.
- Second instance, setup package is part of the BaseOS repository.

- Can also use a wildcard character for filename expansion.

List all packages that contain filenames beginning with "system-config" followed by any number of characters:
```bash
 dnf whatprovides /usr/bin/system-config*
```

To search for all the packages that match the specified string in their
name or summary:
```bash
 dnf search system-config
```


### Package Group Management

- `group` subcommand 
- list, install, query, and remove groups of packages.

### Listing Available and Installed Package Groups

`group list` subcommand:
- list the package groups available for installation from either or both repos
- list the package groups that are already installed on the system.

List all available and installed package groups from all repositories:
```bash
 dnf group list
```

output:
- two categories of package groups: 
	- Environment group
	- Package groups
	
Environment group:
- Larger collection of RHEL packages that provides all necessary software to build the operating system foundation for a desired purpose.

Package group
- Small bunch of RHEL packages that serve a common purpose. 
- Saves time on the deployment of individual and dependent packages. 
- Output shows installed and available package groups.

Display the number of installed and available package groups:
```bash
 sudo dnf group summary
```

List all installed and available package groups including those that
are hidden:
```bash
 sudo dnf group list hidden
```

Try *group list* with \--installed and \--available options to narrow
down the output list.
```bash
 sudo dnf group list --installed
```

List all packages that a specific package group such as Base contains:
```bash
 sudo dnf group info Base
```

-v option with the group info subcommand for more
information. 

Review group list and group info subsections of the `dnf` man pages.

### Installing and Updating Package Groups

- Creates the necessary directory structure for all the packages included in the group and all dependent packages.
- Installs the required files.
- Runs any post-installation steps.
- Attempts to update all the packages included in the group to the latest available versions. 

Install a package group called Emacs. Update if it detects an older version.
```bash
 sudo dnf -y groupinstall emacs
```

Update the *smart card support* package group to the latest version:
```bash
 dnf groupupdate "Smart Card Support"
```

Refer to the *group install* and *group update* subsections of the *dnf* command manual pages for more details.

### Removing Package Groups
- Uninstalls all the included packages and deletes all associated files and directory structure.
- Erases any dependencies

Erase the *smart card support* package group that was installed:

```bash
 sudo dnf -y groupremove 'smart card support'
```

Refer to the *remove* subsection of the *dnf* command manual pages for
more details.

### Lab: Manipulate Package Groups
Perform management operations on a package group called *system tools*. Determine if this group is already installed and if it is available for installation. List the packages it contains and install it. Remove the group along with its dependencies and confirm the removal.

1. Check whether the *system tools* package group is already installed:
```bash
 dnf group list installed
```

2. Determine if the *system tools* group is available for installation:
```bash
 dnf group list available
```

The group name is exhibited at the bottom of the list under the
available groups.

3. Display the list of packages this group contains:
```bash
 dnf group info 'system tools'
```

- All of the packages will be installed as part of the group installation.

4. Install the group:
```bash
 sudo dnf group install 'system tools'
```

5. Remove the group:
```bash
 sudo dnf group remove 'system tools' -y
```

6. Confirm the removal:
```bash
 dnf group list installed
```

