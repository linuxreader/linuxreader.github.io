## Package Integrity and Credibility
- MD5 Checksum for verifying package integrity
- GNU Privacy Guard Public Key (GNU Privacy Guard or GPG) for ensuring credibility of publisher. 
- PGP (Pretty Good Privacy) - commercial version of GPG.
- `--nosignature` 
	- Don't verify package or header signatures when reading.
- `-K` 
	- keep package files after installation
`rpmkeys` command
	- check credibility, import GPG key, and verify packages
- Redhat signs their products and updates with a GPG key. 
	- Files in installation media include public keys in the products for verification.
	- Copied to /etc/pki/rpm-gpg during OS installation. 
	**RPM-GPG-KEY-redhat-release**
		- Used for packages shipped after November 2009 and their updates.
	**RPM-GPG-KEY-redhat-beta**
		- For Beta products shipped after November 2009.
- Import the relevant GPG key and the verify the package to check the credibility of a package. 

### Viewing GPG Keys
- view with rpm command `rpm -q gpg-pubkey`
- `-i` option
	- show info about a key. 

### Verifying Package Attributes
- Compare package file attributes with originals stored in package database at the time of installation.
- `-V` option
	- compare owner, group, permission mode, size, modification time, digest, type, etc. 
	- Returns to prompt if no changes are detected
	- -v or vv for verbose
- `-Vf`
	- run the check directly on the file
- Three columns of output:
	- Column 1
		- 9 fields
			- S = Different file size.
			- M = Mode or permission or file type change.
			- 5 = MD5 Checksum does not match.
			- D = Device file and its major and minor number have changed.
			- L = File is a symlink and it's path has been altered.
			- U = Ownership has changed.
			- G = Group membership has been modified. 
			- T = Timestamp changed.
			- P = Capabilities are altered.
			- . = No modifications detected.
		- 
	- Column 2
		- File type
			- c = Configuration file
			- d = Documentation File
			- g = Ghost FIle
			- l = License file
			- r = Readme file
	- Column 3
		- Full path of file

