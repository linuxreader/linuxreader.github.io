## Shell and Environment Variables
 
 **Current shell** 
- Where a program is executed.

**Sub-shell (child shell)**
- created within a shell to run a program.

- two types of variables: local (or shell) and environment. 
	**local variable** 
	- Private to the shell in which it is created.
	- Only used by programs that are started in that shell. 
	**environment variable**
	- Inherited from the current shell to the sub-shell during the execution of a program
	- Value stored in an environment variable is accessible to the program, as well as any sub-programs that it spawns during its lifecycle. 
	- Any environment variable set in a sub-shell is lost when the sub-shell terminates.
	- `env` or the `printenv` command to view predefined environment variables.
	- Common predefined environment variables:
		- **DISPLAY** 
			- Stores the hostname or IP address for graphical terminal sessions 
		- **HISTFILE** 
			- Defines the file for storing the history of executed commands 
		- **HISTSIZE** 
			- Defines the maximum size for the HISTFILE 
		- **HOME** 
			- Sets the home directory path LOGNAME Retains the login name 
		- **MAIL** 
			- Contains the path to the user mail directory
		- **PATH** 
			- Directories to be searched when executing a command. Eliminates the need to specify the absolute path of a command to run it. 
		- **PPID** 
			- Holds the identifier number for the parent program 
		- **PS1** 
			- Defines the primary command prompt PS2 Defines the secondary command prompt 
		- **PWD** 
			- Stores the current directory location 
		- **SHELL** 
			- Holds the absolute path to the primary primary shell file
		- **TERM** 
			- Holds the terminal type value 
		- **UID** 
			- Holds the logged-in user’s UID 
		- **USER** 
			- Retains the name of the logged-in user

### Setting and unsetting variables
- `export`, `unset`, and `echo` to define and undefine environment variables
- Use uppercase for variables

### echo command
- Restricted to showing the value of a specific variable
### env command
- Displays the environment variables only.

```bash
[root@localhost ~]# env
SHELL=/bin/bash
HISTCONTROL=ignoredups
HISTSIZE=1000
HOSTNAME=localhost
PWD=/root
LOGNAME=root
XDG_SESSION_TYPE=tty
MOTD_SHOWN=pam
HOME=/root
LANG=en_US.UTF-8
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=01;37;41:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.webp=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=01;36:*.au=01;36:*.flac=01;36:*.m4a=01;36:*.mid=01;36:*.midi=01;36:*.mka=01;36:*.mp3=01;36:*.mpc=01;36:*.ogg=01;36:*.ra=01;36:*.wav=01;36:*.oga=01;36:*.opus=01;36:*.spx=01;36:*.xspf=01;36:
SSH_CONNECTION=192.168.0.233 56990 192.168.0.169 22
XDG_SESSION_CLASS=user
SELINUX_ROLE_REQUESTED=
TERM=xterm-256color
LESSOPEN=||/usr/bin/lesspipe.sh %s
USER=root
SELINUX_USE_CURRENT_RANGE=
SHLVL=1
XDG_SESSION_ID=1
XDG_RUNTIME_DIR=/run/user/0
SSH_CLIENT=192.168.0.233 56990 22
which_declare=declare -f
PATH=/root/.local/bin:/root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
SELINUX_LEVEL_REQUESTED=
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/0/bus
MAIL=/var/spool/mail/root
SSH_TTY=/dev/pts/0
BASH_FUNC_which%%=() {  ( alias;
 eval ${which_declare} ) | /usr/bin/which --tty-only --read-alias --read-functions --show-tilde --show-dot $@
}
_=/usr/bin/env
OLDPWD=/dev/vg200

```

### printenv command
- Displays the environment variables only.
### set command
- View both local and environment variables.

### Command and Variable substitutions
- PS1 Environment variable sets what the prompt looks like.
- Default value is \\u@\\h \\W\\$
	- \\u
		- logged-in user name
	- \\h
		- System hostname
	- \\W
		- Working directory
	- \\$
		- End of command prompt
- Command whose output you want assigned to a variable must be encapsulated within either backticks `hostname` or parentheses $(hostname).
