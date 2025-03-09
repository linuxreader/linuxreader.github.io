## Working with Images and Containers 

### Lab: Install Necessary Container Support 

- Install the necessary software to set the foundation for completing the exercises in the remainder of the chapter.
- The standard RHEL 9.1 image includes a package called `container-tools` that consists of all the required components and commands. 
- Use the standard `dnf` command to install the package.

1\. Install the container-tools package:
```bash
 root@server10 ~]# dnf install -y container-tools

 Upgraded:
  aardvark-dns-2:1.10.0-3.el9_4.x86_64 
  buildah-2:1.33.7-3.el9_4.x86_64         
  netavark-2:1.10.3-1.el9.x86_64            
  podman-4:4.9.4-6.el9_4.x86_64                                   
 Installed:
  container-tools-1-14.el9.noarch        
  podman-docker-4:4.9.4-6.el9_4.noarch 
  podman-remote-4:4.9.4-6.el9_4.x86_64    
  python3-podman-3:4.9.0-1.el9.noarch 
  python3-pyxdg-0.27-3.el9.noarch    
  python3-tomli-2.0.1-5.el9.noarch   
  skopeo-2:1.14.3-3.el9_4.x86_64             
  toolbox-0.0.99.5-2.el9.x86_64      
  udica-0.2.8-1.el9.noarch    
```

2\. Verify the package installation:
```bash
 [root@server10 ~]# dnf list container-tools
 Updating Subscription Management repositories.
 Last metadata expiration check: 14:53:32 ago on Wed 31 Jul 2024 05:45:56 PM MST.
 Installed Packages
 container-tools.noarch   1-14.el9    @rhel-9-for-x86_64-appstream-rpms
```
### `podman` Command 

- Finding, inspect, retrieve, and delete images 
- Run, stop, list, and delete containers. 
- Used for most of these operations. 

### Subcommands 

#### Image Management                    

**build**                               
- Builds an image using instructions delineated in a Container file

**images**                              
- Lists downloaded images from local storage

**inspect**                             
- Examines an image and displays its details

**login/logout**                       
- Logs in/out to/from a container registry. A login may be required to access private and protected registries.

**pull**                                
- Downloads an image to local storage from a registry

**rmi**                                 
- Removes an image from local storage

**search**                              
- Searches for an image. The following options can be included with this subcommand:

1. A *partial image name* in the search will produce a list of all images containing the partial name.
2. The `--no-trunc` option makes the command exhibit output without truncating it. 
3. The `--limit <number>` option limits the displayed results to the specified number.

**tag**                                 
- Adds a name to an image. The default is 'latest' to classify the image as the latest version. Older images may have specific version identifiers.

#### Container Management                

**attach**                              
- Attaches to a running container

**exec**                                
- Runs a process in a running container

**generate**                            
- Generates a systemd unit configuration file that can be used to control the operational state of a container. The `--new` option is important and is employed in later exercises.

**info**                                
- Reveals system information, including the defined registries

**inspect**                             
- Exhibits the configuration of a container

**ps**                                  
- Lists running containers (includes stopped containers with the -a option)

**rm**                                  
- Removes a container

**run**                                
- Launches a new container from an image. Some options such as -d (detached), -i (interactive), and -t (terminal) are important and are employed in exercises where needed.

**start/stop/restart**                  
- Starts, stops, or restarts a container

### `skopeo` Command 
- Utilized for interacting with local and remote images and registries. 
- Has numerous subcommands available; however, you will be using only the `inspect` subcommand to examine the details of an image stored in a remote registry. 

*/etc/containers/registries.conf*
- System-wide configuration file for image registries.
- Normal Linux users may store a customized copy of this file, if required, under the *~/.config/containers* directory. 
- Settings stored in the per-user file will take precedence over those stored in the system-wide file. 
	- Useful for running rootless containers.
- Defines searchable and blocked registries. 

```bash
 [root@server10 ~]# grep -Ev '^#|^$' /etc/containers/registries.conf
 unqualified-search-registries = ["registry.access.redhat.com", "registry.redhat.io", "docker.io"]
 short-name-mode = "enforcing"
```

- The output shows three registries. 
- The `podman` command searches these registries for container images in the given order.
- Can add additional registries to the list. 

 Add a private registry called *registry.private.myorg.io* to be added with the highest priority:
```bash
 [root@server10 ~]# vim /etc/containers/registries.conf
```

```bash
 unqualified-search-registries = \["registry.private.myorg.io",
 "registry.access.redhat.com", "registry.redhat.io", "docker.io"\]
```

If this private registry is the only one to be used, you can take the rest of the registry entries out of the list:

```bash
 unqualified-search-registries = \["registry.private.myorg.io"\]
```

