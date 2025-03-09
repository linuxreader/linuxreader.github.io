# Appendix C. Getting Podman

[]{#C.htm#pgfId-1284764}Podman is []{#C.htm#marker-1284763}a great tool
for working with containers, but how do you get it installed on your
system? What packages are required to make it work? This appendix covers
installing or building Podman on your system.

## C.1 Installing Podman {#C.htm#heading_id_3 .fm-head}

[]{#C.htm#pgfId-1284769}Podman
[]{#C.htm#marker-1285884}[]{#C.htm#marker-1285885}is available for
almost all Linux distributions via their package managers. It is also
available on Mac, Windows, and FreeBSD platforms. The official podman.io
site,
[https://podman.io/getting-started/installation](https://podman.io/getting-started/installation){.url},
is regularly updated with new instructions on how to install Podman for
different distributions. Most of the content in this appendix originates
from the podman.io site, as seen in figure C.1.

::: figure
![](images/OEBPS/Images/C-01.png){.calibre18}

Figure C.1 Podman installation instructions website
:::

### []{#C.htm#pgfId-1284778}C.1.1 macOS {#C.htm#heading_id_4 .fm-head1}

[]{#C.htm#pgfId-1284782}Because
[]{#C.htm#marker-1284779}[]{#C.htm#marker-1284780}[]{#C.htm#marker-1284781}Podman
is a tool for running Linux containers, you can use it on a macOS
desktop only if you have access to a Linux box, running either locally
or remotely. To make this process somewhat easier, Podman includes a
command, `podman`{.fm-code-in-text} `machine`{.fm-code-in-text}, to
automatically manage VMs.

[]{#C.htm#pgfId-1284784}Homebrew

[]{#C.htm#pgfId-1284788}The
[]{#C.htm#marker-1284785}[]{#C.htm#marker-1284786}Mac client is
available through Homebrew ([https://brew.sh/](https://brew.sh/){.url}):

``` programlisting
$ brew install podman
```

[]{#C.htm#pgfId-1284790}Podman has the ability to install a VM and run a
Linux instance on your machine using the `podman`{.fm-code-in-text}
`machine`{.fm-code-in-text} command. On a Mac, you must execute the
following commands to install and start the Linux VM to successfully run
containers locally:

``` programlisting
$ podman machine init
$ podman machine start
```

[]{#C.htm#pgfId-1284794}Optionally, you can use the
`podman`{.fm-code-in-text} `system`{.fm-code-in-text}
`connection`{.fm-code-in-text} command[]{#C.htm#marker-1284793} to set
up SSH connections to remote Linux machines running the Podman service.

[]{#C.htm#pgfId-1284795}You can then verify the installation information
using

``` programlisting
$ podman info
```

[]{#C.htm#pgfId-1284797}The Podman command is running natively on the
Mac but communicating with an instance of Podman running within
[]{#C.htm#marker-1284798}[]{#C.htm#marker-1284799}the
[]{#C.htm#marker-1284800}[]{#C.htm#marker-1284801}[]{#C.htm#marker-1284802}VM.

### []{#C.htm#pgfId-1284804}C.1.2 Windows {#C.htm#heading_id_5 .fm-head1}

[]{#C.htm#pgfId-1284808}Because
[]{#C.htm#marker-1284805}[]{#C.htm#marker-1284806}[]{#C.htm#marker-1284807}Podman
is a tool for running Linux containers, you can use it on a Windows
desktop only if you have access to a Linux box, running either locally
or remotely. On Windows, Podman can also utilize the Windows Subsystem
for Linux system.

[]{#C.htm#pgfId-1284810}Windows Remote client

[]{#C.htm#pgfId-1284814}You
[]{#C.htm#marker-1286526}[]{#C.htm#marker-1286527}can retrieve the
latest Windows Remote client on the
[https://github.com/containers/podman/releases](https://github.com/containers/podman/releases){.url}
site.

[]{#C.htm#pgfId-1284815}Once installed, you can configure the Windows
Remote client to connect to a Linux server using the
`podman`{.fm-code-in-text} `system`{.fm-code-in-text}
`connection`{.fm-code-in-text} command[]{#C.htm#marker-1286531}. You can
find out more about this process at
[]{#C.htm#marker-1286532}[]{#C.htm#marker-1286533}[http://mng.bz/M0Kn](http://mng.bz/M0Kn){.url}.

[]{#C.htm#pgfId-1284821}Windows Subsystem for Linux (WSL) 2.0

[]{#C.htm#pgfId-1284824}See
[]{#C.htm#marker-1284822}[]{#C.htm#marker-1284823}Windows documentation
on installing WSL 2.0, and then pick a distribution that includes
Podman, including many described below. Alternatively, the
`podman`{.fm-code-in-text} `machine`{.fm-code-in-text}
`init`{.fm-code-in-text} command[]{#C.htm#marker-1284825} can bootstrap
it all for you by automatically installing and configuring WSL,
downloading and provisioning Fedora Core VM on it, and creating
corresponding SSH connections for the Podman
[]{#C.htm#marker-1284826}[]{#C.htm#marker-1284827}remote
[]{#C.htm#marker-1284828}[]{#C.htm#marker-1284829}[]{#C.htm#marker-1284830}client.

[]{#C.htm#pgfId-1284831}[Note]{.fm-callout-head} WSL 1.0 is not
supported.

### []{#C.htm#pgfId-1284833}C.1.3 Arch Linux and Manjaro Linux {#C.htm#heading_id_6 .fm-head1}

[]{#C.htm#pgfId-1284843}Arch
[]{#C.htm#marker-1284834}[]{#C.htm#marker-1284835}[]{#C.htm#marker-1284836}[]{#C.htm#marker-1284837}Linux
and Manjaro Linux use the `pacman`{.fm-code-in-text}
tool[]{#C.htm#marker-1284838} to install
[]{#C.htm#marker-1284839}[]{#C.htm#marker-1284840}[]{#C.htm#marker-1284841}[]{#C.htm#marker-1284842}software:

``` programlisting
$ sudo pacman -S podman
```

### []{#C.htm#pgfId-1284846}C.1.4 CentOS {#C.htm#heading_id_7 .fm-head1}

[]{#C.htm#pgfId-1284850}Podman
[]{#C.htm#marker-1284847}[]{#C.htm#marker-1284848}[]{#C.htm#marker-1284849}is
available in the default Extras repos for CentOS 7 and in the AppStream
repo for CentOS 8 and
[]{#C.htm#marker-1284851}[]{#C.htm#marker-1284852}[]{#C.htm#marker-1284853}Stream:

``` programlisting
$ sudo yum -y install podman
```

### []{#C.htm#pgfId-1284856}C.1.5 Debian {#C.htm#heading_id_8 .fm-head1}

[]{#C.htm#pgfId-1284864}The
[]{#C.htm#marker-1284857}[]{#C.htm#marker-1284858}[]{#C.htm#marker-1284859}`podman`{.fm-code-in-text}
package[]{#C.htm#marker-1284860} is available in the Debian 11
(bullseye) repositories and
[]{#C.htm#marker-1284861}[]{#C.htm#marker-1284862}[]{#C.htm#marker-1284863}later:

``` programlisting
$ sudo apt-get -y install podman
```

### []{#C.htm#pgfId-1284867}C.1.6 Fedora {#C.htm#heading_id_9 .fm-head1}

``` programlisting
$ sudo dnf -y install podman
```

### []{#C.htm#pgfId-1284876}C.1.7 Fedora-CoreOS, Fedora Silverblue {#C.htm#heading_id_10 .fm-head1}

[]{#C.htm#pgfId-1284885}Podman
[]{#C.htm#marker-1284877}[]{#C.htm#marker-1284878}[]{#C.htm#marker-1284879}[]{#C.htm#marker-1284880}comes
preinstalled on these distributions. There is no need to need to install
[]{#C.htm#marker-1284881}[]{#C.htm#marker-1284882}[]{#C.htm#marker-1284883}[]{#C.htm#marker-1284884}it.

### []{#C.htm#pgfId-1284887}C.1.8 Gentoo {#C.htm#heading_id_11 .fm-head1}

``` programlisting
$ sudo emerge app-emulation/podman
```

### []{#C.htm#pgfId-1284896}C.1.9 OpenEmbedded {#C.htm#heading_id_12 .fm-head1}

[]{#C.htm#pgfId-1284900}BitBake
[]{#C.htm#marker-1284897}[]{#C.htm#marker-1284898}[]{#C.htm#marker-1284899}recipes
for Podman and its dependencies are available in the meta-virtualization
layer ([http://mng.bz/aPzB](http://mng.bz/aPzB){.url}). Add the layer to
your OpenEmbedded build environment, and build Podman
[]{#C.htm#marker-1284902}[]{#C.htm#marker-1284903}[]{#C.htm#marker-1284904}using

``` programlisting
$ bitbake podman
```

### []{#C.htm#pgfId-1284907}C.1.10 openSUSE {#C.htm#heading_id_13 .fm-head1}

``` programlisting
sudo zypper install podman
```

### []{#C.htm#pgfId-1284916}C.1.11 openSUSE Kubic {#C.htm#heading_id_14 .fm-head1}

[]{#C.htm#pgfId-1284920}The
[]{#C.htm#marker-1284917}[]{#C.htm#marker-1284918}[]{#C.htm#marker-1284919}openSUSE
Kubic distribution has Podman built in. There is no need to need to
install
[]{#C.htm#marker-1284921}[]{#C.htm#marker-1284922}[]{#C.htm#marker-1284923}it.

### []{#C.htm#pgfId-1284925}C.1.12 Raspberry Pi OS arm64 {#C.htm#heading_id_15 .fm-head1}

[]{#C.htm#pgfId-1284929}The
[]{#C.htm#marker-1284926}[]{#C.htm#marker-1284927}[]{#C.htm#marker-1284928}Raspberry
Pi OS uses the standard Debian repositories, so it is fully compatible
with Debian's arm64
[]{#C.htm#marker-1284930}[]{#C.htm#marker-1284931}[]{#C.htm#marker-1284932}repository:

``` programlisting
$ sudo apt-get -y install podman
```

### []{#C.htm#pgfId-1284935}C.1.13 Red Hat Enterprise Linux {#C.htm#heading_id_16 .fm-head1}

[]{#C.htm#pgfId-1284937}RHEL7

[]{#C.htm#pgfId-1284945}Make
[]{#C.htm#marker-1284938}[]{#C.htm#marker-1284939}[]{#C.htm#marker-1284940}sure
[]{#C.htm#marker-1284941}[]{#C.htm#marker-1284942}you have a RHEL7
subscription, then enable the extras channel and install
[]{#C.htm#marker-1284943}[]{#C.htm#marker-1284944}Podman:

``` programlisting
$ sudo subscription-manager repos --enable=rhel-7-server-extras-rpms
$ sudo yum -y install podman
```

[]{#C.htm#pgfId-1284948}[Note]{.fm-callout-head} RHEL7 is no longer
receiving updates to the Podman package except for security fixes.

[]{#C.htm#pgfId-1284950}RHEL8

[]{#C.htm#pgfId-1284956}Podman
[]{#C.htm#marker-1284951}[]{#C.htm#marker-1284952}is included in the
`container-tools`{.fm-code-in-text} module[]{#C.htm#marker-1284953}
along with Buildah and
[]{#C.htm#marker-1284954}[]{#C.htm#marker-1284955}Skopeo:

``` programlisting
$ sudo yum module enable -y container-tools:rhel8
$ sudo yum module install -y container-tools:rhel8
```

[]{#C.htm#pgfId-1284960}RHEL9 (and beyond)

``` programlisting
$ sudo yum install podman
```

### []{#C.htm#pgfId-1284970}C.1.14 Ubuntu {#C.htm#heading_id_17 .fm-head1}

[]{#C.htm#pgfId-1284980}The
[]{#C.htm#marker-1284971}[]{#C.htm#marker-1284972}[]{#C.htm#marker-1284973}`podman`{.fm-code-in-text}
package[]{#C.htm#marker-1284974} is available in the official
repositories for Ubuntu 20.10
[]{#C.htm#marker-1284975}[]{#C.htm#marker-1284976}[]{#C.htm#marker-1284977}and
[]{#C.htm#marker-1284978}[]{#C.htm#marker-1284979}newer:

``` programlisting
$ sudo apt-get -y update
$ sudo apt-get -y install podman
```

## []{#C.htm#pgfId-1284984}C.2 Building from source code {#C.htm#heading_id_18 .fm-head}

[]{#C.htm#pgfId-1284987}I
[]{#C.htm#marker-1284985}[]{#C.htm#marker-1284986}usually advise people
to get the packaged versions of Podman because successfully running
Podman on Linux requires having additional tools installed, such as
conmon (container monitor[]{#C.htm#marker-1284988}),
`containernetworking-plugins`{.fm-code-in-text} (network
configuration[]{#C.htm#marker-1284989}), and
`containers-common`{.fm-code-in-text} (general
configuration[]{#C.htm#marker-1284990}). While the process of building
Podman from the source code is not very complicated, the list of
dependencies differs from one Linux distribution to another. You can
always check the latest instructions on the following Podman page:
[]{#C.htm#marker-1284991}[]{#C.htm#marker-1284992}[http://mng.bz/gRDE](http://mng.bz/gRDE){.url}.

## []{#C.htm#pgfId-1284995}C.3 Podman Desktop {#C.htm#heading_id_19 .fm-head}

[]{#C.htm#pgfId-1284998}There
[]{#C.htm#marker-1284996}[]{#C.htm#marker-1284997}is also a GUI, Podman
Desktop, for browsing, managing, and inspecting containers and images
from different container engines, available at
[https://github.com/containers/podman-desktop](https://github.com/containers/podman-desktop){.url}.
Podman Desktop offers the capability to connect to multiple engines at
the same time and provides a unified interface. This is a relatively new
project under heavy development, so expect some rough edges.

[]{#C.htm#pgfId-1285000}To provide some background, in September 2021,
Docker Inc. announced they will begin charging for the previously free
version of Docker Desktop on macOS. The Docker announcement has caused
many people to switch and look for a
[]{#C.htm#marker-1285001}[]{#C.htm#marker-1285002}replacement.

## []{#C.htm#pgfId-1285004}Summary {#C.htm#heading_id_20 .fm-head}

-   []{#C.htm#pgfId-1285005 .calibre17}Podman is a tool for running
    Linux containers, so it runs only on Linux.

-   []{#C.htm#pgfId-1285006 .calibre17}Podman is available in default
    package repositories of most major Linux distributions.

-   []{#C.htm#pgfId-1285007 .calibre17}Podman is available as a remote
    client on Mac and Windows, which connects to either a local or
    remote Linux box.

-   []{#C.htm#pgfId-1285008 .calibre17}Podman provides a special command
    for Linux VM management on macOS and Windows.

-   []{#C.htm#pgfId-1285009 .calibre17}Podman can be built from source
    code, but it requires many other tools to run successfully.

-   []{#C.htm#pgfId-1285011 .calibre17}Podman Desktop is an alternative
    for the popular Docker []{#C.htm#marker-1285010 .calibre17}Desktop.

[]{#D.htm}
