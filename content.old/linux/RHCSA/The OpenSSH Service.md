# The OpenSSH Service 

**Secure Shell (SSH)** 
- Delivers a secure mechanism for data transmission between source and destination systems over IP networks. 
- Designed to replace the old remote login programs that transmitted user passwords in clear text and data unencrypted. 
- Employs digital signatures for user authentication with encryption to secure a communication channel. 
	- this makes it extremely hard for unauthorized people to gain access to passwords or the data in transit. 
- Monitors the data being transferred throughout a session to ensure integrity. 
- Includes a set of utilities `ssh` and `sftp` for remote users to log in, transfer files, and execute commands securely.

### Common Encryption Techniques

- Two common techniques: symmetric and asymetric

**Symmetric Technique**

- Secret key encryption.
- Uses a single key called a secret key that is generated as a result of a negotiation process between two entities at the time of their initial contact. 
- Both sides use the same secret key during subsequent communication for data encryption and decryption.

**Asymmetric Technique**

- Public key encryption
- Combination of private and public keys
	- Randomly generated and mathematically related strings of alphanumeric characters 
	- attached to messages being exchanged. 
- The client transmutes the information with a public key and the server decrypts it with the paired private key.
- Private key must be kept secure since it is private to a single sender
- the public key is disseminated to clients.
- used for channel encryption and user authentication.

### Authentication Methods 

- Encrypted channel is established between the client and server
- Then additional negotiations take place between the two to authenticate the user trying to access the server. 
- Methods listed in the order in which they are attempted during the authentication process:

1. GSSAPI-based ( Generic Security Service Application Program Interface) authentication
2. Host-based authentication
3. Public key-based authentication
4. Challenge-response authentication
5. Password-based authentication

**GSSAPI-Based Authentication** 

- Provides a standard interface that allows security mechanisms, such as Kerberos, to be plugged in. 
- OpenSSH uses this interface and the underlying Kerberos for authentication. 
- Exchange of tokens takes place between the client and server to validate user identity.

**Host-Based Authentication** 

- Allows a single user, a group of users, or all users on the client to be authenticated on the server. 
- A user may be configured to log in with a matching username on the server or as a different user that already exists there. 
- For each user that requires an automatic entry on the server, a \~/.shosts file is set up containing the client name or IP address, and, optionally, a different username.
- The same rule applies to a group of users or all users on the client that require access to the server. 
	- In that case, the setup is done in the /etc/ssh/shosts.equiv file on the server.

**Private/Public Key-Based Authentication** 

- Uses a private/public key combination for user authentication.
- User on the client has a private key and the server stores the corresponding public key. 
- At the login attempt, the server prompts the user to enter the passphrase associated with the key and logs the user in if the passphrase and key are validated.

**Challenge-Response Authentication** 

- Based on the response(s) to one or more arbitrary challenge questions that the user has to answer correctly in order to be allowed to log in to the server.

**Password-Based Authentication** 

- Last fall back option. 
- Server prompts the user to enter their password. 
- Checks the password against the stored entry in the shadow file and allows the user in if the password is confirmed.

### OpenSSH Protocol Version and Algorithms 

- V2
- Supports various algorithms for data encryption and user authentication (digital signatures) such as:

**RSA** (Rivest-Shamir-Adleman)
- More prevalent than the rest
- Supports both encryption and authentication.

**DSA and ECDSA** (Digital Signature Algorithm and Elliptic Curve Digital Signature Algorithm)
- Authentication only.
- Used to generate public and private key pairs for the asymmetric technique.

### OpenSSH Packages 

- Installed during OS installation

**openssh**
- provides the `ssh-keygen` command and some library routines

**openssh-clients**
- includes commands, such as `sftp`, `ssh`, and `ssh-copy-id`, and a client configuration file */etc/ssh/ssh_config*

**openssh-server**
-  contains the sshd service daemon, server configuration file */etc/ssh/sshd_config*, and library routines.

### OpenSSH Server Daemon and Client Commands 

- OpenSSH server program is sshd

**sshd**
- Preconfigured and operational on new RHEL installations
- Allows remote users to log in to the system using an ssh client program such as PuTTY or the ssh command. 
- Daemon listens on TCP port 22 
	- Documented in the /etc/ssh/sshd_config file with the Port directive.

