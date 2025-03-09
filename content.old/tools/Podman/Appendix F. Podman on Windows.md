# Appendix F. Podman on Windows

[]{#F.htm#pgfId-1289588}This appendix []{#F.htm#marker-1290859}covers

-   []{#F.htm#pgfId-1289589 .calibre17}Installing Podman on Windows
-   []{#F.htm#pgfId-1289591 .calibre17}Using the
    `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
    `init`{.fm-code-in-text} command[]{#F.htm#marker-1292811 .calibre17}
    to create a Fedora-based WSL 2 distribution running Podman
-   []{#F.htm#pgfId-1289593 .calibre17}Using the
    `podman`{.fm-code-in-text} command[]{#F.htm#marker-1290864
    .calibre17} on Windows to communicate with the Podman service
    running in the WSL 2 instance
-   []{#F.htm#pgfId-1289595 .calibre17}Starting or stopping the WSL 2
    instance with the `podman`{.fm-code-in-text}
    `machine`{.fm-code-in-text} start/stop
    commands[]{#F.htm#marker-1290867 .calibre17}

[]{#F.htm#pgfId-1289596}Podman is a tool for launching Linux containers.
Linux containers require a Linux kernel. As much as I'd love to convince
the world to move to the Linux desktop like me, most users work on Mac
and Windows operating systems---perhaps even you. If you use the Linux
desktop, hooray! And if you don't use a Windows machine, feel free to
skip this appendix.

[]{#F.htm#pgfId-1289598}Because you did not skip this appendix, I'll
assume you want to create Linux containers without having to
`ssh`{.fm-code-in-text} into a Linux machine and create the container
there. You likely want to use native software development tools and keep
their software local to their machines.

[]{#F.htm#pgfId-1289599}On Linux, Podman can be run as a service
allowing remote connections to launch containers. Then, from another
system, the `podman`{.fm-code-in-text} `--remote`{.fm-code-in-text}
command can be used to communicate with the remote Podman service to
launch a container.

[]{#F.htm#pgfId-1289600}Further, you can use `podman`{.fm-code-in-text}
`system`{.fm-code-in-text} `connection`{.fm-code-in-text} to configure
`podman`{.fm-code-in-text} `--remote`{.fm-code-in-text} to communicate
with a remote Linux machine running the Podman service over SSH, without
providing a URL to every command. The problem with all of this is that
someone has to configure the remote machine with the correct version of
the Podman service, and then you have to configure the SSH session.

[]{#F.htm#pgfId-1289601}Realizing that this experience is not optimal
for new users of Podman on a Windows desktop, Podman added a new
command: `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}. The
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text} command makes it
easy to create and manage a WSL 2-based Linux environment with Podman
preinstalled and configured. The Podman command on Windows is actually a
thinned down Podman command with only `podman`{.fm-code-in-text}
`--remote`{.fm-code-in-text} support[]{#F.htm#marker-1289602}. In this
appendix, you'll learn how to install Podman onto your Windows machine,
and then use the `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
commands to install, configure, and manage the WSL 2 instance.

## []{#F.htm#pgfId-1289603}F.1 First steps {#F.htm#heading_id_3 .fm-head}

[]{#F.htm#pgfId-1289604}The `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} command on Windows accepts all the same
commands as those used on Linux and Mac, with very similar behavior.
Still, there are a few differences, since the underlying backend on
Windows is based on Windows Subsystem for Linux
([https://docs.microsoft.com/en-us/windows/wsl/](https://docs.microsoft.com/en-us/windows/wsl/){.url})
instead of the VM, as in the other operating systems.

[]{#F.htm#pgfId-1289606}WSL 2 involves using the Windows Hyper-V
hypervisor; however, unlike a standard VM-based approach, WSL 2 shares
the same VM and Linux kernel instance across every Linux distribution
instance installed by the user. As an example, if you create two WSL 2
distributions, and you run `dmesg`{.fm-code-in-text} on each instance,
you see the same output, since the same kernel is hosting both.

[]{#F.htm#pgfId-1289607}[Note]{.fm-callout-head} WSL 1 doesn't work with
Podman; you must upgrade your Windows machine to an OS version that
supports WSL 2. For x64 systems, you will need Windows version 1,903 or
higher, with build 18,362 or higher. For arm64 systems, you will need
Windows version 2004 or higher, with build 19,041 or higher.

[]{#F.htm#pgfId-1289608}Running Podman with WSL 2 enables efficient
resource sharing between the host and all running instances in exchange
for less isolation. Keep in mind the `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} command shares the same kernel with any
other distributions you have running, and be cautious when manipulating
any kernel-level setting (e.g., network interfaces and netfilter policy)
in any distribution because you may unintentionally affect containers
executed by Podman.

### []{#F.htm#pgfId-1289610}F.1.1 Prerequisites {#F.htm#heading_id_4 .fm-head1}

[]{#F.htm#pgfId-1289612}Podman []{#F.htm#marker-1289611}for Windows
requires Windows 10 (build 19,041 or later) or Windows 11. As WSL 2 uses
a hypervisor, your computer must have virtualization instructions
enabled (e.g., Intel VT-x or AMD-V). Additionally, the hypervisor
requires second-level address translation
(SLAT[]{#F.htm#marker-1289613}[]{#F.htm#marker-1289614}) support.
Finally, your system must either have internet connectivity or an
offline copy of all software to be fetched by the
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}.

[]{#F.htm#pgfId-1289615}[Note]{.fm-callout-head} If at any time you
experience the errors 0x80070003 or 0x80370102 (or any error indicating
the VM cannot be started), you most likely have virtualization disabled.
Check your BIOS (or WSL 2 instance) settings to verify VT-x/AMD-V/WSL 2
instance and SLAT are enabled.

[]{#F.htm#pgfId-1289616}While not required, installing Windows Terminal
(as opposed to the standard CMD command application or PowerShell) is
strongly recommended (future versions of Windows 11 include it by
default). In addition to having modern terminal features, like
transparent cut and paste and tiled screens, it also offers direct WSL
and PowerShell integration, making it easy to switch between
environments. You can install it via the Windows store or
`winget`{.fm-code-in-text}:

``` programlisting
PS C:\Users\User> winget install Microsoft.WindowsTerminal
```

### []{#F.htm#pgfId-1289620}F.1.2 Installing Podman {#F.htm#heading_id_5 .fm-head1}

[]{#F.htm#pgfId-1289623}Installing
[]{#F.htm#marker-1289621}[]{#F.htm#marker-1289622}Podman is
straightforward. Go to the Podman site or the Podman GitHub repository,
and download the latest Podman MSI Windows Installer in the Releases
section (figure F.1;
[https://github.com/containers/podman/releases](https://github.com/containers/podman/releases){.url}).

::: figure
![](images/OEBPS/Images/F-01.png){.calibre18}

Figure F.1 Downloading and running the Podman installer
:::

[]{#F.htm#pgfId-1289631}After running the installer, open a terminal
(use the `wt`{.fm-code-in-text} command if you installed Windows
Terminal as recommended), and execute your first
`podman`{.fm-code-in-text} command[]{#F.htm#marker-1289632} (figure
F.2).

::: figure
![](images/OEBPS/Images/F-02.png){.calibre18}

Figure F.2 Podman commands running within the Windows Terminal
:::

[]{#F.htm#pgfId-1289641}Automatic WSL installation

[]{#F.htm#pgfId-1289644}If
[]{#F.htm#marker-1292567}[]{#F.htm#marker-1292568}WSL is not installed
on your Windows system, Podman installs it for you. Simply execute the
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`init`{.fm-code-in-text} command[]{#F.htm#marker-1292570} (as
illustrated in figure F.3) to create your first machine instance, and
Podman prompts you for permission to install WSL. The WSL install
process requires a reboot but resumes execution of the machine creation
process. (Be sure to wait a few minutes for the terminal to relaunch and
install.) If you prefer a manual installation, refer to the WSL
[]{#F.htm#marker-1292571}[]{#F.htm#marker-1292572}installation
[]{#F.htm#marker-1292573}[]{#F.htm#marker-1292574}guide:
[https://docs.microsoft.com/en-us/windows/wsl/install](https://docs.microsoft.com/en-us/windows/wsl/install){.url}.

::: figure
![](images/OEBPS/Images/F-03.png){.calibre18}

Figure F.3 The podman machine init starts the WSL installation.
:::

## []{#F.htm#pgfId-1289658}F.2 Using podman machine {#F.htm#heading_id_6 .fm-head}

[]{#F.htm#pgfId-1289661}The
[]{#F.htm#marker-1292506}[]{#F.htm#marker-1292507}setup and use of the
Linux environment is made easy through the use of
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text} commands. On
Windows, these commands create and manage a WSL 2 distribution,
including downloading a base Linux image and packages from the internet
and setting everything up for you. The WSL 2 distribution is
preconfigured with the Podman service, and SSH connection configuration
is automatically added to the `podman`{.fm-code-in-text}
`system`{.fm-code-in-text} `connection`{.fm-code-in-text}
datastore[]{#F.htm#marker-1292509}. The final result is the ability to
easily run Podman commands on your Windows desktop as if it was a Linux
system. Table F.1 lists all of the `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} commands used to manage the lifecycle of the
WSL 2-backed Linux environment.

[]{#F.htm#pgfId-1291337}Table F.1 `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} commands

+-----------------+-----------------------------------------------------+
| [               | []{#F.htm#pgfId-1291343}Description                 |
| ]{#F.htm#pgfId- |                                                     |
| 1291341}Command |                                                     |
+-----------------+-----------------------------------------------------+
| []{#F.htm#pgfI  | []{#F.htm#pgfId-1291347}Initialize a new WSL        |
| d-1291345}`init | 2-based machine instance.                           |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#F.htm# |                                                     |
| marker-1292520} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#F.htm#pgfI  | []{#F.htm#pgfId-1291351}List WSL 2 machines.        |
| d-1291349}`list |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#F.htm# |                                                     |
| marker-1292525} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#F.htm#pg    | []{#F.htm#pgfId-1291355}Remove a WSL 2 machine      |
| fId-1291353}`rm | instance.                                           |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#F.htm# |                                                     |
| marker-1292530} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#F.htm#pgf   | []{#F.htm#pgfId-1291359}Set an updatable WSL        |
| Id-1291357}`set | machine setting.                                    |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#F.htm# |                                                     |
| marker-1292535} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#F.htm#pgf   | []{#F.htm#pgfId-1291363}`ssh`{.fm-code-in-text1}    |
| Id-1291361}`ssh | into a WSL 2 machine instance. This is useful for   |
| `{.fm-code-in-t | entering the WSL 2 instance and running the native  |
| ext1}[]{#F.htm# | Podman commands. Some Podman commands are not       |
| marker-1292540} | supported remotely, and you might want to change    |
|                 | some configurations inside the WSL 2 instance.      |
+-----------------+-----------------------------------------------------+
| []{#F.htm#pgfId | []{#F.htm#pgfId-1291367}Start a WSL 2 machine       |
| -1291365}`start | instance.                                           |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#F.htm# |                                                     |
| marker-1292545} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#F.htm#pgfI  | []{#F.htm#pgfId-1291371}Stop a WSL 2 machine        |
| d-1291369}`stop | instance. If you are not running containers, you    |
| `{.fm-code-in-t | might want to stop to save system resources.        |
| ext1}[]{#F.htm# |                                                     |
| marker-1292550} |                                                     |
+-----------------+-----------------------------------------------------+

[]{#F.htm#pgfId-1289706}After installing Podman (see section F.1.2), the
first step is creating a WSL 2 machine instance on your system. You will
use the `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`init`{.fm-code-in-text} command[]{#F.htm#marker-1289707}, described in
the following section.

### []{#F.htm#pgfId-1289708}F.2.1 podman machine init {#F.htm#heading_id_7 .fm-head1}

[]{#F.htm#pgfId-1289713}As
[]{#F.htm#marker-1293020}[]{#F.htm#marker-1293021}[]{#F.htm#marker-1293022}shown
in figure F.4, you can use the `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `init`{.fm-code-in-text}
command[]{#F.htm#marker-1293023} to automate the installation of a WSL
2-based Linux environment that hosts a Podman service for running
containers. By default, `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `init`{.fm-code-in-text} downloads a known
compatible release of Fedora to create the WSL 2 instance
([https://getfedora.org](https://getfedora.org){.url}). Fedora is used,
since it is well integrated with Podman and is the operating system used
by most of the Podman core developers.

::: figure
![](images/OEBPS/Images/F-04.png){.calibre18}

Figure F.4 The podman machine init command creating the WSL 2
distribution and configuring SSH connections.
:::

[]{#F.htm#pgfId-1289714}[Note]{.fm-callout-head} In addition to the base
image, a number of packages must be downloaded and installed, which can
take several minutes to complete.

[]{#F.htm#pgfId-1289721}The following shows the condensed output from
running the `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`init`{.fm-code-in-text} command[]{#F.htm#marker-1289722}:

``` programlisting
PS C:\Users\User> podman machine init
Downloading VM image: fedora-35.20211125-x86_64.tar.xz: done
Extracting compressed file
Importing operating system into WSL (this may take 5+ minutes on a new WSL 
➥ install)...
Installing packages (this will take awhile)...
Fedora 35 - x86_64                                5.5 MB/s |  79 MB     00:14
Complete!
Configuring system...
Generating public/private ed25519 key pair.
Machine init complete
To start your machine run:
        podman machine start
```

[]{#F.htm#pgfId-1289739}Table F.2 explains the `init`{.fm-code-in-text}
options that allow you to customize the default
[]{#F.htm#marker-1289736}[]{#F.htm#marker-1289737}[]{#F.htm#marker-1289738}settings.
[]{#F.htm#marker-1289740}

[]{#F.htm#pgfId-1291462}Table F.2 `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `init`{.fm-code-in-text} command options

+-----------------+-----------------------------------------------------+
| []{#F.htm#pgfId | []{#F.htm#pgfId-1291468}Description                 |
| -1291466}Option |                                                     |
+-----------------+-----------------------------------------------------+
| []{#F.          | []{#F.htm#pgfId-1291472}Not used                    |
| htm#pgfId-12914 |                                                     |
| 70}`--cpus`{.fm |                                                     |
| -code-in-text1} |                                                     |
| `uint           |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#F.htm# |                                                     |
| marker-1291489} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#F.htm#p     | []{#F.htm#pgfId-1291476}Not used                    |
| gfId-1291474}`- |                                                     |
| -disk-size`{.fm |                                                     |
| -code-in-text1} |                                                     |
| `uint           |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#F.htm# |                                                     |
| marker-1291490} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#F.htm#pg    | []{#F.htm#pgfId-1291480}On Windows, this option     |
| fId-1291478}`-- | refers to the Fedora distribution number (e.g.,     |
| image-path`{.fm | 35). As with Linux and Mac, you can also specify an |
| -code-in-text1} | arbitrary URL or filesystem location with a custom  |
| `string         | image, but Podman expects a Fedora-derived layout.  |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#F.htm# |                                                     |
| marker-1291491} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#F.ht        | []{#F.htm#pgfId-1291484}Not used                    |
| m#pgfId-1291482 |                                                     |
| }`--memory`{.fm |                                                     |
| -code-in-text1} |                                                     |
| `integer        |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#F.htm# |                                                     |
| marker-1291492} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#            | []{#F.htm#pgfId-1291488}Determines whether this     |
| F.htm#pgfId-129 | machine instance should be rootful or rootless      |
| 1486}`--rootful |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#F.htm# |                                                     |
| marker-1291493} |                                                     |
+-----------------+-----------------------------------------------------+

[]{#F.htm#pgfId-1289773}[Note]{.fm-callout-head} The physical limits
specified in table F.2 (e.g., CPU, memory, and disk) are currently
ignored on Windows, since the Windows Subsystem for Linux (WSL) backend
dynamically resizes and shares resources across distributions. If you
need to constrain resources, you can configure those limits in your
users' .wslconfig file. However, they apply globally to all WSL 2
distros, since they share the same underlying VM.

### []{#F.htm#pgfId-1289774}F.2.2 Podman machine SSH configuration {#F.htm#heading_id_8 .fm-head1}

[]{#F.htm#pgfId-1289779}The
[]{#F.htm#marker-1289775}[]{#F.htm#marker-1289776}[]{#F.htm#marker-1289777}`podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `init`{.fm-code-in-text}
command[]{#F.htm#marker-1289778} creates an account within the WSL 2
instance. By default, the user in Fedora is user@localhost. Podman
configures SSH on the client machine and the new user account and root
within the WSL 2 instance. The SSH configuration allows for passwordless
SSH commands to the `user`{.fm-code-in-text}[]{#F.htm#marker-1289780}
and `root`{.fm-code-in-text} accounts[]{#F.htm#marker-1289781} from the
client. The `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`init`{.fm-code-in-text} command[]{#F.htm#marker-1289782} also
configures the Podman system connection information (see section 9.5.4).
The system connection database is configured for both the rootful user
and rootless user within the WSL 2 instance. If you do not have any
existing connections, the `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `init`{.fm-code-in-text}
command[]{#F.htm#marker-1289783} creates and sets as a default one of
the rootless user connections to your WSL 2 instance.

[]{#F.htm#pgfId-1289784}You can examine all of the connections using the
`podman`{.fm-code-in-text} `system`{.fm-code-in-text}
`connection`{.fm-code-in-text} list command[]{#F.htm#marker-1289785}.
The default connection, `podman-machine-default`{.fm-code-in-text}, is
the rootless connection:

``` programlisting
PS C:\Users\User> podman system connection ls
Name                         URI                           Identity 
➥ Default
podman-machine-default       ssh:/ /user@localhost:57051..  podman-machine-
➥ default  true
podman-machine-default-root  ssh:/ /root@localhost:57051..  podman-machine-
➥ default  false
```

[]{#F.htm#pgfId-1289792}Sometimes containers you want to execute require
root privileges and cannot run in rootless modes. You can change the
default connection to rootful by switching the default mode for the
created machine instance. Modify the default to rootful service using
the `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`set`{.fm-code-in-text} command[]{#F.htm#marker-1289793}:

``` programlisting
PS C:\Users\User> podman machine set --rootful
```

[]{#F.htm#pgfId-1289795}View the connections again to confirm the
default is now `podman-machine-default-root`{.fm-code-in-text}:

``` programlisting
PS C:\Users\User> podman system connection ls
Name                         URI                          Identity         
➥ Default
podman-machine-default       ssh:/ /user@localhost:57051.. 
➥ podman-machine-default  false
podman-machine-default-root  ssh:/ /root@localhost:57051.. 
➥ podman-machine-default  true
```

[]{#F.htm#pgfId-1289803}Now all Podman commands connect directly to the
Podman service running within the root account. Change the default
connection back to the rootless user using the
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`set`{.fm-code-in-text} command[]{#F.htm#marker-1289804} again:

``` programlisting
PS C:\Users\User> podman machine set --rootful=false
```

[]{#F.htm#pgfId-1289806}If you attempt to run a Podman container at this
point, it fails because the machine instance is not actually running.
You need to start the machine
[]{#F.htm#marker-1289807}[]{#F.htm#marker-1289808}[]{#F.htm#marker-1289809}instance.

### []{#F.htm#pgfId-1289810}F.2.3 Starting the WSL 2 instance {#F.htm#heading_id_9 .fm-head1}

[]{#F.htm#pgfId-1289815}Attempting
[]{#F.htm#marker-1289811}[]{#F.htm#marker-1289812}[]{#F.htm#marker-1289813}[]{#F.htm#marker-1289814}to
execute the `podman`{.fm-code-in-text} `version`{.fm-code-in-text}
command fails because the WSL 2 instance is not started:

``` programlisting
PS C:\Users\User> podman version
Cannot connect to Podman. Please verify your connection to the Linux system 
using `podman system connection list`, or try `podman machine init` and 
`podman machine start` to manage a new Linux Linux VM
Error: unable to connect to Podman. failed to create sshClient: Connection 
to bastion host (ssh:/ /root@localhost:38243/run/podman/podman.sock) 
failed.: dial tcp [::1]:38243: connect: connection refused
```

[]{#F.htm#pgfId-1289823}As the error points out, the virtualized Linux
environment (the WSL 2 machine instance) is not running and must be
started.

[]{#F.htm#pgfId-1289825}You start a single WSL 2 instance using the
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`start`{.fm-code-in-text} command[]{#F.htm#marker-1289824}. By default,
it starts the default WSL 2 instance:
`podman-machine-default`{.fm-code-in-text}. If you have multiple WSL 2
instances and want to start a different WSL 2 instance, you can specify
the optional machine name for the `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `start`{.fm-code-in-text}
command[]{#F.htm#marker-1289826}:

``` programlisting
PS C:\Users\User> podman machine start
Starting machine "podman-machine-default"
This machine is currently configured in rootless mode. If your containers
require root permissions (e.g. ports < 1024), or if you run into compatibility
issues with non-podman clients, you can switch using the following command:
        podman machine set --rootful
API forwarding listening on: npipe:////./pipe/docker_engine
Docker API clients default to this address. You do not need to set 
DOCKER_HOST.
Machine "podman-machine-default" started successfully
```

[]{#F.htm#pgfId-1289837}You are now ready to begin running Podman
commands on the host that communicates with the Podman service running
in the WSL 2 instance. Run the `podman`{.fm-code-in-text}
`version`{.fm-code-in-text} command[]{#F.htm#marker-1289838} to confirm
the client and server are configured correctly. If not, the Podman
commands instruct you on how to configure the system:

``` programlisting
PS C:\Users\User> podman version
Client:       Podman Engine
Version:      4.0.0-dev
API Version:  4.0.0-dev
Go Version:   go1.17.1
Git Commit:   bac389043f268e632c45fed7b4e88bdefd2d95e6-dirty
Built:        Wed Feb 16 00:33:20 2022
OS/Arch:      windows/amd64
Server:       Podman Engine
Version:      4.0.1
API Version:  4.0.1
Go Version:   go1.16.14
Built:      Fri Feb 25 13:22:13 2022
OS/Arch:    linux/amd64
```

[]{#F.htm#pgfId-1289853}Now you can use the Podman commands you learned
in the previous chapters directly on Windows. Make sure you understand
that Podman on Windows is equivalent to `podman`{.fm-code-in-text}
`--remote`{.fm-code-in-text} talking remotely to the Podman service
within the WSL 2
[]{#F.htm#marker-1289854}[]{#F.htm#marker-1289855}[]{#F.htm#marker-1289856}[]{#F.htm#marker-1289857}instance.

### []{#F.htm#pgfId-1289859}F.2.4 Using podman machine commands {#F.htm#heading_id_10 .fm-head1}

[]{#F.htm#pgfId-1289863}After
[]{#F.htm#marker-1289860}[]{#F.htm#marker-1289861}[]{#F.htm#marker-1289862}your
machine instance is running, you can perform Podman commands in your
PowerShell prompt as if running within
[]{#F.htm#marker-1289864}[]{#F.htm#marker-1289865}[]{#F.htm#marker-1289866}Windows:

``` programlisting
PS C:\Users\User> podman run ubi8-micro date
Thu Jan  6 05:09:59 UTC 2022
```

[]{#F.htm#pgfId-1289869}Stopping the WSL 2 instance

[]{#F.htm#pgfId-1289874}When
[]{#F.htm#marker-1289870}[]{#F.htm#marker-1289871}[]{#F.htm#marker-1289872}[]{#F.htm#marker-1289873}you
are done using containers on your system, you might want to shut down
the WSL 2 instance to save on system resources. Use the
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`stop`{.fm-code-in-text} command[]{#F.htm#marker-1289875} to shut down
all containers within the WSL 2 instance as well as the WSL 2 instance:

``` programlisting
PS C:\Users\User> podman machine stop
```

[]{#F.htm#pgfId-1289877}When you need to start using containers again,
launch the WSL 2 instance with the `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `start`{.fm-code-in-text}
[]{#F.htm#marker-1289878}[]{#F.htm#marker-1289879}[]{#F.htm#marker-1289880}[]{#F.htm#marker-1289881}command[]{#F.htm#marker-1289882}.

[]{#F.htm#pgfId-1289883}[Note]{.fm-callout-head} All of the
`podman`{.fm-code-in-text1} `machine`{.fm-code-in-text1} commands work
on Linux as well and allow you to test different versions of Podman at
the same time. Podman on Linux is the complete command; therefore, you
need to use the `--remote`{.fm-code-in-text1}
option[]{#F.htm#marker-1292703} to communicate with the Podman service
running within the WSL 2 instance launched by the
`podman`{.fm-code-in-text1} `machine`{.fm-code-in-text1} command. On
non-Linux platforms, the `--remote`{.fm-code-in-text1}
option[]{#F.htm#marker-1292704} is not required, since the client is
preconfigured in `--remote`{.fm-code-in-text1}
mode[]{#F.htm#marker-1292705}.

[]{#F.htm#pgfId-1289889}Listing machines

[]{#F.htm#pgfId-1289894}You
[]{#F.htm#marker-1289890}[]{#F.htm#marker-1289891}[]{#F.htm#marker-1289892}can
list the available machine instances using the
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`ls`{.fm-code-in-text} command[]{#F.htm#marker-1289893}. The values
returned by this command on Windows reflect current active usage, as
opposed to fixed resource limits, as is the case on Mac and Linux. Disk
storage reflects the disk space currently allocated to each machine
instance. The CPU values convey the number of CPUs on the Windows host
(unless limited by WSL) repeated per machine instance. The returned
memory values are also repeated (with slight variation from sampling
variability) and reflect the total amount of memory used by the Linux
kernel for all distributions in use (since it is shared). In other
words, for total usage, you sum the disk sizes but not memory and
[]{#F.htm#marker-1289895}[]{#F.htm#marker-1289896}[]{#F.htm#marker-1289897}CPU.

``` programlisting
PS C:\Users\User> podman machine ls
NAME                    VM TYPE     CREATED        LAST UP  CPUS        
➥ MEMORY      DISK SIZE
podman-machine-default  wsl         3 days ago     Running  4           
➥ 528.4MB     845.2MB
other                   wsl         4 minutes ago  Running  4           
➥ 524.5MB     778MB
```

[]{#F.htm#pgfId-1289905}Using Podman at the WSL prompt

[]{#F.htm#pgfId-1289910}In
[]{#F.htm#marker-1289906}[]{#F.htm#marker-1289907}[]{#F.htm#marker-1289908}addition
to the `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`ssh`{.fm-code-in-text} command[]{#F.htm#marker-1289909}, you can also
access the `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`guest`{.fm-code-in-text} using the WSL prompt. If you are running
Windows Terminal, the `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `guests`{.fm-code-in-text} (names prefixed
by Podman) are in the down-arrow dropdown. Alternatively, you can drop
into a WSL shell from any PowerShell prompt by using the
`wsl`{.fm-code-in-text} command and specifying the backing distribution
name. For example, the default instance created by
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`init`{.fm-code-in-text} is `podman-machine-default`{.fm-code-in-text}.
You can use either approach to manage the guest and execute Podman
commands inside a full-featured Linux shell
[]{#F.htm#marker-1289911}[]{#F.htm#marker-1289912}[]{#F.htm#marker-1289913}environment:

``` programlisting
PS C:\Users\User> wsl -d podman-machine-default
[root@WIN10PRO /]# podman version
Client:       Podman Engine
Version:      4.0.1
API Version:  4.0.1
Go Version:   go1.16.14
 
Built:      Fri Feb 25 13:22:13 2022
OS/Arch:    linux/amd64
```

[]{#F.htm#pgfId-1289924}Updating Fedora

[]{#F.htm#pgfId-1289929}Since
[]{#F.htm#marker-1289925}[]{#F.htm#marker-1289926}[]{#F.htm#marker-1289927}[]{#F.htm#marker-1289928}the
Windows machine implementation is based on Fedora, not Fedora CoreOS,
fixes and enhancements are not automatic. They must be explicitly
initiated on the guest using Fedora's package management command:
`dnf`{.fm-code-in-text}. Further, upgrading to a new version of Fedora
requires exporting any data you need to preserve and using
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`init`{.fm-code-in-text} to create a second machine instance (or
replacing the existing one after a `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `rm`{.fm-code-in-text}
command[]{#F.htm#marker-1289930}).

[]{#F.htm#pgfId-1289931}[Note]{.fm-callout-head} Currently, it is
difficult to run Fedora CoreOS inside of WSL, so it was decided to
default to Fedora. If Windows support for CoreOS changes in the future,
`podman`{.fm-code-in-text1} `machine`{.fm-code-in-text1} will move to
Fedora CoreOS.

[]{#F.htm#pgfId-1289932}As an example, to pull the latest packages for
the version of Fedora running on the p`odman`{.fm-code-in-text}
`guest`{.fm-code-in-text}, perform the following
[]{#F.htm#marker-1289933}[]{#F.htm#marker-1289934}[]{#F.htm#marker-1289935}[]{#F.htm#marker-1289936}command:

``` programlisting
PS C:\Users\User> podman machine ssh dnf upgrade -y
Warning: Permanently added '[localhost]:52581' (ED25519) to the list of 
known hosts.
Last metadata expiration check: 1:18:35 ago on Wed Jan  5 21:13:15 2022.
Dependencies resolved.
...
Complete!
```

[]{#F.htm#pgfId-1289945}Advanced stopping and restarting

[]{#F.htm#pgfId-1289951}Normally,
[]{#F.htm#marker-1289946}[]{#F.htm#marker-1289947}[]{#F.htm#marker-1289948}[]{#F.htm#marker-1289949}[]{#F.htm#marker-1289950}to
stop and restart Podman, you would use the respective
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`stop`{.fm-code-in-text}[]{#F.htm#marker-1289952} and
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`start`{.fm-code-in-text} commands[]{#F.htm#marker-1289953}. Stopping
the machine is the preferred approach, since system services can come to
a clean stop. However, in some cases, you may wish to force a hard
restart of the WSL facilities, including the shared Linux kernel, which
stays active even after a machine stop. To kill all processes associated
with a WSL distribution, use the `wsl`{.fm-code-in-text}
`--terminate`{.fm-code-in-text} `<machine`{.fm-code-in-text}
`name>`{.fm-code-in-text} command[]{#F.htm#marker-1289954}. To shut down
the Linux kernel, killing all running distributions, use the
`wsl`{.fm-code-in-text} `--shutdown`{.fm-code-in-text}
command[]{#F.htm#marker-1289955}. After these commands are issued, you
can use a standard `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} `start`{.fm-code-in-text}
command[]{#F.htm#marker-1289956} to relaunch
[]{#F.htm#marker-1289957}[]{#F.htm#marker-1289958}[]{#F.htm#marker-1289959}[]{#F.htm#marker-1289960}[]{#F.htm#marker-1289961}your
[]{#F.htm#marker-1289962}[]{#F.htm#marker-1289963}instance:

``` programlisting
PS C:\Users\User> wsl --shutdown
PS C:\Users\User> podman machine start
Starting machine...
Machine "podman-machine-default" started successfully
```

## []{#F.htm#pgfId-1289968}Summary {#F.htm#heading_id_11 .fm-head}

-   []{#F.htm#pgfId-1289969 .calibre17}Linux containers require a Linux
    kernel, meaning running containers on a Mac or Windows platform
    requires a VM running Linux.

-   []{#F.htm#pgfId-1289970 .calibre17}Podman on Windows does not run
    containers locally on Windows. The Podman command is actually
    `podman`{.fm-code-in-text} `--remote`{.fm-code-in-text}
    communicating with the Podman service running on a Linux machine
    backed by WSL 2.

-   []{#F.htm#pgfId-1289971 .calibre17}The `podman`{.fm-code-in-text}
    `machine`{.fm-code-in-text} `init`{.fm-code-in-text} pulls down and
    installs a virtual Linux environment onto your platform, which runs
    the Podman service.

-   []{#F.htm#pgfId-1289972 .calibre17}The `podman`{.fm-code-in-text}
    `machine`{.fm-code-in-text} `init`{.fm-code-in-text} command also
    sets up the SSH environment required to allow the Podman remote
    client to communicate with the Podman server inside of the WSL 2
    instance.

-   []{#F.htm#pgfId-1289973 .calibre17}Podman on Windows with WSL is the
    full Podman command. WSL is running the Podman commands under the
    Linux kernel, even though it feels like it is running natively on
    the Windows []{#F.htm#marker-1289974 .calibre17}machine.

[]{#index.htm}
