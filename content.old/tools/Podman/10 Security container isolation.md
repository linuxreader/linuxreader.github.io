# 10 Security container isolation

[]{#10.htm#pgfId-1114069}This chapter []{#10.htm#marker-1121657}covers

-   []{#10.htm#pgfId-1114070 .calibre17}All Linux security features used
    to keep containers isolated from each other
-   []{#10.htm#pgfId-1114072 .calibre17}Read-only access to kernel
    filesystems needed for processes within a container but which must
    be blocked from write access
-   []{#10.htm#pgfId-1114073 .calibre17}Masking of kernel filesystems to
    hide information from the host system
-   []{#10.htm#pgfId-1114074 .calibre17}Linux capabilities limiting the
    power of root
-   []{#10.htm#pgfId-1114075 .calibre17}The PID, IPC and network
    namespaces, which hide most of the operating system from processes
    within containers
-   []{#10.htm#pgfId-1114076 .calibre17}The mount namespace, which along
    with SELinux limit the container processes' access to only the
    designated image and volumes
-   []{#10.htm#pgfId-1114077 .calibre17}The user namespace, which allows
    you to write root processes inside of a container that are not root
    outside of a container

[]{#10.htm#pgfId-1114078}In this chapter and chapter 11, I review and
demonstrate some additional security considerations when using Podman to
run containers. Some of the content was covered in other chapters, but I
think it is useful to concentrate on these features from a security
perspective.

[]{#10.htm#pgfId-1114079}One of the most frequent problems I see with
people running containers is that when the container process is denied
some access, the user's first reaction is to run the container in
`--privileged`{.fm-code-in-text} mode[]{#10.htm#marker-1124108}, which
turns off all security separation for your container. Understanding how
to deal with the security features discussed in this chapter helps you
avoid needing to do this.

[]{#10.htm#pgfId-1114081}When I look at containers from a security point
of view, I examine how to protect the host kernel and filesystem from
the processes inside the container. I wrote a coloring book, *The
Container Coloring Book*
([https://red.ht/3gfVlHF](https://red.ht/3gfVlHF){.url}), illustrated by
Máirín Duffy[]{#10.htm#marker-1114083}[]{#10.htm#marker-1114084}
(@marin), describing the security features of containers based on the
three pigs (figure 10.1).

::: figure
![](images/OEBPS/Images/10-01.png){.calibre18}

[]{#10.htm#pgfId-1130030}Figure 10.1 *The Container Coloring Book*
([https://red.ht/3gfVlHF](https://red.ht/3gfVlHF){.url})
:::

[]{#10.htm#pgfId-1114092}The analogy I use in the book is that the three
pigs are applications. I then discuss where they live and their choices
of housing compared to computer systems.

[]{#10.htm#pgfId-1114093}The single-family house is equivalent to one
application on a single isolated node. Living in a duplex is equivalent
to running each application in a separate VM. Living in a hotel or
apartment building is similar to containers, where you get your own
apartment, but you rely on the security of the front desk to control the
access to your living space. If the front desk is compromised, then your
apartment is going to be compromised. Containers are similar to this in
that they rely on the security of the kernel. If one container can take
over the host kernel, then it can take over all of the container
applications running on the system. Also, if they escape to the
underlying filesystem, they might be able to read and write all of the
data of the containers on the system.

[]{#10.htm#pgfId-1114094}From this perspective, I see the number-one
goal of the host as being to protect the host kernel and filesystems
from the container processes. The rest of this chapter describes the
tools used to protect the host kernel and filesystem from container
processes.

[]{#10.htm#pgfId-1114095}Protecting the kernel from potentially hostile
containers is the primary goal of container security. If the kernel is
vulnerable, then the rest of the system and all containers are
vulnerable. In many cases, the only exposure to the host system for a
container is the host kernel itself.

[]{#10.htm#pgfId-1114096}Processes within a container can interact with
the kernel in many different ways. This section examines these
communications and the operating system features used to secure the
container processes.

[]{#10.htm#pgfId-1114098}The Linux kernel provides filesystems that
allow processes to communicate and configure the kernel. Protecting
these filesystems from confined container processes is the first
security feature you will examine.

## []{#10.htm#pgfId-1114100}10.1 Read-only Linux kernel pseudo filesystems {#10.htm#heading_id_3 .fm-head}

[]{#10.htm#pgfId-1114103}These
[]{#10.htm#marker-1114101}[]{#10.htm#marker-1114102}Linux kernel pseudo
filesystems are generally mounted under /proc and /sys. Table 10.1 lists
some of the Linux kernel pseudo filesystems mounted on my
machine.[]{#10.htm#id_yj2rnzt9ihp6}

[]{#10.htm#pgfId-1118386}Table 10.1 Filesystems mounted as read only

+-----------------+-----------------------------------------------------+
| []{#1           | []{#10.htm#pgfId-1118392}Pseudo filesystem          |
| 0.htm#pgfId-111 | description                                         |
| 8390}Filesystem |                                                     |
| mount point     |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.htm#pgf  | []{#10.htm#pgfId-1118396}The sysfs filesystem       |
| Id-1118394}/sys | allows viewing and manipulating objects from        |
|                 | `user-space`{.fm-code-in-text1}, which are created  |
|                 | and destroyed by kernel space.                      |
+-----------------+-----------------------------------------------------+
| []{#10.htm#pgfI | []{#10.htm#pgfId-1118400}The security pseudo        |
| d-1118398}/sys/ | filesystem is used to read and configure general    |
| kernel/security | security modules. An example is the Integrity       |
|                 | Measurement Architecture (IMA) model.               |
+-----------------+-----------------------------------------------------+
| []{#10.ht       | []{#10.htm#pgfId-1118404}The cgroup filesystem is   |
| m#pgfId-1118402 | used to manage control groups.                      |
| }/sys/fs/cgroup |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.ht       | []{#10.htm#pgfId-1118408}The pstore filesystem      |
| m#pgfId-1118406 | stores nonvolatile information useful for           |
| }/sys/fs/pstore | diagnosing the cause of a system crash.             |
+-----------------+-----------------------------------------------------+
| []{#10          | []{#10.htm#pgfId-1118412}The Berkeley Packet Filter |
| .htm#pgfId-1118 | (BPF) filesystem is a mechanism to instrument the   |
| 410}/sys/fs/bpf | Linux kernel with user programs that reveal kernel  |
|                 | information and control the way processes run on a  |
|                 | system.                                             |
+-----------------+-----------------------------------------------------+
| []{#10.htm      | []{#10.htm#pgfId-1118416}The SELinux filesystem is  |
| #pgfId-1118414} | used to configure SELinux in the kernel (see        |
| /sys/fs/selinux | section 10.2.7).                                    |
+-----------------+-----------------------------------------------------+
| []{#10.htm#pg   | []{#10.htm#pgfId-1118420}The configfs filesystem is |
| fId-1118418}/sy | for creating, managing, and destroying kernel       |
| s/kernel/config | objects from `user-space`{.fm-code-in-text1}.       |
+-----------------+-----------------------------------------------------+

[]{#10.htm#pgfId-1114141}Most processes require read access to these
pseudo kernel filesystems to succeed, but only administrator processes
require write access. Normally, the kernel relies on the separation of
root from non-root or possession of the
`CAP_SYS_ADMIN`{.fm-code-in-text} capability[]{#10.htm#marker-1114142}
(see section 10.2.2) to modify these filesystems.

[]{#10.htm#pgfId-1114143}Often containers need to run as root, requiring
container security to use other means to prevent the writing of these
kernel filesystems by the root process. Podman does not mount most of
these advanced kernel pseudo filesystems. It does mount /sys,
/sys/fs/cgroup, and /sys/fs/selinux as read only. When you are in a PID
namespace, the /proc filesystem changes, meaning the /proc inside a
container is not the host's /proc. Processes within the container can
only affect other processes within the container.

[]{#10.htm#pgfId-1114144}The /sys filesystems and the namespaced /proc
filesystem sometimes leak host information into the container. Because
of this, Podman mounts /dev/null over files and mounts read-only tmpfs
filesystems over directories to prevent container access. Podman also
bind mounts certain subdirectories as read only over themselves to
prevent the container process from writing to
them.[]{#10.htm#id_6wlyiab1lct1}[]{#10.htm#id_6ewp0of2k2z2} See table
10.2 for a complete list of files and directories that Podman masks over
for security purposes.

[]{#10.htm#pgfId-1118489}Table 10.2 Filesystem fields masked over with
Podman

+-----------------+-----------------------------------------------------+
| []{#10.htm#pgf  | []{#10.htm#pgfId-1118495}Paths                      |
| Id-1118493}Type |                                                     |
| of masking      |                                                     |
+-----------------+-----------------------------------------------------+
| []{#            | []{#10.htm#pgfId-1118499}/proc/acpi, /proc/kcore,   |
| 10.htm#pgfId-11 | /proc/keys, /proc/latency_stats, /proc/timer_list,  |
| 18497}Read-only | /proc/timer_stats, /proc/sched_debug, /proc/scsi,   |
| tmpfs mounted   | /sys/firmware, /sys/fs/selinux, /sys/dev/block      |
| over the        |                                                     |
| directory       |                                                     |
+-----------------+-----------------------------------------------------+
| []{#            | []{#10.htm#pgfId-1118503} /proc/asound, /proc/bus,  |
| 10.htm#pgfId-11 | /proc/fs, /proc/irq, /proc/sys, /proc/sysrq-trigger |
| 18501}Read-only |                                                     |
| bind mount over |                                                     |
| the directory   |                                                     |
+-----------------+-----------------------------------------------------+

[]{#10.htm#pgfId-1114163}I have found that almost all container images
run fine with this additional security. Sometimes a containerized
application may need additional access to one of these masked-over
directories.

### []{#10.htm#pgfId-1114165}10.1.1 Unmasking the masked paths {#10.htm#heading_id_4 .fm-head1}

[]{#10.htm#pgfId-1114170}Rather
[]{#10.htm#marker-1114166}[]{#10.htm#marker-1114167}[]{#10.htm#marker-1114168}than
force the container to run `--privileged`{.fm-code-in-text}
mode[]{#10.htm#marker-1114169}, you can tell Podman to unmask a
directory. In the following example, you run a container and see there
are no files or directories under /proc/scsi because it is mounted over
with a tmpfs:

``` programlisting
$ podman run --rm ubi8 ls /proc/scsi
```

[]{#10.htm#pgfId-1114173}You can use the
`--security-opt`{.fm-code-in-text} `unmask=/proc/scsi`{.fm-code-in-text}
flag[]{#10.htm#marker-1114172} to remove the mount point and expose the
underlying files and directories:

``` programlisting
$ podman run --rm --security-opt unmask=/proc/scsi ubi8 ls /proc/scsi
device_info
scsi
sg
```

[]{#10.htm#pgfId-1114178}You can even use a `*`{.fm-code-in-text} to
unmount all directories under a certain path:

``` programlisting
$ podman run --rm --security-opt unmask=/proc/* ubi8 ls /proc/scsi
device_info
scsi
sg
```

[]{#10.htm#pgfId-1114183}Unmasking makes your container slightly less
secure, but it is much better than going all the way to
`--privileged`{.fm-code-in-text} and turning off all of the security. In
certain situations, you might want to make the system more secure by
masking over parts of the pseudo filesystems. The
`podman`{.fm-code-in-text} `run`{.fm-code-in-text} man
pages[]{#10.htm#marker-1123817} list the masked
file[]{#10.htm#marker-1123818}[]{#10.htm#marker-1123819}[]{#10.htm#marker-1123820}systems:

``` programlisting
$ man podman run
...
   • unmask=ALL or /path/1:/path/2, or shell expanded paths (/proc/*): 
Paths to unmask separated by a colon. If set to ALL, it will unmask all the 
paths that are masked or made read only by default. The default masked
    paths are /proc/acpi, /proc/kcore, /proc/keys, /proc/latency_stats, 
/proc/sched_debug, /proc/scsi, /proc/timer_list, /proc/timer_stats, 
/sys/firmware, and /sys/fs/selinux. 
    The default paths that are read only are /proc/asound, /proc/bus, 
/proc/fs, /proc/irq, /proc/sys, /proc/sysrq-trigger, /sys/fs/cgroup.
```

### []{#10.htm#pgfId-1114200}10.1.2 Masking additional paths {#10.htm#heading_id_5 .fm-head1}

[]{#10.htm#pgfId-1114205}If
[]{#10.htm#marker-1114201}[]{#10.htm#marker-1114202}[]{#10.htm#marker-1114203}you
are very security conscious or have a container you don't trust with
certain access provided to containers, you can add additional masked
paths with the `--security-opt`{.fm-code-in-text} mask
flag[]{#10.htm#marker-1114206}. For example, if you want to prevent a
container process from seeing the devices in /proc/sys/dev, run the
following:

``` programlisting
$ podman run --rm ubi8 ls /proc/sys/dev
cdrom
hpet
i915
mac_hid
raid
scsi
tty
```

[]{#10.htm#pgfId-1114216}You can mask over it with the
`--security-opt`{.fm-code-in-text}
`mask=/proc/sys/dev`{.fm-code-in-text} flag[]{#10.htm#marker-1114215}:

``` programlisting
$ podman run --rm --security-opt mask=/proc/sys/dev ubi8 ls /proc/sys/dev
```

[]{#10.htm#pgfId-1114218}You saw how Podman prevents root processes from
reading and, more importantly, writing to pseudo filesystems. The
container processes can actually see what is mounted over within the
container by looking at /proc/self/mountinfo.

[]{#10.htm#pgfId-1114219}Listing 10.1 The mount table within a Podman
container

``` programlisting
$ podman run –rm ubi8 cat /proc/self/mountinfo
...
1628 1610 0:5 /null /proc/kcore rw,nosuid – 
➥ devtmpfs devtmpfs rw,seclabel,size=4096k,
➥ nr_inodes=1048576,mode=755,inode64                          ❶
...
1620 1595 0:86 / /sys/firmware ro,relatime - tmpfs tmpfs       ❷
rw,context="system_u:object_r:container_file_t:s0:c406,c915",size=0k,uid=32
➥ 67,gid=3267,inode64
...
```

[]{#10.htm#pgfId-1129169}[❶]{.fm-combinumeral} Shows /dev/null mounted
over /proc/kcore

[]{#10.htm#pgfId-1129190}[❷]{.fm-combinumeral} Shows a tmpfs mounted
read-only over /sys/firmware

[]{#10.htm#pgfId-1114231}You might be asking yourself, "If the container
knows what has been mounted, what prevents the root user within the
container from removing the mounts or remounting filesystems' read/write
and then attacking the
[]{#10.htm#marker-1114232}[]{#10.htm#marker-1114233}[]{#10.htm#marker-1114234}host
[]{#10.htm#marker-1114235}[]{#10.htm#marker-1114236}kernel?

## []{#10.htm#pgfId-1114238}10.2 Linux capabilities {#10.htm#heading_id_6 .fm-head}

[]{#10.htm#pgfId-1114241}Most
[]{#10.htm#marker-1114239}[]{#10.htm#marker-1114240}Linux people
understand Linux has two types of users: root (privileged process) and
everyone else (nonprivileged processes). Root is all powerful, and
non-root has much more limited powers, specifically when configuring and
modifying the kernel. Sometimes a non-privileged process needs
privileges to execute a certain command-line `ping`{.fm-code-in-text} or
`sudo`{.fm-code-in-text}. Linux supports a way to mark these files as
`setuid`{.fm-code-in-text}, and when a nonprivileged process executes
them, the new process gains the privilege.

[]{#10.htm#pgfId-1114242}The binary difference between privileged and
unprivileged processes ended in Linux around 2000. Kernel engineers
broke down the power of root into a group of different privileged
capabilities. Currently, on my system, the Linux kernel supports 41. You
can see the complete list of capabilities using the
`capsh`{.fm-code-in-text} program[]{#10.htm#marker-1114243}. Execute the
`capsh`{.fm-code-in-text} program to see the list of capabilities on
your system. You will see the `current`{.fm-code-in-text} set of
capabilities for your processes as being empty. The
`Bounding`{.fm-code-in-text} set of
capabilities[]{#10.htm#marker-1114244} is the set of capabilities your
process can get from executing a `setuid`{.fm-code-in-text}
program[]{#10.htm#marker-1114245}.

[]{#10.htm#pgfId-1114246}Listing 10.2 `capsh`{.fm-code-in-text}
`–print`{.fm-code-in-text} showing the capabilities available to your
user's process

``` programlisting
$ capsh --print
Current: =                                 ❶
Bounding set =                             ❷
cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,
➥ cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,
➥ cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,
➥ cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,
➥ cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,
➥ cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,
➥ cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,
➥ cap_block_suspend,cap_audit_read,cap_perfmon,cap_bpf,cap_checkpoint_restore
Ambient set =
...
uid=3267(dwalsh) euid=3267(dwalsh)          ❸
gid=3267(dwalsh)
```

[]{#10.htm#pgfId-1128949}[❶]{.fm-combinumeral} The Current set of
capabilities shows no capabilities.

[]{#10.htm#pgfId-1128970}[❷]{.fm-combinumeral} The Bounding set of
capabilities shows all (41) capabilities.

[]{#10.htm#pgfId-1128987}[❸]{.fm-combinumeral} Because you ran the capsh
command as a normal user, you see your UID and GID listed.

[]{#10.htm#pgfId-1114265}This means your user process can execute the
`sudo`{.fm-code-in-text} command[]{#10.htm#marker-1114264} and get the
full set of capabilities as root. You can read information about what
each capability does in the capabilities man page by executing
`man`{.fm-code-in-text} `capabilities`{.fm-code-in-text}. Over the
years, the community has figured out that almost all containers do not
require the full list of capabilities because they seldom modify the
kernel.

### []{#10.htm#pgfId-1114267}10.2.1 Dropped Linux capabilities {#10.htm#heading_id_7 .fm-head1}

[]{#10.htm#pgfId-1114271}Because
[]{#10.htm#marker-1114268}[]{#10.htm#marker-1114269}[]{#10.htm#marker-1114270}container-confined
processes are not supposed to manipulate the operating system, and
specifically the kernel, Podman can run root within its containers with
far fewer capabilities. You can examine the default list of capabilities
available within a Podman container by executing the same
`capsh`{.fm-code-in-text} program[]{#10.htm#marker-1114272}.

[]{#10.htm#pgfId-1114273}Listing 10.3 The default list of capabilities
available within a Podman container

``` programlisting
$ podman run --rm ubi8 capsh --print
Current: =                                                             ❶
cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,
➥ cap_setuid,cap_setpcap,cap_net_bind_service,cap_sys_chroot,
➥ cap_setfcap+eip
Bounding set =                                                         ❷
cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,
➥ cap_setuid,cap_setpcap,cap_net_bind_service,cap_sys_chroot,cap_setfcap
...
uid=0(root)                                                            ❸
gid=0(root)
groups=
```

[]{#10.htm#pgfId-1128609}[❶]{.fm-combinumeral} The Current set of
capabilities shows just 11 capabilities, since the container process is
running as root.

[]{#10.htm#pgfId-1128637}[❷]{.fm-combinumeral} The Bounding set of
capabilities shows the same (11) capabilities.

[]{#10.htm#pgfId-1128654}[❸]{.fm-combinumeral} Because containers
default to running as root, you see the UID and GID as root.

[]{#10.htm#pgfId-1114288}As you observe, Podman, by default, dropped 30
capabilities---from 41 down to 11---when running a container. Even
though the container has root privileges, it is far less powerful than
root on the system.

[]{#10.htm#pgfId-1114289}[Note]{.fm-callout-head} Docker also drops
capabilities but leaves 14 capabilities. Podman runs with tighter
security by dropping the following additional capabilities:
`CAP_ MKNOD`{.fm-code-in-text1}[]{#10.htm#marker-1114290},
`CAP_AUDIT_WRITE`{.fm-code-in-text1}[]{#10.htm#marker-1114291}, and
`CAP_NET_RAW`{.fm-code-in-text1}[]{#10.htm#marker-1114292}.

[]{#10.htm#pgfId-1114293}The list of capabilities still allowed within a
container mainly concern controlling multiple processes; for example,
`CAP_SETUID`{.fm-code-in-text} and `CAP_SETGID`{.fm-code-in-text} allow
processes inside the container to change to different UIDs. An example
of where this is important is running your web application as
`UID=60`{.fm-code-in-text}, but when the container process started, it
needed to run as root for a short time before changing its UID to
`60`{.fm-code-in-text}. If Podman dropped
`CAP_SETUID`{.fm-code-in-text}, then the root process within the
container is not allowed to change to the web services UID.

[]{#10.htm#pgfId-1114294}Another interesting capability Podman allows is
`CAP_NET_BIND_SERVICE`{.fm-code-in-text}, which enables a process to
bind to a network port less than `1024`{.fm-code-in-text}---for example,
port `80`{.fm-code-in-text}. Recall from chapter 2 that you cannot bind
port `80`{.fm-code-in-text} on your host to port `80`{.fm-code-in-text}
within the container. User processes do not have
`CAP_NET_BIND_SERVICE`{.fm-code-in-text}, so they cannot bind to port
`80`{.fm-code-in-text}. Table 10.3 lists the default capabilities
available to root running within a container with Podman. This list can
be modified in the containers.conf file using the
`default_capabilities`{.fm-code-in-text} field under the containers
table.[]{#10.htm#id_tnckf94w0qwk}[]{#10.htm#id_k51d5sudgzug}[]{#10.htm#id_4hrvi2og09zg}[]{#10.htm#id_mav5p9a6hdrw}[]{#10.htm#id_q0cwpuhoolzu}

[]{#10.htm#pgfId-1118917}Table 10.3 Default list of capabilities allowed
root processes in a container

+-----------------+-----------------------------------------------------+
| [               | []{#10.htm#pgfId-1118923}Description                |
| ]{#10.htm#pgfId |                                                     |
| -1118921}Option |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10          | []{#10.htm#pgfId-1118927}Make arbitrary changes to  |
| .htm#pgfId-1118 | file UIDs and GIDs.                                 |
| 925}`CAP_CHOWN` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#10.htm# |                                                     |
| marker-1118968} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.htm#pg   | []{#10.htm#pgfId-1118931}Bypass file read, write,   |
| fId-1118929}`CA | and execute permission checks.                      |
| P_DAC_OVERRIDE` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#10.htm# |                                                     |
| marker-1118969} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.         | []{#10.htm#pgfId-1118935}Bypass permission checks   |
| htm#pgfId-11189 | on operations on the filesystem UID.                |
| 33}`CAP_FOWNER` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#10.htm# |                                                     |
| marker-1118970} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.h        | []{#10.htm#pgfId-1118939}Don\'t clear               |
| tm#pgfId-111893 | `set-user-ID`{.fm-code-in-text1} and                |
| 7}`CAP_SETFSID` | `set-group-ID`{.fm-code-in-text1} mode bits when    |
| {.fm-code-in-te | modifying a file.                                   |
| xt1}[]{#10.htm# |                                                     |
| marker-1118971} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#1           | []{#10.htm#pgfId-1118943}Bypass permission checks   |
| 0.htm#pgfId-111 | for sending signals.                                |
| 8941}`CAP_KILL` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#10.htm# |                                                     |
| marker-1118972} |                                                     |
+-----------------+-----------------------------------------------------+
| []              | []{#10.htm#pgfId-1118947}Bind a socket to internet  |
| {#10.htm#pgfId- | domain privileged ports (port numbers less than     |
| 1118945}`CAP_NE | `1024`{.fm-code-in-text1}).                         |
| T_BIND_SERVICE` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#10.htm# |                                                     |
| marker-1118973} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.h        | []{#10.htm#pgfId-1118951}Set arbitrary capabilities |
| tm#pgfId-111894 | on a file.                                          |
| 9}`CAP_SETFCAP` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#10.htm# |                                                     |
| marker-1118974} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.         | []{#10.htm#pgfId-1118955}Change a process's group   |
| htm#pgfId-11189 | ID (GID) or supplementary GID list.                 |
| 53}`CAP_SETGID` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#10.htm# |                                                     |
| marker-1118975} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.h        | []{#10.htm#pgfId-1118959}Add and drop any           |
| tm#pgfId-111895 | capability from the calling thread\'s bounding set. |
| 7}`SET_SETPCAP` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#10.htm# |                                                     |
| marker-1118976} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.         | []{#10.htm#pgfId-1118963}Make arbitrary             |
| htm#pgfId-11189 | manipulations of the process user ID (UID).         |
| 61}`CAP_SETUID` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#10.htm# |                                                     |
| marker-1118977} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.htm#     | []{#10.htm#pgfId-1118967}Allow                      |
| pgfId-1118965}` | `chroot`{.fm-code-in-text1}, and change mount       |
| CAP_SYS_CHROOT` | namespaces.                                         |
| {.fm-code-in-te |                                                     |
| xt1}[]{#10.htm# |                                                     |
| marker-1118978} |                                                     |
+-----------------+-----------------------------------------------------+

[]{#10.htm#pgfId-1114363}I introduced section 10.2 by asking what
prevents the root process from unmounting or remounting the read-only
filesystems. The answer is Podman dropping the
`CAP_ SYS_ADMIN`{.fm-code-in-text}[]{#10.htm#marker-1122765}[]{#10.htm#marker-1122766}[]{#10.htm#marker-1122767}
c`apability`{.fm-code-in-text}[]{#10.htm#marker-1122768}.

### []{#10.htm#pgfId-1114369}10.2.2 Dropped CAP_SYS_ADMIN {#10.htm#heading_id_8 .fm-head1}

[]{#10.htm#pgfId-1114375}The
[]{#10.htm#marker-1114370}[]{#10.htm#marker-1114371}[]{#10.htm#marker-1114372}most
powerful Linux capability is `CAP_SYS_ADMIN`{.fm-code-in-text}. I
describe this capability in the following way: Imagine you are a kernel
engineer adding a new feature into the kernel, and this feature requires
privilege access. You look to see the list of capabilities, and you
don't find a capability that is a great match for the access. Kernel
engineers can go through the hassle of creating a new capability; or,
say this is something a system administrator needs to do and there is a
`CAP_SYS_ADMIN`{.fm-code-in-text}. I might as well require that
capability. If you look at the man capabilities information, you see
multiple pages of features the `CAP_SYS_ADMIN`{.fm-code-in-text}
capability blocks.

[]{#10.htm#pgfId-1114377}One feature `CAP_SYS_ADMIN`{.fm-code-in-text}
controls is the ability to mount and unmount filesystems. Because this
capability is dropped by default, root processes in Podman containers
cannot unmount or remount the read-only mount points.

[]{#10.htm#pgfId-1114378}As you learned previously, 11 capabilities are
still allowed. In most cases, your containerized process does not even
need those capabilities, meaning you can drop additional
[]{#10.htm#marker-1114379}[]{#10.htm#marker-1114380}[]{#10.htm#marker-1114381}ones.

### []{#10.htm#pgfId-1114384}10.2.3 Dropping capabilities {#10.htm#heading_id_9 .fm-head1}

[]{#10.htm#pgfId-1114388}I
[]{#10.htm#marker-1114385}[]{#10.htm#marker-1114386}[]{#10.htm#marker-1114387}recommend
people run their applications with the least privileges possible. One
way of increasing the security of the system is dropping additional
capabilities.

[]{#10.htm#pgfId-1114389}Imagine your containerized process does not
need to bind to ports \< `1024`{.fm-code-in-text}. You can execute
Podman with the `--cap-drop=CAP_NET_BIND_SERVICE`{.fm-code-in-text}
flag[]{#10.htm#marker-1114390} and drop that capability from your
container.

[]{#10.htm#pgfId-1114391}Listing 10.4 Capabilities inside a container
when you drop `CAP_NET_BIND_SERVICE`{.fm-code-in-text}

``` programlisting
$ podman run --cap-drop CAP_NET_BIND_SERVICE ubi8 capsh --print
Current: =                                                             ❶
cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,
➥ cap_setuid,cap_setpcap,cap_sys_chroot,cap_setfcap+eip
Bounding set =                                                         ❷
cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,
➥ cap_setuid,cap_setpcap,cap_sys_chroot,cap_setfcap
...
```

[]{#10.htm#pgfId-1128376}[❶]{.fm-combinumeral} Notice the list of
Current capabilities no longer includes CAP_NET_BIND_SERVICE.

[]{#10.htm#pgfId-1128404}[❷]{.fm-combinumeral} Notice that the list of
Bounding capabilities no longer includes CAP_NET_BIND_SERVICE.

[]{#10.htm#pgfId-1114404}You can even drop all capabilities using the
`--cap-drop=all`{.fm-code-in-text} flag:

``` programlisting
$ podman run --cap-drop all ubi8 capsh --print
Current: =
Bounding set =
```

[]{#10.htm#pgfId-1114408}Even though your container is running as root,
it has no capabilities to modify the kernel. Sometimes your container
fails to run with the limited list of capabilities provided by Podman;
in this case, you can add required
[]{#10.htm#marker-1114409}[]{#10.htm#marker-1114410}[]{#10.htm#marker-1114411}capabilities.

### []{#10.htm#pgfId-1114413}10.2.4 Adding capabilities {#10.htm#heading_id_10 .fm-head1}

[]{#10.htm#pgfId-1114417}In
[]{#10.htm#marker-1114414}[]{#10.htm#marker-1114415}[]{#10.htm#marker-1114416}some
situations, your container might fail because it does not have a certain
capability. You can simply run `--privileged`{.fm-code-in-text} and turn
off all security in these cases, but a better solution is just adding
required capabilities.

[]{#10.htm#pgfId-1114418}Imagine you have a container that wants to
create a raw IP packet on its namespaced network, which requires
`CAP_NET_RAW`{.fm-code-in-text}. Podman, by default, does not allow
this. Rather than running the container as
`--privileged`{.fm-code-in-text}, you can use the
`--cap-add`{.fm-code-in-text} `CAP_NET_RAW`{.fm-code-in-text}
flag[]{#10.htm#marker-1114419}:

``` programlisting
$ podman run --cap-add CAP_NET_RAW ubi8 capsh --print
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,
➥ cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_
➥ sys_chroot,cap_setfcap+eip
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,
➥ cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_
➥ sys_chroot,cap_setfcap
...
```

[]{#10.htm#marker-1114464}[]{#10.htm#pgfId-1114428}If this is the only
capability needed by your container, you can both drop all capabilities
and just add back in this `CAP_NET_RAW`{.fm-code-in-text} by using the
`--cap-drop`{.fm-code-in-text}[]{#10.htm#marker-1114429} and
`--cap-add`{.fm-code-in-text} flags[]{#10.htm#marker-1114430} at the
same
[]{#10.htm#marker-1114431}[]{#10.htm#marker-1114432}[]{#10.htm#marker-1114433}time:

``` programlisting
$ podman run --cap-drop=all --cap-add CAP_NET_RAW ubi8 capsh --print
Current: = cap_net_raw+eip
Bounding set =cap_net_raw
...
```

### []{#10.htm#pgfId-1114439}10.2.5 No new privileges {#10.htm#heading_id_11 .fm-head1}

[]{#10.htm#pgfId-1114444}Podman
[]{#10.htm#marker-1114440}[]{#10.htm#marker-1114441}[]{#10.htm#marker-1114442}has
an option, `--security-opt`{.fm-code-in-text}
`no-new-privileges`{.fm-code-in-text}[]{#10.htm#marker-1114443}, which
disables the ability for container processes to gain additional
privileges. Basically, it locks the processes into the group of Linux
capabilities they have when they are started. Even if they can execute a
`setuid`{.fm-code-in-text} program, the kernel denies it from gaining
additional capabilities. The `no-new-privileges`{.fm-code-in-text}
option[]{#10.htm#marker-1114445} also affects SELinux and prevents
SELinux label transitions. Even if SELinux had a bug in its rules
database, the container process would not be allowed to change its
[]{#10.htm#marker-1114446}[]{#10.htm#marker-1114447}[]{#10.htm#marker-1114448}label.

### []{#10.htm#pgfId-1114450}10.2.6 Root with no capabilities is still dangerous {#10.htm#heading_id_12 .fm-head1}

[]{#10.htm#pgfId-1114454}Dropping
[]{#10.htm#marker-1114451}[]{#10.htm#marker-1114452}[]{#10.htm#marker-1114453}capabilities
means your container is running much more securely, but running all of
your containers without any Linux capabilities is much more secure.
Another problem to consider when running a container as root, even if
you drop all capabilities, is that the process is still running as root.
The root process is allowed to modify all files on the system that are
owned by root. The root process can modify a system file and trick a
privileged administrator into executing it. Also, some client-server
applications trust the client side of the connection simply if it is
running as root (e.g., Docker). Podman can solve both of these problems
by using the
[]{#10.htm#marker-1114455}[]{#10.htm#marker-1114456}[]{#10.htm#marker-1114457}user
[]{#10.htm#marker-1114458}[]{#10.htm#marker-1114459}namespace.

## []{#10.htm#pgfId-1114461}10.3 UID isolation: User namespace {#10.htm#heading_id_13 .fm-head}

[]{#10.htm#pgfId-1114465}Back
[]{#10.htm#marker-1130059}[]{#10.htm#marker-1130060}[]{#10.htm#marker-1130061}in
section 6.1.1, I introduced the concept of the user namespace. Recall
that UIDs were allocated via the /etc/subuid and /etc/subgid files for a
rootless user. For my accounts, the range of UIDs from
`100000`{.fm-code-in-text}--`165535`{.fm-code-in-text} was allocated
along with my UID `3265`{.fm-code-in-text} and was used by Podman when
launching containers. See figure 10.2 for a description of the user
namespace mapping.

::: figure
![](images/OEBPS/Images/10-02.png){.calibre18}

[]{#10.htm#pgfId-1130085}Figure 10.2 The mapping of UIDs used by
rootless Podman for my account
:::

[]{#10.htm#pgfId-1114472}This user namespace allows my account to have
root access within the container that is not root on the host. Running
containers in a user namespace eliminates the problem of having
processes running as the root user, and the inherent trust is built into
some daemons.

[]{#10.htm#pgfId-1114473}One problem with rootless users is that, by
default, all of the containers run with the same user namespace.
Theoretically, from a user namespace point of view, one container can
attack another container, since they run with duplicate UIDs. Also, if
the container processes break out, they can read/write content in your
home directory, since the root processes within the containers are
running with your UID.

### []{#10.htm#pgfId-1114475}10.3.1 Isolating containers using the - -userns=auto flag {#10.htm#heading_id_14 .fm-head1}

[]{#10.htm#pgfId-1114480}Podman
[]{#10.htm#marker-1114476}[]{#10.htm#marker-1114477}[]{#10.htm#marker-1114478}[]{#10.htm#marker-1114479}has
a feature for allocating unique ranges of UIDs for every container it
launches. Since there are limited UIDs allocated for each user account,
this feature works best when launched by the root user.

[]{#10.htm#pgfId-1114481}To launch multiple containers within their own
user namespace, you need to first allocate the UIDs and GIDs to be used
for these containers. On a Linux system, there are 4 billion UIDs
available. Podman recommends that you allocate the highest 2 billion
UIDs for your containers. You can do this by adding the following
containers line to your /etc/subuid and /etc/subgid file.

[]{#10.htm#pgfId-1114482}Listing 10.5 The contents of the /etc/subuid
and /etc/subgid files

``` programlisting
```bash
# cat /etc/subuid
dwalsh:100000:65536
containers:2147483647:2147483648   ❶
# cat /etc/subgid
dwalsh:100000:65536
containers:2147483647:2147483648   ❶
```

[]{#10.htm#pgfId-1128297}[❶]{.fm-combinumeral} Allocates the top 2
billion UIDs to the container user used by Podman. Adding this line
tells other tools on your system, like useradd, to avoid allocating UIDs
and GIDs within this range.

[]{#10.htm#pgfId-1114490}You[]{#10.htm#marker-1123372} can launch a
container within a unique user namespace using the
`--userns=auto`{.fm-code-in-text} option. Podman allocates the UIDs for
the container starting with UID `2147483647`{.fm-code-in-text}, which
you specified in the /etc/subuid file. Podman then examines the
container image for all UIDs defined within it as well as the
/etc/passwd file if it exists in the image and then uses this to
allocate the number of UIDs required to run the container with a default
minimum of `1024`{.fm-code-in-text}:

``` programlisting
# podman run --userns=auto ubi8 cat /proc/self/uid_map
     0 2147483647   1024
```

[]{#10.htm#pgfId-1114493}If I run a second container with a specific
user `2000`{.fm-code-in-text}, then the allocation of UIDs reflects
this. You see that the number of UIDs allocated is
`2001`{.fm-code-in-text}---UID `2000`{.fm-code-in-text} plus one for the
root user:

``` programlisting
# podman run --user=2000 --userns=auto ubi8 cat /proc/self/uid_map
     0 2147484671   2001
```

[]{#10.htm#pgfId-1114496}Also, note that the starting UID for the first
container was `2147483647,`{.fm-code-in-text} while the starting UID for
the second container was `2147484671`{.fm-code-in-text}. Subtracting the
first UID `2147483647`{.fm-code-in-text} from the second UID
`2147484671`{.fm-code-in-text} gives you `1024`{.fm-code-in-text}, which
is the number of UIDs allocated for the first container. No UID within
the first container overlaps with the second container, meaning no
process within the first container can attack processes within the
second container, and vice versa.

[]{#10.htm#pgfId-1114497}You can override the default size of the user
namespace used within the container with a size option if Podman does
not allocate enough UIDs or GIDs for your container. In this example,
you tell Podman to allocate `5000`{.fm-code-in-text} UIDs for the
container with `--userns=auto:size=5000`{.fm-code-in-text}:

``` programlisting
# podman run --userns=auto:size=5000 ubi8 cat /proc/self/uid_map
     0 2147486672   5000
```

[]{#10.htm#pgfId-1114500}When containers are removed, Podman reclaims
the UIDs used for the deleted containers and uses those UIDs for the
next container created with the `--userns=auto`{.fm-code-in-text}
`flag`{.fm-code-in-text}. You see this when you launch back-to-back
containers with the `--rm`{.fm-code-in-text}
option[]{#10.htm#marker-1114501}. Notice that they start with the same
UID. In the following example, both containers start with UID
`2147491672`{.fm-code-in-text}:

``` programlisting
# podman run --rm --userns=auto ubi8 cat /proc/self/uid_map
     0 2147491672   1024
# podman run --rm --userns=auto ubi8 cat /proc/self/uid_map
     0 2147491672   1024
```

[]{#10.htm#pgfId-1114506}The name used in /etc/subuid and the minimum
and maximum number of UIDs used for user namespaces is defined in the
storage.conf file described in
[]{#10.htm#marker-1114507}[]{#10.htm#marker-1114508}[]{#10.htm#marker-1114509}[]{#10.htm#marker-1114510}table
10.4.[]{#10.htm#id_z912g9l4dv}

[]{#10.htm#pgfId-1119178}Table 10.4  The fields used within storage.conf
files to override the user namespace auto settings

+-----------------+-----------------------------------------------------+
| [               | []{#10.htm#pgfId-1119184}Description                |
| ]{#10.htm#pgfId |                                                     |
| -1119182}Option |                                                     |
+-----------------+-----------------------------------------------------+
| []{             | []{#10.htm#pgfId-1119188}Defines the username used  |
| #10.htm#pgfId-1 | to look up one or more UID/GID ranges in the        |
| 119186}`root-au | /etc/subuid and /etc/subgid file. These ranges are  |
| to-userns-user` | partitioned into containers configured to create a  |
| {.fm-code-in-te | user namespace automatically. Containers configured |
| xt1}[]{#10.htm# | to automatically create a user namespace can still  |
| marker-1121477} | overlap with containers with an explicit mapping    |
|                 | set. The `root-auto-userns-user`{.fm-code-in-text1} |
|                 | setting is ignored by rootless users. It defaults   |
|                 | to `containers`{.fm-code-in-text1}.                 |
+-----------------+-----------------------------------------------------+
| []              | []{#10.htm#pgfId-1119192}Defines the minimum size   |
| {#10.htm#pgfId- | for a user namespace created automatically. It      |
| 1119190}`auto-u | defaults to `1024`{.fm-code-in-text1}.              |
| serns-min-size` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#10.htm# |                                                     |
| marker-1121482} |                                                     |
+-----------------+-----------------------------------------------------+
| []              | []{#10.htm#pgfId-1119196}Defines the maximum size   |
| {#10.htm#pgfId- | for a user namespace created automatically. It      |
| 1119194}`auto-u | defaults to `65536`{.fm-code-in-text1}.             |
| serns-max-size` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#10.htm# |                                                     |
| marker-1121487} |                                                     |
+-----------------+-----------------------------------------------------+

### []{#10.htm#pgfId-1114536}10.3.2 User-namespaced Linux capabilities {#10.htm#heading_id_15 .fm-head1}

[]{#10.htm#pgfId-1114542}In
[]{#10.htm#marker-1114537}[]{#10.htm#marker-1114538}[]{#10.htm#marker-1114539}[]{#10.htm#marker-1114540}[]{#10.htm#marker-1114541}section
10.2 you learned about Linux capabilities and how they are used to break
up the power of root. When a container is launched within a user
namespace, it can have Linux capabilities. These capabilities can only
affect the UIDs and GIDs mapped into the user namespace. Capabilities
that do not involve UIDs and GIDs are limited. Usually, they only affect
the other namespaces that are mapped with the user namespace.

[]{#10.htm#pgfId-1114543}For example, `CAP_NET_ADMIN`{.fm-code-in-text}
is the capability that allows you to manipulate the network stack. It
allows a process to set up firewall rules and network routing tables. A
process with a namespaced `CAP_NET_ADMIN`{.fm-code-in-text} is only
allowed to modify the namespaced network assigned to the user namespace,
not the host's network namespace.

[]{#10.htm#pgfId-1114544}In the following example, the list of
capabilities within a user-namespaced container is the same as when you
launch one without a user namespace. In the second command using the
`--userns=auto`{.fm-code-in-text} flag[]{#10.htm#marker-1114545}, the
capabilities are namespaced capabilities:

``` programlisting
```bash
# podman run --rm ubi8 capsh --print | grep Current
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,
➥ cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_sys_chroot,
➥ cap_setfcap+eip
# podman run --rm --userns=auto ubi8 capsh --print | grep Current
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,
➥ cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_sys_chroot,
➥ cap_setfcap+eip
```

[]{#10.htm#pgfId-1114554}To prove this, attempt to
`chown`{.fm-code-in-text} a file within a container to a nonexistent
UID. It fails because the `CAP_CHOWN`{.fm-code-in-text}
capability[]{#10.htm#marker-1114555} only allows the root process inside
a container to `chown`{.fm-code-in-text} files to any UID as long as the
UID is mapped to the user namespace:

``` programlisting
# podman run --rm --userns=auto:size=5000 ubi8 chown 6000 /etc/motd
chown: changing ownership of '/etc/motd': Invalid argument
```

[]{#10.htm#pgfId-1114558}It succeeds if you `chown`{.fm-code-in-text} to
a UID mapped within the user namespace:

``` programlisting
# podman run --rm --userns=auto:size=5000 ubi8 chown 4000 /etc/motd
```

[]{#10.htm#pgfId-1114560}Suppose you launch all of your system
containers with the `--userns=auto`{.fm-code-in-text}
`flag`{.fm-code-in-text}. In that case, you get the benefit of running
the container within its unique user namespace isolated from all other
containers and UIDs on the host system. You also get root privileges
with limited capabilities, and these processes outside the container
have no capabilities on the host
[]{#10.htm#marker-1114561}[]{#10.htm#marker-1114562}[]{#10.htm#marker-1114563}[]{#10.htm#marker-1114564}[]{#10.htm#marker-1114565}system.

### []{#10.htm#pgfId-1114567}10.3.3 Rootless Podman with the - -userns=auto flag {#10.htm#heading_id_16 .fm-head1}

[]{#10.htm#pgfId-1114572}The[]{#10.htm#marker-1114568}[]{#10.htm#marker-1114569}[]{#10.htm#marker-1114570}[]{#10.htm#marker-1114571}
`--userns=auto`{.fm-code-in-text} works with rootless containers, based
on the number of UIDs available to the user. But this number is very
limited. You can run the previous examples and see that the user
namespaces start at UID `1`{.fm-code-in-text}. UID `1`{.fm-code-in-text}
is relative to the user namespace of the rootless user:

``` programlisting
$ podman run --userns=auto ubi8 cat /proc/self/uid_map
     0   1      1024
$ podman run --userns=auto ubi8 cat /proc/self/uid_map
     0   1025   1024
```

[]{#10.htm#pgfId-1114577}If you examine your user namespace, you'll see
that UID `1`{.fm-code-in-text} in your user namespace is
`100000`{.fm-code-in-text}:

``` programlisting
$ podman run --rm ubi8 cat /proc/self/uid_map
     0   3267    1
     1   100000    65536
```

[]{#10.htm#pgfId-1114581}This means the first rootless user-namespace
container is running UID `0`{.fm-code-in-text} mapped to UID
`1`{.fm-code-in-text} in the rootless user namespace. UID
`1`{.fm-code-in-text} is the rootless UID `100000`{.fm-code-in-text} on
the host system. A couple of problems with rootless users of
`--userns=auto`{.fm-code-in-text} is that since the default user only
gets 65,536 UIDS, at max, you can launch 64 containers, and you cannot
run any containers that require more than 65,536
[]{#10.htm#marker-1114582}[]{#10.htm#marker-1114583}[]{#10.htm#marker-1114584}[]{#10.htm#marker-1114585}UIDs.

[]{#10.htm#pgfId-1114587}[Note]{.fm-callout-head} If you launch a
container without using the `--userns=auto`{.fm-code-in-text1}
flag[]{#10.htm#marker-1114586}, the UIDs mapped to the user namespace
can and probably do overlap with the UIDs in the user-namespaced
isolated containers. You need to be careful that none of the UIDs used
within such containers use those UIDs because those UIDs are vulnerable
to attack from a UID perspective. To avoid overlaps, I suggest using a
high range of UIDs.

### []{#10.htm#pgfId-1114589}10.3.4 User volumes with the - -userns=auto flag {#10.htm#heading_id_17 .fm-head1}

[]{#10.htm#pgfId-1114594}When
[]{#10.htm#marker-1114590}[]{#10.htm#marker-1114591}[]{#10.htm#marker-1114592}[]{#10.htm#marker-1114593}using
the user namespace, it is difficult to determine which user's UID needs
to own the volume you're mounting into a container to allow access. In
the following example, you first create a directory and then volume
mount it into the container and attempt to create a file in it.

[]{#10.htm#pgfId-1114595}Listing 10.6 Drawbacks of using volumes within
a user namespace

``` programlisting
```bash 
# mkdir /mnt/test
# ls -ld /mnt/test
drwxr-xr-x. 2 root root 6 Feb 8 16:23 /mnt/test                           ❶
# podman run --rm -v /mnt/test:/mnt/test --userns=auto ubi8 ls -ld /mnt/test
drwxr-xr-x. 2 nobody nobody 6 Feb 8 21:23 /mnt/test                       ❷
# podman run --rm -v /mnt/test:/mnt/test:Z --userns=auto ubi8 touch /mnt/test
touch: setting times of '/mnt/test': 
➥ Permission denied                                                      ❸
```

[]{#10.htm#pgfId-1128036}[❶]{.fm-combinumeral} The directory is owned by
root on the host.

[]{#10.htm#pgfId-1128057}[❷]{.fm-combinumeral} The directory is listed
as the user nobody, since the root UID=0 is not mapped into the user
namespace. All files and directories owned by UIDs not mapped to the
container are treated as the nobody user. The :Z tells Podman to relabel
for SELinux.

[]{#10.htm#pgfId-1128081}[❸]{.fm-combinumeral} Even root is not allowed
to write into a directory of an unmapped user, unless the directory is
world writable.

[]{#10.htm#pgfId-1114608}Podman supports a special option on the
`--volume`{.fm-code-in-text} flag
`U`{.fm-code-in-text}[]{#10.htm#marker-1114607}`,`{.fm-code-in-text}
which tells Podman to `chown`{.fm-code-in-text} all files or directories
in the source directory to match the UID of the container's primary
process:

``` programlisting
# ls -ld /mnt/test
drwxr-xr-x. 2 root root 6 Feb 8 16:38 /mnt/test
# podman run --rm -v /mnt/test:/mnt/test:Z,U 
➥ --userns=auto ubi8 touch /mnt/test/test1       ❶
# ls -ld /mnt/test
drwxr-xr-x. 2 2147503960 2147503960 
➥ 19 Feb 8 16:38 /mnt/test                       ❷
```

[]{#10.htm#pgfId-1127904}[❶]{.fm-combinumeral} After adding the U
option, processes within the container can write to the volume.

[]{#10.htm#pgfId-1127925}[❷]{.fm-combinumeral} Podman chowned the source
volume to 2147503960 to match the root user mapping in the container.

[]{#10.htm#pgfId-1114618}A new, advanced feature of the Linux kernel is
called `idmapped`{.fm-code-in-text} `mounts`{.fm-code-in-text}. It
allows users to remap the UIDs inside a source volume to match the user
namespace without actually `chown`{.fm-code-in-text}ing the files on
disk. In the next example, you will recreate the /mnt/test directory
and, this time, mount it with the `idmap`{.fm-code-in-text}
option[]{#10.htm#marker-1114619}. When the ID-mapped volume shows up
inside the container, the files appear to be owned by the root of the
user namespace, and you are allowed to read and write the files based on
standard permissions. When you finish writing the files, they are mapped
back correctly into the user namespace, unlike the `U`{.fm-code-in-text}
option, which writes them back based on the real UID of the container
process:

``` programlisting
# chown -R root:root /mnt/test                         ❶
# podman run --rm -v /mnt/test:/mnt/test:idmap,Z 
➥ --userns=auto ubi8 ls -ld /mnt/test                 ❷
drwxr-xr-x. 2 root root 31 Feb 9 11:56 /mnt/test
# podman run --rm -v /mnt/test:/mnt/test:idmap,Z 
➥ --userns=auto ubi8 touch /mnt/test/test             ❸
# ls -l /mnt/test                                      ❹
total 0
-rw-r--r--. 1 root root 0 Feb 9 06:57 test
-rw-r--r--. 1 root root 0 Feb 8 17:02 test1
```

[]{#10.htm#pgfId-1127607}[❶]{.fm-combinumeral} Reset the source volume
to root ownership.

[]{#10.htm#pgfId-1127628}[❷]{.fm-combinumeral} Mount the source volume
/mnt/test into the container with the idmap option. Notice the path is
owned by root within the container.

[]{#10.htm#pgfId-1127655}[❸]{.fm-combinumeral} Create a file within the
source directory to prove the container can write to the directory.

[]{#10.htm#pgfId-1127672}[❹]{.fm-combinumeral} Notice on the host system
that the newly created file is owned by the real root.

[]{#10.htm#pgfId-1114635}[Note]{.fm-callout-head} The
`idmap`{.fm-code-in-text1} features[]{#10.htm#marker-1114634} are brand
new as of writing and are not available on all filesystems. It is only
supported in privileged mode at this time, but hopefully, this changes
soon. Currently, the OCI runtime that supports this feature is
`crun`{.fm-code-in-text1}.

[]{#10.htm#pgfId-1114636}Understanding the security benefits of running
containers with user namespaces is very important. Next, I'll show you
some security benefits in the
[]{#10.htm#marker-1114637}[]{#10.htm#marker-1114638}[]{#10.htm#marker-1114639}[]{#10.htm#marker-1114640}other
[]{#10.htm#marker-1114641}[]{#10.htm#marker-1114642}[]{#10.htm#marker-1114643}namespaces.

## []{#10.htm#pgfId-1114645}10.4 Process isolation: PID namespace {#10.htm#heading_id_18 .fm-head}

[]{#10.htm#pgfId-1114649}I
[]{#10.htm#marker-1114646}[]{#10.htm#marker-1114647}[]{#10.htm#marker-1114648}often
say that namespaces were not intended as a security mechanism, but in
reality, they do provide additional security via isolation and
information masking. The PID namespace hides the fact that there are
other processes running on a system. Being aware that a particular
application is running on a system can be valuable to someone hacking a
container. When you run a container within its own PID namespace, it is
only able to see the other processes running within the container. By
default, Podman runs containers within their own PID namespaces.

[]{#10.htm#pgfId-1114650}Some applications shipped as container images
require additional access to the system. If you have such an application
that needs to monitor the processes on the host, you'll need to turn off
the PID namespace to expose all the processes on the system. Turning off
the PID namespace with Podman is simple: just add the
`--pid=host`{.fm-code-in-text} flag[]{#10.htm#marker-1114651}. In the
next couple of examples, you see that with the PID namespace, you only
see the container process within the container. The second command
exposes all processes within the system to the
[]{#10.htm#marker-1114652}[]{#10.htm#marker-1114653}[]{#10.htm#marker-1114654}container.

[]{#10.htm#pgfId-1114655}Listing 10.7 The differences between using the
`pid`{.fm-code-in-text} namespace and disabling it

``` programlisting
$ podman run --rm ubi8 find /proc -maxdepth 1 
➥ -type d -regex ".*/[0-9]*"                     ❶
/proc/1
$ podman run --rm --pid=host ubi8 find 
➥ /proc -maxdepth 1 -type d -regex ".*/[0-9]*"   ❷
/proc/1
/proc/2
/proc/3
/proc/4
...
```

[]{#10.htm#pgfId-1127480}[❶]{.fm-combinumeral} Running the find command
looking for all processes within the container, you see only one
process.

[]{#10.htm#pgfId-1127501}[❷]{.fm-combinumeral} Running the find command
in a \--pid=host container, you see all of the processes on the system.

[]{#10.htm#pgfId-1114669}[Note]{.fm-callout-head} On an SELinux system,
exposing the host's processes via the `--pid=host`{.fm-code-in-text1}
option[]{#10.htm#marker-1114668} also has a side effect of disabling
SELinux separation. SELinux blocks access to the host's processes and
causes problems when processes within the container interact with these
processes. Other security mechanisms, like dropped capabilities and user
namespaces, are not dropped and can block access to the processes.

## []{#10.htm#pgfId-1114671}10.5 Network isolation: Network namespace {#10.htm#heading_id_19 .fm-head}

[]{#10.htm#pgfId-1114675}The
[]{#10.htm#marker-1114672}[]{#10.htm#marker-1114673}[]{#10.htm#marker-1114674}network
namespace sets up isolation from the host network. It allows Podman to
set up virtual private networks to control which containers can talk to
other containers. Podman has the ability to create multiple networks and
then assign containers within those networks. By default, all containers
run within the host network. But it is simple to set up additional
networks using the `podman`{.fm-code-in-text}
`network`{.fm-code-in-text} `create`{.fm-code-in-text}
command[]{#10.htm#marker-1114676}. In the next example, you will create
two networks---`net1`{.fm-code-in-text}[]{#10.htm#marker-1114677} and
`net2`{.fm-code-in-text}[]{#10.htm#marker-1114678}:

``` programlisting
$ podman network create net1
net1
$ podman network create net2
net2
```

[]{#10.htm#pgfId-1114683}When you create new containers, you can assign
them to a specific network with the `--network`{.fm-code-in-text}
`net1`{.fm-code-in-text} option[]{#10.htm#marker-1114684}:

``` programlisting
$ podman run -d --network net1 --name 
➥ cnet1 ubi8 sleep 1000                                 ❶
74ce5b2396f77fce8c499b121aeb8731f1e1b22e363a6a72d243487cf93a5897
$ podman run --network net1 alpine 
➥ ping -c 1 cnet1                                       ❷
PING cnet1 (10.89.0.4): 56 data bytes
64 bytes from 10.89.0.4: seq=0 ttl=42 time=0.077 ms
```

[]{#10.htm#pgfId-1127304}[❶]{.fm-combinumeral} Start a background
container in network net1.

[]{#10.htm#pgfId-1127325}[❷]{.fm-combinumeral} Make sure the container
is reachable from another container within the network.

[]{#10.htm#pgfId-1114694}If you attempt to ping the network from the
default network namespace via the container name, or even the IP
address, it fails:

``` programlisting
$ podman run --rm alpine ping -c 1 cnet1
ping: bad address 'cnet1'
$ podman run alpine ping -c 1 10.89.0.4         ❶
PING 10.89.0.4 (10.89.0.4): 56 data bytes
64 bytes from 10.89.0.4: seq=0 ttl=42 time=0.073 ms
```

[]{#10.htm#pgfId-1127243}[❶]{.fm-combinumeral} Make sure the cnet1
container is still available by the IP address.

[]{#10.htm#pgfId-1114701}Similarly, if you attempt to ping it from a
different network, `--network`{.fm-code-in-text}
`net2`{.fm-code-in-text}, it also fails:

``` programlisting
$ podman run --rm --network net2 alpine ping -c 1 cnet1
ping: bad address 'cnet1'
```

[]{#10.htm#pgfId-1114704}Creating private networks for your containers
allows you to isolate them from each other, even over the network, using
the network namespace.

[]{#10.htm#pgfId-1114705}[Note]{.fm-callout-head} For these examples, I
used the alpine image because it comes with the ping package installed,
while the ubi8 image does not include it. You can easily add the ping
executable to ubi8 via a Containerfile and `podman`{.fm-code-in-text1}
`build`{.fm-code-in-text1}.

[]{#10.htm#pgfId-1114706}You can expose your host network to the
container using the `--net=host`{.fm-code-in-text} option, allowing a
container to bind to ports on the host. In certain situations, you can
get better performance when you eliminate the network
[]{#10.htm#marker-1114707}[]{#10.htm#marker-1114708}[]{#10.htm#marker-1114709}namespace.

## []{#10.htm#pgfId-1114711}10.6 IPC isolation: IPC namespace {#10.htm#heading_id_20 .fm-head}

[]{#10.htm#pgfId-1114715}The
[]{#10.htm#marker-1123073}[]{#10.htm#marker-1123074}[]{#10.htm#marker-1123075}inter-process
communication (IPC) namespace isolates certain IPC resources, namely,
System V IPC objects and POSIX message queues. It also isolates the
/dev/shm tmpfs from the host and other containers. The IPC namespace
allows containers to create named IPCs with the same name as other
containers on the same system, without causing a conflict.

[]{#10.htm#pgfId-1114716}Thus, IPC isolation prevents one container from
attacking another via an IPC or /dev/shm. You can join two
IPC-namespaced containers together using the
`--ipc=container:NAME`{.fm-code-in-text} or run them within a pod. They
share the same IPC namespace. They can use IPC together but are still
isolated from the host.

[]{#10.htm#pgfId-1114717}Listing 10.8 IPC namespace keeping /dev/shm
private to each container

``` programlisting
$ podman run -d --rm --name ipc1 ubi8 bash 
➥ -c "touch /dev/shm/ipc1; sleep 1000"                              ❶
93df44264dd4b87d24f59dfffb92a6a0b6359bc5bcf94213d5e38499a10d3f3e
$ podman run --rm ubi8 ls /dev/shm                                   ❷
$ podman run --rm --ipc=container:ipc1 ubi8 ls /dev/shm              ❸
ipc1
```

[]{#10.htm#pgfId-1127009}[❶]{.fm-combinumeral} Create a container named
ipc1, touch /dev/shm/ipc1, and then go to sleep.

[]{#10.htm#pgfId-1127030}[❷]{.fm-combinumeral} Run a second container to
see that the /dev/shm/ipc does not exist because the container is
running in a separate IPC namespace.

[]{#10.htm#pgfId-1127047}[❸]{.fm-combinumeral} Run a container with a
shared IPC namespace, and you will see that the /dev/shm is shared and
the IPC file exists.

[]{#10.htm#pgfId-1114727}You can share the host's IPC namespace with
your container by executing the `--ipc=host`{.fm-code-in-text}
[]{#10.htm#marker-1114728}[]{#10.htm#marker-1114729}[]{#10.htm#marker-1114730}option.

[]{#10.htm#pgfId-1114731}[Note]{.fm-callout-head} On SELinux systems,
Podman modifies all containers that share the same IPC namespace to
share the same SELinux label. Otherwise, SELinux blocks the IPC
communications between containers when the labels do not match. Using
the `--ipc=host`{.fm-code-in-text1} option causes SELinux separation to
be disabled; otherwise, SELinux blocks access to the host's IPC.

## []{#10.htm#pgfId-1114733}10.7 Filesystem isolation: Mount namespace {#10.htm#heading_id_21 .fm-head}

[]{#10.htm#pgfId-1114738}The
[]{#10.htm#marker-1114734}[]{#10.htm#marker-1114735}[]{#10.htm#marker-1114736}[]{#10.htm#marker-1114737}next,
and perhaps most important, namespace isolation is the mount namespace.
The mount namespace hides the entire host filesystem from the container
processes. The container processes only see the filesystem content
defined to be in the mount namespace. Podman creates the filesystem
mount point `rootfs`{.fm-code-in-text}[]{#10.htm#marker-1114739} and
bind mounts all volumes onto it. Podman then executes the OCI runtime,
which then executes the `pivot_root`{.fm-code-in-text}
syscall[]{#10.htm#marker-1114740}, which in turn changes the root mount
in the mount namespace of the calling process. It moves the root mount
to the rootfs directory. Thus, all of the content of the host operating
system disappears, and the container processes only see the provided
content. By dropping the `CAP_SYS_ADMIN`{.fm-code-in-text}
capability[]{#10.htm#marker-1114741}, the processes inside the container
have no ability to affect the mounts of the rootfs to expose the
underlying filesystems.

[]{#10.htm#pgfId-1114743}[Note]{.fm-callout-head} Read the
`pivot_root(2)`{.fm-code-in-text1} man page[]{#10.htm#marker-1114742} to
find out more about the `pivot_ root`{.fm-code-in-text1} system call:
`man`{.fm-code-in-text1} `2`{.fm-code-in-text1}
`pivot_root`{.fm-code-in-text1}.

[]{#10.htm#pgfId-1114744}While the mount namespace and the lack of
`CAP_SYS_ADMIN`{.fm-code-in-text} provide excellent isolation, there
have been some container escapes to the underlying filesystem, which is
where SELinux steps in. One example of this was a flaw in the OCI
runtime `runc`{.fm-code-in-text}[]{#10.htm#marker-1114745}
(CVE-2019-5736), which allowed container processes to overwrite the
`runc`{.fm-code-in-text} executable[]{#10.htm#marker-1114746} in rootful
containers. This exploit allowed containers to escape their containment
and take over users' systems. This exploit affected all container
engines, including Podman, Docker, CRI-O, and containerd. The good news
is that well-configured SELinux can stop it. Podman is mainly run in
rootless mode, and rootless Podman is protected in two ways: SELinux and
not running as root. I wrote about this exploit in this "Latest
container exploit (runc) can be blocked by SELinux" blog post, available
on Red Hat's website
[]{#10.htm#marker-1114747}[]{#10.htm#marker-1114748}[]{#10.htm#marker-1114749}[]{#10.htm#marker-1114750}([http://mng.bz/Qn6j](http://mng.bz/Qn6j){.url}).

## []{#10.htm#pgfId-1114752}10.8 Filesystem isolation: SELinux {#10.htm#heading_id_22 .fm-head}

[]{#10.htm#pgfId-1114757}SELinux
[]{#10.htm#marker-1114753}[]{#10.htm#marker-1114754}[]{#10.htm#marker-1114755}[]{#10.htm#marker-1114756}is
a labeling system, where every process and filesystem object gets
labeled. Then rules are written to the kernel about how the process
labels interact with the filesystem labels as well as other process
labels. SELinux supports multiple different security mechanisms;
containers take advantage of two of these. The first is called *type
enforcement*, with which SELinux controls what processes can do based on
their type. The second is called *MCS enforcement*, and it additionally
uses categories assigned to processes.

[]{#10.htm#pgfId-1114758}SELinux is not supported on all distributions.
Fedora, RHEL, and other Red Hat distributions support SELinux, while
Debian-based distributions, like Ubuntu, often do not. If your Linux
distribution does not support SELinux, you might want to skip this
section.

### []{#10.htm#pgfId-1114760}10.8.1 SELinux type enforcement {#10.htm#heading_id_23 .fm-head1}

[]{#10.htm#pgfId-1114766}The
[]{#10.htm#marker-1120087}[]{#10.htm#marker-1120088}[]{#10.htm#marker-1120089}[]{#10.htm#marker-1120090}[]{#10.htm#marker-1120091}SELinux
labels have four components: the SELinux user, role, type, and MCS level
(see table 10.5).[]{#10.htm#id_hi509vooki1y}

[]{#10.htm#pgfId-1119978}Table 10.5 SELinux label type examples

+-------------+-------------+-------------+-------------+-------------+
| []{#10.ht   | []{#10.     | []{#10.     | []{#10.     | []{#10      |
| m#pgfId-111 | htm#pgfId-1 | htm#pgfId-1 | htm#pgfId-1 | .htm#pgfId- |
| 9988}Object | 119990}User | 119992}Role | 119994}Type | 1119996}MCS |
|             |             |             |             | level       |
+-------------+-------------+-------------+-------------+-------------+
| [           | []{#10.htm  | []{#10.htm  | []          | []{#10.htm  |
| ]{#10.htm#p | #pgfId-1120 | #pgfId-1120 | {#10.htm#pg | #pgfId-1120 |
| gfId-111999 | 000}`system | 002}`system | fId-1120004 | 006}`s0:c1, |
| 8}Container | _u`{.fm-cod | _r`{.fm-cod | }`container | c2`{.fm-cod |
| process     | e-in-text1} | e-in-text1} | _t`{.fm-cod | e-in-text1} |
|             |             |             | e-in-text1} |             |
+-------------+-------------+-------------+-------------+-------------+
| [           | []{#10.htm  | []{#10.htm  | []          | []{         |
| ]{#10.htm#p | #pgfId-1120 | #pgfId-1120 | {#10.htm#pg | #10.htm#pgf |
| gfId-112000 | 010}`system | 012}`system | fId-1120014 | Id-1120016} |
| 8}Container | _u`{.fm-cod | _r`{.fm-cod | }`container | `s0:c361,c8 |
| process     | e-in-text1} | e-in-text1} | _t`{.fm-cod | 71`{.fm-cod |
|             |             |             | e-in-text1} | e-in-text1} |
+-------------+-------------+-------------+-------------+-------------+
| [           | []{#10.htm  | []{#10.htm  | []{#10.     | []{#10.htm  |
| ]{#10.htm#p | #pgfId-1120 | #pgfId-1120 | htm#pgfId-1 | #pgfId-1120 |
| gfId-112001 | 020}`system | 022}`object | 120024}`con | 026}`s0:c1, |
| 8}Container | _u`{.fm-cod | _r`{.fm-cod | tainer_file | c2`{.fm-cod |
| file        | e-in-text1} | e-in-text1} | _t`{.fm-cod | e-in-text1} |
|             |             |             | e-in-text1} |             |
+-------------+-------------+-------------+-------------+-------------+
| [           | []{#10.htm  | []{#10.htm  | []{#10.     | []{         |
| ]{#10.htm#p | #pgfId-1120 | #pgfId-1120 | htm#pgfId-1 | #10.htm#pgf |
| gfId-112002 | 030}`system | 032}`object | 120034}`con | Id-1120036} |
| 8}Container | _u`{.fm-cod | _r`{.fm-cod | tainer_file | `s0:s361,c8 |
| file        | e-in-text1} | e-in-text1} | _t`{.fm-cod | 71`{.fm-cod |
|             |             |             | e-in-text1} | e-in-text1} |
+-------------+-------------+-------------+-------------+-------------+
| []{         | []{#10.htm  | []{#10.htm  | []{#10.htm  | []{#        |
| #10.htm#pgf | #pgfId-1120 | #pgfId-1120 | #pgfId-1120 | 10.htm#pgfI |
| Id-1120038} | 040}`system | 042}`object | 044}`shadow | d-1120046}` |
| /etc/shadow | _u`{.fm-cod | _r`{.fm-cod | _t`{.fm-cod | s0`{.fm-cod |
| label       | e-in-text1} | e-in-text1} | e-in-text1} | e-in-text1} |
+-------------+-------------+-------------+-------------+-------------+
| [           | []{#10.htm  | []{#10.htm  | []{#10.     | []{#        |
| ]{#10.htm#p | #pgfId-1120 | #pgfId-1120 | htm#pgfId-1 | 10.htm#pgfI |
| gfId-112004 | 050}`system | 052}`system | 120054}`spc | d-1120056}` |
| 8}Container | _u`{.fm-cod | _r`{.fm-cod | _t`{.fm-cod | s0`{.fm-cod |
| process     | e-in-text1} | e-in-text1} | e-in-text1} | e-in-text1} |
+-------------+-------------+-------------+-------------+-------------+
| []{#10.     | []{         | []{         | []{         | []{#1       |
| htm#pgfId-1 | #10.htm#pgf | #10.htm#pgf | #10.htm#pgf | 0.htm#pgfId |
| 120058}User | Id-1120060} | Id-1120062} | Id-1120064} | -1120066}`s |
| process     | `unconfined | `unconfined | `unconfined | 0-s0:c0.c10 |
|             | _u`{.fm-cod | _r`{.fm-cod | _t`{.fm-cod | 23`{.fm-cod |
|             | e-in-text1} | e-in-text1} | e-in-text1} | e-in-text1} |
+-------------+-------------+-------------+-------------+-------------+

[]{#10.htm#marker-1125639}[]{#10.htm#pgfId-1114852}In this section, you
will concentrate on the SELinux type. I wrote *The SELinux Coloring
Book* to explain the labeling, using the analogy of cats and dogs
(figure 10.3).

::: figure
![](images/OEBPS/Images/10-03.png){.calibre18}

[]{#10.htm#pgfId-1130140}Figure 10.3 *The SELinux Coloring
Book*[]{#10.htm#id_1vp0m2w67vu8}[]{#10.htm#marker-1130141}
([http://mng.bz/Xay6](http://mng.bz/Xay6){.url})
:::

[]{#10.htm#pgfId-1114859}As the coloring book explains, imagine you have
a group of processes labeled as `cat`{.fm-code-in-text} types and
another group of processes labeled as `dog`{.fm-code-in-text} types.
Imagine you also have objects on the filesystem labeled as
`dog`{.fm-code-in-text} `food`{.fm-code-in-text} type and
`cat`{.fm-code-in-text} `food`{.fm-code-in-text} type. Finally, imagine
you write rules to the kernel saying that `cat`{.fm-code-in-text} types
are allowed to eat `cat`{.fm-code-in-text} `food`{.fm-code-in-text}
types, and `dog`{.fm-code-in-text} types can eat `dog`{.fm-code-in-text}
`food`{.fm-code-in-text} types. With SELinux, anything that is not
explicitly allowed is denied. The `cat`{.fm-code-in-text} processes can
eat the `cat`{.fm-code-in-text} `food`{.fm-code-in-text}, and the
`dog`{.fm-code-in-text} processes can eat the `dog`{.fm-code-in-text}
`food`{.fm-code-in-text}, but if a `dog`{.fm-code-in-text} type attempts
to eat `cat`{.fm-code-in-text} `food`{.fm-code-in-text}, the Linux
kernel steps in and blocks the access.

[]{#10.htm#pgfId-1114860}Containers work the same way. Podman labels
each container process with the `container_t`{.fm-code-in-text}
type[]{#10.htm#marker-1114861}. All the files within the container are
labeled as a `container_file_t`{.fm-code-in-text}
type[]{#10.htm#marker-1114862}. Rules are written into the kernel,
saying the `container_t`{.fm-code-in-text} processes are allowed to
read, write, and execute files labeled with the
`container_file_t`{.fm-code-in-text} type[]{#10.htm#marker-1114863}.

[]{#10.htm#pgfId-1114864}[Note]{.fm-callout-head} SELinux does not care
about ownerships and permissions, so you can, for example, define a
process type that has access to all filesystem types and is not confined
by SELinux, often called an *unconfined type*. You can see a couple of
unconfined types running on your Linux system. The
`id`{.fm-code-in-text1} `-Z`{.fm-code-in-text1} command shows your user
processes are running with the `unconfined_t`{.fm-code-in-text1}
type[]{#10.htm#marker-1114865} and a privileged container runs with the
`spc_t`{.fm-code-in-text1} type.

[]{#10.htm#pgfId-1114866}When Podman constructs the rootfs for the
container, it labels all of the files in the rootfs as
`container_file_t`{.fm-code-in-text}. This means the container process
can read, write, and execute all of the files within the container's
rootfs, but if they escape to the host filesystem, the SELinux kernel
blocks access to the host filesystem objects. In the next few examples,
you can examine what is happening in containers with SELinux. In this
first example, you can see the label of the containerized process;
notice the type is
`container_t`{.fm-code-in-text}[]{#10.htm#marker-1114867}. But when you
run with the `--privileged`{.fm-code-in-text}
flag[]{#10.htm#marker-1114868}, Podman changes the label to
`spc_t`{.fm-code-in-text}, an unconfined domain:

``` programlisting
$ podman run --rm ubi8 cat /proc/self/attr/current
system_u:system_r:container_t:s0:c694,c944
$ podman run --rm --privileged ubi8 cat /proc/self/attr/current
unconfined_u:system_r:spc_t:s0
```

[]{#10.htm#pgfId-1114874}Examine the files within the container, using
the `ls`{.fm-code-in-text} `-Z`{.fm-code-in-text}
command[]{#10.htm#marker-1114873}. You see the files are all labeled as
`container_file_t`{.fm-code-in-text}:

``` programlisting
$ podman run --rm ubi8 ls -Z /
system_u:object_r:container_file_t:s0:c88,c191 bin
system_u:object_r:container_file_t:s0:c88,c191 boot
system_u:object_r:container_file_t:s0:c88,c191 dev
system_u:object_r:container_file_t:s0:c88,c191 etc
system_u:object_r:container_file_t:s0:c88,c191 home
system_u:object_r:container_file_t:s0:c88,c191 lib
...
```

[]{#10.htm#pgfId-1114883}Because Podman configured the SELinux
environment properly, container processes have full access to all of the
objects within the container's rootfs, and SELinux pretty much stays out
of the way, unless something else breaks down and somehow the container
process escapes out of the rootfs into the host operating system. At
that point, SELinux starts blocking access. Imagine the container
process you are running on your system broke out of the container and
attempted to read the SSH keys in your home directory. Let's look at the
labels on those files. You see that those files are labeled with the
`ssh_home_t`{.fm-code-in-text} type[]{#10.htm#marker-1114884}:

``` programlisting
$ ls -1Z $HOME/.ssh/
unconfined_u:object_r:ssh_home_t:s0 authorized_keys
unconfined_u:object_r:ssh_home_t:s0 authorized_keys2
unconfined_u:object_r:ssh_home_t:s0 config
...
```

[]{#10.htm#pgfId-1114891}Because there is no rule in SELinux policy
allowing a `container_t`{.fm-code-in-text}
process[]{#10.htm#marker-1114890} to read an
`ssh_home_t`{.fm-code-in-text} file, the SELinux kernel blocks access.
You can demonstrate this by volume mounting the .ssh directory into a
container. When you attempt to list the directory, the container process
gets `Permission`{.fm-code-in-text} `denied`{.fm-code-in-text}:

``` programlisting
$ podman run -v $HOME/.ssh:/.ssh ubi8 ls /.ssh
ls: cannot open directory '/.ssh': Permission denied
```

[]{#10.htm#pgfId-1114894}As you learned in section 3.1.2, Podman has
SELinux volume options `z`{.fm-code-in-text} and `Z`{.fm-code-in-text},
which tell SELinux to relabel the content of the source volume to make
it usable inside of the container. This is not a good idea to do with
the .ssh directory.

[]{#10.htm#pgfId-1114895}Instead, let's create a temporary file and show
the SELinux labels in action. First, create a temporary file in your
home directory named foo. Label it `user_home_t`{.fm-code-in-text}.
Volume mount it into the container, and see that the container process
is denied access.

[]{#10.htm#pgfId-1114896}Listing 10.9 How SELinux works with volumes
inside Podman containers

``` programlisting
$ mkdir foo
$ ls -Zd foo                                          ❶
unconfined_u:object_r:user_home_t:s0 foo
$ podman run -v ./foo:/foo ubi8 touch /foo/bar        ❷
touch: cannot touch '/foo/bar': Permission denied
$ podman run --privileged -v ./foo:/foo ubi8 touch 
➥ /foo/bar                                           ❸
$ ls -Z foo                                           ❹
unconfined_u:object_r:user_home_t:s0 bar
$ rm foo/bar
$ podman run -v ./foo:/foo:Z ubi8 touch /foo/bar      ❺
$ ls -Z ./foo                                         ❻
system_u:object_r:container_file_t:s0:c454,c510 bar
```

[]{#10.htm#pgfId-1126543}[❶]{.fm-combinumeral} Files created in your
home directory default to the user_home_t type.

[]{#10.htm#pgfId-1126567}[❷]{.fm-combinumeral} By default, container
processes are not allowed to write to content in the user\'s home
directory. Podman does not change the labels on volumes by default.

[]{#10.htm#pgfId-1126584}[❸]{.fm-combinumeral} The \--privileged flag
causes SELinux separation to be disabled, running the container with an
unconfined type (spc_t). The command simulates a container escape,
showing that without SELinux, an escaped container is allowed to write
to the filesystem.

[]{#10.htm#pgfId-1126601}[❹]{.fm-combinumeral} The file created by the
privileged container has the label of the user home directory
(user_home_t).

[]{#10.htm#pgfId-1126618}[❺]{.fm-combinumeral} The :Z option on the
volume mount tells Podman to relabel the content of the directory to
match the labels of files within the rootfs (container_file_t).

[]{#10.htm#pgfId-1126635}[❻]{.fm-combinumeral} The labels of the newly
created file match the label within the container.

[]{#10.htm#pgfId-1114916}SELinux type enforcement has shown itself to be
invaluable in blocking container escape when no other mechanism was
available. Table 10.6 shows a list of container escapes that have been
blocked by SELinux.[]{#10.htm#id_k8ptbkj4chg7}

[]{#10.htm#pgfId-1114946}SELinux type enforcement does a great job
protecting the host operating system from container processes. The
problem is that `type`{.fm-code-in-text} `enforcement`{.fm-code-in-text}
does not protect you from one container attacking
[]{#10.htm#marker-1114947}[]{#10.htm#marker-1114948}[]{#10.htm#marker-1114949}[]{#10.htm#marker-1114950}[]{#10.htm#marker-1114951}another.

[]{#10.htm#pgfId-1120491}Table 10.6 Major container exploits blocked by
SELinux

+-----------------+-----------------------------------------------------+
| [               | []{#10.htm#pgfId-1120497}Description                |
| ]{#10.htm#pgfId |                                                     |
| -1120495}Common |                                                     |
| vulnerabilities |                                                     |
| and exposures   |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.h        | []{#10.htm#pgfId-1120501}Execution of malicious     |
| tm#pgfId-112049 | containers allows for container escape and access   |
| 9}CVE-2019-5736 | to the host filesystem.                             |
+-----------------+-----------------------------------------------------+
| []{#10.h        | []{#10.htm#pgfId-1120505}Insecure opening of        |
| tm#pgfId-112050 | file-descriptor 1, leading to privilege escalation  |
| 3}CVE-2015-3627 |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.h        | []{#10.htm#pgfId-1120509}Read/write proc paths      |
| tm#pgfId-112050 | allow host modification and information disclosure. |
| 7}CVE-2015-3630 |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.h        | []{#10.htm#pgfId-1120513}Volume mounts allow Linux  |
| tm#pgfId-112051 | Security Modules (LSM) profile escalation.          |
| 1}CVE-2015-3631 |                                                     |
+-----------------+-----------------------------------------------------+
| []{#10.h        | []{#10.htm#pgfId-1120517}`runc`{.fm-code-in-text1}  |
| tm#pgfId-112051 | exec vulnerability                                  |
| 5}CVE-2016-9962 |                                                     |
+-----------------+-----------------------------------------------------+

### []{#10.htm#pgfId-1114953}10.8.2 SELinux Multi-Category Security separation {#10.htm#heading_id_24 .fm-head1}

[]{#10.htm#pgfId-1114960}SELinux[]{#10.htm#marker-1114954}
[]{#10.htm#marker-1114955}[]{#10.htm#marker-1114956}[]{#10.htm#marker-1114957}[]{#10.htm#marker-1114958}[]{#10.htm#marker-1114959}does
not block processes of one type from attacking other processes of the
same type. One way to think about this is going back to the cats and
dogs analogy. Type enforcement prevents the `dog`{.fm-code-in-text} from
eating the `cat`{.fm-code-in-text} `food`{.fm-code-in-text}, but it does
not prevent `cat-A`{.fm-code-in-text} from eating
`cat-B`{.fm-code-in-text}'s `cat`{.fm-code-in-text}
`food`{.fm-code-in-text}.

[]{#10.htm#pgfId-1114961}Recall when I introduced this section, I said
there were two types of SELinux security Podman takes advantage of.
SELinux has a mechanism to enforce process separation based on the
Multi-Category Security (MCS) level field. SELinux defines 1,024
categories, which can be combined together to give a level to each
container. Podman allocates two categories for each container and then
makes sure the process label level matches the filesystem label levels.
Then the SELinux kernel enforces the MCS levels matching, or the access
is denied.

[]{#10.htm#pgfId-1114962}[Note]{.fm-callout-head} MCS Separation is
actually about dominance. Each category must dominate the MCS level. A
level of `S0:C1,C2`{.fm-code-in-text1} can write to levels
`S0:C1,C2`{.fm-code-in-text1}, `S0:C1`{.fm-code-in-text1},
`S0:C2`{.fm-code-in-text1}, and `S0`{.fm-code-in-text1}. But the
`S0:C1,C2`{.fm-code-in-text1} is not allowed to write to
`S0:C1,C3`{.fm-code-in-text1}, since the original label does not include
the `C3`{.fm-code-in-text1}. In practice, Podman only uses two
categories or no categories. When you use the `:z`{.fm-code-in-text1}
option on a volume, Podman relabels the source directory with the level
`s0`{.fm-code-in-text1}---no categories. The `s0`{.fm-code-in-text1}
allows processes from any container to read and write filesystem objects
with this level, from an SELinux perspective.

[]{#10.htm#pgfId-1114963}Revisit table 10.4, but this time concentrate
on the MCS level field (table 10.7).[]{#10.htm#id_vyh8g5fd3gfv}

[]{#10.htm#pgfId-1120627}Table 10.7 Container process labels, with MCS
level highlighted

+-------------+-------------+-------------+-------------+-------------+
| []{#10.ht   | []{#10.     | []{#10.     | []{#10.     | []{#10      |
| m#pgfId-112 | htm#pgfId-1 | htm#pgfId-1 | htm#pgfId-1 | .htm#pgfId- |
| 0637}Object | 120639}User | 120641}Role | 120643}Type | 1120645}MCS |
|             |             |             |             | level       |
+-------------+-------------+-------------+-------------+-------------+
| [           | []{#10.htm  | []{#10.htm  | []          | []{#10.htm  |
| ]{#10.htm#p | #pgfId-1120 | #pgfId-1120 | {#10.htm#pg | #pgfId-1120 |
| gfId-112064 | 649}`system | 651}`system | fId-1120653 | 655}`s0:c1, |
| 7}Container | _u`{.fm-cod | _r`{.fm-cod | }`container | c2`{.fm-cod |
| process     | e-in-text1} | e-in-text1} | _t`{.fm-cod | e-in-text1} |
|             |             |             | e-in-text1} |             |
+-------------+-------------+-------------+-------------+-------------+
| [           | []{#10.htm  | []{#10.htm  | []          | []{         |
| ]{#10.htm#p | #pgfId-1120 | #pgfId-1120 | {#10.htm#pg | #10.htm#pgf |
| gfId-112065 | 659}`system | 661}`system | fId-1120663 | Id-1120665} |
| 7}Container | _u`{.fm-cod | _r`{.fm-cod | }`container | `s0:c361,c8 |
| process     | e-in-text1} | e-in-text1} | _t`{.fm-cod | 71`{.fm-cod |
|             |             |             | e-in-text1} | e-in-text1} |
+-------------+-------------+-------------+-------------+-------------+
| [           | []{#10.htm  | []{#10.htm  | []{#10.     | []{#10.htm  |
| ]{#10.htm#p | #pgfId-1120 | #pgfId-1120 | htm#pgfId-1 | #pgfId-1120 |
| gfId-112066 | 669}`system | 671}`object | 120673}`con | 675}`s0:c1, |
| 7}Container | _u`{.fm-cod | _r`{.fm-cod | tainer_file | c2`{.fm-cod |
| file        | e-in-text1} | e-in-text1} | _t`{.fm-cod | e-in-text1} |
|             |             |             | e-in-text1} |             |
+-------------+-------------+-------------+-------------+-------------+
| [           | []{#10.htm  | []{#10.htm  | []{#10.     | []{         |
| ]{#10.htm#p | #pgfId-1120 | #pgfId-1120 | htm#pgfId-1 | #10.htm#pgf |
| gfId-112067 | 679}`system | 681}`object | 120683}`con | Id-1120685} |
| 7}Container | _u`{.fm-cod | _r`{.fm-cod | tainer_file | `s0:s361,c8 |
| file        | e-in-text1} | e-in-text1} | _t`{.fm-cod | 71`{.fm-cod |
|             |             |             | e-in-text1} | e-in-text1} |
+-------------+-------------+-------------+-------------+-------------+
| []{         | []{#10.htm  | []{#10.htm  | []{#10.htm  | []{#        |
| #10.htm#pgf | #pgfId-1120 | #pgfId-1120 | #pgfId-1120 | 10.htm#pgfI |
| Id-1120687} | 689}`system | 691}`object | 693}`shadow | d-1120695}` |
| /etc/shadow | _u`{.fm-cod | _r`{.fm-cod | _t`{.fm-cod | s0`{.fm-cod |
| label       | e-in-text1} | e-in-text1} | e-in-text1} | e-in-text1} |
+-------------+-------------+-------------+-------------+-------------+
| [           | []{#10.htm  | []{#10.htm  | []{#10.     | []{#        |
| ]{#10.htm#p | #pgfId-1120 | #pgfId-1120 | htm#pgfId-1 | 10.htm#pgfI |
| gfId-112069 | 699}`system | 701}`system | 120703}`spc | d-1120705}` |
| 7}Container | _u`{.fm-cod | _r`{.fm-cod | _t`{.fm-cod | s0`{.fm-cod |
| process     | e-in-text1} | e-in-text1} | e-in-text1} | e-in-text1} |
+-------------+-------------+-------------+-------------+-------------+
| []{#10.     | []{         | []{         | []{         | []{#1       |
| htm#pgfId-1 | #10.htm#pgf | #10.htm#pgf | #10.htm#pgf | 0.htm#pgfId |
| 120707}User | Id-1120709} | Id-1120711} | Id-1120713} | -1120715}`s |
| process     | `unconfined | `unconfined | `unconfined | 0-s0:c0.c10 |
|             | _u`{.fm-cod | _r`{.fm-cod | _t`{.fm-cod | 23`{.fm-cod |
|             | e-in-text1} | e-in-text1} | e-in-text1} | e-in-text1} |
+-------------+-------------+-------------+-------------+-------------+

[]{#10.htm#pgfId-1115049}Now look at how MCS leveling works with Podman.
If you run containers back to back and examine the SELinux label, you
notice that each container's MCS level is unique:

``` programlisting
$ podman run --rm ubi8 cat /proc/self/attr/current
System_u:system_r:container_t:s0:c648,c1009
$ podman run --rm ubi8 cat /proc/self/attr/current
system_u:system_r:container_t:s0:c393,c834
```

[]{#10.htm#pgfId-1115054}This MCS level prevents the processes from
attacking each other. Recall that in section 10.2.8, you created the
foo/bar file with a container private label. If you volume mount this
file into another container and then try to write to the file, you get
permission denied.

[]{#10.htm#pgfId-1115055}Listing 10.10 SELinux preventing different
containers from sharing a volume

``` programlisting
$ ls -Z ./foo                                            ❶
system_u:object_r:container_file_t:s0:c454,c510 bar
$ podman run -v ./foo:/foo ubi8 touch /foo/bar           ❷
touch: cannot touch '/foo/bar': Permission denied
$ podman run --security-opt label=level:s0:c454,c510 
➥ -v ./foo:/foo ubi8 touch /foo/bar                     ❸
```

[]{#10.htm#pgfId-1126309}[❶]{.fm-combinumeral} The file foo/bar has a
private MCS level, which Podman does not give to another container.

[]{#10.htm#pgfId-1126330}[❷]{.fm-combinumeral} Other containers are not
allowed to access the foo/bar file based on having a different MCS
level.

[]{#10.htm#pgfId-1126347}[❸]{.fm-combinumeral} If you force the
container MCS level to match the previous container's label, SELinux
allows the access.

[]{#10.htm#pgfId-1115065}Recall that the `Z`{.fm-code-in-text} volume
option tells Podman to label the container private to the container,
while the `z`{.fm-code-in-text} volume option tells Podman to label the
container shared for all containers. You can use this option if you have
a directory you want to allow multiple containers to use.

[]{#10.htm#pgfId-1115066}Listing 10.11 Volume option
`z`{.fm-code-in-text} causing Podman to relabel volumes with a shared
label

``` programlisting
$ podman run -v ./foo:/foo:z ubi8 touch /foo/bar       ❶
$ ls -Z foo/                                           ❷
system_u:object_r:container_file_t:s0 bar
$ podman run --rm -v ./foo:/foo ubi8 touch /foo/bar    ❸
```

[]{#10.htm#pgfId-1126108}[❶]{.fm-combinumeral} The -v ./foo:/foo:z tells
Podman to label the volume as shared.

[]{#10.htm#pgfId-1126129}[❷]{.fm-combinumeral} Podman uses the :s0 MCS
level because all containers are allowed to write to it.

[]{#10.htm#pgfId-1126146}[❸]{.fm-combinumeral} Other containers with
different MCS levels can successfully modify the content.

[]{#10.htm#pgfId-1115074}[Note]{.fm-callout-head} SELinux has 1,024
categories, and Podman chooses two categories for each container. Level
`s0:c1,c1`{.fm-code-in-text1} is not allowed. These categories must not
match, and the order is not important. Level
`s0:c1,c2`{.fm-code-in-text1} is the same as
`s0:c2,c1`{.fm-code-in-text1}. There are 1024 x 1024 ÷ 2 -- 1024 =
\~500,000 unique combinations available, meaning you can create half a
million unique containers on your system.

[]{#10.htm#pgfId-1115075}Sometimes it is necessary to disable SELinux
container separation for your container. For example, you might want to
share your home directory within a container. It is a bad idea to
relabel your home directory with the `Z`{.fm-code-in-text} or
`z`{.fm-code-in-text} options. Recall that when relabeling volumes, they
need to be private to the container. Relabeling the home directory can
cause other SELinux problems with other confined domains. You can run
the container with the `--privileged`{.fm-code-in-text} flag, but you
probably want the other security mechanisms to still be enforced. To
achieve this, you can use the `--security-opt`{.fm-code-in-text}
`label:disable`{.fm-code-in-text} flag[]{#10.htm#marker-1115076}:

``` programlisting
$ podman run --rm --security-opt label=disable ubi8 cat 
➥ /proc/self/attr/current
unconfined_u:system_r:spc_t:s0
$ podman run --rm -v $HOME/.ssh:/ssh --security-opt label=disable ubi8 ls /ssh
authorized_keys
authorized_keys2
config
fedora_rsa
fedora_rsa.pub
...
```

[]{#10.htm#pgfId-1115087}[Note]{.fm-callout-head} The udica project's
([https://github.com/containers/udica](https://github.com/containers/udica){.url})
goal is to generate SELinux policies for containers. Basically, Udica
examines a container you have created via `podman`{.fm-code-in-text1}
`inspect`{.fm-code-in-text1} and then writes a policy type that allows
access to the volumes you want to mount into the container.

[]{#10.htm#pgfId-1115088}SELinux is a very powerful tool for protecting
the host operating system from the containers. It is easy to deal with
for containers as long as you understand how to handle volumes.
Understanding how to protect the filesystem, it is time now to look at
protecting the Linux kernel from potentially
vulnerable[]{#10.htm#marker-1115089}
[]{#10.htm#marker-1115090}[]{#10.htm#marker-1115091}[]{#10.htm#marker-1115092}[]{#10.htm#marker-1115093}[]{#10.htm#marker-1115094}system
[]{#10.htm#marker-1115095}[]{#10.htm#marker-1115096}[]{#10.htm#marker-1115097}[]{#10.htm#marker-1115098}calls.

## []{#10.htm#pgfId-1115100}10.9. System call isolation seccomp {#10.htm#heading_id_25 .fm-head}

[]{#10.htm#pgfId-1115107}A
[]{#10.htm#marker-1115101}[]{#10.htm#marker-1115103}[]{#10.htm#marker-1115104}[]{#10.htm#marker-1115105}*system
call*, often called a *syscall*[]{#10.htm#marker-1115106}, is how a
computer program requests a service from the kernel of the operating
system on which it is executed. Common syscalls are
`open`{.fm-code-in-text}[]{#10.htm#marker-1115108},
`read`{.fm-code-in-text}[]{#10.htm#marker-1115109},
`write`{.fm-code-in-text}[]{#10.htm#marker-1115110},
`fork`{.fm-code-in-text}[]{#10.htm#marker-1115111}, and
`exec`{.fm-code-in-text}[]{#10.htm#marker-1115112}. In Linux, there are
over 700 system calls.

[]{#10.htm#pgfId-1115113}Recall from the beginning of this chapter that
the Linux kernel is the single point of failure hostile containers can
attack to escape confinement. If a bug exists in the Linux kernel that
can be attacked via a system call, the container processes might escape.
The Linux kernel feature seccomp[]{#10.htm#marker-1115114} allows
processes to voluntarily limit the number of system calls they and their
children can make. Podman, by default, eliminates hundreds of the system
calls using this feature. Suppose the Linux kernel has a flaw in one of
its system calls, which a container process can use to escape, but
Podman eliminated it from the table of system calls available to the
container. In that case, the container is blocked from using it.

[]{#10.htm#pgfId-1115116}Podman's seccomp
filters[]{#10.htm#marker-1115115} are stored as a JSON file in the
/usr/share/containers/seccomp.json file. Podman also modifies the list
of seccomp filters[]{#10.htm#marker-1115117} based on the capabilities
you allow to the container. When you add a capability, Podman adds the
system calls required for that capability. Capabilities and seccomp are
both enforced separately; Podman just tries to make it easier for the
user. If the user provides their own seccomp JSON file, it needs to be
similar to the default one for the capability modifications to work.

[]{#10.htm#pgfId-1115118}You can modify the seccomp filter by editing
this file. In the following example, you remove the
`mkdir`{.fm-code-in-text} syscall[]{#10.htm#marker-1115119} from
seccomp.json, and then run a container in which you try to make a
directory. The seccomp filter blocks the syscall, and your container
fails.

[]{#10.htm#pgfId-1115121}Listing 10.12 How seccomp filters can block
syscalls within a Podman container

``` programlisting
$ sed '/mkdir/d' /usr/share/containers
➥ /seccomp.json > /tmp/seccomp.json                    ❶
$ diff /usr/share/containers/seccomp.json/ 
➥ tmp/seccomp.json                                     ❷
249,250d248
<        "mkdir",
<        "mkdirat",
$ podman run --rm --security-opt seccomp=/
➥ tmp/seccomp.json ubi8 mkdir /foo                     ❸
mkdir: cannot create directory '/foo': Function not implemented
$ podman run --rm ubi8 mkdir /foo                       ❹
```

[]{#10.htm#pgfId-1125832}[❶]{.fm-combinumeral} Use the sed command to
delete all entries that make mkdir and create /tmp/seccomp.json.

[]{#10.htm#pgfId-1125868}[❷]{.fm-combinumeral} Use the diff command to
show the removed mkdir entries.

[]{#10.htm#pgfId-1125885}[❸]{.fm-combinumeral} Use the \--security-opt
seccomp=/tmp/seccomp.json flag to use an alternative seccomp filter; the
mkdir command fails because the mkdir system call is not available.

[]{#10.htm#pgfId-1125833}[❹]{.fm-combinumeral} Run the same command
again with a default filter to show the mkdir succeeds.

[]{#10.htm#pgfId-1115137}[Note]{.fm-callout-head} Not many people modify
the seccomp filters because it is difficult to figure out the number of
system calls required by a container. There are tools to generate this
list of system calls using the Berkeley Packet Filter
(BPF[]{#10.htm#marker-1115138}). The package at the following webpage is
a hook that monitors a container and automatically generates a
seccomp.json file to use later to lock down the container:
[https://github.com/containers/oci-seccomp-bpf-hook/](https://github.com/containers/oci-seccomp-bpf-hook/){.url}.

[]{#10.htm#pgfId-1115139}Sometimes the default container seccomp.json
file is too tight. Your container might not work because it needs a
system call that is not available. In this case, you can disable seccomp
filtering by using the `--security-opt`{.fm-code-in-text}
`seccomp=unconfined`{.fm-code-in-text} flag[]{#10.htm#marker-1115140}.

[]{#10.htm#pgfId-1115141}As you see, system call filtering is powerful
and can really limit the container processes' access to the host kernel.
The next level is to use KVM
[]{#10.htm#marker-1115142}[]{#10.htm#marker-1115144}[]{#10.htm#marker-1115145}[]{#10.htm#marker-1115146}isolation.

## []{#10.htm#pgfId-1115148}10.10 Virtual machine isolation {#10.htm#heading_id_26 .fm-head}

[]{#10.htm#pgfId-1115151}At
[]{#10.htm#marker-1115149}[]{#10.htm#marker-1115150}the beginning of
this chapter, I compared process isolation based on where the three pigs
chose to live. They could live in separate houses, a duplex, or a
condominium. Each one was getting slightly less secure. Container
security, by default, is living in a condo. But you can use VM
isolation, which basically puts your container into a VM, to get better
isolation.

[]{#10.htm#pgfId-1115152}In appendix B, I cover how different OCI
runtimes, Kata and libkrun, take advantage of Kernel-based Virtual
Machine (KVM[]{#10.htm#marker-1115153}) to run their containers within a
lightweight virtual machine. These virtual machines run their own kernel
and initialization tools to launch the container. By doing this, almost
all of the host kernels' system calls are eliminated, making it much
more difficult to escape confinement.

[]{#10.htm#pgfId-1115154}The problem with this isolation is that it
comes at a cost. As with a duplex, you end up sharing fewer services
between your containers. Memory management, CPU, and other resources are
harder to share. Sharing volumes into a container is also going to
perform worse.

[]{#10.htm#pgfId-1115155}Now you've finished examining Podman security
features used for container isolation. Next let's look at other security
[]{#10.htm#marker-1115156}[]{#10.htm#marker-1115157}features.

## []{#10.htm#pgfId-1115159}Summary {#10.htm#heading_id_27 .fm-head}

-   []{#10.htm#pgfId-1115160 .calibre17}Container security is all about
    protecting the Linux kernel and host filesystem from hostile
    container processes.

-   []{#10.htm#pgfId-1115161 .calibre17}Defense in depth means your
    container tooling takes advantage of as many security mechanisms as
    possible. If one security mechanism fails, others might still
    protect your []{#10.htm#marker-1115162 .calibre17}system.

[]{#11.htm}
