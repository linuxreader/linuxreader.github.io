## The dnf Command

- introduced in RHEL 8 
- can use interchangeably with yum in RHEL
	 - yum is a soft link to the *dnf* utility. 
- requires the system to have access to either:
	- a local or remote software repository
	- a local installable package file. 

Subscription Management* (RHSM) service 
- available in the Red Hat Customer Portal 
- offers access to official Red Hat software repositories.

- Other web-based repositories that host packages are available 
- You can also set up a local, custom repository on your system and add packages of your choice to it.

Primary benefit of using `dnf` over `rpm`: 
- Resolve dependencies automatically 
	- by identifying and installing any additional required packages 
-  With multiple repositories set up, `dnf` extracts the software from wherever it finds it.

- Perform abundant software administration tasks. 
- Invokes the `rpm` utility in the background 
- Can perform a number of operations on individual packages, package groups, and modules:
	- listing
	- querying
	- installing
	- removing
	- enabling and disabling specific module streams.

Software handling tasks that dnf can perform on packages: 
- clean and repolist are specific to repositories. 
- Refer to the manual pages of *dnf* for additional subcommands, operators, options, examples, and other details.

| **Subcommand** | **Description**                                                      |
| -------------- | -------------------------------------------------------------------- |
| check-update   | Checks if updates are available for installed packages               |
| clean          | Removes cached data                                                  |
| history        | Display previous dnf activities as recorded in /var/lib/dnf/history/ |
| info           | Show details for a package                                           |
| install        | Install or update a package                                          |
| list           | List installed and available packages                                |
| provides       | Search for packages that contain the specified file or feature       |
| reinstall      | Reinstall the exact version of an installed package                  |
| remove         | Remove a package and its dependencies                                |
| repolist       | List enabled repositories                                            |
| repoquery      | Runs queries on available packages                                   |
| search         | Searches package metadata for the specified string                   |
| upgrade        | Updates each installed package to the latest version                 |

dnf subcommands that are intended for operations on package
groups and modules:

| **Subcommand** | **Description**                                                         |
| -------------- | ----------------------------------------------------------------------- |
| group install  | Install or updates a package group                                      |
| group info     | Return details for a package group                                      |
| group list     | List available package groups                                           |
| group remove   | Remove a package group                                                  |
| module disable | Disable a module along with all the streams it contains                 |
| module enable  | Enable a module along with all the streams it contains                  |
| module install | Install a module profile including its packages                         |
| module info    | Show details for a module                                               |
| module list    | Lists all available module streams along with their profiles and status |
| module remove  | Removes a module profile including its packages                         |
| module reset   | Resets a module so that it is neither in enable nor in disable state    |
| module update  | Updates packages in a module profile                                    |

For labs, you'll need to create a definition file and configure access to the two repositories available on the RHEL 8 ISO
image.

### Lab: Configure Access to Pre-Built Repositories

Set up access to the two dnf repositories that are available on RHEL 9 image. (You should have already configured an automatic mounting of RHEL 9 image on */mnt*.) Create a definition file for the repositories and confirm.

1. Verify that the image is currently mounted:

`df -h | grep mnt`

2. Create a definition file called local.repo in /etc/yum.repos.d/ using the *vim* editor and define the following data for both repositories in it:

```bash
[BaseOS] 
name=BaseOS 
baseurl=file:///mnt/BaseOS 
gpgcheck=0 

[AppStream] 
name=AppStream 
baseurl=file:///mnt/AppStream 
gpgcheck=0
```

3. Confirm access to the repositories:

`sudo dnf repolist`

- Ignore lines 1-4 in the output that are related to subscription and
system registration. 
- Lines 5 and 6 show the rate at which the command read the repo data. 
- Line 7 displays the timestamp of the last metadata check.
- last two lines show the repo IDs, repo names, and a count of packages they hold.
- AppStream repo consists of 4,672 packages
- BaseOS repo contains 1,658 packages. 
- Both repos are enabled by default and are ready for use.

### dnf yum Repository

