## Secure Shell Service Review Questions 

Q. Name the SSH client-side configuration file.
A\. /etc/ssh/ssh_config

---

Q\. What would the command `ssh server10 ls` do?
A\. The command provided will run the ls command on the specified remote ssh server without the need for the user to log in.

---

Q\. What would the command `ssh-keygen -N ""` do?
A\. The command provided will generate a passwordless ssh key pair using the default RSA algorithm.

---

Q\. What is the default location to store user SSH keys?
A\. Under the \~/.ssh directory.

---

Q\. What would the command `ssh-copy-id` do?
A\. Distribute the public key to remote systems.

---

Q\. Which two of the five authentication methods mentioned in this chapter are more prevalent?
A\. The public key-based and password-based authentication methods are more prevalent.

---

Q\. What is the use of the `ssh-keygen` command?
A\. Generate public/private key combination for use with ssh.

---

Q\. Name the default algorithm used with SSH.
A\. The default algorithm used with ssh is RSA.

---

Q\. The primary secure shell server configuration file is ssh_config. True or False?
A\. False. The primary secure shell configuration file is sshd_config.

---

Q\. Which three common algorithms are used with SSH version 2 for encryption and/or authentication?
A\. The SSH version 2 uses RSA, DSA, and ECDSA algorithms.

---

Q\. What kind of information does the \~/.ssh/known_hosts file store?
A\. The \~/.ssh/known_hosts file stores fingerprints of remote servers.

---

Q\. List the two encryption techniques described in this chapter.
A\. The two encryption techniques are symmetric (secret key) and asymmetric (public key).

---

Q\. What is the default port used by the secure shell service?
A\. The default port used by the secure shell service is 22.

---

Q\. Which log file stores authentication messages?
A\. The /var/log/secure file stores authentication messages.

---

Q\. The ssh tool provides a non-secure tunnel over a network for accessing a RHEL system. True or False?
A\. False. The ssh command provides a secure tunnel over a network.































