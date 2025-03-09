## Advanced Container Management 

- Preset environment variables may be passed when launching containers or new variables may be set for containerized applications to consume for proper operation. 
- Information stored during an application execution is lost when a container is restarted or erased. 
- This behavior can be overridden by making a directory on the host available inside the container for saving data persistently. 
- Containers may be configured to start and stop with the transitioning of the host system via the systemd service. These advanced tasks are also performed with the podman command. 

## Containers and Port Mapping 

- Applications running in different containers often need to exchange data for proper operation. 
- For instance, a containerized Apache web server may need to talk to a MySQL database instance running in a different container. 
- It may also need to talk to the outside world over a port such as 80 or 8080. 
- To support this traffic flow, appropriate port mappings are established between the host system and each container.

EXAM TIP: As a normal user, you cannot map a host port below 1024 to a container port.

### Lab: Configure Port Mapping 

- Launch a container called rhel7-port-map in detached mode (as a daemon) with host port 10000 mapped to port 8000 inside the container. 
- Use a version of the RHEL 7 image with Apache web server software pre-installed. 
- This image is available from a Red Hat container registry.
- List the running container and confirm the port mapping.

1\. Search for an Apache web server image for RHEL 7 using podman
search:
```bash
[user1@server30 ~]$ podman search registry.redhat.io/rhel7/httpd
NAME                                      DESCRIPTION
registry.redhat.io/rhscl/httpd-24-rhel7   Apache HTTP 2.4 Server
```

2\. Log in to registry.redhat.io using the Red Hat credentials to access
the image:
```bash
[user1@server30 ~]$ podman login registry.redhat.io
Username: tdavetech@gmail.com
Password: 
Login Succeeded!
```

3\. Download the latest version of the Apache image using podman pull:
```bash
[user1@server30 ~]$ podman pull registry.redhat.io/rhscl/httpd-24-rhel7
Trying to pull registry.redhat.io/rhscl/httpd-24-rhel7:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob fd77da0b900b done   | 
Copying blob 7f2c2c4492b6 done   | 
Copying blob ea092d7970b2 done   | 
Copying config 847db19d6c done   | 
Writing manifest to image destination
Storing signatures
847db19d6cbc726106c901a7713d30dccc9033031ec812037c4c458319a1b328
```

4\. Verify the download using podman images:
```bash
[user1@server30 ~]$ podman images
REPOSITORY                               TAG         IMAGE ID      CREATED       SIZE
registry.redhat.io/rhscl/httpd-24-rhel7  latest      847db19d6cbc  2 months ago  332 MB
```

5\. Launch a container named (`\--name`) *rhel7-port-map* in detached mode
(`-d`) to run the containerized Apache web server with host port 10000
mapped to container port 8000 (`-p`). Use the `podman run` command with
appropriate flags.
```bash
[user1@server30 ~]$ podman run -dp 10000:8000 --name rhel7-port-map httpd-24-rhel7
cd063dff352dfbcd57dd417587513b12ca4033ed657f3baaa28d54df19d4df1c
```

6\. Verify that the container was launched successfully using podman ps:
```bash
[user1@server30 ~]$ podman ps
CONTAINER ID  IMAGE                                           COMMAND               CREATED         STATUS         PORTS                    NAMES
cd063dff352d  registry.redhat.io/rhscl/httpd-24-rhel7:latest  /usr/bin/run-http...  36 seconds ago  Up 36 seconds  0.0.0.0:10000->8000/tcp  rhel7-port-map
```

7\. You can also use `podman port` to view the mapping:
```bash
[user1@server30 ~]$ podman port rhel7-port-map
8000/tcp -> 0.0.0.0:10000
```

- Now any inbound web traffic on host port 10000 will be redirected to the container.

### Exercise 22-7: Stop, Restart, and Remove a Container 

- Stop the container, restart it, stop it again, and then erase it. 
- Use appropriate `podman` subcommands and verify each transition.

1\. Verify the current operational state of the container rhel7-port-map:
```bash
[user1@server30 ~]$ podman ps
CONTAINER ID  IMAGE                                           COMMAND               CREATED        STATUS        PORTS                    NAMES
cd063dff352d  registry.redhat.io/rhscl/httpd-24-rhel7:latest  /usr/bin/run-http...  3 minutes ago  Up 3 minutes  0.0.0.0:10000->8000/tcp  rhel7-port-map
```

