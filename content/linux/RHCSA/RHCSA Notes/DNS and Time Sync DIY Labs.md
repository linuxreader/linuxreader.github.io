## DNS and Time Sync DIY Labs

### Lab: Configure Chrony 

- Install Chrony and mark the service for autostart on reboots. 
`systemctl enable --now chronyd`

- Edit the Chrony configuration file and comment all line entries that begin with "pool" or "server". 
```bash
[root@server10 ~]# vim /etc/chrony.conf
```

- Go to the end of the file, and add a new line "server 127.127.1.0". 

- Start the Chrony service and run chronyc sources to confirm the binding. 
```bash
[root@server10 ~]# systemctl restart chronyd

[root@server10 ~]# chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? 127.127.1.0                   0   6     0     -     +0ns[   +0ns] +/-    0ns
```

### Lab: Modify System Date and Time 

- Execute the date and timedatectl commands to check the current system date and time. 
```bash
[root@server10 ~]# date
Mon Jul 22 11:37:54 AM MST 2024

[root@server10 ~]# timedatectl
               Local time: Mon 2024-07-22 11:37:59 MST
           Universal time: Mon 2024-07-22 18:37:59 UTC
                 RTC time: Mon 2024-07-22 18:37:59
                Time zone: America/Phoenix (MST, -0700)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: no
```

- Identify the distinctions between the two outputs. 

- Use `timedatectl` and change the system date to a future date. 
```bash
[root@server10 ~]# timedatectl set-time 2024-07-23
Failed to set time: Automatic time synchronization is enabled

[root@server10 ~]# timedatectl set-ntp false

[root@server10 ~]# timedatectl set-time "2024-07-23"
```
- Issue the `date` command and change the system time to one hour ahead of the current time. 
```bash
[root@server10 ~]# date -s "2024-07-22 12:41"
Mon Jul 22 12:41:00 PM MST 2024
```

- Observe the new date and time with both commands. 
```bash
[root@server10 ~]# date -s "2024-07-22 12:41"
Mon Jul 22 12:41:00 PM MST 2024

[root@server10 ~]# date
Mon Jul 22 12:41:39 PM MST 2024
[root@server10 ~]# timedatectl
               Local time: Mon 2024-07-22 12:41:41 MST
           Universal time: Mon 2024-07-22 19:41:41 UTC
                 RTC time: Tue 2024-07-23 07:01:41
                Time zone: America/Phoenix (MST, -0700)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
```

- Reset the date and time to the current actual time by disabling and re-enabling the NTP service using the `timedatectl` command.
```bash
[root@server10 ~]# timedatectl set-ntp true
```