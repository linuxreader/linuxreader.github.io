## System Tuning

System tuning service 
- Monitor connected devices
- Tweak their parameters to improve performance or conserve power. 
- recommended tuning profile may be identified and activated for optimal performance and power saving.
tuned
- system tuning service 
- monitor storage, networking, processor, audio, video, and a variety of other connected devices
- Adjusts their parameters for better performance or power saving based on a chosen profile. 
- Several predefined tuning profiles may be activated either statically or dynamically.

*tuned* service 
- static behavior (default)
	- activates a selected profile at service startup and continues to use it until it is switched to a different profile. 

- dynamic 
	- adjusts the system settings based on the live activity data received from monitored system components

### tuned tuning Profiles

- Nine profiles to support a variety of use cases.
- Can create custom profiles from nothing or by using one of the existing profiles as a template. 
- Must to store the custom profile in /etc/tuned/ 

Three groups: 
(1) Performance
(2) Power consumption
(3) Balanced


| **Profile**               | **Description**                                                                                             |
| ------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **Performance**           |                                                                                                             |
| Desktop                   | Based on the balanced profile for desktop systems. Offers improved throughput for interactive applications. |
| Latency-performance       | For low-latency requirements                                                                                |
| Network-latency           | Based on the latency-performance for faster network throughput                                              |
| Network-throughput        | Based on the throughput-performance profile for maximum network throughput                                  |
| Virtual-guest             | Optimized for virtual machines                                                                              |
| Virtual-host              | Optimized for virtualized hosts                                                                             |
| **Power Saving**          |                                                                                                             |
| Powersave                 | Saves maximum power at the cost of performance                                                              |
| **Balanced/Max Profiles** |                                                                                                             |
| Balanced                  | Preferred choice for systems that require a balance between performance and power saving                    |
| Throughput-performance    | Provides maximum performance and consumes maximum power                                                     |

### Tuning Profiles


Predefined profiles are located in /usr/lib/tuned/ in subdirectories matching their names. 

View predefined profiles:
`ls -l /usr/lib/tuned` 

The default active profile set on *server1* and *server2* is the
*virtual-guest* profile, as the two systems are hosted in a VirtualBox virtualized environment.

### The tuned-adm Command

- single profile management command that comes with tuned
- can list active and available profiles, query current settings, switch between profiles, and turn the tuning off. 
- Can recommend the best profile for the system based on many system attributes. 

View the man pages:
`man tuned-adm`

### Lab 12-2: Manage Tuning Profiles

- install the *tuned* service
- start it now
- enable it for auto-restart upon future system reboots.
- display all available profiles and the current active profile.
- switch to one of the available profiles and confirm.
- determine the recommended profile for the system and switch to it. 
- deactivate tuning and reactivate it. 
- confirm the activation

1. Install the *tuned* package if it is not already installed:
`dnf install tuned` 

2. Start the *tuned* service and set it to autostart at reboots:
`systemctl --now enable tuned` 

3. Confirm the startup:
`systemctl status tuned` 

4. Display the list of available and active tuning profiles:
`tuned-adm list` 

5. List only the current active profile:
`tuned-adm active` 

6. Switch to the *powersave* profile and confirm:
```bash
tuned-adm profile powersave
tuned-adm active
``` 

7. Determine the recommended profile for *server1* and switch to it:
```bash
[root@localhost ~]# tuned-adm recommend
virtual-guest
[root@localhost ~]# tuned-adm profile virtual-guest
[root@localhost ~]# tuned-adm active
Current active profile: virtual-guest
``` 

8. Turn off tuning:
```bash
[root@localhost ~]# tuned-adm off
[root@localhost ~]# tuned-adm active
No current active profile.
``` 

9. Reactivate tuning and confirm:
```bash
[root@localhost ~]# tuned-adm profile virtual-guest
[root@localhost ~]# tuned-adm active
Current active profile: virtual-guest
``` 
