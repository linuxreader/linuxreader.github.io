# Appendix E. Podman on macOS

[]{#E.htm#pgfId-1286425}This appendix []{#E.htm#marker-1288008}covers

-   []{#E.htm#pgfId-1286426 .calibre17}Installing Podman on macOS
-   []{#E.htm#pgfId-1286428 .calibre17}Using the
    `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
    `init`{.fm-code-in-text} command[]{#E.htm#marker-1288011 .calibre17}
    to download a VM with a Podman service installed
-   []{#E.htm#pgfId-1286431 .calibre17}Using the
    `podman`{.fm-code-in-text} command[]{#E.htm#marker-1288014
    .calibre17} to communicate with the Podman service running in the VM
-   []{#E.htm#pgfId-1286433 .calibre17}Starting or stopping the VM with
    the `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
    start/stop commands[]{#E.htm#marker-1288016 .calibre17}

[]{#E.htm#pgfId-1286434}Podman is a tool for launching Linux containers.
Linux containers require a Linux kernel. As much as I'd love to convince
the world to move to the Linux Desktop like I use, most users work on
macOS and Windows operating systems---perhaps even you. If you use the
Linux Desktop, hooray! And if you don't use a macOS machine, feel free
to skip this appendix.

[]{#E.htm#pgfId-1286435}Because you did not skip this appendix, I'll
assume you want to create Linux containers without having to
`ssh`{.fm-code-in-text} into a Linux box. You likely want to use native
software development tools and keep development local.

[]{#E.htm#pgfId-1286436}One way to achieve this would be running Podman
as a service on a Linux box and using the `podman`{.fm-code-in-text}
`--remote`{.fm-code-in-text} command to communicate with this service.
Podman provides the `podman`{.fm-code-in-text}
`system`{.fm-code-in-text} `connection`{.fm-code-in-text}
command[]{#E.htm#marker-1286437} to configure how Podman communicates
with a Linux box. However, the problem with this approach is that it is
a meticulous process and requires a number of manual steps. Please refer
to this web page for an updated tutorial on this process:
[http://mng.bz/69ro](http://mng.bz/69ro){.url}.

[]{#E.htm#pgfId-1286438}A better way would be using a new command,
`podman`{.fm-code-in-text} `machine,`{.fm-code-in-text} which
encapsulates all these steps and improves your experience with managing
a Linux box to be used for `podman-remote`{.fm-code-in-text}. In this
appendix, you'll learn how to install Podman on macOS and then use the
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text} commands to
install, configure, and manage the VM to allow you to use the native
Podman client to launch containers.

[]{#E.htm#pgfId-1286440}The first step to launching Podman on a macOS is
installing it. The macOS client is available through Homebrew
([https://brew.sh/](https://brew.sh/){.url}).

[]{#E.htm#pgfId-1286442}[Note]{.fm-callout-head} Homebrew describes
itself as "\... the easiest and most flexible way to install the UNIX
tools Apple didn't include with macOS"
([https://docs.brew.sh/Manpage](https://docs.brew.sh/Manpage){.url}).

[]{#E.htm#pgfId-1286444}Homebrew is the best way to get open source
software installed on your macOS. If you do not currently have Homebrew
installed on your macOS, open a terminal, and install it with the
following command at the prompt:

``` programlisting
$ /bin/bash -c "$(curl -fsSL 
➥ https:/ /raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

[]{#E.htm#pgfId-1286448}Now run the following `brew`{.fm-code-in-text}
command[]{#E.htm#marker-1286447} to install a trimmed-down version of
Podman, with only `--remote`{.fm-code-in-text}
support[]{#E.htm#marker-1286449}, into the /opt/homebrew/bin directory:

``` programlisting
$ brew install podman
```

[]{#E.htm#pgfId-1286451}If you don't have access to a Linux VM or a
remote Linux server, Podman allows you to create a locally running VM
using the `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
command. It makes this easy by creating and configuring a VM with a
Podman service enabled.

[]{#E.htm#pgfId-1286452}[Note]{.fm-callout-head} If you have an existing
Linux machine, you can use the Podman system connection commands to set
up connections to those machines.

## []{#E.htm#pgfId-1286454}E.1 Using podman machine {#E.htm#heading_id_3 .fm-head}

[]{#E.htm#pgfId-1286457}The
[]{#E.htm#marker-1286455}[]{#E.htm#marker-1286456}`podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} commands allow you to pull a VM from the
internet and start it, stop it, or remove it. This VM is preconfigured
with the Podman service. Additionally, this command creates the SSH
connection and adds this information to the `podman`{.fm-code-in-text}
`system`{.fm-code-in-text} `connection`{.fm-code-in-text}
datastore[]{#E.htm#marker-1286458}, greatly simplifying the process of
setting up a `podman-remote`{.fm-code-in-text}
environment[]{#E.htm#marker-1286459}. Table E.1 lists all of the
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text} subcommands used
to manage the life cycle of the Podman virtual machine. The first step
is initializing a new VM in your system using the
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`init`{.fm-code-in-text} command[]{#E.htm#marker-1286460}, described in
the following section.

[]{#E.htm#pgfId-1288214}Table E.1 Podman machine commands

+-----------------+-----------------------------------------------------+
| [               | []{#E.htm#pgfId-1288220}Description                 |
| ]{#E.htm#pgfId- |                                                     |
| 1288218}Command |                                                     |
+-----------------+-----------------------------------------------------+
| []{#E.htm#pgfI  | []{#E.htm#pgfId-1288224}Initialize a new virtual    |
| d-1288222}`init | machine.                                            |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#E.htm# |                                                     |
| marker-1288245} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#E.htm#pgfI  | []{#E.htm#pgfId-1288228}List virtual machines.      |
| d-1288226}`list |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#E.htm# |                                                     |
| marker-1288246} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#E.htm#pg    | []{#E.htm#pgfId-1288232}Remove a virtual machine.   |
| fId-1288230}`rm |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#E.htm# |                                                     |
| marker-1288247} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#E.htm#pgf   | []{#E.htm#pgfId-1288236}`ssh`{.fm-code-in-text1}    |
| Id-1288234}`ssh | into a virtual machine. This is useful for entering |
| `{.fm-code-in-t | the virtual machine and running the native Podman   |
| ext1}[]{#E.htm# | commands. Some Podman commands are not supported    |
| marker-1288248} | remotely, and you might want to change some         |
|                 | configurations inside the VM.                       |
+-----------------+-----------------------------------------------------+
| []{#E.htm#pgfId | []{#E.htm#pgfId-1288240}Start a virtual machine.    |
| -1288238}`start |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#E.htm# |                                                     |
| marker-1288249} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#E.htm#pgfI  | []{#E.htm#pgfId-1288244}Stop a virtual machine. If  |
| d-1288242}`stop | you are not running containers, you might want to   |
| `{.fm-code-in-t | shut down the VM to save system resources.          |
| ext1}[]{#E.htm# |                                                     |
| marker-1288250} |                                                     |
+-----------------+-----------------------------------------------------+

### []{#E.htm#pgfId-1286501}E.1.1 podman machine init {#E.htm#heading_id_4 .fm-head1}

[]{#E.htm#pgfId-1286503}The `podman machine`{.fm-code-in-text}
`init`{.fm-code-in-text} command[]{#E.htm#marker-1286502} downloads and
configures a VM on your macOS system (figure E.1). By default, it
downloads the latest released `fedora-coreos`{.fm-code-in-text}
image[]{#E.htm#marker-1286504}
([https://getfedora.org/en/coreos](https://getfedora.org/en/coreos){.url})
if it was not downloaded before. Fedora CoreOS is a minimal operating
system designed to run containers.

[]{#E.htm#pgfId-1286505}[Note]{.fm-callout-head} The VM is relatively
large and takes a few minutes to download.

::: figure
![](images/OEBPS/Images/E-01.png){.calibre18}

Figure E.1 The podman `machine init`{.fm-code-in-text} command pulling
the VM and configuring the SSH
:::

[]{#E.htm#pgfId-1286513}Listing E.1 Podman downloading a VM onto the Mac
and preparing it for execution

``` programlisting
$ podman machine init
Downloading VM image: fedora-coreos-35.20211215.2.
➥ 0-qemu.x86_64.qcow2.xz                                    ❶
[=========>------------------------------------------------] 111.0MiB /
➥ 620.7MiB
Downloading VM image: fedora-coreos-35.20211215.2.0-qemu.x86_64.qcow2.xz: done
Extracting compressed file                                   ❷
```

[]{#E.htm#pgfId-1290305}[❶]{.fm-combinumeral} Podman finds and downloads
the latest fedora-coreos qcow image onto your system.

[]{#E.htm#pgfId-1290306}[❷]{.fm-combinumeral} After downloading the
image, Podman decompresses the image and configures qemu to be ready to
execute it. It also configures the SSH connection to the Podman system
connection datastore.

[]{#E.htm#pgfId-1286523}Podman preconfigures the VM with the amount of
memory, disk size, and CPUs for it to use. These values can be
configured using `init`{.fm-code-in-text} subcommand
options[]{#E.htm#marker-1286524}. Table E.2 describes these options.

[]{#E.htm#pgfId-1288316}Table E.2 Podman machine
`init`{.fm-code-in-text} command options

+-----------------+-----------------------------------------------------+
| []{#E.htm#pgfId | []{#E.htm#pgfId-1288322}Description                 |
| -1288320}Option |                                                     |
+-----------------+-----------------------------------------------------+
| []{#E.          | []{#E.htm#pgfId-1288326}Number of CPUs (the default |
| htm#pgfId-12883 | is 1)                                               |
| 24}`--cpus uint |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#E.htm# |                                                     |
| marker-1288339} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#E.htm#p     | []{#E.htm#pgfId-1288330}Disk size in GB (the        |
| gfId-1288328}`- | default is 10). This is an important setting to     |
| -disk-size uint | consider, since it limits the number of containers  |
| `{.fm-code-in-t | and images allowed to be used within the VM. If you |
| ext1}[]{#E.htm# | have the space, I recommend increasing the field.   |
| marker-1288340} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#E.htm#pgfI  | []{#E.htm#pgfId-1288334}Path to                     |
| d-1288332}`--im | `qcow`{.fm-code-in-text1} image (the default is     |
| age-path string | `testing`{.fm-code-in-text1}). Podman has two       |
| `{.fm-code-in-t | built-in Fedora CoreOS images it can pull:          |
| ext1}[]{#E.htm# | `testing`{.fm-code-in-text1} and                    |
| marker-1288341} | `stable`{.fm-code-in-text1}. You can also select    |
|                 | other OSs and VMs to download, but the VMs must     |
|                 | support CoreOS/Ignition files                       |
|                 | ([https://coreos.github.io/ign                      |
|                 | ition/](https://coreos.github.io/ignition/){.url}). |
+-----------------+-----------------------------------------------------+
| []{#E.htm#p     | []{#E.htm#pgfId-1288338}Memory in MB (the default   |
| gfId-1288336}`- | is 2048). The VM requires a certain amount of       |
| -memory integer | memory to run, and depending on the containers you  |
| `{.fm-code-in-t | want to run within the VM, you might need more.     |
| ext1}[]{#E.htm# |                                                     |
| marker-1288343} |                                                     |
+-----------------+-----------------------------------------------------+

[]{#E.htm#pgfId-1286554}Once `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `init`{.fm-code-in-text} finishes
downloading and installing the VM, you can view the VM with the
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`list`{.fm-code-in-text} command[]{#E.htm#marker-1286555}. Notice the
`*`{.fm-code-in-text} indicates the default VM to be used. The
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text} command currently
only supports running one VM at a time:

``` programlisting
$ podman machine list
NAME                    VM TYPE   CREATED      LAST UP     CPUS   
➥ MEMORY    DISK SIZE
podman-machine-default  qemu      2 minutes ago  2 minutes ago  1     
➥ 2.147GB   10.74GB
```

[]{#E.htm#pgfId-1286561}In the next section, you'll examine the
automatically created SSH connection.

### []{#E.htm#pgfId-1286563}E.1.2 Podman machine SSH configuration {#E.htm#heading_id_5 .fm-head1}

[]{#E.htm#pgfId-1286568}The
[]{#E.htm#marker-1286564}[]{#E.htm#marker-1286565}[]{#E.htm#marker-1286566}`podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `init`{.fm-code-in-text}
command[]{#E.htm#marker-1286567} provides the OS with the Ignition
config, which includes an SSH key for the core user. Then Podman adds
SSH connections on the client machine for the rootless and rootful
modes, configures the user account, and adds required packages and
configurations within the VM. The SSH configuration allows password-less
SSH commands to the `core`{.fm-code-in-text}[]{#E.htm#marker-1286569}
and `root`{.fm-code-in-text} accounts[]{#E.htm#marker-1286570} from the
client. The `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`init`{.fm-code-in-text} command[]{#E.htm#marker-1286571} also
configures the Podman system connection information (see section 9.5.4).
The system connection database is configured for both the rootful user
and the rootless user within the VM. If no previous connections are
present, the `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`init`{.fm-code-in-text} command[]{#E.htm#marker-1286572} will make the
newly created connection a default one.

[]{#E.htm#pgfId-1286573}You can examine all the connections using the
`podman`{.fm-code-in-text} `system`{.fm-code-in-text}
`connection`{.fm-code-in-text} `list`{.fm-code-in-text}
command[]{#E.htm#marker-1289513}. The default connection,
`podman-machine-default`{.fm-code-in-text}, is the rootless connection:

``` programlisting
$ podman system connection list
Name                       URI                                                     
Identity                                   Default
podman-machine-default
➥ ssh:/ /core@localhost:50107/run/user/501/podman/podman.sock  
➥ /Users/danwalsh/.ssh/podman-machine-default true
podman-machine-default-root  
➥ ssh:/ /root@localhost:50107/run/podman/podman.sock       
➥ /Users/danwalsh/.ssh/podman-machine-default  false
```

[]{#E.htm#pgfId-1286584}Sometimes containers you want to execute require
root privileges and cannot run in rootless modes. For this, you can
modify the system connection to default to the rootful service using the
`podman`{.fm-code-in-text} `system`{.fm-code-in-text}
`connection`{.fm-code-in-text} `default`{.fm-code-in-text} command:

``` programlisting
$ podman system connection default podman-machine-default-root
```

[]{#E.htm#pgfId-1286586}View the connections again to confirm the
default connection is now
`podman-machine-default-root`{.fm-code-in-text}:

``` programlisting
$ $ podman system connection list
Name                       URI                                                     
➥ Identity                                   Default
podman-machine-default   
➥ ssh:/ /core@localhost:50107/run/user/501/podman/podman.sock  
➥ /Users/danwalsh/.ssh/podman-machine-default  false
podman-machine-default-root 
➥ ssh:/ /root@localhost:50107/run/podman/podman.sock       
➥ /Users/danwalsh/.ssh/podman-machine-default true
n-machine-default ssh:/ /root@localhost:38243/run/podman/podman.sock
```

[]{#E.htm#pgfId-1286593}Now all Podman commands connect directly to the
Podman service running within the root account. Change the default
connection back to the rootless user using the Podman system connection
default command again:

``` programlisting
$ podman system connection default podman-machine-default
```

[]{#E.htm#pgfId-1286595}If you attempt to run a Podman container at this
point, it fails because the VM is not actually running. You need to
start the
[]{#E.htm#marker-1286596}[]{#E.htm#marker-1286597}[]{#E.htm#marker-1286598}VM.

### []{#E.htm#pgfId-1286600}E.1.3 Starting the VM {#E.htm#heading_id_6 .fm-head1}

[]{#E.htm#pgfId-1286605}After
[]{#E.htm#marker-1286601}[]{#E.htm#marker-1286602}[]{#E.htm#marker-1286603}[]{#E.htm#marker-1286604}adding
a VM and setting a specific connection as a default one, try running a
`podman`{.fm-code-in-text} command[]{#E.htm#marker-1286606}:

``` programlisting
$ podman version
Cannot connect to Podman. Please verify your connection to the Linux system 
using `podman system connection list`, or try `podman machine init` and 
`podman machine start` to manage a new Linux VM
Error: unable to connect to Podman. failed to create sshClient: Connection 
to bastion host (ssh:/ /root@localhost:38243/run/podman/podman.sock) 
failed.: dial tcp [::1]:38243: connect: connection refused
```

[]{#E.htm#pgfId-1286614}As the error points out, the VM is not running
and must be started.

[]{#E.htm#pgfId-1286616}You start a single VM using the
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`start`{.fm-code-in-text} command[]{#E.htm#marker-1286615}. Podman only
supports running one VM at a time. By default, the start command starts
the default VM. If you have multiple VMs and want to start a different
VM, you can specify the optional machine name:

``` programlisting
$ podman machine start
INFO[0000] waiting for clients...   
INFO[0000] listening tcp:/ /127.0.0.1:7777
INFO[0000] new connection from @ to /run/user/3267/podman/
➥ qemu_podman-machine-default.sock
Waiting for VM ...
macOShine "podman-machine-default" started successfully
```

[]{#E.htm#pgfId-1286624}You are now ready to begin running Podman
commands on the Linux box that runs the Podman service. Run the
`podman`{.fm-code-in-text} `version`{.fm-code-in-text}
command[]{#E.htm#marker-1286625} to confirm the client and server are
configured correctly. If not, the Podman commands should instruct you on
configuring the system:

``` programlisting
$ podman version
Client:
Version:      4.1.0
API Version:  4.1.0
Go Version:   go1.18.1
Built:        Thu May  5 16:07:47 2022
OS/Arch:      darwin/arm64
Server:
Version:      4.1.0
API Version:  4.1.0
Go Version:   go1.18
Built:        Fri May  6 12:16:38 2022
OS/Arch:      linux/arm64
```

[]{#E.htm#pgfId-1286639}Now you can use the Podman commands you learned
in the previous chapters directly on macOS. When you are done working
with containers in the VM, you probably should shut it down to save
[]{#E.htm#marker-1286640}[]{#E.htm#marker-1286641}[]{#E.htm#marker-1286642}[]{#E.htm#marker-1286643}resources.

[]{#E.htm#pgfId-1286644}[Note]{.fm-callout-head} Podman is supported on
M1 arm64 machines as well as the x86 platforms.
`podman`{.fm-code-in-text1} `machine`{.fm-code-in-text1}
`init`{.fm-code-in-text1} downloads the matching architecture VM,
allowing you to build images for that architecture. Support for building
images on other architectures is being worked on as of this writing.

### []{#E.htm#pgfId-1286646}E.1.4 Stopping the VM {#E.htm#heading_id_7 .fm-head1}

[]{#E.htm#pgfId-1286652}The
[]{#E.htm#marker-1286647}[]{#E.htm#marker-1286648}[]{#E.htm#marker-1286649}[]{#E.htm#marker-1286650}`podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `stop`{.fm-code-in-text}
command[]{#E.htm#marker-1286651} allows you to shut down all containers
within the VM as well as the VM itself:

``` programlisting
$ podman machine stop
```

[]{#E.htm#pgfId-1286658}When you need to start using containers again,
launch the VM with the
[]{#E.htm#marker-1286654}[]{#E.htm#marker-1286655}[]{#E.htm#marker-1286656}[]{#E.htm#marker-1286657}`podman`{.fm-code-in-text}
`machine`{.fm-code-in-text}
[]{#E.htm#marker-1286659}[]{#E.htm#marker-1286660}`start`{.fm-code-in-text}
command[]{#E.htm#marker-1286661}.

[]{#E.htm#pgfId-1286662}[Note]{.fm-callout-head} All of the
`podman`{.fm-code-in-text1} `machine`{.fm-code-in-text1} commands work
on Linux as well and allow you to test different versions of Podman at
the same time. Podman on Linux is the complete command; therefore, you
need to use the `--remote`{.fm-code-in-text1}
option[]{#E.htm#marker-1289943} to communicate with the Podman service
running within the VM launched by the Podman machine. On non-Linux
platforms the `--remote`{.fm-code-in-text1}
option[]{#E.htm#marker-1289944} is not required, since the client is
preconfigured in `--remote`{.fm-code-in-text1}
mode[]{#E.htm#marker-1289945}.

## []{#E.htm#pgfId-1286667}Summary {#E.htm#heading_id_8 .fm-head}

-   []{#E.htm#pgfId-1286668 .calibre17}Linux containers require a Linux
    kernel, meaning running containers on a macOS requires a virtual
    machine running Linux.

-   []{#E.htm#pgfId-1286669 .calibre17}Podman on a macOS is not running
    containers locally on the macOS. The Podman command is actually
    communicating with the Podman service running on a Linux machine.

-   []{#E.htm#pgfId-1286670 .calibre17}The `podman`{.fm-code-in-text}
    `machine`{.fm-code-in-text} `init`{.fm-code-in-text} command pulls
    down and installs a Fedora CoreOS VM onto your platform, which is
    running the Podman service.

-   []{#E.htm#pgfId-1286671 .calibre17}The `podman`{.fm-code-in-text}
    `machine`{.fm-code-in-text} `init`{.fm-code-in-text} command also
    sets up the SSH environment required to allow the Podman remote
    client to communicate with the Podman server inside the
    []{#E.htm#marker-1286672 .calibre17}VM.

[]{#F.htm}
