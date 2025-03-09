## Application Streams and Modules

**Application Streams**
- Introduced in RHEL 8. 
- Employs a modular approach to organize multiple versions of a software application alongside its dependencies to be available for installation from a single repository. 
**module**
- logical set of application packages that includes everything required to install it, including the executables, libraries, documentation, tools, and utilities as well as dependent components. 
- Modularity gives the flexibility to choose the version of software based on need.

* In older RHEL releases, each version of a package would have to come from a separate repository. (This has changed in RHEL 8.)
* Now modules of a single application with different versions can be stored and made available for installation from a common repository. 
* The package management tool has also been enhanced to manipulate modules.

- RHEL 9 is shipped with two core repositories called _BaseOS_ and _Application Stream_ (AppStream).

**BaseOS** repository
- Includes the core set of RHEL 9 components
- kernel, modules, bootloader, and other foundational software packages.
- Lays the foundation to install and run software applications and programs. 
- Available in the traditional rpm format.

**AppStream** repository
- Comes standard with core applications,
- Plus several add-on applications
- Rpm and modular format
- Include web server software, development languages, database software, etc.

#### Benefits of Segregation

Why separate BaseOS components from other applications?

(1) Separates application components from the core operating system elements.  
(2) Allows publishers to deliver and administrators to apply application updates more frequently. 

In previous RHEL versions, an OS update would update all installed components including the kernel, service, and application
components to the latest versions by default. 

This could result in an unstable system or a misbehaving application due to an unwanted upgrade of one or more packages. 

By detaching the base OS components from the applications, either of the two can be updated independent of the other.

This provides enhanced flexibility in tailoring the system components and application workloads without impacting the underlying stability of the system.

### Module Streams

- Collection of packages organized by version
- Each module can have multiple streams
- Each stream receives updates independent of the other streams
- Stream can be enabled or disabled.
**enabled** stream
- Allows the packages it contains to be queried or installed
- Only one stream of a specific module can be enabled at a time
- Each module has a default stream, which provides the latest or the recommended version.i

### Module Profiles

- List of recommended packages organized for purpose-built, convenient deployments to support a variety of use cases
such as:
- Minimal, development, common, client, server, etc. 
- A profile may also include packages from the BaseOS repository or the dependencies of the stream
- Each module stream can have zero, one, or more profiles associated with it with only one of them marked as the default

### Module Management

**Modules** are special package groups usually representing an application, a language runtime, or a set of tools. They are available in one or **multiple streams** which usually represent a major version of a piece of software, giThey are available in one or **multiple streams** whichving you an option to choose what versions of packages you want to consume.
https://docs.fedoraproject.org/en-US/modularity/using-modules/

Modules are a way to deliver different versions of software (such as programming languages, databases, or web servers) independently of the base operating system's release cycle.

Each module can contain multiple streams, representing different versions or configurations of the software. For example, a module for Python might have streams for Python 2 and Python 3.

 *module* dnf subcommand 
 - list, enable, install, query, remove, and disable
modules.

### Listing Available and Installed Modules

List all modules along with their stream, profile, and summary
information available from all configured repos:
`dnf module list`

Limit the output to a list of modules available from a specific
repo such as AppStream by adding \--repo AppStream:
`dnf module lisy --repo AppStream`

Output: 
- default (d)
- enabled (e)
- disabled (x)
- installed (i)

List all the streams for a specific module such as `ruby` and display their status:
`dnf module list ruby`

Modify the above and list only the specified stream 3.3 for the module `ruby`
`dnf module list ruby:3.3

List all enabled module streams:
`dnf module list --enabled`

Similarly, you can use the \--installed and \--disabled options with
**dnf module list** to output only the installed or the disabled
streams.

Refer to the *module list* subsection of the *dnf* command manual pages.

### Installing and Updating Modules

Installing a module 
- creates directory tree for all packages included in the module and all dependent packages
- installs required files for the selected profile
- runs any post-installation steps. 
- If module being loaded or a part of it is already present, the command attempts to update all the packages included in the profile to the latest available versions. 

Install the *perl* module using its default stream and default
profile:
`sudo dnf -y module install perl`

Update a module called *squid* to the latest version:
`sudo dnf module update squid -y`

Install the profile "common" with stream "rhel9" for the *container-tools* module: (module:stream/profile)
`sudo dnf module install container-tools:rhel9/common`

### Displaying Module Information

- Shows 
	- Name, stream, version, list of profiles, default profile, repo name module was installed or is available from
	- Summary, description, and artifacts. 
- Can be viewed by supplying module info with dnf. 

List all profiles available for the module ruby:
`dnf module info --profile ruby`

Limit the output to a particular stream such as 3.1:
`dnf module info --profile ruby:3.1`

Refer to the *module info* subsection of the *dnf* command manual pages for more details.

### Removing Modules

Removing a module will:
- uninstall all the included packages and 
- delete all associated files and directory structure. 
- erases any dependencies as part of the deletion process.

Remove the ruby module with "3.1" stream:
`sudo dnf module remove ruby:3.1

Refer to the *module remove* subsection of the *dnf* command manual pages: 

### Lab: Manipulate Modules

- Perform management operations on a module called *postgresql*. 
- determine if this module is already installed and if it is available for installation
- show its information and install the default profile for stream "10".
- remove the module profile along with any dependencies
- confirm the removal.

1. Check whether the *postgresql* module is already installed(i):
`dnf module list postgresql`

2. Display detailed information about the default stream of the module:
`dnf module info postgresql:15`

3. Install the module with default profile for stream "15":
`sudo dnf -y module install --profile postgresql:15`

4. Display the module information again:
`dnf module info postgresql:15`

5. Erase the module profile for the stream:
`dnf module remove -y postgresql:15`

6. Confirm the removal (back to (d)):
`dnf module info postgresql:15`

### Switching Module Streams

- Typically performed to upgrade or downgrade the version of an installed module. 

process:
- uninstall the existing version provided by a stream alongside any dependencies that it has, 
- switch to the other stream
- install the desired version. 
- Installing a module from a stream automatically enables the stream if it was previously disabled
- you can manually enable or disable it with the *dnf* command.
- Only one stream of a given module enabled at a time.
- Attempting to enable another one for the same module automatically
disables the current enabled stream.

- dnf module list and dnf module info expose the enable/disable status of the module stream. 

### Lab: Install a Module from an Alternative Stream

- Downgrade a module to a lower version. 
- Remove the stream `ruby` 3.3 and 
- Confirm its removal. 
- manually enable the stream *perl* 5.24 and confirm its new status. 
- install the new version of the module and display its information.

1. Check the current state of all *perl* streams:
`dnf module list perl`

2. Remove perl 5.26:
`sudo dnf module remove perl -y`

3. Confirm the removal:
`dnf module list ruby`

4. Reset the module so that neither stream is enabled or disabled. This will remove the enabled (e) indication from ruby 3.3
`sudo dnf module reset ruby`

5. Install the non-default profile "minimal" for ruby stream 3.1. This will auto-enable the stream.

--allowerasing 
- Will instruct the command to remove installed packages for dependency resolution.
`sudo dnf module install ruby:3.1 --allowerasing`

6. Check the status of the module:
`dnf module list perl`