**dnf repository** (*yum repository* or a *repo*)
- Digital library for storing software packages
- Repository is accessed for package retrieval, query, update, and installation
- The two repositories
	- BaseOS and AppStream
		- come preconfigured with the RHEL 9 ISO image.
- Number of other repositories available on the Internet that are maintained by software publishers such as Red Hat and CentOS. 
- Can build private custom repositories for internal IT use for stocking and delivering software.
	- Good practice for an organization with a large Linux server base, as it manages dependencies automatically and aids in maintaining software consistency across the board.
- Can also be used to store in-house developed packages.

- It is important to obtain software packages from authentic and reliable sources such as Red Hat to prevent potential damage to your system and
to circumvent possible software corruption.

- There is a process to create repositories and to access preconfigured repositories. 
- There are two pre-set repositories available on the RHEL 9 image. You will configure access to them via a definition file to support the exercises and lab environment.

### Repository Definition File
- Repo definition files are located in /etc/yum.repos.d/
- Can create local.repo file in this directory to specify local repos
- See dnf.conf man page

Sample repo definition file and key directives:

```bash
[BaseOS_RHEL_9]
name= RHEL 9 base operating system components
baseurl=file://*mnt*BaseOS
enabled=1
gpgcheck=0
```


**EXAM TIP:** 
- Knowing how to configure a dnf/yum repository using a URL plays an important role in completing some of the RHCSA exam tasks successfully. 
- Use two forward slash characters (//) with the baseurl directive for an FTP, HTTP, or HTTPS source.

Five lines from a sample repo file:
Line 1 defines an exclusive ID within the square brackets. 
Line 2 is a brief description of the repo with the "name" directive. 
Line 3 is the location of the repodata directory with the "baseurl" directive. 
Line 4 shows whether this repository is active. 
Line 5 shows if packages are to be GPGchecked for authenticity.

- Each repository definition file must have:
	- Unique ID
	- Description
	- Baseurl directive defined 
	- Other directives are set as required. 
	
- The baseurl directive for a local directory path is defined as file:///local_path
	- The first two forward slash characters represent the URL convention, and the third forward slash is for the absolute path to the destination directory)
	- FTP and 
		- \ftp://hostname/network_path
	- HTTP(S) 
		- http(s)://hostname/network_path
	- network path must include a resolvable hostname or an IP address.

### Software Management with dnf

- Tools are available to work with individual packages as well as package groups and modules. 
- `rpm` command is limited to managing one package at a time.
- `dnf` has an associated configuration file that can define settings to control its behavior.

#### dnf Configuration File

- Key configuration file: /etc/dnf/dnf.conf
- "main" section
		- sets directives that have a global effect on dnf operations.
- Can define separate sections for each custom repository that you plan to set up on the system.
- Preferred location to store configuration for each custom repository in their own definition files is in /etc/yum.repos.d
	- default location created for this purpose.
	
Default content of this configuration file:
```bash
cat /etc/dnf/dnf.conf
```

```bash
[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
best=True
skip_if_unavailable=False
```

The above and a few other directives that you may define in the file:

| Directive                    | Description                                                                                                         |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| best                         | Whether to install (or upgrade to) the latest available version.                                                    |
| clean_requirements_on_remove | Whether to remove dependencies during a package removal process that are no longer in use.                          |
| debuglevel                   | Sets debug from 1 (minimum) and 10 (maximum). Default is 2. A value of 0 disables this feature.                     |
| gpgcheck                     | Whether to check the GPG signature for package authenticity. Default is 1 (enabled).                                |
| installonly_limit            | Count of packages that can be installed concurrently. Default is 3.                                                 |
| keepcache                    | Defines whether to store the package and header cache following a successful installation. Default is 0 (disabled). |
| logdir                       | Sets the directory location to store the log files. Default is **var*log.*                                          |
| obsoletes                    | Checks and removes any obsolete dependent packages during installs and updates. Default is 1 (enabled).             |
  ------------------------------ ------------------------------------------------------------------------------------------------------------------------------------------------------------


For other directives: `man 5 dnf.conf`




