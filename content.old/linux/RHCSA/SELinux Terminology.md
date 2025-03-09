## SELinux Terminology 

- Implementation of the Mandatory Access Control (MAC) architecture
	- developed by the U.S. National Security Agency (NSA)
	- flexible, enriched, and granular security controls in Linux. 
	- integrated into the Linux kernel as a set of patches using the Linux Security Modules (LSM) framework
		- allows the kernel to support various security implementations
	- provides an added layer of protection above and beyond the standard Linux Discretionary Access Control (DAC) security architecture. 
	- DAC includes:
		- traditional file and directory permissions
		- extended attribute settings
		- setuid/setgid bits
		- su/sudo privileges
		- etc.
	- Limits the ability of a **subject** (Linux user or process) to access an **object** (file, directory, file system, device, network interface/connection, port, pipe, socket, etc.) 
		- To reduce or eliminate the potential damage the subject may be able to inflict on the system if compromised.

- **MAC controls**
	- Fine-grained
	- Protect other services in the event one service is negotiated. 
	- Example:
		- If the HTTP service process is compromised, the attacker can only damage the files the hacked process will have access to, and not the other processes running on the system, or the objects the other processes will have access to. 
	- To ensure this coarse-grained control, MAC uses a set of defined authorization rules called **policy** to examine security attributes associated with subjects and objects when a subject tries to access an object, and decides whether to permit the access attempt. 
	- These attributes are stored in **contexts** (a.k.a. **labels**), and are applied to both subjects and objects.

- SELinux decisions are stored in a special cache area called **Access**
**Vector Cache** (**AVC**). 
- This cache area is checked for each access attempt by a process to determine whether the access attempt was previously allowed. 
- With this mechanism in place, SELinux does not have to check the policy ruleset repeatedly, thus improving performance.

- SELinux is enabled by default 
	- Confines processes to the bare minimum privileges that they need to function.

### Key Terms

**Subject**
- Any user or process that accesses an object. 
- Examples:
	- *system_u* for the SELinux system user
	- *unconfined_u* for subjects that are not bound by the SELinux policy 
- Stored in field 1 of the context.

**Object** 
- A resource, such as a file, directory, hardware device, network interface/connection, port, pipe, or socket, that a subject accesses. 
- Examples:
	- *object_r* for general objects
	- *system_r* for system-owned objects
	- *unconfined_r* for objects that are not bound by the SELinux policy.

**Access** 
- An action performed by the subject on an object. 
- Examples:
	- creating, reading, or updating a file
	- creating or navigating a directory
	- accessing a network port or socket

**Policy** 
- A defined ruleset that is enforced system-wide
- Used to analyze security attributes assigned to subjects and objects. 
- Referenced to decide whether to permit a subject's access attempt to an object, or a subject's attempt to interact with another subject.
- The default behavior of SELinux in the absence of a rule is to deny the access. 
- Standard preconfigured policies:
	- **targeted** (default)
		- Any process that is targeted runs in a confined domain
		- Any process that is not targeted runs in an unconfined domain.
		- Example:
			- SELinux runs logged-in users in the unconfined domain, and the httpd process in a confined domain by default. 
			- Any subject running unconfined is more vulnerable than the one running confined.
	- **mls**
		- Places tight security controls at deeper levels.
	- **minimum**
		- light version of the targeted policy
		- designed to protect only selected processes.


**Context** (label)
- A tag to store security attributes for subjects and objects. 
- Every subject and object has a context assigned
- Consists of a SELinux user, role, type (or domain), and sensitivity level.
- Used by SELinux  to make access control decisions.

**Labeling** 
- Mapping of files with their stored contexts.

**SELinux User** 
- Several predefined SELinux user identities that are authorized for a particular set of roles. 
- Linux user to SELinux user identity mapping to place SELinux user restrictions on Linux users. 
- Controls what roles and levels a process (with a particular SELinux user identity) can enter. 
- Example:
	- A Linux user cannot run the `su` and `sudo` commands or the programs located in their home directories if they are mapped to the SELinux user *user_u*.

**Role** 
- An attribute of the **Role-Based Access Control (RBAC)** security model that is part of SELinux.
- Classifies who (subject) is allowed to access what (domains or types). 
- SELinux users are authorized for roles, and roles are authorized for domains and types. 
- Each subject has an associated role to ensure that the system and user processes are separated. 
- A subject can transition into a new role to gain access to other domains and types. 
- Examples:
	- *user_r* for ordinary users
	- *sysadm_r* for administrators
	- *system_r* for processes that initiate under the *system_r* role
- Stored in field 2 of the context.

**Type Enforcement** (TE)
- Identifies and limits a subject's ability to access domains for processes, and types for files. 
- References the contexts of the subjects and objects for this enforcement.

**Type**
- Attribute of type enforcement.
- Group of objects based on uniformity in their security requirements. 
- Objects such as files and directories with common security requirements, are grouped within a specific type. 
- Examples:
	- *user_home_dir_t* for objects located in user home directories
	- *usr_t* for most objects stored in the /usr directory. 
- Stored in field 3 of a file context.

**Domain**
- Determines the type of access that a process has. 
- Processes with common security requirements are grouped within a specific domain type, and they run confined within that domain. 
- Examples:
	- *init_t* for the systemd process
	- *firewalld_t* for the firewalld process
	- *unconfined_t* for all processes that are not bound by SELinux policy.
- Stored in field 3 of a process context.

**Rules** 
- Outline how types can access each other, domains can access types, and domains can access each other.

**Level** 
- An attribute of **Multi-Level Security (MLS)** and **Multi-Category** **Security (MCS)**. 
- Pair of *sensitivity:category* values that defines the level of security in the context. 
- *category* may be defined as a single value or a range of values, such as c0.c4 to represent c0 through c4. 
- In RHEL 9, the targeted policy is used as the default, which enforces **MCS** (MCS supports only one sensitivity level (s0) with 0 to 1023 different categories).

 
