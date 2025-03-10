## Containers DIY Labs 

### Lab: Launch Named Root Container with Port Mapping 

- Create a new user account called conadm on server30 and give them full sudo rights. 
```bash
 [root@se

 -bash: 3: command not found
 rver30 ~]# adduser conadm
 [root@server30 ~]# visudo
```

```bash
 conadm ALL=(ALL)        ALL
```

- As conadm with sudo (where required) on server30, inspect the latest version of ubi9 and then download it to your computer. 
```bash
 [root@server30 ~]# dnf install container-tools
 [root@server30 ~]# podman login registry.redhat.io
 [conuser1@server30 ~]$ podman pull ubi9
 Resolved "ubi9" as an alias  (/etc/containers/registries.conf.d/001-rhel- shortnames.conf)
 Trying to pull  registry.access.redhat.com/ubi9:latest...
 Getting image source signatures
 Checking if image destination supports signatures
 Copying blob cc296d75b612 done   | 
 Copying config 159a1e6731 done   | 
 Writing manifest to image destination
 Storing signatures  159a1e67312ef50059357047ebe2a365afea904504fca9561abb3 85ecd942d62
 [conuser1@server30 ~]$ podman inspect ubi9

```

- Launch a container called *rootful-cont-port* in attached terminal mode with host port 80 mapped to container port 8080. 
```bash
 sudo podman run -it --name rootful-cont-port -p  80:8080 ubi9
```

- Run a few basic Linux commands such as `ls`, `pwd`, `df`, `cat` */etc/redhat-release*, and *os-release* while in the container. 
```bash
 [root@349163a6e431 /]# ls
 afs  boot  etc	 lib	lost+found  mnt  proc  run    srv  tmp  var
 bin  dev   home  lib64	media	    opt  root  sbin   sys  usr

 [root@349163a6e431 /]# pwd
 /

 [root@349163a6e431 /]# df -hT
 Filesystem     Type      Size  Used Avail Use%  Mounted on
 overlay        overlay    17G  4.3G   13G  26% /
 tmpfs          tmpfs      64M     0   64M   0% /dev
 shm            tmpfs      63M     0   63M   0%  /dev/shm
 tmpfs          tmpfs     356M  6.0M  350M   2%  /etc/hosts
 devtmpfs       devtmpfs  4.0M     0  4.0M   0%  /proc/keys

 [root@349163a6e431 /]# cat /etc/redhat-release
 Red Hat Enterprise Linux release 9.4 (Plow)

```
- Check to confirm the port mapping from server30.
```bash
 [conadm@server30 ~]$ sudo podman port rootful-cont- port
 8080/tcp -> 0.0.0.0:80
```

- Do not remove the container yet.

### Lab: Launch Nameless Rootless Container with Two Variables 

- As conadm on server30, launch a container using the latest version of ubi8 in interactive mode (-it) with two environment variables VAR1=lab1 and VAR2=lab2 defined. 
```bash
 [conadm@server30 ~]$ podman run -d -e VAR1="lab1" -e VAR2="lab2" --name variables8 ubi8 
```

- Check the variables from within the container. 
```bash
 [root@803642faea28 /]# echo $VAR1
 lab1
 [root@803642faea28 /]# echo $VAR2
 lab2
```

- Delete the container and the image when done. 

### Lab: Launch Named Rootless Container with Persistent Storage 

- As conadm with sudo (where required) on server30, create a directory called */host_perm1* with full permissions, and a file called *str1* in it.
```bash
 [conadm@server30 ~]$ sudo mkdir /host_perm1
 [sudo] password for conadm: 
 [conadm@server30 ~]$ sudo chmod 777 /host_perm1
 [conadm@server30 ~]$ sudo touch /host_perm1/str1
```

- Launch a container called *rootless-cont-str* in attached terminal mode with the created directory mapped to */cont_perm1* inside the container.
```bash
 [conadm@server30 ~]$ sudo podman run --name  rootless-cont-str -v /host_perm1:/cont_perm1:Z -it  ubi8
 [root@a1326200eae1 /]# 
```

- While in the container, check access to the directory and the presence of the file. 
```bash
 [root@a1326200eae1 /]# ls /cont_perm1
 str1
```

- Create a sub-directory and a file under */cont_perm1* and exit out of the container shell. 
```bash
 [root@a1326200eae1 cont_perm1]# mkdir permdir2
 [root@a1326200eae1 cont_perm1]# ls
 permdir2  str1
 [root@a1326200eae1 cont_perm1]# exit
 exit
 [conadm@server30 ~]$ 
```

