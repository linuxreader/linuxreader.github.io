## Logical Constructs 

- Use test conditions, which decides what to do next based on the true or false status of the condition.

The shell offers two logical constructs: 
**if-then-fi**
**case**

**Exit Codes** (exit values)
- Refer to the value returned by a command when it finishes execution. 
	- Value is based on the outcome of the command. 
- If the command runs successfully, you typically get a zero exit code(return code), otherwise you get a non-zero value. 
- Return code is stored in a special shell parameter called ? (question mark). 
 
Let's look at the following two examples to understand their usage:
```bash
[root@server30 ~]# pwd
/root
[root@server30 ~]# echo $?
0
[root@server30 ~]# man
What manual page do you want?
For example, try 'man man'.
[root@server30 ~]# echo $?
1
```

- Exit code was returned and stored in the ?. 
- Non-zero exit code was stored in ? with an error.
- Can define exit codes within a script at different locations to help debug the script by knowing where exactly it terminated.

**Test Conditions** 
- Used in logical constructs to decide what to do next. 
- Can be set on integer values, string values, or files using the `test` command or by enclosing them within the square brackets \[\].

**Operation on Integer Value**   

`integer1 -eq (-ne) integer2`         
- Integer1 is equal (not equal) to integer2

`integer1 -lt (-gt) integer2 `        
- Integer1 is less (greater) than integer2

`integer1 -le (-ge) integer2`         
- Integer1 is less (greater) than or equal to integer2
  

**Operation on String Value**

`string1=(!=)string2`                 
- Tests whether the two strings are identical (not identical)

`-l string or -z string`              
- Tests whether the string length is zero

`string or -n string`                 
- Tests whether the string length is non-zero


**Operation on File**

`-b (-c) file`                        
- Tests whether the file is a block (character) device file

`-d (-f) file`                        
- Tests whether the file is a directory (normal file)

`-e (-s) file`                        
- Tests whether the file exists (non-empty)

`-L file`                             
- Tests whether the file is a symlink

`-r (-w) (-x) file`                   
- Tests whether the file is readable (writable) (executable)

`-u (-g) (-k) file`                   
- Tests whether the file has the setuid (setgid) (sticky) bit

`file1 -nt (-ot) file2`               
- Tests whether file1 is newer (older) than file2


**Logical Operators**

`!`                                  
- The logical NOT operator

 `-a` or `&&` (two ampersand characters) 
- The logical AND operator. Both operands must be true for the condition to be true. Syntax: `[ -b file1 && -r file1 ]`

`-o ` or ` ||` (two pipe characters)    
- The logical OR operator. Either of the two or both operands must be true for the condition to be true. Syntax: `[ (x == 1 -o y == 2) ]`

### if-then-fi Construct 

- Evaluates the condition for true or false. 
- Executes the specified action if the condition is true
- otherwise, it exits the construct. 
- Begins with an `if` and ends with a `fi`
-  can execute an action only if the specified condition is true. It quits the statement if the condition is untrue.

![](image-7FX8FVMW%201.jpg)

The general syntax of this statement is as follows:

if condition > then > action > fi

### Script07: The if-then-fi Construct

Create *if_then_fi.sh* to determine the number of arguments and print an error message if there are none provided:
```bash
[root@server30 ~]# vim /usr/local/bin/if_then_fi.sh

#!/bin/bash 
if [ $# -ne 2 ] # Ensure there is a space after [ and before ] 
then 
        echo "Error: Invalid number of arguments supplied" 
        echo "Usage: $0 source_file destination_file" 
exit 2 
fi 
echo "Script terminated"
```

```bash
[root@server30 ~]# chmod +x /usr/local/bin/if_then_fi.sh

[root@server30 ~]# if_then_fi.sh
Error: Invalid number of arguments supplied
Usage: /usr/local/bin/if_then_fi.sh source_file destination_file
```


This script will display the following messages on the screen if it is
executed without exactly two arguments specified at the command line:
```bash
[root@server30 ~]# if_then_fi.sh
Error: Invalid number of arguments supplied
Usage: /usr/local/bin/if_then_fi.sh source_file destination_file
```

Return code value reflects the exit code in the script .

```bash
[root@server30 ~]# echo $?
2
```

Return code will be 0 if you supply a pair of arguments:
```bash
[root@server30 ~]# if_then_fi.sh a b
Script terminated
[root@server30 ~]# echo $?
0
```


### if-then-else-fi Construct 

- Can execute an action if the condition is true and another action if the condition is false. 