2\. Stop the container and confirm using the `stop` and `ps` subcommands
(the `-a` option with `ps` also includes the stopped containers in the output):
```bash
[user1@server30 ~]$ podman stop rhel7-port-map
rhel7-port-map

[user1@server30 ~]$ podman ps -a
CONTAINER ID  IMAGE                                           COMMAND               CREATED        STATUS                    PORTS                    NAMES
cd063dff352d  registry.redhat.io/rhscl/httpd-24-rhel7:latest  /usr/bin/run-http...  6 minutes ago  Exited (0) 5 seconds ago  0.0.0.0:10000->8000/tcp  rhel7-port-map
```

3\. Start the container and confirm with the `start` and `ps` subcommands:
```bash
[user1@server30 ~]$ podman start rhel7-port-map
rhel7-port-map
[user1@server30 ~]$ podman ps -a
CONTAINER ID  IMAGE                                           COMMAND               CREATED        STATUS         PORTS                    NAMES
cd063dff352d  registry.redhat.io/rhscl/httpd-24-rhel7:latest  /usr/bin/run-http...  8 minutes ago  Up 11 seconds  0.0.0.0:10000->8000/tcp  rhel7-port-map
```

4\. Stop the container and remove it using the `stop` and `rm` subcommands:
```bash
[user1@server30 ~]$ podman rm rhel7-port-map
rhel7-port-map
```

5\. Confirm the removal using the `-a` option with `podman ps` to also include any stopped containers:
```bash
[user1@server30 ~]$ podman ps -a
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

## Containers and Environment Variables 

- Many times it is necessary to pass a host's pre-defined environment variable, such as PATH, to a containerized application for consumption.
- Moreover, it may also be necessary at times to set new variables to inject debugging flags or sensitive information such as passwords, access keys, or other secrets for use inside containers. 
- Passing host environment variables or setting new environment variables is done at the time of launching a container. 
- The `podman` command allows multiple variables to be passed or set with the `-e` option.

EXAM TIP: Use the `-e` option with each variable that you want to pass or set.

### Lab: Pass and Set Environment Variables 

- Launch a container using the latest version of a ubi for RHEL 9 available in a Red Hat container registry. 
- Inject the HISTSIZE environment variable, and a variable called SECRET with a value "secret123". 
- Name this container rhel9-env-vars and have a shell terminal opened to check the variable settings.
- Remove this container.

1\. Launch a container with an interactive terminal session (`-it`) and inject (`-e`) variables **HISTSIZE** and **SECRET** as directed. Use the specified container image.
```bash
[user1@server30 ~]$ podman run -it -e HISTSIZE -e SECRET="secret123" --name rhel9-env-vars ubi9
Resolved "ubi9" as an alias (/etc/containers/registries.conf.d/001-rhel-shortnames.conf)
Trying to pull registry.access.redhat.com/ubi9:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob cc296d75b612 done   | 
Copying config 159a1e6731 done   | 
Writing manifest to image destination
Storing signatures
[root@b587355b8fc1 /]# 
```

2\. Verify both variables using the echo command:
```bash
[root@b587355b8fc1 /]# echo $HISTSIZE $SECRET
1000 secret123
[root@b587355b8fc1 /]# 
```

3\. Disconnect from the container using the `exit` command, and stop and remove it using the `stop` and `rm` subcommands:
```bash
[user1@server30 ~]$ podman stop rhel9-env-vars
rhel9-env-vars
[user1@server30 ~]$ podman rm rhel9-env-vars
rhel9-env-vars
```

Confirm the deletion by running `podman ps -a`.
```bash
[user1@server30 ~]$ podman ps -a
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

## Containers and Persistent Storage 

- Containers are normally launched for a period of time to run an application and then stopped or deleted when their job is finished. 
- Any data that is produced during runtime is lost on their restart, failure, or termination.
- This data may be saved for persistence on a host directory by attaching the host directory to a container. 
- The containerized application will see the attached directory just like any other local directory and will use it to store data if it is configured to do so. 
- Any data that is saved on the directory will be available even after the container is rebooted or removed. 
- Later, this directory can be re-attached to other containers to give them access to the stored data or to save their own data. 
- The source directory on the host may itself may exist on any local or remote file system.

EXAM TIP: Proper ownership, permissions, and SELinux file type must be set to ensure persistent storage is accessed and allows data writes without issues.