- List */host_perm1* on server30 to verify the sub-directory and the file. 
```bash
 [conadm@server30 ~]$ sudo ls /host_perm1
 permdir2  str1
```

- Stop and delete the container. 
```bash
 [conadm@server30 ~]$ podman stop rootless-cont-str
 rootless-cont-str
 [conadm@server30 ~]$ podman rm rootless-cont-str
 rootless-cont-str
```

- Remove */host_perm1*.
```bash
 [conadm@server30 ~]$ sudo rm -r /host_perm1
```

### Lab: Launch Named Rootless Container with Port Mapping, Environment Variables, and Persistent Storage

- As conadm with sudo (where required) on server30, launch a named rootless container called *rootless-cont-adv* in attached mode with two variables (**HISTSIZE**=100 and **MYNAME**=RedHat), host port 9000 mapped to container port 8080, and */host_perm2* mounted at */cont_perm2*
```bash
 [conadm@server30 ~]$ podman run --name rootless-cont-adv -v ~/host_perm2:/cont_perm2:Z -e HISTSIZE="100" -e MYNAME="RedHat" -p 9000:8080 -it --replace ubi8
 [root@79e965cd1436 /]# 
```

- Check and confirm the settings while inside the container. 
```bash
 [root@79e965cd1436 /]# echo $HISTSIZE
 100

 [root@79e965cd1436 /]# echo $MYNAME
 RedHat

 [root@79e965cd1436 /]# ls -ld /cont_perm2 
 drwxrwxrwx. 2 root root 6 Aug  4 02:16 /cont_perm2

 [conadm@server30 ~]$ podman port rootless-cont-adv
 8080/tcp -> 0.0.0.0:9000

```
- Exit out of the container. 
```bash
 [root@5d510a1b2293 /]# exit
 exit
 [conadm@server30 ~]$ 
```
- Do not remove the container yet. 

### Lab 22-5: Control Rootless Container States via systemd 

- As conadm on server30, use the *rootless-cont-adv* container launched in the last lab as a template and generate a systemd service configuration file and store the file in the appropriate directory. 
```bash
 [conadm@server30 ~]$ podman run --name rootless-cont-adv -v ~/host_perm2:/cont_perm2:Z -e HISTSIZE="100" -e MYNAME="RedHat" -p 9000:8080 -dt --replace ubi8
 da8faf434813242985b8e332dc06b0e6da78e7125bc36579ffc8d82b0bcafb8e
 [conadm@server30 ~]$ podman generate systemd --new --name rootless-cont-adv >  ~/.config/systemd/user/rootless-container.service

 DEPRECATED command:
 It is recommended to use Quadlets for running  containers and pods under systemd.

 Please refer to podman-systemd.unit(5) for details.
```

- Stop and remove the source container rootless-cont-adv. 
```bash
 [conadm@server30 ~]$ podman stop rootless-cont-adv
 rootless-cont-adv
 [conadm@server30 ~]$ podman rm rootless-cont-adv
 rootless-cont-adv
```

- Add the support for the new service to systemd and enable the new service to auto-start at system reboots.
```bash
 [conadm@server30 ~]$ systemctl --user daemon-reload

 [conadm@server30 user]$ systemctl --user enable -- now rootless-container.service
 Created symlink  /home/conadm/.config/systemd/user/default.target.want s/rootless-container.service →  /home/conadm/.config/systemd/user/rootless- container.service.
```

- Perform the required setup to ensure the container is launched without the need for the conadm user to log in. 
```bash
 [conadm@server30 user]$ loginctl enable-linger

 [conadm@server30 user]$ loginctl show-user conadm |  grep -i linger
 Linger=yes
```

- Reboot server30 and confirm a successful start of the container service and the container. 
```bash
[root@rhcsa3 ~]# systemctl --user --machine=conadm@ list-units --type=service
  UNIT                           LOAD   ACTIVE SUB     DESCRIPTION           >
  dbus-broker.service            loaded active running D-Bus User Message Bus
  rootless-cont-adv.service      loaded active running Podman container-rootl>
  systemd-tmpfiles-setup.service loaded active exited  Create User's Volatile>

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
3 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
[root@rhcsa3 ~]# sudo -i -u conadm podman ps -a
CONTAINER ID  IMAGE                                   COMMAND     CREATED         STATUS         PORTS                   NAMES
a48fd2c25be4  registry.access.redhat.com/ubi9:latest  /bin/bash   10 minutes ago  Up 10 minutes  0.0.0.0:9000->8080/tcp  rootless-cont-adv

```

### Lab 22-6: Control Rootful Container States via systemd 