![](image-CMLV644G%201.jpg)

The general syntax of this statement is as follows:

if condition > then > action1 > else > action2 > fi

### Script08: The if-then-else-fi Construct

Create a script called *if_then_else_fi.sh* that will accept an integer value as an argument and tell if the value is positive or negative:
```bash
vim /usr/local/bin/if_then_else_fi.sh
```

```
#!/bin/bash
if [ $1 -gt 0 ]
then
        echo "$1 is a positive integer value"
else
        echo "$1 is a negative integer value"
fi
```

```bash
[root@server30 ~]# chmod +x /usr/local/bin/if_then_else_fi.sh
[root@server30 ~]# if_then_else_fi.sh
/usr/local/bin/if_then_else_fi.sh: line 2: [: -gt: unary operator expected
 is a negative integer value
[root@server30 ~]# if_then_else_fi.sh 3
3 is a positive integer value
[root@server30 ~]# if_then_else_fi.sh -3
-3 is a negative integer value
```

```bash
[root@server30 ~]# if_then_else_fi.sh a
/usr/local/bin/if_then_else_fi.sh: line 2: [: a: integer expression expected
a is a negative integer value

[root@server30 ~]# echo $?
0
```

### The if-then-elif-fi Construct 

- Can define multiple conditions and associate an action with each one of them. 
- The action corresponding to the true condition is performed. 

![](image-TNYN7SP2%201.jpg)

The general syntax of this statement is as follows:

if  condition1 > then action1 > elif condition2 > then action2 > elif  condition3 > then action3 > else > action(n) > fi                                  
### Script09: The if-then-elif-fi Construct (Example 1)

Create *if_then_elif_fi.sh* script to accept an integer value as an argument and tell if the integer is positive, negative, or zero. If a non-integer value or no argument is supplied, the script will complain. Employ the exit command after each action to help you identify where it exited.
```bash
[root@server30 ~]# vim /usr/local/bin/if_then_elif_fi.sh
```

```bash
#!/bin/bash
if [ $1 -gt 0 ]
then
        echo "$1 is a positive integer value"
exit 1
elif [ $1 -eq 0 ]
then
        echo "$1 is a zero integer value"
exit 2
elif [ $1 -lt 0 ]
then
        echo "$1 is a negative integer value"
exit 3
else
        echo "$1 is not an integer value. Please supply an i
nteger."
exit 4
fi
```



```bash
[root@server30 ~]# if_then_elif_fi.sh -0
-0 is a zero integer value

[root@server30 ~]# echo $?
2

[root@server30 ~]# if_then_elif_fi.sh -1
-1 is a negative integer value

[root@server30 ~]# echo $?
3

[root@server30 ~]# if_then_elif_fi.sh 10
10 is a positive integer value

[root@server30 ~]# echo $?
1

[root@server30 ~]# if_then_elif_fi.sh abd
/usr/local/bin/if_then_elif_fi.sh: line 2: [: abd: integer expression expected
/usr/local/bin/if_then_elif_fi.sh: line 6: [: abd: integer expression expected
/usr/local/bin/if_then_elif_fi.sh: line 10: [: abd: integer expression expected
abd is not an integer value. Please supply an i
nteger.>)

[root@server30 ~]# echo $?
4
```

### Script10: The if-then-elif-fi Construct (Example 2)

Create *ex200_ex294.sh* to display the name of the Red Hat exam RHCSA or RHCE in the output based on the input argument (ex200 or ex294). If a random or no argument is provided, it will print "Usage: Acceptable values are ex200 and ex294". Add white spaces in the conditions.

```bash
[root@server30 ~]# vim /usr/local/bin/ex200_ex294.sh
```

```bash
#!/bin/bash
if [ "$1" = ex200 ]
then
        echo "RHCSA"
elif [ "$1" = ex294 ]
then
        echo "RHCE"
else
        echo "Usage: Acceptable values are ex200 and ex294"
fi

```


```bash
[root@server30 ~]# chmod +x /usr/local/bin/ex200_ex294.sh

[root@server30 ~]# ex200_ex294.sh ex200
RHCSA

[root@server30 ~]# ex200_ex294.sh ex294
RHCE

[root@server30 ~]# ex200_ex294.sh frog
Usage: Acceptable values are ex200 and ex294
```

### Looping Constructs 

