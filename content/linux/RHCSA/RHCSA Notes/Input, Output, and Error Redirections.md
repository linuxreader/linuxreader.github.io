## Input, Output, and Error Redirections

- Input, output, and error character streams:
	- **standard input (or stdin)**
		- Input redirection
		- <
	- **standard output (or stdout)**
		- >
		- >>
			- Append instead of overwrite
		- &>
			- Redirect standard error and output
	 - **standard error (or stderr)**
		 - 2>
- File descriptors:
	- 0, 1, and 2
		- 1
			- Represents standard output location
	- Can use these to represent character streams instead of <, and >
- **noclobber** feature
	- prevent overwriting of the output file
	- `set -o noclobber`
		- activates the feature
	- `set +o noclobber`
		- deactivate the feature
```bash
[root@localhost ~]# vim test.txt
[root@localhost ~]# set -o noclobber
[root@localhost ~]# echo "Hello" > test.txt
-bash: test.txt: cannot overwrite existing file
[root@localhost ~]# set +o noclobber
[root@localhost ~]# echo "Hello" > test.txt
[root@localhost ~]# cat test.txt
Hello
```