- There are a few simple steps that should be performed to configure a host directory before it can be attached to a container. 
- These steps include the correct ownership, permissions, and SELinux type (*container_file_t*). 
- The special SELinux file type is applied to prevent containerized applications (especially those running in root containers) from gaining undesired privileged access to host files and processes, or other running containers on the host if compromised.

### Lab: Attach Persistent Storage and Access Data Across Containers 

- Set up a directory on server20 and attach it to a new container. 
- Write some data to the directory while in the container. 
- Delete the container and launch another container with the same directory attached. 
- Observe the persistence of saved data in the new container and that it is accessible. 
- Remove the container to mark the completion of this exercise.

1\. Create a directory called /host_data, set full permissions on it, and confirm:
```bash
[user1@server30 ~]$ sudo mkdir /host_data
[sudo] password for user1: 
[user1@server30 ~]$ sudo chmod 777 /host_data/
[user1@server30 ~]$ ll -d /host_data/
drwxrwxrwx. 2 root root 6 Aug  1 22:59 /host_data/
```

2\. Launch a root container called *rhel9-persistent-data* (`--name`) in interactive mode (`-it`) using the latest ubi9 image. Specify the attachment point (*/container_data*) to be used inside the container for the host directory (*/host_data*) with the `-v` (volume) option. Add `:Z` as shown to ensure the SELinux type container_file_t is automatically set on the directory and files within.
```bash
[user1@server30 ~]$ sudo podman run --name rhel9-persistent-data -v /host_data:/container_data:Z -it ubi9
Resolved "ubi9" as an alias (/etc/containers/registries.conf.d/001-rhel-shortnames.conf)
Trying to pull registry.access.redhat.com/ubi9:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob cc296d75b612 done   | 
Copying config 159a1e6731 done   | 
Writing manifest to image destination
Storing signatures
```

3\. Confirm the presence of the directory inside the container with `ls` on /container_data:
```bash
[root@e8711892370f /]# ls -ldZ /container_data
drwxrwxrwx. 2 root root system_u:object_r:container_file_t:s0:c376,c965 6 Aug  2 05:59 /container_data
```

4\. Create a file called testfile with the echo command under /container_data:
```bash
[root@e8711892370f /]# echo "This is persistent storage." > /container_data/testfile
```

5\. Verify the file creation and the SELinux type on it:
```bash
[root@e8711892370f /]# ls -lZ /container_data/
total 4
-rw-r--r--. 1 root root system_u:object_r:container_file_t:s0:c376,c965 28 Aug  2 06:03 testfile
```

6\. Exit out of the container and check the presence of the file in the host directory:
```bash
[root@e8711892370f /]# exit
exit
[user1@server30 ~]$ ls -lZ /host_data/
total 4
-rw-r--r--. 1 root root system_u:object_r:container_file_t:s0:c376,c965 28 Aug  1 23:03 testfile
```

7\. Stop and remove the container using the `stop` and `rm` subcommands:
```bash
[user1@server30 ~]$ sudo podman stop rhel9-persistent-data
rhel9-persistent-data
[user1@server30 ~]$ sudo podman rm rhel9-persistent-data
rhel9-persistent-data
```

8\. Launch a new root container called *rhel8-persistent-data* (`--name`) in interactive mode (`-it`) using the latest ubi8 image from any of the defined registries. Specify the attachment point (*/container_data2*) to be used inside the container for the host directory (*/host_data*) with the `-v` (volume) option. Add `:Z` as shown to ensure the SELinux type *container_file_t* is automatically set on the directory and files within.
```bash
[user1@server30 ~]$ sudo podman run -it --name rhel8-persistent-data -v /host_data:/container_data2:Z ubi8
Resolved "ubi8" as an alias (/etc/containers/registries.conf.d/001-rhel-shortnames.conf)
Trying to pull registry.access.redhat.com/ubi8:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 8694db102e5b done   | 
Copying config 269749ad51 done   | 
Writing manifest to image destination
Storing signatures
[root@af6773299c7e /]# 
```

9\. Confirm the presence of the directory inside the container with ls on /container_data2:
```bash
[root@af6773299c7e /]# ls -ldZ /container_data2/
drwxrwxrwx. 2 root root system_u:object_r:container_file_t:s0:c198,c914 22 Aug  2 06:03 /container_data2/
[root@af6773299c7e /]# ls -lZ /container_data2/
total 4
-rw-r--r--. 1 root root system_u:object_r:container_file_t:s0:c198,c914 28 Aug  2 06:03 testfile
[root@af6773299c7e /]# cat /container_data2/testfile
This is persistent storage.
```

