## Sysinit, Logging, and Tuning Labs

### Lab: Modify Default Boot Target

- Modify the default boot target from graphical to multi-user, and reboot the system to test it. 
`systemctl set-default multi-user`
- Run the `systemctl` and `who` commands after the reboot for validation. 
- Restore the default boot target back to graphical and reboot to verify. 

### Lab: Record Custom Alerts

- Write the message "This is $LOGNAME adding this marker on $(date)" to /var/log/messages.
`logger -i "This is $LOGNAME adding this marker on $(date)"`
- Ensure that variable and command expansions work. Verify the entry in the file.
`tail -l /var/log/messages`
### Lab: Apply Tuning Profile

- identify the current system tuning profile with the *tuned-adm* command. 
`tuned-adm active`

- List all available profiles. 
`tuned-adm list`
- List the recommended profile for *server1*. 
`tuned-adm recommend`
- Apply the "balanced" profile and verify with *tuned-adm*. 
`tuned-adm profile balanced`
`tuned-adm active`
