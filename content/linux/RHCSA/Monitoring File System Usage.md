## Monitoring File System Usage 

### df (disk free) command 
- reports usage details for mounted file systems.
- reports the numbers in KBs unless the -m or -h option is specified to view the sizes in MBs or human-readable format.

Let's run this command with the -h option on server2:
```bash
[root@server2 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               4.0M     0  4.0M   0% /dev
tmpfs                  888M     0  888M   0% /dev/shm
tmpfs                  356M  5.1M  351M   2% /run
/dev/mapper/rhel-root   17G  2.0G   15G  12% /
tmpfs                  178M     0  178M   0% /run/user/0
/dev/sda1              960M  344M  617M  36% /boot

```

Column 1:
- file system device file or type

Columns 2, 3, 4, 5, 6
- total, used, and available spaces in and the usage percentage and mount point

Useful flags

`-T `
- Add the file system type to the output (example: df -hT)

`-x` 
- Exclude the specified file system type from the output (example: df -hx tmpfs)

`-t `
- Limit the output to a specific file system type (example: df -t xfs)

`-i`
- show inode information (example: df -hi)

### Calculating Disk Usage 

### du command
- reports the amount of space a file or directory occupies. 
- -m or -h option to view the output in MBs or human-readable format. In addition, you can 
- view a usage summary with the -s switch and a grand total with -c.

Run this command on the /usr/bin directory to view the usage summary:
```bash
[root@server2 ~]# du -sh /usr/bin
151M	/usr/bin

```

Add a "total" row to the output and with numbers displayed in KBs:
```bash
[root@server2 ~]# du -sc /usr/bin
154444	/usr/bin
154444	total
```

```bash
[root@server2 ~]# du -sch /usr/bin
151M	/usr/bin
151M	total
```

Try this command with different options on the /usr/sbin/lvm file and observe the results.