- Perform certain task on a number of given elements.
- Or repeatedly until a specified condition becomes true or false. 
- Examples:
	- if plenty of disks need to be initialized for use in LVM, you can either run the `pvcreate` command on each disk one at a time manually or employ a loop to do it for you. 
	- Based on a condition, you may want a program to continue to run until that condition becomes true or false.

Three looping constructs:
**for-do-done**
- **for loop** is also referred to as the **foreach** loop.
- iterates on a list of given values until the list is exhausted.
**while-do-done**
- while loop runs repeatedly until the specified condition becomes false. 
**until-do-done**
- until loop does just the opposite of the while loop
- Performs an operation repeatedly until the specified condition becomes true.

### Test Conditions 

`let` command 
- Used in looping constructs to evaluate a condition at each iteration.
- compares the value stored in a variable against a pre-defined value
- Each time the loop does an iteration, the variable value is altered. 
- Can enclose the condition for arithmetic evaluation within a pair of parentheses (( )) or quotation marks (" ") instead of using the let command explicitly.

Operators used in test conditions                 

**!**                                   
- Negation

**\+ / -- / \* / /**                    
- Addition / subtraction /multiplication / division

**%**                                  
- Remainder

**< / \<=**                            
- Less than / less than or equal to

**\> / \>=**                            
- Greater than / greater than or equal to

**=**                                   
- Assignment

**== / !=**                             
- Comparison for equality /non-equality

### The for Loop 

- Executed on an array of elements until all the elements in the array are consumed. 
- Each element is assigned to a variable one after the other for processing. 

![](image-BPQ7RWER%201.jpg)


The general syntax of this construct is as follows:

for VAR in list > do > action > done

### Script11: Print Alphabets Using for Loop

Create script *for_do_done.sh* script that initializes the variable COUNT to 0. The for loop will read each letter sequentially from the range placed within curly brackets (no spaces before the letter A and after the letter Z), assign it to another variable LETTER, and display the value on the screen. The `expr` command is an arithmetic processor, and it is used here to increment the COUNT by 1 at each loop iteration.
```bash
[root@server10 ~]# vim /usr/local/bin/for_do_done.sh
```

```bash
#!/bin/bash
COUNT=0
for LETTER in {A..Z}
do
        COUNT=`/usr/bin/expr $COUNT + 1`
        echo "Letter $COUNT is [$LETTER]"
done
```

```bash
[root@server10 ~]# chmod +x /usr/local/bin/for_do_done.sh
```

```bash
[root@server10 ~]# for_do_done.sh
Letter 1 is [A]
Letter 2 is [B]
Letter 3 is [C]
Letter 4 is [D]
Letter 5 is [E]
Letter 6 is [F]
Letter 7 is [G]
Letter 8 is [H]
Letter 9 is [I]
Letter 10 is [J]
Letter 11 is [K]
Letter 12 is [L]
Letter 13 is [M]
Letter 14 is [N]
Letter 15 is [O]
Letter 16 is [P]
Letter 17 is [Q]
Letter 18 is [R]
Letter 19 is [S]
Letter 20 is [T]
Letter 21 is [U]
Letter 22 is [V]
Letter 23 is [W]
Letter 24 is [X]
Letter 25 is [Y]
Letter 26 is [Z]
```


### Script12: Create Users Using for Loop

Create script *create_user.sh* script to create several Linux user accounts. As each account is created, the value of the variable ? is checked. If the value is 0, a message saying the account is created successfully will be displayed, otherwise the script will terminate. In case of a successful
account creation, the `passwd` command will be invoked to assign the user the same password as their username.

```bash
[root@server10 ~]# vim /usr/local/bin/create_user.sh
```

```bash
#!/bin/bash

for USER in user{10..12}
do
        echo "Create account for user $USER"
        /usr/sbin/useradd $USER
if [ $? = 0 ]
then
        echo $USER | /usr/bin/passwd --stdin $USER
        echo "$USER is created successfully"
else
        echo "Failed to create account $USER"
exit
fi
done
```

```bash
[root@server10 ~]# chmod +x /usr/local/bin/create_user.sh

[root@server10 ~]# create_user.sh
Create account for user user10
Changing password for user user10.
passwd: all authentication tokens updated successfully.
user10 is created successfully
Create account for user user11
Changing password for user user11.
passwd: all authentication tokens updated successfully.
user11 is created successfully
Create account for user user12
Changing password for user user12.
passwd: all authentication tokens updated successfully.
user12 is created successfully
```

Script fails if ran again:

```bash
[root@server10 ~]# create_user.sh
Create account for user user10
useradd: user 'user10' already exists
Failed to create account user10
```