10\. Create a file called testfile2 with the echo command under /container_data2:
```bash
[root@af6773299c7e /]# echo "This is persistent storage2." > /container_data2/testfile2

[root@af6773299c7e /]# ls -lZ /container_data2/
total 8
-rw-r--r--. 1 root root system_u:object_r:container_file_t:s0:c198,c914 28 Aug  2 06:03 testfile
-rw-r--r--. 1 root root system_u:object_r:container_file_t:s0:c198,c914 29 Aug  2 06:10 testfile2
```

11\. Exit out of the container and confirm the existence of both files in the host directory:
```bash
[root@af6773299c7e /]# exit
exit

[user1@server30 ~]$ ls -lZ /host_data/
total 8
-rw-r--r--. 1 root root system_u:object_r:container_file_t:s0:c198,c914 28 Aug  1 23:03 testfile
-rw-r--r--. 1 root root system_u:object_r:container_file_t:s0:c198,c914 29 Aug  1 23:10 testfile2
```

12\. Stop and remove the container using the stop and rm subcommands:
```bash
[user1@server30 ~]$ sudo podman stop rhel8-persistent-data
rhel8-persistent-data
[user1@server30 ~]$ sudo podman rm rhel8-persistent-data
rhel8-persistent-data
```

13\. Re-check the presence of the files in the host directory:
```bash
[user1@server30 ~]$ ll /host_data
total 8
-rw-r--r--. 1 root root 28 Aug  1 23:03 testfile
-rw-r--r--. 1 root root 29 Aug  1 23:10 testfile2
```

## Container State Management with systemd 

- Multiple containers run on a single host and it becomes a challenging task to change their operational state or delete them manually. 
- In RHEL 9, these administrative functions can be automated via the systemd service 

- There are several steps that need to be completed to configure container state management via systemd. 
- These steps vary for rootful and rootless container setups and include the creation of service unit files and their storage in appropriate directory locations (*~/.config/systemd/user* for rootless containers and */etc/systemd/system* for rootful containers). 
- Once setup and enabled, the containers will start and stop automatically as a systemd service with the host state transition or manually with the `systemctl` command.

- The `podman` command to start and stop containers is no longer needed if the systemd setup is in place. 
- You may experience issues if you continue to use `podman` for container state transitioning alongside.

- The start and stop behavior for rootless containers differs slightly from that of rootful containers. 
- For the rootless setup, the containers are started when the relevant user logs in to the host and stopped when that user logs off from all their open terminal sessions;
- However, this default behavior can be altered by enabling **lingering** for that user with the `loginctl` command.

- User lingering is a feature that, if enabled for a particular user, spawns a user manager for that user at system startup and keeps it running in the background to support long-running services configured for that user. 
- The user need not log in.

EXAM TIP: Make sure that you use a normal user to launch rootless containers and the root user (or sudo) for rootful containers.

- Rootless setup does not require elevated privileges of the root user.

### Lab: Configure a Rootful Container as a systemd Service 

- Create a systemd unit configuration file for managing the state of your rootful containers. 
- Launch a new container and use it as a template to generate a service unit file. 
- Stop and remove the launched container to avoid conflicts with new containers that will start. 
- Use the `systemctl` command to verify the automatic container start, stop, and deletion.

1\. Launch a new container called *rootful-container* (`--name`) in detached mode (`-d`) using the latest ubi9:
```bash
[user1@server30 ~]$ sudo podman run -dt --name rootful-container ubi9
[sudo] password for user1: 
0ed04dcedec418068acd14c864e95e78f56a38dd57d2349cf2c46b0de1a1bf1b
```

2\. Confirm the new container using podman ps. Note the container ID.
```bash
[user1@server30 ~]$ sudo podman ps
CONTAINER ID  IMAGE                                   COMMAND     CREATED         STATUS         PORTS       NAMES
0ed04dcedec4  registry.access.redhat.com/ubi9:latest  /bin/bash   20 seconds ago  Up 20 seconds              rootful-container
```