EXAM TIP: As there is no Internet access provided during Red Hat exams, you may have to access a network-based registry to download images.

### Viewing Podman Configuration and Version 

- The podman command references various system runtime and configuration files and runs certain Linux commands in the background to gather and display information. 
- For instance, it looks for registries and storage data in the system-wide and per-user configuration files, pulls memory information from the */proc/meminfo* file, executes `uname -r`to obtain the kernel version, and so on. 
- podman's `info` subcommand shows all this information. 

Here is a sample when this command is executed as a normal
user (user1):
```bash
 [[user1@server10 root]$ podman info
 ERRO[0000] XDG_RUNTIME_DIR directory "/run/user/0" is not owned by the current user](<[user1@server10 ~]$ podman info
 host:
  arch: amd64
  buildahVersion: 1.33.8
  cgroupControllers:
  - memory
  - pids
  cgroupManager: systemd
  cgroupVersion: v2
  conmon:
 ...
```


- Re-run the command as root (preceded by sudo if running as user1) and compare the values for the settings "rootless" under host and "ConfigFile" and "ImageStore" under store. 

- The differences lie between where the root and rootless (normal) users store and obtain configuration data, the number of container images they have locally available, and so on.

```bash
 [root@server10 ~]# podman info
 host:
  arch: amd64
  buildahVersion: 1.33.8
  cgroupControllers:
  - cpuset
  - cpu
  - io
  - memory
  - hugetlb
  - pids
  - rdma
  - misc
...
```

Similarly, you can run the podman command as follows to check its
version:
```bash
 [root@server10 ~]# podman version
 Client:       Podman Engine
 Version:      4.9.4-rhel
 API Version:  4.9.4-rhel
 Go Version:   go1.21.11 (Red Hat 1.21.11-1.el9_4)
 Built:        Mon Jul  1 03:27:14 2024
 OS/Arch:      linux/amd64
```
### Image Management 

Container images 
- Are available from numerous private and public registries. 
- They are pre-built for a variety of use cases. 
- You can search through registries to find the one that suits your needs. 
- You can examine their metadata before downloading them for consumption.
- Downloaded images can be removed when no longer needed to conserve local storage. 
- The same pair of commands---`podman` and `skopeo`---is employed for these operations.

### Lab: Search, Examine, Download, and Remove an Image 

- Log in to the registry.access.redhat.com registry
- Look for an image called *mysql-80* in the registry, examine its details, pull it to your system, confirm the retrieval, and finally erase it from the local storage. 

1\. Log in to the specified Red Hat registry:
```bash
 [user1@server10 ~]$ podman login registry.redhat.io
```

2\. Confirm a successful login:
```bash
 [user1@server10 ~]$ podman login registry.redhat.io --get-login
```

3\. Find the mysql-80 image in the specified registry. Add the
`--no-trunc` option to view full output.
```bash
 [user1@server10 ~]$ podman search registry.redhat.io/mysql-80 --no-trunc
 NAME                                     DESCRIPTION
 registry.redhat.io/rhel8/mysql-80        This container image provides a containerized packaging of the MySQL mysqld daemon and client application. The  mysqld server daemon accepts connections from clients and provides access to content from MySQL databases on behalf of the clients.
...
```

4\. Select the second image rhel9/mysql-80 for this exercise. Inspect the image without downloading it using skopeo inspect. A long output
will be generated. The command uses the docker:// mechanism to access
the image.
```bash
 [user1@server10 ~]$ skopeo inspect docker://registry.redhat.io/rhel9/mysql-80
 {
    "Name": "registry.redhat.io/rhel9/mysql-80",
    "Digest": "sha256:247903d2103a3c1db9401f6340ecdcd97c6244480b7a3419e6303dda650491dc",
    "RepoTags": [
        "1",
        "1-190",
        "1-190.1655192188",
        "1-190.1655192188-source",
        "1-190-source",
        "1-197",
        "1-197-source",
        "1-206",
...
```

Output:
- Shows older versions under RepoTags
- Creation time for the latest version
- Build date of the image
- description
- other information. 

- It is a good practice to analyze the metadata of an image prior to downloading and consuming it.

5\. Download the image by specifying the fully qualified image name using podman pull:
```bash
 [user1@server10 ~]$ podman pull docker://registry.redhat.io/rhel9/mysql-80
 Trying to pull registry.redhat.io/rhel9/mysql-80:latest...
 Getting image source signatures
 Checking if image destination supports signatures
 Copying blob 846c0bdf4e30 done   | 
 Copying blob cc296d75b612 done   | 
 Copying blob db22e630b1c7 done   | 
 Copying config b5782120a3 done   | 
 Writing manifest to image destination
 Storing signatures
 b5782120a320e5915d86555e661c357cfa56dd8320ba4c54a58caa1e1c91925f
```

