## History Substitution
- Command history or history expansion.
- Can disable and re-enable if required.
- Values may be altered for individual users by editing *.bashrc* or *.bash_profile* in the user's home directory.
- Three variables
	- **HISTFILE**
		- Defines the name and location of the history file to be used to store command history,
		- Default is .bash_history in the user’s home directory.
	- **HISTSIZE**
		- Size of the history buffer for the current shell.
	- **HISTFILESIZE**
		- Sets the maximum number of commands allowed for storage in the history file at the beginning of the current session and are written to the HISTFILE from memory at the end of the current terminal session.
		- Usually, HISTSIZE and HISTFILESIZE are set to a common value.

### `history` command
- Displays or reruns previously executed commands.
- Gets the history data from the system memory as well as from the .bash_history file.
- Shows all entries by default.
- `set +o history`
	- disable history expansion
- `set -o history`
	- re-enable history expansion

```bash
[root@localhost ~]# set +o history

[root@localhost ~]# history | tail 
  126  ls
  127  vim test.txt
  128  set -o noclobber
  129  echo "Hello" > test.txt
  130  set +o noclobber
  131  echo "Hello" > test.txt
  132  cat test.txt
  133  history | tail 
  134  set +0 history
  135  set +o history
  
[root@localhost ~]# vim test2.txt

[root@localhost ~]# history | tail 
  126  ls
  127  vim test.txt
  128  set -o noclobber
  129  echo "Hello" > test.txt
  130  set +o noclobber
  131  echo "Hello" > test.txt
  132  cat test.txt
  133  history | tail 
  134  set +0 history
  135  set +o history
  
[root@localhost ~]# set -o history

[root@localhost ~]# vim test2.txt

[root@localhost ~]# history | tail 
  128  set -o noclobber
  129  echo "Hello" > test.txt
  130  set +o noclobber
  131  echo "Hello" > test.txt
  132  cat test.txt
  133  history | tail 
  134  set +0 history
  135  set +o history
  136  vim test2.txt
  137  history | tail 
  ```