3\. Create (generate) a service unit file called *rootful-container.service* under */etc/systemd/system* while ensuring that the next new container that will be launched based on this configuration file will not require the source container (`--new`) to work. The `tee` command will show the generated file content on the screen as well as store it in the specified file.
```bash
[user1@server30 ~]$ sudo podman generate systemd --new --name rootful-container | sudo tee /etc/systemd/system/rootful-container.service

[Unit]
Description=Podman container-rootful-container.service
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
	--replace \
	-dt \
	--name rootful-container ubi9
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

- The unit file has the same syntax as any other systemd service configuration file.
- There are three sections---Unit, Service, and Install. 
	- (1) The **unit** section provides a short description of the service, the manual page location, and the dependencies (wants and after). 
	- (2) The **service** section highlights the full commands for starting (**ExecStart**) and stopping (**ExecStop**) containers. 
		- It also highlights the commands that will be executed before the container start (**ExecStartPre**) and after the container stop (**ExecStopPost**). 
		- There are a number of options and arguments with the commands to ensure a proper transition. 
		- The **restart** on-failure stipulates that systemd will try to restart the container in the event of a failure. 
	- (3) The **install** section identifies the operational target the host needs to be running in before this container service can start.  

4\. Stop and delete the source container (rootful-container) using the `stop` and `rm` subcommands:
```bash
[user1@server30 ~]$ sudo podman stop rootful-container
[sudo] password for user1: 
WARN[0010] StopSignal SIGTERM failed to stop container rootful-container in 10 seconds, resorting to SIGKILL 
rootful-container

[user1@server30 ~]$ sudo podman rm rootful-container
rootful-container
```

Verify the removal by running sudo podman ps -a:
```bash
[user1@server30 ~]$ sudo podman ps -a
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

5\. Update systemd to bring the new service under its control (reboot the system if required):
```bash
[user1@server30 ~]$ sudo systemctl daemon-reload

```

6\. Enable (`enable`) and start (`--now`) the container service using the `systemctl` command:
```bash
[user1@server30 ~]$ sudo systemctl enable --now rootful-container
Created symlink /etc/systemd/system/default.target.wants/rootful-container.service → /etc/systemd/system/rootful-container.service.
```

7\. Check the running status of the new service with the `systemctl` command:
```bash
[user1@server30 ~]$ sudo systemctl status rootful-container
● rootful-container.service - Podman container-rootful-container.s>
     Loaded: loaded (/etc/systemd/system/rootful-container.service>
     Active: active (running)
```

8\. Verify the launch of a new container (compare the container ID with that of the source root container):
```bash
[user1@server30 ~]$ sudo podman ps
CONTAINER ID  IMAGE                                   COMMAND     CREATED             STATUS             PORTS       NAMES
440a57c26186  registry.access.redhat.com/ubi9:latest  /bin/bash   About a minute ago  Up About a minute              rootful-container
```

9\. Restart the container service using the systemctl command:
```bash
[user1@server30 ~]$ sudo systemctl restart rootful-container
sudo systemctl status rootful-

[user1@server30 ~]$ sudo systemctl status rootful-container
● rootful-container.service - Podman container-rootful-container.s>
     Loaded: loaded (/etc/systemd/system/rootful-container.service>
     Active: active (running)
```

10\. Check the status of the container again. Observe the removal of the previous container and the launch of a new container (compare container IDs).
```bash
[user1@server30 ~]$ sudo podman ps
CONTAINER ID  IMAGE                                   COMMAND     CREATED         STATUS             PORTS       NAMES
0a980537b83a  registry.access.redhat.com/ubi9:latest  /bin/bash   59 seconds ago  Up About a minute              rootful-container
```

- Each time the rootful-container service is restarted or server20 is rebooted, a new container will be launched. 

### Lab: Configure Rootless Container as a systemd Service 

- Create a systemd unit configuration file for managing the state of your rootless containers. 
- Launch a new container as conuser1 (create this user) and use it as a template to generate a service unit file. 
- Stop and remove the launched container to avoid conflicts with new containers that will start. 
- Use the `systemctl` command as conuser1 to verify the automatic container start, stop, and deletion.

1\. Create a user account called conuser1 and assign a simple password:
```bash
[user1@server30 ~]$ sudo useradd conuser1

[user1@server30 ~]$ echo conuser1 | sudo passwd --stdin conuser1
Changing password for user conuser1.
passwd: all authentication tokens updated successfully.
```

2\. Open a new terminal window on server20 and log in as conuser1. Create directory *~/.config/systemd/user* to store a service unit file:
`[conuser1@server30 ~]$ mkdir ~/.config/systemd/user -p`