- As conadm with sudo where required on server10, use the rootful-cont-port container launched in Lab 22-1 as a template and generate a systemd service configuration file and store the file in the appropriate directory. 
```bash
 [root@server30 ~]# podman generate systemd --new -- name rootful-cont-port | tee  /etc/systemd/system/rootful-cont-port.service
 DEPRECATED command:
 It is recommended to use Quadlets for running  containers and pods under systemd.

 Please refer to podman-systemd.unit(5) for details.
 # container-rootful-cont-port.service
 # autogenerated by Podman 4.9.4-rhel
 # Sat Aug  3 20:49:32 MST 2024

 [Unit]
 Description=Podman container-rootful-cont- port.service
 Documentation=man:podman-generate-systemd(1)
 Wants=network-online.target
 After=network-online.target
 RequiresMountsFor=%t/containers

 [Service]
 Environment=PODMAN_SYSTEMD_UNIT=%n
 Restart=on-failure
  TimeoutStopSec=70
 ExecStart=/usr/bin/podman run \
	--cidfile=%t/%n.ctr-id \
	--cgroups=no-conmon \
	--rm \
	--sdnotify=conmon \
	-d \
	--replace \
	-it \
	--name rootful-cont-port \
	-p 80:8080 ubi9
 ExecStop=/usr/bin/podman stop \
	--ignore -t 10 \
	--cidfile=%t/%n.ctr-id
 ExecStopPost=/usr/bin/podman rm \
	-f \
	--ignore -t 10 \
	--cidfile=%t/%n.ctr-id
 Type=notify
 NotifyAccess=all

 [Install]
 WantedBy=default.target
```

- Stop and remove the source container rootful-cont-port. 
```bash
 [root@server30 ~]# podman stop rootful-cont-port
 WARN[0010] StopSignal SIGTERM failed to stop  container rootful-cont-port in 10 seconds, resorting  to SIGKILL 
 rootful-cont-port
 [root@server30 ~]# podman ps
 CONTAINER ID  IMAGE                                   COMMAND     CREATED         STATUS         PORTS       NAMES
 fe0d07718dda  registry.access.redhat.com/ubi9:latest  /bin/bash   16 minutes ago  Up 16 minutes              rootful-container
 [root@server30 ~]# podman rm rootfil-cont-port
 Error: no container with ID or name "rootfil-cont- port" found: no such container
 [root@server30 ~]# podman rm rootful-cont-port
 rootful-cont-port
```
- Add the support for the new service to systemd and enable the service to auto-start at system reboots. 
```bash
 [root@server30 ~]# systemctl daemon-reload
 [root@server30 ~]# systemctl enable --now rootful- cont-port
 Created symlink  /etc/systemd/system/default.target.wants/rootful- cont-port.service → /etc/systemd/system/rootful-cont- port.service.
```

- Reboot server10 and confirm a successful start of the container service and the container.
```bash
 [root@server30 ~]# reboot

 [root@server30 ~]# podman ps
 CONTAINER ID  IMAGE                                   COMMAND     CREATED             STATUS              PORTS                 NAMES
 5c030407a7d6   registry.access.redhat.com/ubi9:latest  /bin/bash    About a minute ago  Up About a minute  0.0.0.0:80- >8080/tcp  rootful-cont-port
 9d1e8a429ac6   registry.access.redhat.com/ubi9:latest  /bin/bash    About a minute ago  Up About a minute                        rootful-container
 [root@server30 ~]# 
```

### Lab 22-7: Build Custom Image Using Containerfile 

- As conadm on server10, write a containerfile to use the latest version of ubi8 and create a user account called user-in-container in the resultant custom image. 
```bash
 [conadm@server30 ~]$ vim containerfile
```

```bash
 FROM registry.access.redhat.com/ubi8/ubi:latest

 RUN useradd -ms /bin/bash -u 1001 user-in-container

 USER 1001
```

```bash
 [conadm@server30 ~]$ podman image build -f  containerfile --no-cache -t ubi8-user .
 STEP 1/3: FROM  registry.access.redhat.com/ubi8/ubi:latest
 STEP 2/3: RUN useradd -ms /bin/bash -u 1001 user-in- container
 --> b330095e91eb
 STEP 3/3: USER 1001
 COMMIT ubi8-user
 --> e8cde30fc020
 Successfully tagged localhost/ubi8-user:latest
 e8cde30fc020051caa2a4e2f58aaaf90f088709462a1314b936fd608facfdb5e

```

- Test the image by launching a container in interactive mode and verifying the user. 

```bash
 [conadm@server30 ~]$ podman run -ti --name test12  ubi8-user
 [user-in-container@30558ffcb227 /]$
```
