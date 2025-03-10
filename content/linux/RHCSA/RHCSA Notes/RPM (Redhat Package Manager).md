## RPM (Redhat Package Manager)
- Specially formatted File(s) packaged together with the .rpm extension. 
- Packages included or available for RHEL are in rpm format.
- Metadata info gets updated whenever a package is updated.

### rpm command
- Install, Upgrade, remove, query, freshen, or decompress packages.
- Validate package authenticity and integrity.

### Packages
- Two types of packages binary (or installable) and source.

**Binary packages**
- Installation ready
- Bundled for distribution.
- Have .rpm extension.
- Contain:
	- install scripts (pre and post)
	- Executables
	- Configuration files
	- Library files
	- Dependency information
	- Where to install files
	- Documentation
		- How to install/uninstall
		- Man pages for config files/commands
		- Other install and usage info
	- Metadata
		- Stored in central location
		- Includes:
			- Package version
			- Install location
			- Checksum values
			- List of included files and their attributes
- Package intelligence
	- Used by package administration toolset for successful completion of the package installation process. 
	- May include info on:
		- prerequisites
		- User account setup
		- Needed directories/ soft links
	- Includes reverse process for uninstall

### Package Naming
5 parts to a package name:
	1. Name
	2. Version
	3. release (revision or build)
	4. Linux version 	
	5. Processor Architecture
		- noarch
			- platform independant
		- src
			- Source code packages
- Always has .rpm extension
- .rpm is removed after install
Example:
	openssl-1.1.1-8.el8.x86_64.rpm,

### Package Dependency
- Dependency info is in the metadata
	- Read by package handling utilities

### Package Database
- Metadata for installed packages and package files is stored in /var/lib/rpm/
	- Package database
	- Referenced by package manipulation utilities to obtain:
		- package name and version data
		- Info about owerships, permissions, timestamps, and file sizes that are part of the package.
		- Contain info on dependencies.
		- Aids management commands in:
			- listing and querying packages
			- Verifying dependencies and file attributes.
			- Installing new packages.
			- Upgrading and uninstalling packages. 
		- Removes and replaces metadata when a package is replaced. 
		- Can maintain multiple version of a single package. 

---

# Left off here
### Package Management Tools
- rpm (redhat package manager)
	- Does not automatically resolve dependencies.
- yum (yellowdog update, modified)
	- Find, get, and install dependencies automatically.
	- softlink to dnf now.
- dnf (dandified yum)

### Package management with rpm
rpm package management tasks:
	- query
	- install
	- upgrade
	- freshen
	- overwrite
	- remove
	- extract
	- validate
	- verify
- Works with installed and installable packages.

### rpm command
###### Query options
Query and display packages\
`-q (--query)` \
\
List all installed packages\
`-qa (--query --all)`\
\
List config files in a package\
`-qc (--query --config-files)`\
\
List documentation files in a package\
`-qd (--query --docfiles)` \
\
Exhibit what package a file comes from\
`-qf (--query --file)` \
\
Show installed package info (Version, Size, Installation status, Date, Signature, Description, etc.)	
`-qi (--query --info)` 
	
Show installable package info (Version, Size, Installation status, Date, Signature, Description, etc.)
`-qip (--query --info --package)`

List all files in a package.\
`-ql (--query --list)`

List files and packages a package depends on. \
`-qR (--query --requires)`

List packages that provide the specified package or file.\
`-q --whatprovides`

List packages that require the specified package or file.\
`-q --whatrequires`

###### Package installation options
Remove a package\
`-e (--erase)`

Upgrades installed package. Or loads if not installed.\
`-U (--upgrade)`

Display detailed information\
`-v (--verbose or -vv)`

Verify integrity of a package or package files\
`-V (--verify)`

### Querying packages
Query packages in the package database or at a specified location. 

### Installing a package
- Creates directory structure needed
- Installs files
- Runs needed post installation steps
- Installing package will fail if missing dependencies.
- Error message will show missing dependencies.

### Upgrading a package
- Installs the package if previous version does not exist. (-U)
- Makes backup of effected configuration files and adds .rpmsave extension.

### Freshening a package
- Older version must exist. 
- -F option
- Will only work if a newer version of a package is available.

### Overwriting a Package
- Replaces existing files of a package with the same version.
- --replacepkgs option.
- Useful when you suspect corruption.

### Removing a Package
- Uninstalls package and associated files/ directories
- -e Option
- Checks to see if this package is a dependency for another program and fails if it is. 

### Extracting Files from an Installable Package
- `rpm2cpio` command
- -i (extract)
- -d create directory structure.
Useful for:
	- Examining package contents.
	- Replacing corrupt or lost command.
	- Replace critical configuration file to it's original state