- Use sftp instead of scp do to scp security flaws. 

`sftp`                              
- Secure remote file transfer program

`ssh`                         
 - Secure remote login command

`ssh-copy-id`                       
  - Copies public key to remote systems

`ssh-keygen`                          
  - Generates and manages private and public key pairs

### Server Configuration File 

**/etc/ssh/sshd_config**

**/var/log/secure** 
- log file is used to capture authentication messages.

View directives listed in */etc/ssh/sshd_config*:
```bash
[root@server30 tmp]# cat /etc/ssh/sshd_config
```


**Port**                                
  - Port number to listen on. Default is 22.

**Protocol**                            
  - Default protocol version to use.

**ListenAddress**                       
  - Sets the local addresses the sshd service should listen on. 
  - Default is to listen on all local addresses.

**SyslogFacility**                      
  - Defines the facility code to be used when logging messages to the /var/log/secure file. This is based on the configuration in the /etc/rsyslog.conf file. Default is **AUTHPRIV.**

**LogLevel**                            
  Identifies the level of criticality for the messages to be logged. Default is **INFO**.

**PermitRootLogin**                     
  Allows or disallows the root user to log in directly to the system. Default is yes.

**PubKeyAuthentication**                
  Enables or disables public key-based authentication. Default is yes.

 **AuthorizedKeysFile**                  
  Sets the name and location of the file containing a user's authorized keys. Default is \~/.ssh/authorized_keys.

**PasswordAuthentication**              
  Enables or disables local password authentication. Default is yes.

**PermitEmptyPasswords**                
  Allows or disallows the use of null passwords. Default is no.

**ChallengeResponseAuthentication**     
  Enables or disables challenge-response authentication mechanism. Default is yes.

**UsePAM**                              
  Enables or disables user authentication via PAM. If enabled, only root will be able to run the sshd daemon. Default is yes.

**X11Forwarding**                       
  Allows or disallows remote access to graphical applications. Default is yes.

### Client Configuration File 

**/etc/ssh/ssh_config**
- Local configuration file that directs how the client should behave. This file, , is located in the /etc/ssh directory.
- Directives preset in this file that affect all outbound ssh communication.

View the default directive settings:
`[root@server30 tmp]# cat /etc/ssh/sshd_config`

**Host**                                
  - Container that declares directives applicable to one host, a group of hosts, or all hosts. 
  - Ends when another occurrence of Host or Match is encountered. Default is \*,  (all hosts)

**ForwardX11**                          
  - Enables or disables automatic redirection of X11 traffic over SSH connections. 
  - Default is no.

**PasswordAuthentication**              
  - Allows or disallows password authentication. 
  - Default is yes.

**StrictHostKeyChecking**               
  - Whether to add host keys (host fingerprints) to  \~/.ssh/known_hosts when accessing a host for the first time
  - What to do when the keys of a previously accessed host mismatch with what is stored in \~/.ssh/known_hosts. 
- no: 
	 - Adds new host keys and ignores changes to existing keys.

- yes: 
	- Adds new host keys and disallows connections to hosts with non-matching keys. 

- accept-new: 
	- Adds new host keys and disallows connections to hosts with non-matching keys. 
	
- ask (default): 
	- Prompts whether to add new host keys and disallows connections to hosts with non-matching keys.

**IdentityFile**                        
  - Defines the name and location of a file that stores a user's private key for their identity validation. 
  - Defaults are:
	- id_rsa, id_dsa, and  id_ecdsa based on the type of algorithm used.
	- Corresponding public key files with .pub extension are also stored at the same directory location.

**Port**                                
  Sets the port number to listen on. Default is 22.

**Protocol**                            
  Specifies the default protocol version to use

\~/.ssh/
- does not exist by default
- created when:
	- a user executes the `ssh-keygen` command for the first time to generate a key pair
	- A user connects to a remote ssh server and accepts its host key for the first time.
		- The client stores the server's host key locally in a file called known_hosts along with its hostname or IP address. 
		- On subsequent access attempts, the client will use this information to verify the server's authenticity.