3\. Launch a new container called rootless-container (`--name`) in detached mode (`-d`) using the latest ubi8:
```bash
[conuser1@server30 ~]$ podman run -dt --name rootless-container ubi8
Resolved "ubi8" as an alias (/etc/containers/registries.conf.d/001-rhel-shortnames.conf)
Trying to pull registry.access.redhat.com/ubi8:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 8694db102e5b done   | 
Copying config 269749ad51 done   | 
Writing manifest to image destination
Storing signatures
381d46ae9a3e11723c3bde35090782129e6937c461f8c2621bc9725f6b9efc27
```

4\. Confirm the new container using podman ps. Note the container ID.
```bash
[conuser1@server30 ~]$ podman ps
CONTAINER ID  IMAGE                                   COMMAND     CREATED         STATUS         PORTS       NAMES
381d46ae9a3e  registry.access.redhat.com/ubi8:latest  /bin/bash   27 seconds ago  Up 27 seconds              rootless-container
```

5\. Create (generate) a service unit file called *rootless-container.service* under *~/.config/systemd/user* while ensuring that the next new container that will be launched based on this configuration will not require the source container (`--new`) to work:
```bash
[conuser1@server30 ~]$ podman generate systemd --new --name rootless-container > ~/.config/systemd/user/rootless-container.service

DEPRECATED command:
It is recommended to use Quadlets for running containers and pods under systemd.

Please refer to podman-systemd.unit(5) for details.
```

6\. Display the content of the unit file:
```bash
[conuser1@server30 ~]$ cat ~/.config/systemd/user/rootless-container.service 
# container-rootless-container.service
# autogenerated by Podman 4.9.4-rhel
# Thu Aug  1 23:42:11 MST 2024

[Unit]
Description=Podman container-rootless-container.service
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
	--replace \
	-dt \
	--name rootless-container ubi8
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

7\. Stop and delete the source container rootless-container using the stop and rm subcommands:
```bash
[conuser1@server30 ~]$ podman stop rootless-container
rootless-container

[conuser1@server30 ~]$ podman rm rootless-container
rootless-container
```

Verify the removal by running podman ps -a:
```bash
[conuser1@server30 ~]$ podman ps -a
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

8\. Update systemd to bring the new service to its control (reboot the system if required). Use the \--user option with the systemctl command.
```bash
[conuser1@server30 ~]$ systemctl --user daemon-reload
```

9\. Enable (`enable`) and start (`--now`) the container service using the `systemctl` command:
```bash
[conuser1@server30 ~]$ systemctl --user enable --now rootless-container.service 
Created symlink /home/conuser1/.config/systemd/user/default.target.wants/rootless-container.service → /home/conuser1/.config/systemd/user/rootless-container.service.
```

10\. Check the running status of the new service using the `systemctl` command:
```bash
conuser1@server30 ~]$ systemctl --user status rootless-container
● rootless-container.service - Podman container-rootless-container>
     Loaded: loaded (/home/conuser1/.config/systemd/user/rootless->
     Active: active (running)
```

11\. Verify the launch of a new container (compare the container ID with that of the source rootless container):
```bash
[conuser1@server30 ~]$ podman ps
CONTAINER ID  IMAGE                                   COMMAND     CREATED             STATUS             PORTS       NAMES
57f946085605  registry.access.redhat.com/ubi8:latest  /bin/bash   About a minute ago  Up About a minute              rootless-container
```

12\. Enable the container service to start and stop with host transition using the `loginctl` command (systemd login manager) and confirm:
```bash
[conuser1@server30 ~]$ loginctl enable-linger
[conuser1@server30 ~]$ loginctl show-user conuser1 | grep -i linger
Linger=yes
```

13\. Restart the container service using the `systemctl` command:
```bash
[conuser1@server30 ~]$ systemctl --user restart rootless-container
[conuser1@server30 ~]$ systemctl --user status rootless-container
● rootless-container.service - Podman container-rootless-container>
     Loaded: loaded (/home/conuser1/.config/systemd/user/rootless->
     Active: active (running)
```

14\. Check the status of the container again. Observe the removal of the previous container and the launch of a new container (compare container IDs).
```bash
[conuser1@server30 ~]$ podman ps
CONTAINER ID  IMAGE                                   COMMAND     CREATED         STATUS         PORTS       NAMES
4dec33db41b5  registry.access.redhat.com/ubi8:latest  /bin/bash   41 seconds ago  Up 41 seconds              rootless-container
```

- Each time the rootless-container service is restarted or server20 is rebooted, a new container will be launched. You can verify this by comparing their container IDs.