6\. List the image to confirm the retrieval using podman images:
```bash
 [user1@server10 ~]$ podman images
 REPOSITORY                         TAG         IMAGE ID      CREATED      SIZE
 registry.redhat.io/rhel9/mysql-80  latest      b5782120a320  2 weeks ago  555 MB
```

7\. Display the image's details using `podman inspect`:
```bash
 [user1@server10 ~]$ podman inspect mysql-80
 [
     {
          "Id": "b5782120a320e5915d86555e661c357cfa56dd8320ba4c54a58caa1e1c91925f",
          "Digest": "sha256:247903d2103a3c1db9401f6340ecdcd97c6244480b7a3419e6303dda650491dc",
          "RepoTags": [
               "registry.redhat.io/rhel9/mysql-80:latest"
          ],
```

8\. Remove the mysql-80 image from local storage:
```bash
 [user1@server10 ~]$ podman rmi mysql-80
 Untagged: registry.redhat.io/rhel9/mysql-80:latest
 Deleted: b5782120a320e5915d86555e661c357cfa56dd8320ba4c54a58caa1e1c91925f
```

- Shows the ID of the image after deletion.

9\. Confirm the removal:
```bash
 [user1@server10 ~]$ podman images
 REPOSITORY  TAG         IMAGE ID    CREATED     SIZE
```

### Containerfile 
- Can build a custom image by outlining the steps you need to be run in a file called Containerfile. 
- The podman command can then be used to read those instructions and executes them to produce a new image.
- File name *containerfile* is widespread; but you can use any name of your liking.

Instructions that may be utilized inside a Containerfile to perform specific functions during the build process:

**CMD**                                 
- Runs a command

**COPY**                                
- Copies files to the specified location

**ENV**                                 
- Defines environment variables to be used during the build process

**EXPOSE**                              
- A port number that will be opened when a container is launched using this image

**FROM**                                
- Identifies the base container image to use

**RUN**                                 
- Executes the specified commands

**USER**                                
- Defines a non-root user to run the commands as

**WORKDIR**                             
- Sets the working directory. This directory is automatically created if it does not already exist.

A sample container file is presented below:
```bash
 [user1@server10 ~]$ vim containerfile
```

```bash
 # Use RHEL9 base image
 FROM registry.redhat.io/ubi9/ubi

 # Install Apache web server software
 RUN dnf -y install httpd

 # Copy the website
 COPY ./index.html /var/www/html/

 # Expose Port 80/tcp
 EXPOSE 80

 # Start Apache web server
 CMD ["httpd"]
```

- The index.html file may contain a basic statement such as "This is a custom-built Apache web server container image based on RHEL 9".

### Lab: Use Containerfile to Build Image 

- Use a containerfile to build a custom image based on the latest version of the RHEL 9 universal base image (ubi) available from a Red Hat container registry. 
- Confirm the image creation. 
- Use the podman command for these activities.

1\. Log in to the specified Red Hat registry:
```bash
 [user1@server10 ~]$ podman login registry.redhat.io
 Authenticating with existing credentials for registry.redhat.io
 Existing credentials are valid. Already logged in to registry.redhat.io
```

2\. Confirm a successful login:
```bash
 [user1@server10 ~]$ podman login registry.redhat.io --get-login
```

3\. Create a file called containerfile with the following code:
```bash
 [user1@server10 ~]$ vim containerfile2
```

```bash
 # Use RHEL9 base image
 FROM registry.redhat.io/ubi9/ubi

 # Count the number of characters
 CMD echo "RHCSA exam is hands-on." | wc

 # Copy a local file to /tmp
 COPY ./testfile /tmp

```

4\. Create a file called testfile with some random text in it and place it in the same directory as the containerfile.
```bash
 [user1@server10 ~]$ echo "boo bee doo bee doo" >> testfile
 [user1@server10 ~]$ cat testfile 
 boo bee doo bee doo
```

5\. Build an image by specifying the containerfile name and an image tag such as `ubi9-simple-image`. The period character at the end represents the current directory and this is where both containerfile and testfile are located.
```bash
 [user1@server10 ~]$ podman image build -f containerfile2 -t ubi9-simple-image .
 STEP 1/3: FROM registry.redhat.io/ubi9/ubi
 Trying to pull registry.redhat.io/ubi9/ubi:latest...
 Getting image source signatures
 Checking if image destination supports signatures
 Copying blob cc296d75b612 done   | 
 Copying config 159a1e6731 done   | 
 Writing manifest to image destination
 Storing signatures
 STEP 2/3: CMD echo "RHCSA exam is hands-on." | wc
 --> 4c005bfd0b34
 STEP 3/3: COPY ./testfile /tmp
 COMMIT ubi9-simple-image
 --> a2797b06a129
 Successfully tagged localhost/ubi9-simple-image:latest
 a2797b06a1294ed06edab2ba1c21d2bddde3eb3af1d8ed286781837f62992622
```

