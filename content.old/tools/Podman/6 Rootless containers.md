# 6 Rootless containers 
[]{#06.htm#pgfId-1110286}This chapter []{#06.htm#marker-1111672}covers

-   []{#06.htm#pgfId-1110287 .calibre17}Why rootless mode is more secure
-   []{#06.htm#pgfId-1110288 .calibre17}How Podman works with the user
    and mount namespaces
-   []{#06.htm#pgfId-1110289 .calibre17}The architecture of Podman
    running in rootless mode

[]{#06.htm#pgfId-1110290}In this chapter, you will take a deep dive into
what is going on when running Podman in rootless mode. I believe it is
helpful to understand what is happening when you run rootless containers
and learn about the problems that running in rootless mode can cause.
With the introduction of containerized applications over the last few
years, certain highly secure environments were not able to take
advantage of the new technology.

[]{#06.htm#pgfId-1110292}High performance computing
(HPC[]{#06.htm#marker-1110291}) systems run the fastest computers in the
world. These tend to be at national labs and universities and deal with
high-security information. They also handle some of the most secure data
in the world and expressly forbid the use of rootful containers. HPC
systems deal with huge datasets, including artificial intelligence,
nuclear weapons, global weather patterns, and medical research. These
systems tend to have thousands of shared computers, and they need to be
locked down because of their multi-user shared environments. HPC
computing believes running daemons as root is too insecure. If a rogue
container process breaks out of confinement and gains root access, it
can access highly sensitive data. Administrators of HPC environments
couldn't use Open Container Initiative (OCI) containers until Podman
came along. The HPC community is now working to move to rootless Podman.

[]{#06.htm#pgfId-1110293}Similarly, large financial company
administrators do not allow users and developers access to root on their
shared computer systems, out of concern for the financial data involved.
The largest financial firms in the world were having difficulty fully
adopting OCI containers. Figure 6.1 shows that even though the Docker
client can be run as non-root, it connects to a root running daemon,
giving full root access to the host OS.

::: figure
![](images/OEBPS/Images/06-01.png){.calibre18}

[]{#06.htm#pgfId-1115608}Figure 6.1 Multiple users' workloads sharing
the same daemon running as root is inherently insecure.
:::

[]{#06.htm#pgfId-1110300}The bottom line is that allowing users on a
shared computing system to run container workloads accessing the same
root-running daemon is too insecure. Running each user's containers in
rootless mode under different users' accounts is more secure. Figure 6.2
shows multiple users running Podman independent of each other, without
any root access.

::: figure
![](images/OEBPS/Images/06-02.png){.calibre18}

[]{#06.htm#pgfId-1115646}Figure 6.2 Each workload running within its
unique user space is more secure.
:::

[]{#06.htm#pgfId-1110307}Linux was designed from the ground up with a
separation between privileged mode (rootful) and unprivileged mode
(rootless). In Linux almost all tasks run without being privileged.
Privileged operations are only required for modifications to the core
operating system. Almost all applications that run in containers, web
servers, databases, and user tools run without requiring root. The
applications do not modify core parts of the system. Sadly, most of the
images you will find on container registries are built to require root
privileges or at least start as root and then drop privileges.

[]{#06.htm#pgfId-1110308}In the corporate world, administrators are very
reluctant to give out root access to their users. If you receive a
corporate laptop from your employer, usually you are not granted any
root access. Administrators need to control what is installed on their
systems because of scale, and they need to be able to update hundreds to
thousands of machines at the same time, so controlling what is in the OS
is critical. If someone else is administering your machine, they need to
control who gets root access.

[]{#06.htm#pgfId-1110309}As a security person, I still flinch a little
when I see sudo without a password. When I first started working with
Docker, I was shocked that it was encouraging the use of the Docker
group, giving users full root access on the host, without a password.
The holy grail of hackers is to get a root exploit; this means the
hackers gain full control over the system.

[]{#06.htm#pgfId-1110310}Bottom line is that if you have a container
escape, as bad as that is, you are better off in rootless mode. This is
because the hackers have control over only nonprivileged processes, as
opposed to a root exploit, where they have full control over the system
and all of the data (ignoring other security mechanisms like SELinux).
Podman's design goals include the ability to run as many workloads as
possible without being root and push the core OS to make it easier for
you to run in this more secure mode.

## []{#06.htm#pgfId-1110312}6.1 How does rootless Podman work? {#06.htm#heading_id_3 .fm-head}

[]{#06.htm#pgfId-1110313}Have you ever wondered what happens behind the
scenes of a rootless Podman container? In chapter 2, all of the Podman
examples were running in rootless mode. Let's take a look at what
happens under the hood of rootless Podman containers. I'll explain each
component and then break down all of the steps involved.

[]{#06.htm#pgfId-1110314}[Note]{.fm-callout-head} Some of this section
is copied and rewritten from the "What Happens behind the Scenes of a
Rootless Podman Container?" blog
([https://www.redhat.com/sysadmin/behind-scenes-podman](https://www.redhat.com/sysadmin/behind-scenes-podman){.url}),
written by myself and coworkers Matthew Heon and Giuseppe Scrivano.

[]{#06.htm#pgfId-1110316}First, let's first clear out all storage, so
you can get a fresh environment, and then run a container on
quay.io/rhatdan/myimage. (Remember that the `podman`{.fm-code-in-text}
`rmi`{.fm-code-in-text} `--all`{.fm-code-in-text}
`--force`{.fm-code-in-text} command[]{#06.htm#marker-1110317} removes
all images and containers from storage.)

``` programlisting
$ podman rmi --all --force
Untagged: registry.access.redhat.com/ubi8/httpd-24:latest
Untagged: registry.access.redhat.com/ubi8-init:latest
Untagged: localhost/myimage:latest
Untagged: quay.io/rhatdan/myimage:latest
Deleted: d2244a4379d6f1981189d35154beaf4f9a17666ae3b9fba680ddb014eac72adc
Deleted: 82eb390304938f16dd707f32abaa8464af8d4a25959ab342e25696a540ec56b5
Deleted: 8773554aad01d4b8443d979cdd509e7b8fa88ddbc966987fe91690d05614c961
```

[]{#06.htm#pgfId-1110327}Now that you have a clean system, you need to
retrieve the application image, quay.io/ rhatdan/myimage, from the
container registry you pushed it to in chapter 2. In the following
command, re-create the application on your machine. The command pulls
the image back from the container registry and starts the
`myapp`{.fm-code-in-text} container[]{#06.htm#marker-1110328} on your
host.

``` programlisting
$ podman run -d -p 8080:8080 --name myapp quay.io/rhatdan/myimage
Trying to pull quay.io/rhatdan/myimage:latest...
...  
2f111737752dcbf1a1c7e15e807fb48f55362b67356fc10c2ade24964e99fa09
```

[]{#06.htm#pgfId-1110333}Now let's dig deep into what just happened when
you ran a rootless Podman container. The first thing that happened was
Podman needed to set up the user namespace. In the next section, I
explain why, and how it works.

### []{#06.htm#pgfId-1110335}6.1.1 Images contain content owned by multiple user identifiers (UIDs) {#06.htm#heading_id_4 .fm-head1}

[]{#06.htm#pgfId-1110340}In
[]{#06.htm#marker-1113923}[]{#06.htm#marker-1113924}[]{#06.htm#marker-1113925}Linux,
user identifiers (UIDs) and group identifiers
(GIDs[]{#06.htm#marker-1113926}) are assigned to processes and stored on
filesystem objects. The filesystem objects also have permission values
assigned to them. Linux controls the processes' access to the filesystem
based on these UIDs and GIDs. This access is called *discretionary
access control*
(DAC[]{#06.htm#marker-1113928}[]{#06.htm#marker-1113929}). When you log
in to a Linux machine, your rootless user processes run with a single
UID---say, `1000`{.fm-code-in-text}---but container images usually come
with multiple different UIDs in their image layers. Let's examine the
UIDs needed to run our image. In this example, you examine all the UIDs
defined within the container image by running another container.

[]{#06.htm#pgfId-1110343}In the following command, launch a container
with the quay.io/rhatdan/myimage image. You need to run the container as
root (`- -user=root`{.fm-code-in-text}) inside the container to examine
every file within the image.

``` programlisting
$ podman run --user=root --rm quay.io/rhatdan/myimage -- bash -c "find / 
➥ -mount -printf \”%U=%u\n\” | sort -un" 2>/dev/null
```

[]{#06.htm#pgfId-1110347}Since this is only a temporary container, use
the `--rm`{.fm-code-in-text} option[]{#06.htm#marker-1110346} to make
sure the container is removed when it finishes running. The container
runs a Bash script, which finds all of the UIDs and users associated
with every file/directory in the container. The script pipes the output
to show unique entries and redirects `stderr`{.fm-code-in-text} to
/dev/null to eliminate any errors.

``` programlisting
$ podman run --user=root --rm quay.io/rhatdan/myimage -- bash -c "find /
➥ -mount -printf \”%U=%u\n\” | sort -un" 2>/dev/null
0=root
48=apache
1001=default
65534=nobody
```

[]{#06.htm#pgfId-1110354}As you can see from the output, our container
image uses four different UIDs, shown in table 6.1.

[]{#06.htm#pgfId-1112391}Table 6.1 Unique UIDs required to run the
container image

+-----------------+-----------------+-----------------------------------+
| []{#06.htm#pg   | []{#06.htm#pgf  | []{                               |
| fId-1112397}UID | Id-1112399}Name | #06.htm#pgfId-1112401}Description |
+-----------------+-----------------+-----------------------------------+
| []              | []{#0           | []{#06.htm#pgfId-1112407}Owns     |
| {#06.htm#pgfId- | 6.htm#pgfId-111 | most of the content within the    |
| 1112403}`0`{.fm | 2405}`root`{.fm | container image                   |
| -code-in-text1} | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{             | []{#06.         | []{#06.htm#pgfId-1112413}Owns all |
| #06.htm#pgfId-1 | htm#pgfId-11124 | of the Apache content             |
| 112409}`48`{.fm | 11}`apache`{.fm |                                   |
| -code-in-text1} | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#0           | []{#06.h        | []{#06.htm#pgfId-1112419}Default  |
| 6.htm#pgfId-111 | tm#pgfId-111241 | user the container runs as        |
| 2415}`1001`{.fm | 7}`default`{.fm |                                   |
| -code-in-text1} | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#06          | []{#06.         | []{#06.htm#pgfId-1112425}Assigned |
| .htm#pgfId-1112 | htm#pgfId-11124 | to any UID that is not mapped     |
| 421}`65634`{.fm | 23}`nobody`{.fm | into the container                |
| -code-in-text1} | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+

[]{#06.htm#pgfId-1110389}For you to pull a container image to your home
directory, Podman needs to store at least three different UIDs:
`0`{.fm-code-in-text}, `48`{.fm-code-in-text}, and
`1001`{.fm-code-in-text}. Since the Linux kernel prevents nonprivileged
accounts from using more than a single UID, you are prevented from
creating files with different UIDs. You will need to take advantage of
the user namespace.

[]{#06.htm#pgfId-1110391}User namespace

[]{#06.htm#pgfId-1110396}Linux[]{#06.htm#marker-1110392}[]{#06.htm#marker-1110393}[]{#06.htm#marker-1110394}[]{#06.htm#marker-1110395}
supports the concept of user namespaces, which is a mapping of UID/GIDs
from the host to different UIDs and GIDs inside the namespace. Here is
how the man page describes it:

``` programlisting
$ man user namespaces
...
```

[]{#06.htm#pgfId-1110399}User namespaces isolate security-related
identifiers and attributes---in particular, user IDs and group IDs (see
`credentials(7)`{.fm-code-in-text}), the root directory, keys (see
`keyrings(7)`{.fm-code-in-text}), and capabilities (see
`capabilities(7)`{.fm-code-in-text}). A process's user and group IDs can
be different inside and outside a user namespace. In particular, a
process can have a normal, unprivileged user ID outside a user
namespace, while at the same time having a user ID of
`0`{.fm-code-in-text} inside the namespace; in other words, the process
has full privileges for operations inside the user namespace but is
unprivileged for operations outside the namespace.

[]{#06.htm#pgfId-1110400}Since your container requires more than one
UID, the Podman process first creates and enters a user namespace, where
it has access to more UIDs. Podman must also mount several filesystems
to run a container. These mount commands are not allowed outside a user
namespace (along with a mount namespace). Figure 6.3 shows the UIDs used
within a user namespace.

::: figure
![](images/OEBPS/Images/06-03.png){.calibre18}

[]{#06.htm#pgfId-1115684}Figure 6.3 User namespace mapping for
containers
:::

[]{#06.htm#pgfId-1110408}When I created my system, I used the
`useradd`{.fm-code-in-text} program[]{#06.htm#marker-1110407} to create
my account. It assigned me `3267`{.fm-code-in-text} as my UID and GID,
defined in /etc/passwd and /etc/group. It also allocated UID
`100000-1065535`{.fm-code-in-text}---additional UIDs and GIDs for me
defined in /etc/ subuid and /etc/subgid. Let's see the content of these
files:

``` programlisting
$ cat /etc/subuid
dwalsh:100000:65536
Testuser:165536:65536
$ cat /etc/subgid
dwalsh:100000:65536
Testuser:165536:65536
```

[]{#06.htm#pgfId-1110415}You can cat these files on your system, and
you'll see something similar. On my system I also have a
`testuser`{.fm-code-in-text} account; `useradd`{.fm-code-in-text} also
added UIDs/GIDs for that user, starting right after my allocation.

[]{#06.htm#pgfId-1110416}Within a user namespace, I have access to UIDs
`3267`{.fm-code-in-text} (my UID) as well as `100000`{.fm-code-in-text},
`100001`{.fm-code-in-text}, `100002`{.fm-code-in-text}, \...,
`165535`{.fm-code-in-text}, for a total of 65,537 UIDs. A root user can
modify the /etc/subuid and /etc/subgid files to increase or decrease
this number.

[]{#06.htm#pgfId-1110418}The `useradd`{.fm-code-in-text}
command[]{#06.htm#marker-1110417} starts at UID
`100000`{.fm-code-in-text} to allow you to have around 99,000 regular
users plus 1,000 UIDs reserved for system services on a Linux system.
The kernel supports more than 4 billion UIDs (2^32^ = 4,294,967,296).
Since `useradd`{.fm-code-in-text} allocates 65,537 per user, Linux can
support more than 60,000 users. The 65,536 (2^16^) number was picked
because up until the Linux kernel 2.4, this was the maximum number of
users on a Linux system. Let's look deeper into the user namespace.

[]{#06.htm#pgfId-1110419}Every process on a Linux system is in a
namespace, including the init process and systemd. These are the host
namespaces. Therefore, every process is in a user namespace. You can see
the user namespace mapping for your process by examining the /proc
filesystem. The /proc/PID/uid_map and /proc/PID/gid_map contain the user
namespace mappings for each process on the OS. /proc/self/uid_map
contains the UID map of the current process:

``` programlisting
$ cat /proc/self/uid_map 
      0        0 4294967295
```

[]{#06.htm#pgfId-1110422}The mapping means UIDs starting at UID
`0`{.fm-code-in-text} are mapped to UID `0`{.fm-code-in-text} for a
range of 4,294,967,295 UIDs.

[]{#06.htm#pgfId-1110423}Another way of looking at this mapping is

``` programlisting
UID 0->0, 1->1,...3267->3267,...,4294967294->4294967294.
```

[]{#06.htm#pgfId-1110425}Basically, there is no mapping, so root is
root. And my UID `3267`{.fm-code-in-text} is mapped to
`3267`{.fm-code-in-text}---itself.

[]{#06.htm#pgfId-1110426}Now let's enter the user namespace and see what
is mapped. Podman has a special command, `podman`{.fm-code-in-text}
`unshare`{.fm-code-in-text}, which allows you to enter a user namespace
without launching a container. It allows you to examine what is going on
within the user namespace, while still running as a regular process on
your system.

[]{#06.htm#pgfId-1110427}In the following command, I run
`podman`{.fm-code-in-text} `unshare`{.fm-code-in-text} to launch the cat
/proc/self/ uid_map within the default user namespace for my account:

``` programlisting
$ podman unshare cat /proc/self/uid_map 
       0     3267        1
       1   100000    65536
```

[]{#06.htm#pgfId-1110431}The mappings show that UID
`0`{.fm-code-in-text} is mapped to UID `3267`{.fm-code-in-text} (my UID)
for a range of `1`{.fm-code-in-text}. Then UID `1`{.fm-code-in-text} is
mapped to UID `100000`{.fm-code-in-text} for a range of
`65536`{.fm-code-in-text} UIDS.

[]{#06.htm#pgfId-1110432}Any UID not mapped to the user namespace is
reported within the user namespace as the `nobody`{.fm-code-in-text}
user[]{#06.htm#marker-1110433}. You saw this earlier when you searched
for the UIDs within the container image:

``` programlisting
$ podman run --user=root --rm quay.io/rhatdan/myimage -- bash -c "find / 
➥ -mount -exec stat -c %u=%U {} \; | sort -un" 2>/dev/null
0=root
48=apache
1001=default
65534=nobody
```

[]{#06.htm#pgfId-1110440}If you look at `/`{.fm-code-in-text} on the
host, you see it is owned by the real root:

``` programlisting
$ ls -l -ld /
dr-xr-xr-x. 18 root root 242 Sep 21 22:32 /
```

[]{#06.htm#pgfId-1110443}If you examine the same directory within the
user namespace, you see it is owned by the `nobody`{.fm-code-in-text}
user[]{#06.htm#marker-1110444}:

``` programlisting
$ podman unshare ls -ld /
dr-xr-xr-x. 18 nobody nobody 242 Sep 21 22:32 /
```

[]{#06.htm#pgfId-1110447}Since the host's UID `0`{.fm-code-in-text} is
not mapped into the user namespace, the kernel reports the UID as the
`nobody`{.fm-code-in-text} user[]{#06.htm#marker-1110448}. Processes
within the user namespace only have access to `nobody`{.fm-code-in-text}
files based on only the
`other`{.fm-code-in-text}[]{#06.htm#marker-1110449} or
`world`{.fm-code-in-text} permissions[]{#06.htm#marker-1110450}. In the
example that follows, you will launch a Bash script that shows the user
is root within the user namespace but sees /etc/passwd as owned by the
user `nobody`{.fm-code-in-text}. You can read the file with the grep
command because /etc/passwd is world readable. But the touch command
fails because even root cannot modify files owned by UIDs not mapped to
the user namespace:

``` programlisting
$ podman unshare bash -c "id ; ls -l /etc/passwd; grep dwalsh 
➥ /etc/passwd; touch /etc/passwd"
uid=0(root) gid=0(root) groups=0(root),65534(nobody)
-rw-r--r--. 1 nobody nobody 2942 Sep 28 07:08 /etc/passwd
dwalsh:x:3267:3267:Dan Walsh:/home/dwalsh:/bin/bash
touch: cannot touch '/etc/passwd': Permission denied
```

[]{#06.htm#pgfId-1110457}Looking at your home directory on the host
versus inside of the user namespace, you see that the same files are
reported as being owned by your UID:

``` programlisting
$ ls -ld /home/dwalsh
drwx------. 365 dwalsh dwalsh 24576 Sep 28 07:30 /home/dwalsh
```

[]{#06.htm#pgfId-1110460}Within the user namespace, they are owned by
root:

``` programlisting
$ podman unshare ls -ld /home/dwalsh
drwx------. 365 root root 24576 Sep 28 07:30 /home/dwalsh
```

[]{#06.htm#pgfId-1110463}By default, Podman maps your UID to root within
the user namespace. Podman defaults to root because, as I specified at
the beginning of this chapter, the majority of container images assume
they start with root.

[]{#06.htm#pgfId-1110464}I'll give one last example. Create a directory
and a file within the directory while in the user namespace, and use the
`chown`{.fm-code-in-text} command[]{#06.htm#marker-1110465} to change
the contents UIDs to `1:1`{.fm-code-in-text}:

``` programlisting
$ podman unshare bash -c "mkdir test;touch test/testfile; chown -R 1:1 test"
```

[]{#06.htm#pgfId-1110467}Outside the user namespace, you see the test
file is owned by UID `100000`{.fm-code-in-text}:

``` programlisting
$ ls -l test
total 0
-rw-r--r--. 1 100000 100000 0 Sep 28 07:53 testfile
```

[]{#06.htm#pgfId-1110471}When you create the test file and
`chown`{.fm-code-in-text} it to UID/GID `1:1`{.fm-code-in-text} within
the user namespace, the on-disk owner is actually UID
`100000`{.fm-code-in-text}/`100000`{.fm-code-in-text}. Remember, within
the user namespace, UID `1`{.fm-code-in-text} is mapped to UID
`100000`{.fm-code-in-text}, so when you create a UID
`1`{.fm-code-in-text} file within the user namespace, the OS actually
creates UID `100000`{.fm-code-in-text}.

[]{#06.htm#pgfId-1110472}If you attempt to remove the file outside of
the user namespace, you get an error:

``` programlisting
$ rm -rf test
rm: cannot remove 'test/testfile': Permission denied
```

[]{#06.htm#pgfId-1110475}Outside the user namespace, you have access to
only your UID; you don't have access to the additional UIDs.

[]{#06.htm#pgfId-1110476}[Note]{.fm-callout-head} In section 3.1.2, I
showed how user namespace mappings can be problematic with container
volumes and discussed ways you can handle them.

[]{#06.htm#pgfId-1110477}Reentering the user namespace, you can remove
the file:

``` programlisting
$ podman unshare rm -rf test
```

[]{#06.htm#pgfId-1110479}Hopefully, you are starting to get a feel for
the user namespace; the `podman`{.fm-code-in-text}
`unshare`{.fm-code-in-text} command[]{#06.htm#marker-1110480} makes it
easy to explore your system within the user namespace and understand
what is happening in rootless containers. When running a rootless
container, Podman needs more than just to run as root; it also needs
access to some of the special powers of root called Linux capabilities.

[]{#06.htm#pgfId-1110481}In Linux, the root processes actually are not
all equally powerful. Linux breaks root privileges into a series of
Linux capabilities. A root process with all Linux capabilities is all
powerful, while a root process without Linux capabilities is not allowed
to manipulate a lot of the system. For example, it cannot read non-root
files, unless those files have permission flags that allow all UIDs on
the system to read (world readable).

[]{#06.htm#pgfId-1110482}Let's see how capabilities work with the user
namespace:

``` programlisting
$ man capabilities
...
DESCRIPTION
For the purpose of performing permission checks, traditional UNIX 
implementations distinguish two categories of processes: privileged 
processes (whose effective user ID is 0, referred to as superuser or root), 
and unprivileged processes (whose effective UID is nonzero). Privileged 
processes bypass all kernel permission checks, while unprivileged processes 
are subject to full permission checking based on the process's credentials 
(usually: effective UID, effective GID, and supplementary group list).
Starting with kernel 2.2, Linux divides the privileges traditionally 
associated with superuser into distinct units, known as capabilities, which 
can be independently enabled and disabled. Capabilities are a per-thread 
attribute.
```

[]{#06.htm#pgfId-1110498}Linux currently has around 40 capabilities.
Examples include
`CAP_SETUID`{.fm-code-in-text}[]{#06.htm#marker-1110497} and
`CAP_SETGID`{.fm-code-in-text}[]{#06.htm#marker-1110499}, which allow
processes to change their UIDs and GIDs.
`CAP_NET_ADMIN`{.fm-code-in-text}[]{#06.htm#marker-1110500} allows you
to manage the network stack.

[]{#06.htm#pgfId-1110502}Another capability called
`CAP_CHOWN`{.fm-code-in-text}[]{#06.htm#marker-1110501} allows processes
to change the UID/GID of files on disk. In the preceding example, when
you `chown`{.fm-code-in-text}ed the test directory to
`1:1`{.fm-code-in-text}, you used the `CAP_CHOWN`{.fm-code-in-text}
capability[]{#06.htm#marker-1110503} within the user namespace:

``` programlisting
$ podman unshare bash -c "mkdir test;touch test/testfile; chown -R 1:1 test"
```

[]{#06.htm#pgfId-1110505}When you run within a user namespace, you are
using namespaced capabilities. The root user within your user namespace
has these capabilities beyond the UIDs and GIDs defined within the
namespace. Processes with the namespaced capability,
`CAP_CHOWN`{.fm-code-in-text}, are allowed to `chown`{.fm-code-in-text}
files owned within your user namespace to UIDs that are also within the
user namespace. If a process within a user namespace attempts to
`chown`{.fm-code-in-text} a file not mapped to the user namespace, owned
by the `nobody`{.fm-code-in-text} user, the process is denied
permission. Likewise, a process attempting to `chown`{.fm-code-in-text}
a file with a UID not defined within the user namespace also gets
denied. Similarly, the `CAP_SETUID`{.fm-code-in-text} capability only
allows processes to change UIDs to those defined within the user
namespace.

[]{#06.htm#pgfId-1110508}When Podman runs a container, it needs to mount
several filesystems for the container. In Linux, the
`CAP_SYS_ADMIN`{.fm-code-in-text} capability[]{#06.htm#marker-1110509}
is required for mounting filesystems. From a security point of view,
mounting filesystems can be a dangerous thing to do on Linux. The kernel
adds additional controls on which types of filesystems can be mounted
and requires your user-namespaced processes to also be in a unique mount
namespace. In chapter 10, you will see how Podman limits the number of
Linux capabilities available to the namespaced root within a
[]{#06.htm#marker-1110510}[]{#06.htm#marker-1110511}[]{#06.htm#marker-1110512}[]{#06.htm#marker-1110513}container.

[]{#06.htm#pgfId-1110515}Mount namespace

[]{#06.htm#pgfId-1110520}Mount
[]{#06.htm#marker-1110516}[]{#06.htm#marker-1110517}[]{#06.htm#marker-1110518}[]{#06.htm#marker-1110519}namespaces
allow processes within them to mount filesystems, where the mount points
are not seen by processes outside the mount namespace. Inside a mount
namespace, you can mount a `tmpfs`{.fm-code-in-text} on /tmp, which
blocks the processes within the namespaces view of /tmp. Outside the
mount namespace, processes still see the original mount and files within
/tmp, but they do not see your mount.

[]{#06.htm#pgfId-1110521}In rootless containers, Podman needs to mount
the content in the container images as well as /proc, /sys, devices from
/dev, and some `tmpfs`{.fm-code-in-text} filesystems. For that, Podman
needs to create a mount namespace:

``` programlisting
$ man mount namespaces
...
Mount namespaces provide isolation of the list of mount points seen by the 
processes in each namespace instance. Thus, the processes in each of the 
mount namespace instances see distinct single-directory hierarchies.
```

[]{#06.htm#pgfId-1110528}When you execute the `podman`{.fm-code-in-text}
`unshare`{.fm-code-in-text} command,[]{#06.htm#marker-1110527} you are
actually entering a different mount namespace as well as a different
user namespace.

[]{#06.htm#pgfId-1110529}You can examine a process's namespaces by
listing the /proc/self/ns/ directory as follows:

``` programlisting
$ ls -l /proc/self/ns/user /proc/self/ns/mnt
lrwxrwxrwx. 1 dwalsh dwalsh 0 Sep 28 09:17 /proc/self/ns/mnt -> 
➥ 'mnt:[4026531840]'
lrwxrwxrwx. 1 dwalsh dwalsh 0 Sep 28 09:17 /proc/self/ns/user -> 
➥ 'user:[4026531837]'
```

[]{#06.htm#pgfId-1110535}Notice that when you enter the user namespace
and mount namespace, the identifiers change:

``` programlisting
$ podman unshare ls -l /proc/self/ns/user /proc/self/ns/mnt
lrwxrwxrwx. 1 root root 0 Sep 28 09:17 /proc/self/ns/mnt -> 
➥ 'mnt:[4026533087]'
lrwxrwxrwx. 1 root root 0 Sep 28 09:17 /proc/self/ns/user -> 
➥ 'user:[4026533086]'
```

[]{#06.htm#pgfId-1110541}In the following test, you can create a file on
/tmp and then attempt to bind mount it onto /etc/shadow. Outside the
namespaces, the kernel rightly prevents you from mounting the file, as
you can see in the following output:

``` programlisting
$ echo hello > /tmp/testfile
$ mount --bind /tmp/testfile /etc/shadow
mount: /etc/shadow: must be superuser to use mount.
 
Once you enter the user namespace and mount namespace, your namespaced 
process can successfully mount over the /etc/shadow file. You can see when 
you run the following command that /etc/shadow is actually modified:
$ podman unshare bash -c "mount -o bind /tmp/testfile /etc/shadow; cat 
/etc/shadow" 
hello
```

[]{#06.htm#pgfId-1110556}Once you exit the `unshare`{.fm-code-in-text},
everything is back to
[]{#06.htm#marker-1110552}[]{#06.htm#marker-1110553}[]{#06.htm#marker-1110554}[]{#06.htm#marker-1110555}normal.

[]{#06.htm#pgfId-1110558}User namespace and mount namespace

[]{#06.htm#pgfId-1110564}As
[]{#06.htm#marker-1110559}[]{#06.htm#marker-1110560}[]{#06.htm#marker-1110561}[]{#06.htm#marker-1110562}[]{#06.htm#marker-1110563}you
saw previously, when you over-mount the /etc/shadow file, you might
trick some `setuid`{.fm-code-in-text} applications, like /bin/su or
/bin/sudo, into giving you full root. The reason rootless users are not
allowed to mount filesystems is to prevent this type of attack.

[]{#06.htm#pgfId-1110565}As you have seen, the separate mount namespace
prevents you from affecting the host's view of the system, and anything
you mount is seen only within the mount namespace. Within the user
namespace, the container already has a namespaced root. Attacks on your
mount points can be escalated to root only within the user
namespace---not real root on the host. Containerized processes cannot
change their UID (`setuid`{.fm-code-in-text}) to real root or any other
UID not mapped into the user namespace.

[]{#06.htm#pgfId-1110566}Even with the namespaces, the Linux kernel only
allows you to mount certain filesystem types. Many filesystem types are
too dangerous to allow for rootless users because they gain access to
sensitive parts of the kernel. I work with filesystem kernel engineers
to see if there are ways to lock down other filesystem types that could
be allowed to be mounted in rootless mode, without affecting the
security of the system.

[]{#06.htm#pgfId-1110567}As of kernel 5.13, the kernel engineers added
native overlay mounts to the list of allowed mounts. The filesystem
types currently allowed are
[]{#06.htm#marker-1110568}[]{#06.htm#marker-1110569}[]{#06.htm#marker-1110570}[]{#06.htm#marker-1110571}[]{#06.htm#marker-1110572}listed
[]{#06.htm#marker-1110573}[]{#06.htm#marker-1110574}[]{#06.htm#marker-1110575}in
table 6.2.

[]{#06.htm#pgfId-1112521}Table 6.2 Filesystem mounts currently supported
in rootless mode

+-----------------+-----------------------------------------------------+
| []{#06.htm#pgfI | []{#06.htm#pgfId-1112527}Description                |
| d-1112525}Mount |                                                     |
| type            |                                                     |
+-----------------+-----------------------------------------------------+
| [               | []{#06.htm#pgfId-1112531}Used heavily in rootless   |
| ]{#06.htm#pgfId | containers. Because rootless users are not allowed  |
| -1112529}`bind` | to create devices, Podman `bind`{.fm-code-in-text1} |
| {.fm-code-in-te | mounts /dev on the host into the container. Podman  |
| xt1}[]{#06.htm# | also uses `bind`{.fm-code-in-text1} mounts to       |
| marker-1112572} | obscure content within the host filesystem from     |
|                 | containers. Podman also `bind`{.fm-code-in-text1}   |
|                 | mounts /dev/null over files in /proc and /sys to    |
|                 | hide content. Volume mounts, described in chapter   |
|                 | 3, also use `bind`{.fm-code-in-text1} mounts.       |
+-----------------+-----------------------------------------------------+
| []{#0           | []{#06.htm#pgfId-1112535}Filesystem for the Android |
| 6.htm#pgfId-111 | binder IPC mechanism. It is not supported by        |
| 2533}`binderfs` | Podman.                                             |
| {.fm-code-in-te |                                                     |
| xt1}[]{#06.htm# |                                                     |
| marker-1112573} |                                                     |
+-----------------+-----------------------------------------------------+
| []{             | []{#06.htm#pgfId-1112539}Virtual filesystem mounted |
| #06.htm#pgfId-1 | at /dev/pts. It contains device files used for      |
| 112537}`devpts` | terminal emulators                                  |
| {.fm-code-in-te |                                                     |
| xt1}[]{#06.htm# |                                                     |
| marker-1112574} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#0           | []{#06.htm#pgfId-1112543}Kernel filesystem used to  |
| 6.htm#pgfId-111 | manipulate cgroups; rootless containers can use     |
| 2541}`cgroupfs` | `cgroupfs`{.fm-code-in-text1} to manipulate cgroups |
| {.fm-code-in-te | in cgroups v2. On v1 this is not supported. This is |
| xt1}[]{#06.htm# | mounted at /sys/fs/cgroups.                         |
| marker-1112575} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#0           | []{#06.htm#pgfId-1112547}Used to mount container    |
| 6.htm#pgfId-111 | images using the                                    |
| 2545}`FUSE`{.fm | `fuse-overlayfs`{.fm-code-in-text1} in rootless     |
| -code-in-text1} | mode. Prior to kernel 5.13, this was the only way   |
| []{#06.htm#     | to use an overlay filesystem in rootless mode.      |
| marker-1112576} |                                                     |
+-----------------+-----------------------------------------------------+
| []{             | []{#06.htm#pgfId-1112551}Mounted at /proc within    |
| #06.htm#pgfId-1 | the container. You can examine processes within the |
| 112549}`procfs` | container.                                          |
| {.fm-code-in-te |                                                     |
| xt1}[]{#06.htm# |                                                     |
| marker-1112577} |                                                     |
+-----------------+-----------------------------------------------------+
| []{             | []{#06.htm#pgfId-1112555}Implements the POSIX       |
| #06.htm#pgfId-1 | message queues API. Podman mounts this filesystem   |
| 112553}`mqueue` | at /dev/mqueue.                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#06.htm# |                                                     |
| marker-1112578} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#06          | []{#06.htm#pgfId-1112559}Used for mounting the      |
| .htm#pgfId-1112 | image. Performs better in the                       |
| 557}`overlayfs` | `fuse-overlayfs`{.fm-code-in-text1} filesystem. In  |
| {.fm-code-in-te | certain use cases, it provides benefits over native |
| xt1}[]{#06.htm# | overlay, such as NFS home directories.              |
| marker-1112579} |                                                     |
+-----------------+-----------------------------------------------------+
| []              | []{#06.htm#pgfId-1112563}Dynamically resizable,     |
| {#06.htm#pgfId- | ram-based Linux filesystem, currently not used with |
| 1112561}`ramfs` | Podman.                                             |
| {.fm-code-in-te |                                                     |
| xt1}[]{#06.htm# |                                                     |
| marker-1112580} |                                                     |
+-----------------+-----------------------------------------------------+
| []              | []{#06.htm#pgfId-1112567}Mounted at /sys.           |
| {#06.htm#pgfId- |                                                     |
| 1112565}`sysfs` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#06.htm# |                                                     |
| marker-1112581} |                                                     |
+-----------------+-----------------------------------------------------+
| []              | []{#06.htm#pgfId-1112571}Used to obscure kernel     |
| {#06.htm#pgfId- | filesystem directories from containers in /proc and |
| 1112569}`tmpfs` | /sys.                                               |
| {.fm-code-in-te |                                                     |
| xt1}[]{#06.htm# |                                                     |
| marker-1112582} |                                                     |
+-----------------+-----------------------------------------------------+

## []{#06.htm#pgfId-1110640}6.2 Rootless Podman under the covers {#06.htm#heading_id_5 .fm-head}

[]{#06.htm#pgfId-1110641}Now that you have some understanding of how the
user namespace and mount namespace work and why they are needed, let's
dig deeper into what Podman does when it runs a container. The first
time you run a Podman container after logging in, Podman reads the
/etc/subuid and /etc/subgid files, looking for your username or UID.
Once Podman finds the entry, it uses the contents as well as your
current UID/GID to generate a user namespace for you. Podman then
launches the `podman`{.fm-code-in-text} `pause`{.fm-code-in-text}
process[]{#06.htm#marker-1110642} to hold open the user and mount
namespaces (figure 6.4).

::: figure
![](images/OEBPS/Images/06-04.png){.calibre18}

[]{#06.htm#pgfId-1115722}Figure 6.4 Podman launches the pause process to
hold open the user and mount namespaces.
:::

[]{#06.htm#pgfId-1110649}Users commonly report that after they run
Podman containers, they see a `podman`{.fm-code-in-text} process still
running when they run the following command:

``` programlisting
$ ps -e | grep podman
  2541 ?     00:00:00 podman pause 
```

[]{#06.htm#pgfId-1110652}Subsequent running of the Podman commands joins
the namespaces of the `podman`{.fm-code-in-text}
`pause`{.fm-code-in-text} process[]{#06.htm#marker-1110653}. Podman does
this to avoid race conditions when user namespaces are coming up and
going down. The `pause`{.fm-code-in-text} process remains running until
you log out. You can also execute the `podman`{.fm-code-in-text}
`system`{.fm-code-in-text} `migrate`{.fm-code-in-text}
command[]{#06.htm#marker-1110654} to remove it. The
`pause`{.fm-code-in-text} process's role is keeping the user namespace
alive, as all rootless containers must be run in the same user
namespace. If they were not, sharing content and other namespaces (like
sharing the network namespace from another container) is impossible.

[]{#06.htm#pgfId-1110655}[Note]{.fm-callout-head} I often have users
report that when changing the /etc/subuid and /etc/subgid files, their
containers don't reflect the changes right away. Since the pause process
was launched with the previous user namespace settings, it needs to be
removed. Executing the `podman`{.fm-code-in-text1}
`system`{.fm-code-in-text1} `migrate`{.fm-code-in-text1} command
restarts the pause process within the user namespace.

[]{#06.htm#pgfId-1110656}You can kill the `pause`{.fm-code-in-text}
process at any time, but Podman re-creates it on the next run. By
default each rootless user has their own user namespace, and all of
their containers run within the same user namespace. You can subdivide
the user namespace and run containers with different user namespaces,
but realize, by default, you only have 65,000 UIDs to work with. Running
multiple containers in different user namespaces is much easier to do
when running rootful containers. Now that the user namespace and mount
namespace are created, Podman creates storage for the container's image
and sets up a mount point to start storing the image.

### []{#06.htm#pgfId-1110658}6.2.1 Pulling the image {#06.htm#heading_id_6 .fm-head1}

[]{#06.htm#pgfId-1110668}When
[]{#06.htm#marker-1110665}[]{#06.htm#marker-1110666}[]{#06.htm#marker-1110667}pulling
the image (figure 6.5), Podman checks if the container image quay.io/
rhatdan/myimage exists in local container storage. If it does, Podman
sets up the container network (see section 6.2.3). However, if the
container image does not exist, Podman uses the containers/image library
to pull the image. Following are the steps Podman takes while pulling
the image:

1.  []{#06.htm#pgfId-1110669 .calibre17}Resolve the IP address for the
    registry: quay.io.

2.  []{#06.htm#pgfId-1110670 .calibre17}Connect to the IP address via
    the HTTPS port (`443`{.fm-code-in-text}).

3.  []{#06.htm#pgfId-1110671 .calibre17}Begin pulling the manifest, all
    layers, and the config of the image using the HTTP protocol.

4.  []{#06.htm#pgfId-1110672 .calibre17}Find the multiple layers or
    blobs of quay.io/rhatdan/myimage.

5.  []{#06.htm#pgfId-1110673 .calibre17}Copy all layers simultaneously
    from the container registry to the host.

::: figure
![](images/OEBPS/Images/06-05.png){.calibre18}

[]{#06.htm#pgfId-1115763}Figure 6.5 Podman pulls an image off a
container registry and stores it in the container storage.
:::

[]{#06.htm#marker-1110696}[]{#06.htm#pgfId-1110674}As each layer is
copied to the host, Podman uses the containers/storage library to
reassemble the layers in order, creating an overlay mount point for each
of them on top of the previous one in
\~/.local/share/containers/storage. If there is no previous layer, it
creates the initial layer.

[]{#06.htm#pgfId-1110675}Next, containers/storage untars the contents of
the layer into the new storage layer. As the layers are untarred,
containers/storage `chown`{.fm-code-in-text}s the UID/GIDs of files in
the tarball into the home directory. Podman takes advantage of the user
namespace `CAP_CHOWN`{.fm-code-in-text}, as explained in previous
sections. Remember that Podman fails to create content if the UID or GID
specified in the TAR file was not mapped into the user
[]{#06.htm#marker-1110676}[]{#06.htm#marker-1110677}[]{#06.htm#marker-1110678}namespace.

### []{#06.htm#pgfId-1110680}6.2.2 Creating a container {#06.htm#heading_id_7 .fm-head1}

[]{#06.htm#pgfId-1110682}Once []{#06.htm#marker-1110681}the
containers/storage library finishes downloading the image and creating
the storage, Podman creates a new container based on the image. Podman
adds the container to Podman's internal database. It then tells
containers/storage to create writable space on disk and use the default
storage driver, usually `overlayfs`{.fm-code-in-text}, to mount this
space as a new container layer. The new container layer acts as the
final read/write layer and is mounted on top of the image.

[]{#06.htm#pgfId-1110683}[Note]{.fm-callout-head} Rootful containers
default to using native Linux overlay mounts. In rootless mode, kernel
versions newer than 5.13 or with the rootless overlay feature backported
(RHEL 8.5 kernels or later also have this feature) use the native
overlay mounts. On older kernels, Podman uses the
`fuse-overlayfs`{.fm-code-in-text1} executable[]{#06.htm#marker-1110684}
to create the layer. In Podman, `overlay`{.fm-code-in-text1} and
`overlay2`{.fm-code-in-text1} are the same drivers.

[]{#06.htm#pgfId-1110686}At this point, Podman needs to configure the
network inside the network []{#06.htm#marker-1110685}namespace.

### []{#06.htm#pgfId-1110688}6.2.3 Setting up the network {#06.htm#heading_id_8 .fm-head1}

[]{#06.htm#pgfId-1110697}In
[]{#06.htm#marker-1115791}[]{#06.htm#marker-1115792}rootless Podman, you
cannot create full, separate networking for containers because rootless
processes are not allowed to create network devices and modify the
firewall rules. Rootless Podman uses slirp4netns
([https://github.com/rootless-containers/slirp4netns](https://github.com/rootless-containers/slirp4netns){.url})
to configure the host network and simulate a VPN for the container.
Slirp4netns provides user-mode networking (slirp) for unprivileged
network namespaces. See figure 6.6.

::: figure
![](images/OEBPS/Images/06-06.png){.calibre18}

[]{#06.htm#pgfId-1115814}Figure 6.6 Podman creates a network namespace
and launches slirp4netns to relay network connections.
:::

[]{#06.htm#pgfId-1110698}[Note]{.fm-callout-head} In rootful containers,
Podman uses the CNI plugins to configure networking devices. In rootless
mode, even though the user is allowed to create and join a network
namespace, they are not allowed to create network devices. The
slirp4netns program emulates a virtual network to connect host
networking to the container networking. More advanced networking setups
require rootful containers.

[]{#06.htm#pgfId-1110699}Remember that in our original example, you
specified the `8080:8080`{.fm-code-in-text} port mapping as follows:

``` programlisting
$ podman run -d -p 8080:8080 --name myapp 
     registry.access.redhat.com/ubi8/httpd-24
```

[]{#06.htm#marker-1110731}[]{#06.htm#pgfId-1110701}Podman configures the
slirp4netns program to listen on the host network at port
`8080`{.fm-code-in-text} and allow the container process to bind to port
`8080`{.fm-code-in-text}. The slirp4netns command creates a tap device
that is injected inside the new network namespace, where the container
lives. Each packet is read back from slirp4netns and emulates a TCP/IP
stack in user space. Each connection outside the container network's
namespace is converted in a socket operation the unprivileged user can
run in the host network's namespace.

[]{#06.htm#pgfId-1110702}[Note]{.fm-callout-head} Linux TAP devices
create a user space network bridge. In user space, TAP devices can
simulate network devices inside of a network namespace. Processes within
the namespace interact with the network device. Packets read/written
from the network device are routed via the TUN/TAP device to the user
space program: slirp4netns.

[]{#06.htm#pgfId-1110703}Now that the storage and network are
configured, Podman is ready to finally start the container
[]{#06.htm#marker-1110704}[]{#06.htm#marker-1110705}process.

### []{#06.htm#pgfId-1110707}6.2.4 Starting the container monitor: conmon {#06.htm#heading_id_9 .fm-head1}

[]{#06.htm#pgfId-1110716}Podman
[]{#06.htm#marker-1110714}[]{#06.htm#marker-1110715}now executes conmon
(container monitor) for the container, telling it to use its configured
OCI runtime, usually `crun`{.fm-code-in-text}[]{#06.htm#marker-1110717}
or `runc`{.fm-code-in-text}[]{#06.htm#marker-1110718}. It also executes
the `podman`{.fm-code-in-text} `container`{.fm-code-in-text}
`cleanup`{.fm-code-in-text} `$CTRID`{.fm-code-in-text}
command[]{#06.htm#marker-1110719} when the container exits (see figure
6.7). conmon is described in
[]{#06.htm#marker-1110720}[]{#06.htm#marker-1110721}section 4.1.

::: figure
![](images/OEBPS/Images/06-07.png){.calibre18}

[]{#06.htm#pgfId-1115852}Figure 6.7 Podman launches the container
monitor, which launches the OCI runtime.
:::

### []{#06.htm#pgfId-1110723}6.2.5 Launching the OCI runtime {#06.htm#heading_id_10 .fm-head1}

[]{#06.htm#pgfId-1110732}The
[]{#06.htm#marker-1115925}[]{#06.htm#marker-1115926}OCI runtime reads
the OCI spec file and configures the kernel to run the container (see
figure 6.8). OCI runtimes do the following:

1.  []{#06.htm#pgfId-1110733 .calibre17}Set up the additional namespaces
    for the container.

2.  []{#06.htm#pgfId-1110734 .calibre17}Configure cgroups v2 (cgroups v1
    is not supported for rootless containers).

3.  []{#06.htm#pgfId-1110735 .calibre17}Set up the SELinux label for
    running the container.

4.  []{#06.htm#pgfId-1110736 .calibre17}Load the
    /usr/share/containers/seccomp.json seccomp rules into the kernel.

5.  []{#06.htm#pgfId-1110737 .calibre17}Set the environment variables
    for the container.

6.  []{#06.htm#pgfId-1110738 .calibre17}Bind mount any volumes onto the
    paths in the rootfs.

7.  []{#06.htm#pgfId-1110739 .calibre17}Switch the current
    `/`{.fm-code-in-text} to the rootfs `/`{.fm-code-in-text}.

8.  []{#06.htm#pgfId-1110740 .calibre17}Fork the container process.

9.  []{#06.htm#pgfId-1110741 .calibre17}Execute any OCI hook programs,
    passing them the rootfs as well as the container's PID 1.

10. []{#06.htm#pgfId-1110742 .calibre17}Execute the command specified by
    the image.

11. []{#06.htm#pgfId-1110743 .calibre17}Exit the OCI runtime, leaving
    conmon to monitor the container.

::: figure
![](images/OEBPS/Images/06-08.png){.calibre18}

[]{#06.htm#pgfId-1116007}Figure 6.8 conmon launches the OCI runtime,
which configures the kernel.
:::

[]{#06.htm#marker-1110753}[]{#06.htm#pgfId-1110744}And finally, conmon
reports the success back to Podman (see figure 6.9).

::: figure
![](images/OEBPS/Images/06-09.png){.calibre18}

[]{#06.htm#pgfId-1116045}Figure 6.9 Podman and OCI runtime exit, leaving
the container running with conmon monitoring it and slirp4netns
providing the network.
:::

[]{#06.htm#pgfId-1110754}The Podman command now exits because it ran in
`--detach`{.fm-code-in-text} (`-d`{.fm-code-in-text})
[]{#06.htm#marker-1115991}[]{#06.htm#marker-1115992}mode[]{#06.htm#marker-1115993}.

``` programlisting
$ podman run -d -p 8080:8080 --name myapp 
     registry.access.redhat.com/ubi8/httpd-24
```

[]{#06.htm#pgfId-1110756}[Note]{.fm-callout-head} If later you want
Podman to interact with the detached container, use the
`podman`{.fm-code-in-text1} `attach`{.fm-code-in-text1}
command[]{#06.htm#marker-1110757}, which connects to the conmon socket.
conmon allows Podman to interact with the container process through the
`STDIN`{.fm-code-in-text1}[]{#06.htm#marker-1110758},
`STDOUT`{.fm-code-in-text1}[]{#06.htm#marker-1110759}, and
`STDERR`{.fm-code-in-text1} file descriptors[]{#06.htm#marker-1110760},
which conmon has been monitoring.

### []{#06.htm#pgfId-1110762}6.2.6 The containerized application runs until completion {#06.htm#heading_id_11 .fm-head1}

[]{#06.htm#pgfId-1110764}The []{#06.htm#marker-1110763}application
process can exit on its own, or you can stop the container by executing
the `podman`{.fm-code-in-text} `stop`{.fm-code-in-text}
command[]{#06.htm#marker-1110765}:

``` programlisting
$ podman stop myapp
```

[]{#06.htm#pgfId-1110768}When the container process exits, the kernel
sends a `SIGCHLD`{.fm-code-in-text} to the `conmon`{.fm-code-in-text}
process[]{#06.htm#marker-1110767}. In turn, conmon does the following:

1.  []{#06.htm#pgfId-1110769 .calibre17}Records the container's exit
    code

2.  []{#06.htm#pgfId-1110770 .calibre17}Closes the container's logfile

3.  []{#06.htm#pgfId-1110771 .calibre17}Closes the Podman command's
    `STDOUT`{.fm-code-in-text}/`STDERR`{.fm-code-in-text}

4.  []{#06.htm#pgfId-1110773 .calibre17}Executes the
    `podman`{.fm-code-in-text} `container`{.fm-code-in-text}
    `cleanup`{.fm-code-in-text} `$CTRID`{.fm-code-in-text}
    command[]{#06.htm#marker-1110772 .calibre17}

5.  []{#06.htm#pgfId-1110774 .calibre17}Exits itself

[]{#06.htm#pgfId-1110776}The `podman`{.fm-code-in-text}
`container`{.fm-code-in-text} `cleanup`{.fm-code-in-text}
command[]{#06.htm#marker-1110775} takes down the slirp4netns network and
unmounts all of the container mount points. If you specify the
`--rm`{.fm-code-in-text} option[]{#06.htm#marker-1110777}, the container
is entirely removed---layers are removed from containers/storage, and
the container definition is removed from []{#06.htm#marker-1110778}the
DB.

## []{#06.htm#pgfId-1110780}Summary {#06.htm#heading_id_12 .fm-head}

-   []{#06.htm#pgfId-1110781 .calibre17}Running rootless containers is
    more secure than running rootful containers.

-   []{#06.htm#pgfId-1110782 .calibre17}The user namespace gives
    ordinary users the ability to manipulate more than one UID and is
    key to running containers.

-   []{#06.htm#pgfId-1110783 .calibre17}The mount namespace allows
    Podman to mount filesystems within the user namespace.

-   []{#06.htm#pgfId-1110784 .calibre17}Podman uses slirp4netns for
    providing network access to containers.

-   []{#06.htm#pgfId-1110786 .calibre17}Podman launches the
    `conmon`{.fm-code-in-text} process to monitor the
    []{#06.htm#marker-1110785 .calibre17}container.

[]{#p3.htm}

## Part 3. Advanced topics 

[]{#p3.htm#pgfId-1016264}[I]{.fm-part-initial-cap}n part 3 of the book,
you learn about advanced ways you can use Podman. This part discusses
integrating Podman into your system and how Podman can work with other
tools and orchestrators.

[]{#p3.htm#pgfId-1016265}In chapter 7, I introduce systemd integration.
Podman was developed to fully integrate into the system and takes
advantage of the init system: systemd. Systemd can easily be run within
Podman containers, and this chapter shows you how. Podman, likewise, can
be run within systemd services and provides commands that allow you to
automatically create the service configuration files to make this
happen.

[]{#p3.htm#pgfId-1016266}Chapter 8 shows you how Podman works with
Kubernetes. Podman is not a container engine under Kubernetes but can
work with Kubernetes YAML files. Because Kubernetes YAML files are used
to define applications that run within Kubernetes, Podman makes it easy
to move applications to and from a fully orchestrated environment back
to a single node. This feature makes it easier for you to develop
applications that eventually run under Kubernetes or debug problems that
happen under Kubernetes by running these applications locally on your
laptop. Kubernetes YAML is a great alternative to
`docker-compose`{.fm-code-in-text} YAML when running a group of
containers on a single node.

[]{#p3.htm#pgfId-1016267}Chapter 9 introduces the concept of Podman as a
service, which allows tools written to use a RESTful API to generate and
manage pods and containers with Podman. Tools like
`docker-compose`{.fm-code-in-text} and other Python tools built on
docker-py can interface with the Podman service, eliminating the need
for Docker altogether. The Podman service even allows Podman running on
remote systems, such as Windows, macOS, and Linux, to work with Linux
Podman containers.[]{#p3.htm#id_u6n877dddx1z}

[]{#07.htm}
