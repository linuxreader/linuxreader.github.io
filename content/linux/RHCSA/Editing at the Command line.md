## Editing at the Command line

### Common key combinations:
- **Ctrl+a / Home** 
	- Moves the cursor to the beginning of the command line 
- **Ctrl+e / End** 
	- Moves the cursor to the end of the command line 
- **Ctrl+u** 
	- Erase everything at and before cursor
- **Ctrl+k** 
	- Erase Everything at cursor and after
- **Alt+f** 
	- Moves the cursor to the right one word at a time 
- **Alt+b** 
	- Moves the cursor to the left one word at a time 
- **Ctrl+f / Right arrow** 
	- Moves the cursor to the right one character at a time
- **Ctrl+b / Left arrow** 
	- Moves the cursor to the left one character at a time

### Tab completion
- Hitting the Tab key twice automatically completes the entire name.
- In case of multiple possibilities matching the entered characters, it completes up to the point they have in common and prints the rest of the possibilities on the screen.

### Tilde Substitution (tilde expansion)
- Performed on words that begin with the tilde character (~).
~ 
	- refers to user's home directory.

~+
	- refers to current directory

~-
	- Refers to previous working directory.

~USER
	- Refers to specific user's home directory.

### Alias Substitution (command aliasing or alias)
- Define shortcuts for commands.
- Bash shell includes several predefined aliases that are set during user login.
- Shell gives precedent to an alias if an alias matches a command or program.
	- can still run the command without using the alias but preceding command with a backslash. \

### alias command
- Set an alias.
- Internal shell command.

```bash
[root@localhost ~]# alias
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias xzegrep='xzegrep --color=auto'
alias xzfgrep='xzfgrep --color=auto'
alias xzgrep='xzgrep --color=auto'
alias zegrep='zegrep --color=auto'
alias zfgrep='zfgrep --color=auto'
alias zgrep='zgrep --color=auto'

```

```
[root@localhost ~]# alias frog='pwd'
[root@localhost ~]# frog
/root
```

### unalias command
- Unset an alias.
- Internal shell command.
```bash
[root@localhost ~]# unalias frog
[root@localhost ~]# frog
-bash: frog: command not found
```