6\. Confirm image creation:
```bash
 [user1@server10 ~]$ podman image ls
 REPOSITORY                   TAG         IMAGE ID      CREATED        SIZE
 localhost/ubi9-simple-image  latest      a2797b06a129  2 minutes ago  220 MB
 registry.redhat.io/ubi9/ubi  latest      159a1e67312e  2 weeks ago    220 MB
```

Output: 
- downloaded image
- new custom image along with their image IDs, creation time, and size. 

- Do not remove the custom image yet as you will be using it to launch a container in the next section.

### Basic Container Management 

- Starting, stopping, listing, viewing information about, and deleting them. 
- Depending on the use case, containers can be launched in different ways. 
- They can:
	- Have a name assigned or be nameless
	- Have a terminal session opened for interaction
	- Execute an entry point command (the command specified at the launch time) and be auto-terminated right after.
	- etc. 
- Running containers can be stopped and restarted, or discarded if no longer needed. 
- The podman command is utilized to start containers and manage their lifecycle. 
- This command is also employed to list stopped and running containers, and view their details.

### Lab: Run, Interact with, and Remove a Named Container 

- Run a container based on the latest version of the RHEL 8 ubi available in the Red Hat container registry.
- Assign this container a name and run a few native Linux commands in a terminal window interactively. 
- Exit out of the container to mark the completion of the exercise.

1\. Launch a container using ubi8 (RHEL 8). Name this container rhel8-base-os and open a terminal session for interaction:
```bash
 [user1@server10 ~]$ podman run -ti --name rhel8-base-os ubi8
 Resolved "ubi8" as an alias (/etc/containers/registries.conf.d/001-rhel-shortnames.conf)
 Trying to pull registry.access.redhat.com/ubi8:latest...
 Getting image source signatures
 Checking if image destination supports signatures
 Copying blob 8694db102e5b done   | 
 Copying config 269749ad51 done   | 
 Writing manifest to image destination
 Storing signatures
 [root@30c7cccd8490 /]# 
```

- Downloaded the latest version of the specified image automatically even though no FQIN was provided. 
	- This is because it searched through the registries listed in the */etc/containers/registries.conf* file and retrieved the image from wherever it found it first (registry.access.redhat.com). 
- Opened a terminal session inside the container as the root user to interact with the containerized RHEL 8 OS. 
- The container ID is reflected as the hostname in the container's command prompt (last line in the output). This is an auto-generated ID.

- If you encounter any permission issues, delete the /etc/docker directory (if it exists) and try again.

2\. Run a few basic commands such as pwd, ls, cat, and date inside the
container for verification:
```bash
 [root@30c7cccd8490 /]# pwd
 /
 [root@30c7cccd8490 /]# ls
 bin   dev  home  lib64	     media  opt   root	sbin  sys  usr
 boot  etc  lib	 lost+found  mnt    proc  run	srv   tmp  var
 [root@30c7cccd8490 /]# cat /etc/redhat-release
 Red Hat Enterprise Linux release 8.10 (Ootpa)
 [root@30c7cccd8490 /]# date
 Thu Aug  1 21:09:13 UTC 2024
```

3\. Close the terminal session when done:
```bash
 [root@30c7cccd8490 /]# exit
 exit
 [user1@server10 ~]$ 
```

4\. Delete the container using podman rm:
```bash
 [user1@server10 ~]$ podman rm rhel8-base-os
 rhel8-base-os
```

Confirm the removal with podman ps.
```bash
 [user1@server10 ~]$ podman ps
 CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

### Lab: Run a Nameless Container and Auto-Remove it After Entry Point Command Execution 

- Launch a container based on the latest version of RHEL 7 ubi available in a Red Hat container registry.
	- This image provides the base operating system layer to deploy containerized applications. 
- Enter a Linux command at the command line for execution inside the container as an entry point command and the container should be automatically deleted right after that.

1\. Start a container using ubi7 (RHEL 7) and run ls as an entry point command. Remove the container as soon as the entry point command has finished running.
```bash
 [user1@server10 ~]$ podman run --rm ubi7 ls
 Resolved "ubi7" as an alias (/etc/containers/registries.conf.d/001-rhel-shortnames.conf)
 Trying to pull registry.access.redhat.com/ubi7:latest...
 Getting image source signatures
 Checking if image destination supports signatures
 Copying blob 7f2c2c4492b6 done   | 
 Copying config a084eb42a5 done   | 
 Writing manifest to image destination
 Storing signatures
 bin
 boot
 dev
 etc
 home
...
```

2\. Confirm the container removal with podman ps:
```bash
podman ps
 CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```
