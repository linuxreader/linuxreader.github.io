# Shell Parameters 

- An entity that holds a value such as a name, special character, or number. 
- The parameter that holds a **name** is referred to as a **variable**
- a **special character** is referred to as a **special parameter**
	- represents the command or script itself (\$0), count of supplied arguments (\$\* or \$@), all arguments (\$#), and PID of the process (\$\$)
- one or more digits, except for 0 is referred to as a **positional parameter** (a command line argument). 
	- (\$1, \$2, \$3 . . .) is an argument supplied to a script at the time of its invocation
	- position is determined by the shell based on its location with respect to the calling script.
	- Positional parameters beyond 9 are to be enclosed in curly brackets. 


![](image-G1VF6C4I%201.jpg)

- Just like the variable and command substitutions, the shell uses the dollar (\$) sign for special and positional parameter expansions as well.

### Script05: Using Special and Positional Parameters

Create *com_line_arg.sh* to show the supplied arguments, total count, value of the first argument, and PID of the script:
```bash
[root@server30 ~]# vim /usr/local/bin/com_line_arg.sh

#!/bin/bash
echo "There are $# arguments specified at the command line"
echo "The arguments supplied are: $*"
echo "The first argument is: $1"
echo "The Process ID of the script is: $$"                                          
```

```bash
[root@server30 ~]# chmod +x /usr/local/bin/com_line_arg.sh

[root@server30 ~]# com_line_arg.sh
There are 0 arguments specified at the command line
The arguments supplied are: 
The first argument is: 
The Process ID of the script is: 1935

[root@server30 ~]# com_line_arg.sh the dog jumped over the frog
There are 6 arguments specified at the command line
The arguments supplied are: the dog jumped over the frog
The first argument is: the
The Process ID of the script is: 1936
```


### Script06: Shifting Command Line Arguments

`shift` command
- Used to move arguments one position to the left. 
- During this move, the value of the first argument is lost. 

```bash
[root@server30 ~]# vim /usr/local/bin/com_line_arg_shift.sh

#!/bin/bash
echo "There are $# arguments specified at the command line"
echo "The arguments supplied are: $*"
echo "The first argument is: $1"
echo "The Process ID of the script is: $$"
shift
echo "The new first argument after the first shift is: $1"
shift
echo "The new first argument after the second shift is: $1"
```

```bash
[root@server30 ~]# chmod +x /usr/local/bin/com_line_arg_shift.sh

[root@server30 ~]# com_line_arg_shift.sh
There are 0 arguments specified at the command line
The arguments supplied are: 
The first argument is: 
The Process ID of the script is: 1941
The new first argument after the first shift is: 
The new first argument after the second shift is: 

[root@server30 ~]# com_line_arg_shift.sh the dog jumped over the frog
There are 6 arguments specified at the command line
The arguments supplied are: the dog jumped over the frog
The first argument is: the
The Process ID of the script is: 1942
The new first argument after the first shift is: dog
The new first argument after the second shift is: jumped
```

- Multiple shifts in a single attempt may be performed by furnishing a count of desired shifts to the shift command as an argument. For example, "shift 2" will carry out two shifts, "shift 3" will make three shifts, and so on.




