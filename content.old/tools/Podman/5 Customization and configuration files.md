# 5 Customization and configuration files 

[]{#05.htm#pgfId-1105487}This chapter []{#05.htm#marker-1111462}covers

-   []{#05.htm#pgfId-1105488 .calibre17}Using Podman configuration files
    based on libraries used
-   []{#05.htm#pgfId-1105490 .calibre17}Configuring the storage.conf
    file
-   []{#05.htm#pgfId-1105491 .calibre17}Using the registries.conf and
    policy.json files for configuration
-   []{#05.htm#pgfId-1105492 .calibre17}Using the containers.conf file
    to configure other defaults
-   []{#05.htm#pgfId-1105493 .calibre17}Using system configuration files
    to allow non-root users namespace access

[]{#05.htm#pgfId-1105494}Container engines like Podman have dozens of
hardcoded defaults built into them. These defaults determine many
aspects of the functional and nonfunctional behaviors of Podman, such as
network and security settings. Podman developers try to pick the maximum
amount of security but still allow most containers to run successfully.
Similarly, I want as much isolation from the host as possible.

[]{#05.htm#pgfId-1105496}The security defaults include which Linux
capabilities to use, which SELinux labels to set, and the set of
syscalls available to the containers. There are defaults for resource
constraints, like memory usage and maximum processes allowed within a
container. Other defaults include the local path for storing images, the
list of container registries, and even system configuration to allow
rootless mode to work. The Podman developers wanted to allow users to
have ultimate control over these defaults, so the container engine
configuration files provide a mechanism for customizing the way Podman
and other container engines run.

[]{#05.htm#pgfId-1105499}The problem with defaults is that they are
best-guess estimates from developers. While most users run Podman in
default configuration, sometimes there is a need to change the
configuration. Not every environment has the same configuration, and you
might want to default certain machines to different levels of security
and different registry configurations. Even rootless users might need
different configurations than rootful users. In this chapter, I will
show you how to customize different parts of Podman and explain where
you can find more information about all of the different knobs available
to you.[]{#05.htm#id_aocnkyr9vpgv}[]{#05.htm#id_qrx0m8lodld}

[]{#05.htm#pgfId-1105502}As you have learned in previous chapters,
Podman uses multiple libraries to perform different tasks when working
with containers. Table 5.1 describes the different libraries Podman
uses.[]{#05.htm#id_md98emseev64}

[]{#05.htm#pgfId-1108137}Table 5.1 Container libraries used by Podman

+-----------------+-----------------------------------------------------+
| []              | []{#05.htm#pgfId-1108143}Description                |
| {#05.htm#pgfId- |                                                     |
| 1108141}Library |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.ht       | []{#05.htm#pgfId-1108147}Defines the storage of     |
| m#pgfId-1108145 | container images and other basic storage used by    |
| }containers/sto | container engines                                   |
| rage[]{#05.htm# |                                                     |
| marker-1108160} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.         | []{#05.htm#pgfId-1108151}Defines the mechanisms     |
| htm#pgfId-11081 | used to move container images from different types  |
| 49}containers/i | of storage; usually used between container          |
| mage[]{#05.htm# | registries and local container storage              |
| marker-1108161} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.h        | []{#05.htm#pgfId-1108155}Defines all of the default |
| tm#pgfId-110815 | configuration options for container engines not     |
| 3}containers/co | defined in containers/storage or containers/image   |
| mmon[]{#05.htm# |                                                     |
| marker-1108162} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.ht       | []{#05.htm#pgfId-1108159}As explained in chapter 2, |
| m#pgfId-1108157 | it is used for building container images into local |
| }containers/bui | storage using rules defined in a Containerfile or   |
| ldah[]{#05.htm# | Dockerfile; for more information on Buildah, see    |
| marker-1108163} | appendix A.                                         |
+-----------------+-----------------------------------------------------+

[]{#05.htm#pgfId-1105533}Each of these libraries has separate
configuration files used to set the default settings for the particular
library, with the exception of Buildah. The container engines, Podman,
and Buildah share the containers/common configuration file
containers.conf, described in section 5.3.

[]{#05.htm#pgfId-1105534}[Note]{.fm-callout-head} All of the nonsystem
configuration files used by Podman use the TOML format. TOML's syntax
consists of name = "value" pairs, \[section names\], and \# comments.
The format of TOML can be simplified to the following:

-   []{#05.htm#pgfId-1105535 .calibre17}`[table]`{.fm-code-in-text1}

-   []{#05.htm#pgfId-1105536 .calibre17}`option`{.fm-code-in-text1}
    `=`{.fm-code-in-text1} `value`{.fm-code-in-text1}

-   []{#05.htm#pgfId-1105537
    .calibre17}`[table.subtable1]`{.fm-code-in-text1}

-   []{#05.htm#pgfId-1105538 .calibre17}`option`{.fm-code-in-text1}
    `=`{.fm-code-in-text1} `value`{.fm-code-in-text1}

-   []{#05.htm#pgfId-1105539
    .calibre17}`[table.subtable2]`{.fm-code-in-text1}

-   []{#05.htm#pgfId-1105540 .calibre17}`option`{.fm-code-in-text1}
    `=`{.fm-code-in-text1} `value`{.fm-code-in-text1}

[]{#05.htm#pgfId-1105542}See [https://toml.io](https://toml.io){.url}
for a more complete explanation of the TOML language. When configuring
Podman, usually one of the first concerns is where you are going to
store your containers and images.

## []{#05.htm#pgfId-1105544}5.1 Configuration files for storage {#05.htm#heading_id_3 .fm-head}

[]{#05.htm#pgfId-1105548}Podman
[]{#05.htm#marker-1105545}[]{#05.htm#marker-1105546}uses the
[github.com/containers/storage](http://github.com/containers/storage){.url}
library[]{#05.htm#marker-1105547}, which provides methods for storing
filesystem layers, container images, and containers. Configuration of
this library is done using the storage.conf configuration file, which
can be stored in multiple different directories.

[]{#05.htm#pgfId-1105549}Linux distributions often provide a
/usr/share/containers/storage.conf file, which can be overridden by
creating a /etc/containers/storage.conf file. Rootless users can store
their configuration in the \$XDG_CONFIG_HOME/containers/storage.conf
file; if the \$XDG_CONFIG_HOME environment variable is not set, the
\$HOME/.config/ containers/storage.conf file is used. Most users will
never change the storage.conf file, but in a few situations, advanced
users need to do some customizations. The most common reason for changes
is relocating the container's storage.

[]{#05.htm#pgfId-1105550}[Note]{.fm-callout-head} When using Podman in
remote mode, for example on a Mac or Windows box, the Podman service
uses the storage.conf files located in the Linux box. To modify them,
you need to enter the VM. When using the Podman machine, execute the
`podman`{.fm-code-in-text1} `machine`{.fm-code-in-text1}
`ssh`{.fm-code-in-text1} command[]{#05.htm#marker-1105551} to enter the
VM. See appendixes E and F for more information.

[]{#05.htm#pgfId-1105553}Podman reads only one storage.conf and ignores
all subsequent ones. Podman first attempts to use the storage.conf from
your home directory; next goes the /etc/storage/ storage.conf; and
finally, if both files do not exist, Podman reads the /usr/share/
containers/storage.conf file. You can see the storage.conf file your
Podman command is using via the `podman`{.fm-code-in-text}
`info`{.fm-code-in-text} command[]{#05.htm#marker-1105554}:

``` programlisting
$ podman info --format '{{ .Store.ConfigFile }}'
/home/dwalsh/.config/containers/storage.conf
```

### []{#05.htm#pgfId-1105558}5.1.1 Storage location {#05.htm#heading_id_4 .fm-head1}

[]{#05.htm#pgfId-1105562}By
[]{#05.htm#marker-1109582}[]{#05.htm#marker-1109583}[]{#05.htm#marker-1109584}default
rootless Podman is configured to store your images in the \$HOME/.local/
share/containers/storage directory. The default rootful storage location
is /var/lib/ containers/storage.

[]{#05.htm#pgfId-1105563}Sometimes you need to change this default
location. Perhaps you don't have enough disk space in /var or in the
user's home directory, so you want to store your images on a different
disk. The storage.conf file calls the storage location the
`graphRoot`{.fm-code-in-text}[]{#05.htm#marker-1105564}, and it can be
overridden in /etc/containers/storage.conf for rootful containers.

[]{#05.htm#pgfId-1105565}In this section, you will modify the location
of the graph driver to /var/mystorage. First, become root and make sure
the /etc/containers/storage.conf file exists. If it does not exist, just
copy the /usr/share/containers/storage.conf file into it:

``` programlisting
$ sudo cp /usr/share/containers/storage.conf /etc/containers/storage.conf
```

[]{#05.htm#pgfId-1105567}[Note]{.fm-callout-head} Some distributions
just ship the /etc/containers/storage.conf.

[]{#05.htm#pgfId-1105568}Now, make a backup, and open
/etc/containers/storage.conf file for editing:

``` programlisting
$ sudo cp /etc/containers/storage.conf /etc/containers/storage.conf.orig
$ sudo vi /etc/containers/storage.conf 
```

[]{#05.htm#pgfId-1105571}Set the `graphdriver`{.fm-code-in-text}
variable `graphroot`{.fm-code-in-text} `=`{.fm-code-in-text}
`"/var/lib/containers/storage"`{.fm-code-in-text} to
`graphroot`{.fm-code-in-text} `=`{.fm-code-in-text}
`"/var/mystorage"`{.fm-code-in-text}, and save the file.

[]{#05.htm#pgfId-1105572}Your storage.conf file should include the
following:

``` programlisting
$ grep -B 1 graph /etc/containers/storage.conf 
# Primary Read/Write location of container storage
graphroot = "/var/mystorage"
```

[]{#05.htm#pgfId-1105576}Execute `podman`{.fm-code-in-text}
`info`{.fm-code-in-text} to see if the change took place:

``` programlisting
$ sudo podman info
...
Store:
 configFile: /etc/containers/storage.conf
...
 graphDriverName: overlay
 graphOptions:
  overlay.mountopt: nodev,metacopy=on
 graphRoot: /var/mystorage
...
 volumePath: /var/mystorage/volumes
```

[]{#05.htm#pgfId-1105588}Notice in the storage section that the
`graphRoot`{.fm-code-in-text} is now /var/mystorage. All images and
containers will be stored in this directory.

[]{#05.htm#pgfId-1105590}Now run the `podman`{.fm-code-in-text}
`info`{.fm-code-in-text} command[]{#05.htm#marker-1105589} in rootless
mode. The storage location will not change; it is still
/home/dwalsh/.local/share/containers/storage:

``` programlisting
$ podman info
store:
 configFile: /home/dwalsh/.config/containers/storage.conf
 containerStore:
  number: 27
  paused: 0
  running: 0
  stopped: 27
 graphDriverName: overlay
 graphOptions: {}
 graphRoot: /home/dwalsh/.local/share/containers/storage
```

[]{#05.htm#pgfId-1105602}You can create a
\$HOME/.config/containers/storage.conf and change it there, but this
does not scale well for systems with multiple users. The key
`rootless_storage_ path`{.fm-code-in-text}[]{#05.htm#marker-1105603}
allows you to change the location for all users on your system.

[]{#05.htm#pgfId-1105604}This time, uncomment and modify the
`rootless_storage_path`{.fm-code-in-text} line:

``` programlisting
$ sudo vi /etc/containers/storage.conf 
```

[]{#05.htm#pgfId-1105606}Modify the
`rootless_storage_path`{.fm-code-in-text} line in storage.conf from

``` programlisting
# rootless_storage_path = "$HOME/.local/share/containers/storage"
```

[]{#05.htm#pgfId-1105608}Change it to

``` programlisting
rootless_storage_path = "/var/tmp/$UID/var/mystorage"
```

[]{#05.htm#pgfId-1105610}Save the storage.conf file. When you are done,
it should look like this:

``` programlisting
$ grep -B 3 rootless_storage_path /etc/containers/storage.conf
# Storage path for rootless users
#
rootless_storage_path = "/var/tmp/$UID/var/mystorage"
```

[]{#05.htm#pgfId-1105615}Now run `podman`{.fm-code-in-text}
`info`{.fm-code-in-text} to see the changes. Notice that the
`graphRoot`{.fm-code-in-text} now points at the
/var/tmp/3267/var/mystorage directory:

``` programlisting
$ podman info
...
store:
 configFile: /home/dwalsh/.config/containers/storage.conf
...
 graphOptions: {}
 graphRoot: /var/tmp/3267/var/mystorage
```

[]{#05.htm#pgfId-1105625}Container/storage supports expanding the
`$HOME`{.fm-code-in-text}[]{#05.htm#marker-1105623} and
`$UID`{.fm-code-in-text} environment variables[]{#05.htm#marker-1105624}
for this path. To revert changes, copy and restore the original
storage.conf file:

``` programlisting
$ sudo cp /etc/containers/storage.conf.orig /etc/containers/storage.conf
```

[]{#05.htm#pgfId-1105627}[Note]{.fm-callout-head} If you are running on
an SELinux system and change the default location of storage, you need
to inform SELinux about it, using the following
`semanage`{.fm-code-in-text1} command[]{#05.htm#marker-1105628}. This
will tell SELinux to label the new location as if it was in the old
location. Next, you will need to change the labeling on disk using the
`restorecon`{.fm-code-in-text1} command[]{#05.htm#marker-1105629}. You
can do this with the following commands:

``` programlisting
sudo semanage fcontext -a -e /var/lib/containers/storage /var/mystorage
  sudo restorecon -R -v /var/mystorage
```

[]{#05.htm#pgfId-1105632}In rootless mode you need to do the following:

``` programlisting
sudo semanage fcontext -a -e $HOME/.local/share/containers/storage/
➥ var/tmp/3267/var/mystorage
sudo restorecon -R -v /var/tmp/3267/var/mystorage
```

[]{#05.htm#pgfId-1105636}Sometimes you might want to change the storage
driver or, more likely, the configuration of the storage
[]{#05.htm#marker-1105637}[]{#05.htm#marker-1105638}[]{#05.htm#marker-1105639}driver.

### []{#05.htm#pgfId-1105641}5.1.2 Storage drivers {#05.htm#heading_id_5 .fm-head1}

[]{#05.htm#pgfId-1105645}Recall
[]{#05.htm#marker-1105642}[]{#05.htm#marker-1105643}[]{#05.htm#marker-1105644}the
wedding cake illustration from chapter 2. This illustration shows that
images are often made of multiple layers. These layers are stored on
disk by the container/ storage library, but when you are running a
container on them, each layer needs to be mounted on the previous layer
(figure 5.1).

::: figure
![](images/OEBPS/Images/05-01.png){.calibre18}

[]{#05.htm#pgfId-1111790}[]{#05.htm#id_mnifp03xfnzo}Figure 5.1 Layered
images stacked on one another are reassembled and mounted using
container/storage.
:::

[]{#05.htm#pgfId-1105653}Container/storage uses a Linux kernel
filesystem concept called a *layered
filesystem*[]{#05.htm#marker-1105652} to do this. Podman, using
container/storage, supports multiple different types of layered
filesystems. In Linux, these filesystems are called *copy-on-write
(CoW*[]{#05.htm#marker-1105654}*)* filesystems. In containers/storage,
these different filesystem types are called *drivers*. By default Podman
uses the `overlay`{.fm-code-in-text} storage
driver[]{#05.htm#marker-1105655}.

[]{#05.htm#pgfId-1105656}[Note]{.fm-callout-head} Docker supports two
types of overlay drivers: `overlay`{.fm-code-in-text1} and
`overlay2`{.fm-code-in-text1}. `overlay2`{.fm-code-in-text1} was an
improvement over `overlay`{.fm-code-in-text1}, and the original
`overlay`{.fm-code-in-text1} driver is rarely used any more. In
contrast, Podman uses the newer `overlay2`{.fm-code-in-text1} driver and
just calls it `overlay`{.fm-code-in-text1}. You can select the
`overlay`{.fm-code-in-text1} driver in Podman, but this is just an alias
for `overlay2`{.fm-code-in-text1}.

[]{#05.htm#pgfId-1105657}Table 5.2 lists all of the storage drivers
Podman and containers/storage support. I recommend you just stick to the
`overlay`{.fm-code-in-text} driver, since this is the driver the vast
majority of the world uses.[]{#05.htm#id_4qwxaoy9s9gr}

[]{#05.htm#pgfId-1108284}Table 5.2 Container storage drivers

+-----------------+-----------------------------------------------------+
| []              | []{#05.htm#pgfId-1108290}Description                |
| {#05.htm#pgfId- |                                                     |
| 1108288}Storage |                                                     |
| drivers         |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05          | []{#05.htm#pgfId-1108294}This is the default        |
| .htm#pgfId-1108 | driver, and I strongly recommend its use. It is     |
| 292}`overlay (o | based on the Linux kernel overlay filesystem.       |
| verlay2`{.fm-co | `overlay`{.fm-code-in-text1} and                    |
| de-in-text1}[]{ | `overlay2`{.fm-code-in-text1} are exactly the same  |
| #05.htm#marker- | in Podman. It is the most tested driver, which the  |
| 1108315}`)`{.fm | overwhelming majority of users use.                 |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.htm#pgfI | []{#05.htm#pgfId-1108298}This is the simplest       |
| d-1108296}`vfs` | driver; it creates full copies of each lower layer  |
| {.fm-code-in-te | up onto the next layer. It works everywhere but is  |
| xt1}[]{#05.htm# | slow and very disk intensive.                       |
| marker-1108316} |                                                     |
+-----------------+-----------------------------------------------------+
| []{             | []{#05.htm#pgfId-1108302}This driver was heavily    |
| #05.htm#pgfId-1 | used when Docker first became popular---before the  |
| 108300}`devmapp | `overlay`{.fm-code-in-text1} driver was available.  |
| er`{.fm-code-in | It reallocates the size of each layer at a maximum  |
| -text1}[]{#05.h | size. It is not recommended any longer.             |
| tm#id_mnp47wrp8 |                                                     |
| 7ki}[]{#05.htm# |                                                     |
| marker-1109998} |                                                     |
+-----------------+-----------------------------------------------------+
| [               | []{#05.htm#pgfId-1108306}This driver was never      |
| ]{#05.htm#pgfId | merged into the upstream kernel, so it is only      |
| -1108304}`aufs` | available on a few Linux distributions.             |
| {.fm-code-in-te |                                                     |
| xt1}[]{#05.htm# |                                                     |
| marker-1108319} |                                                     |
+-----------------+-----------------------------------------------------+
| []              | []{#05.htm#pgfId-1108310}This driver allows storage |
| {#05.htm#pgfId- | on btrfs snapshots based on the Btrfs filesystem.   |
| 1108308}`btrfs` | Some users have had success using this filesystem.  |
| {.fm-code-in-te |                                                     |
| xt1}[]{#05.htm# |                                                     |
| marker-1108320} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.htm#pgfI | []{#05.htm#pgfId-1108314}This driver uses the ZFS   |
| d-1108312}`zfs` | filesystem, which is a proprietary filesystem and   |
| {.fm-code-in-te | not available on most distributions.                |
| xt1}[]{#05.htm# |                                                     |
| marker-1108321} |                                                     |
+-----------------+-----------------------------------------------------+

[]{#05.htm#pgfId-1105699}overlay storage options

[]{#05.htm#pgfId-1105701}The
`overlay`{.fm-code-in-text}[]{#05.htm#marker-1105700} driver has some
interesting customization options. These options are located in the
storage.conf \[storage.options.overlay\] table.

[]{#05.htm#pgfId-1105702}There are several advanced options available
for configuring the overlay driver. I'll quickly mention a few to
describe use cases.

[]{#05.htm#pgfId-1105704}The `mount_program`{.fm-code-in-text}
option[]{#05.htm#marker-1105703} allows you to specify an executable to
use instead of the kernel overlay driver. Podman usually ships with the
`fuse-overlayfs`{.fm-code-in-text} executable[]{#05.htm#marker-1105705},
which provides a `FUSE`{.fm-code-in-text} (userspace) overlay driver.
Podman automatically fails over to the
`fuse-overlayfs`{.fm-code-in-text} `mount_program`{.fm-code-in-text} if
it is installed on systems where rootless native overlay is not
supported. Most kernels support native overlay; however, there are use
cases when you might want to configure the
`mount_program`{.fm-code-in-text}. The
`fuse-overlayfs`{.fm-code-in-text} has advanced features not currently
supported in the native overlay.

[]{#05.htm#pgfId-1105707}Podman is quickly being adopted by the
high-performance computing (HPC[]{#05.htm#marker-1105706}) community.
The HPC community does not allow rootful containers, and in many cases
it allows workloads to run only with a single UID. This means some HPC
systems do not allow user namespaces with multiple UIDs. Since many
images come with multiple UIDs, Podman added an
`ignore_chown_errors`{.fm-code-in-text} option[]{#05.htm#marker-1105708}
to containers/storage to allow images with files with different UIDs to
be flattened into a single UID. Table 5.3 lists all the current storage
options supported by container storage.[]{#05.htm#id_41jizfpj6s0d}

[]{#05.htm#pgfId-1105757}[Note]{.fm-callout-head} You have examined a
few of the storage.conf fields, but there are many more. Use the
containers-storage.conf man page to explore all of them:

``` programlisting
https:/ /github.com/containers/storage/blob/main/docs/containers-storage.conf.5.md 
$ man containers-storage.conf
```

[]{#05.htm#pgfId-1108407}Table 5.3 Container storage drivers

+-----------------+-----------------------------------------------------+
| []              | []{#05.htm#pgfId-1108413}Description                |
| {#05.htm#pgfId- |                                                     |
| 1108411}Storage |                                                     |
| drivers         |                                                     |
+-----------------+-----------------------------------------------------+
| [               | []{#05.htm#pgfId-1108417}Ignore                     |
| ]{#05.htm#pgfId | `chown`{.fm-code-in-text1}ing file UIDs for         |
| -1108415}`ignor | rootless containers with a single UID. There is no  |
| e_chown_errors` | entry in /etc/subuid.                               |
| {.fm-code-in-te |                                                     |
| xt1}[]{#05.htm# |                                                     |
| marker-1110057} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.htm      | []{#05.htm#pgfId-1108421}Path to a helper program   |
| #pgfId-1108419} | to use for mounting the filesystem instead of using |
| `mount_program` | a kernel overlay to mount it. Older kernels did not |
| {.fm-code-in-te | support rootless overlay.                           |
| xt1}[]{#05.htm# |                                                     |
| marker-1110062} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.ht       | []{#05.htm#pgfId-1108425}Comma-separated list of    |
| m#pgfId-1108423 | mount options to be passed to the kernel. It        |
| }`mountopt`{.fm | defaults to                                         |
| -code-in-text1} | `"nodev,metacopy=on"`{.fm-code-in-text1}.           |
+-----------------+-----------------------------------------------------+
| []{#05.htm#p    | []{#05.htm#pgfId-1108429}Do not create              |
| gfId-1108427}`s | `PRIVATE`{.fm-code-in-text1} bind mounts on the     |
| kip_mount_home` | storage home directory.                             |
| {.fm-code-in-te |                                                     |
| xt1}[]{#05.htm# |                                                     |
| marker-1110072} |                                                     |
+-----------------+-----------------------------------------------------+
| []              | []{#05.htm#pgfId-1108433}Maximum number of inodes   |
| {#05.htm#pgfId- | in a container image                                |
| 1108431}`inode` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#05.htm# |                                                     |
| marker-1110077} |                                                     |
+-----------------+-----------------------------------------------------+
| [               | []{#05.htm#pgfId-1108437}Maximum size of a          |
| ]{#05.htm#pgfId | container image                                     |
| -1108435}`size` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#05.htm# |                                                     |
| marker-1110082} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.         | []{#05.htm#pgfId-1108441}Permissions mask for new   |
| htm#pgfId-11084 | files and directories in an image. The values are   |
| 39}`force_mask` | the following:                                      |
| {.fm-code-in-te |                                                     |
| xt1}[]{#05.htm# | -   []{#05.ht                                       |
| marker-1110087} | m#pgfId-1108480}`private`{.fm-code-in-text1}---This |
|                 |     sets all filesystem objects to                  |
|                 |     `0700`{.fm-code-in-text1}. No other users on    |
|                 |     the system can access the files.                |
|                 |                                                     |
|                 | -   []{#05.h                                        |
|                 | tm#pgfId-1108487}`shared`{.fm-code-in-text1}---This |
|                 |     is equivalent to `0755`{.fm-code-in-text1}.     |
|                 |     Everyone on the system can read, access, and    |
|                 |     execute files in the image. This is useful for  |
|                 |     sharing container storage with other users.     |
|                 |                                                     |
|                 | []{#05.htm#pgfId-1108490}All files within the image |
|                 | are made readable and executable by any user on the |
|                 | system. Even /etc/shadow within your image is now   |
|                 | readable by any user.                               |
|                 |                                                     |
|                 | []{#05.htm#pgfId-1108493}When                       |
|                 | `force_mask`{.fm-code-in-text1} is set, the         |
|                 | original permission mask is stored in               |
|                 | `xattr`{.fm-code-in-text1}s, and the                |
|                 | `mount_program`{.fm-code-in-text1}, like            |
|                 | /usr/bin/fuse-overlayfs, presents the               |
|                 | `xattr`{.fm-code-in-text1} permissions to processes |
|                 | within containers.                                  |
+-----------------+-----------------------------------------------------+

[]{#05.htm#pgfId-1105760}Now you know about configuring the container
storage! The next configuration you will look at is
[]{#05.htm#marker-1105761}container
[]{#05.htm#marker-1105762}[]{#05.htm#marker-1105763}[]{#05.htm#marker-1105764}registry
[]{#05.htm#marker-1105765}[]{#05.htm#marker-1105766}access.

## 5.2 Configuration files for registries

[]{#05.htm#pgfId-1105771}Podman
[]{#05.htm#marker-1105769}[]{#05.htm#marker-1105770}uses the
github.com/containers/image library for pulling and pushing container
images, usually from container registries. Podman uses the
registries.conf configuration file to specify registries and the
policy.json file for signature verification of images. As with the
container storage storage.conf, most users never modify these files and
just use the distribution defaults.

### []{#05.htm#pgfId-1105773}5.2.1 registries.conf {#05.htm#heading_id_7 .fm-head1}

[]{#05.htm#pgfId-1105775}The []{#05.htm#marker-1105774}registries.conf
configuration file is a system-wide configuration file for container
image registries. Podman uses \$HOME/.config/containers/registries.conf
if it exists; otherwise, it uses /etc/containers/registries.conf.

[]{#05.htm#pgfId-1105776}[Note]{.fm-callout-head} When using Podman in
remote mode, for example on a Mac or Windows box, registries.conf files
are stored in the Linux box on the server side. You need to
`ssh`{.fm-code-in-text1} into the Linux box to make the changes. With a
Podman machine, you can execute `podman`{.fm-code-in-text1}
`machine`{.fm-code-in-text1} `ssh`{.fm-code-in-text1}. See appendixes E
and F for more information.

[]{#05.htm#pgfId-1105777}The main key value to use with the
registries.conf file is
`unqualified-search-registries`{.fm-code-in-text}. This field specifies
an array of `host[:port]`{.fm-code-in-text} registries to try when
pulling via short names, in order. If you specify only one registry in
the `unqualified-search-registries`{.fm-code-in-text}
option[]{#05.htm#marker-1105778}, Podman will work similarly to Docker
and force a single registry on the user.

[]{#05.htm#pgfId-1105779}In this exercise, you will modify the default
search registries to be used by Podman. First, you need to make a backup
of the /etc/containers/registries.conf file, and then remove docker.io
and add example.com:

``` programlisting
$ sudo cp /etc/containers/registries.conf /etc/containers/registries.conf.orig
$ sudo vi /etc/containers/registries.conf 
```

[]{#05.htm#pgfId-1105782}Modify the following line:

``` programlisting
unqualified-search-registries = ["registry.fedoraproject.org", "registry.access.redhat.com", "docker.io", "quay.io"]
```

[]{#05.htm#pgfId-1105784}Change the line to

``` programlisting
unqualified-search-registries = ["registry.fedoraproject.org", "registry.access.redhat.com", "example.com", "quay.io"]
```

[]{#05.htm#pgfId-1105786}Save the file, then execute
`podman`{.fm-code-in-text} `info`{.fm-code-in-text} to verify the
changes:

``` programlisting
$ podman info
registries:
  search:
  - registry.fedoraproject.org
  - registry.access.redhat.com
  - example.com
  - quay.io
```

[]{#05.htm#pgfId-1105794}Now, if you attempt to pull via an unknown
short name, you should see the following prompt:

``` programlisting
$ podman pull foobar
? Please select an image: 
  ▸ registry.fedoraproject.org/foobar:latest
    registry.access.redhat.com/foobar:latest
    example.com/foobar:latest
    quay.io/foobar:latest
```

[]{#05.htm#pgfId-1105801}Copy the original to the registries.conf file:

``` programlisting
$ sudo cp /etc/containers/registries.conf.orig /etc/containers/registries.conf
```

[]{#05.htm#pgfId-1105803}Table 5.4 describes all of the options
available in registries.conf files.[]{#05.htm#id_ampolw88x8jd}

[]{#05.htm#pgfId-1108571}Table 5.4 Container registries.conf global
fields

+-----------------+-----------------------------------------------------+
| [               | []{#05.htm#pgfId-1108577}Description                |
| ]{#05.htm#pgfId |                                                     |
| -1108575}Fields |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.htm#pgfI | []{#05.htm#pgfId-1108581}An array of                |
| d-1108579}`unqu | `host[:port]`{.fm-code-in-text1} registries to try  |
| alified-search- | when pulling an unqualified image, in order.        |
| registries`{.fm |                                                     |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+
| [               | []{#05.htm#pgfId-1108585}Determines how Podman      |
| ]{#05.htm#pgfId | should handle short names. The values include the   |
| -1108583}`short | following:                                          |
| -name-mode`{.fm |                                                     |
| -code-in-text1} | -   []{#05.ht                                       |
|                 | m#pgfId-1108612}`enforcing`{.fm-code-in-text1}---If |
|                 |     there is one unqualified search registry, use   |
|                 |     it. If there are two or more registries defined |
|                 |     and you are running Podman in a terminal,       |
|                 |     prompt the user to select one of the search     |
|                 |     registries; otherwise, there will be an error.  |
|                 |                                                     |
|                 | -   []{#05.htm#pgfI                                 |
|                 | d-1108615}`permissive`{.fm-code-in-text1}---Behaves |
|                 |     as `enforcing`{.fm-code-in-text1} but does not  |
|                 |     lead to an error if no terminal: just uses each |
|                 |     entry in unqualified search registries until    |
|                 |     success.                                        |
|                 |                                                     |
|                 | -   []{#05.ht                                       |
|                 | m#pgfId-1108618}`disabled`{.fm-code-in-text1}---Use |
|                 |     all unqualified search registries without       |
|                 |     prompting.                                      |
+-----------------+-----------------------------------------------------+
| []{#            | []{#05.htm#pgfId-1108589}An array of default        |
| 05.htm#pgfId-11 | credential helpers is used as external credential   |
| 08587}`credenti | stores. Note that containers-auth.json is a         |
| al-helpers`{.fm | reserved value to use auth files as specified in    |
| -code-in-text1} | `containers-auth.json(5)`{.fm-code-in-text1}. The   |
|                 | credential helpers are set to                       |
|                 | `["containers-auth.json"]`{.fm-code-in-text1} if    |
|                 | none are specified.                                 |
+-----------------+-----------------------------------------------------+

[]{#05.htm#pgfId-1105829}Blocking pulling from container registries

[]{#05.htm#pgfId-1105832}Another
[]{#05.htm#marker-1105830}[]{#05.htm#marker-1105831}interesting thing
you can configure in registries.conf is the ability to block users from
pulling from a container registry. In the following example, you will
configure registries.conf to block pulls from docker.io. The
registries.conf file has a specific `[[registry]]`{.fm-code-in-text}
table entry that can specify how to handle individual container
registries. You can add this table multiple times---once per registry:

``` programlisting
$ sudo vi /etc/containers/registries.conf
```

[]{#05.htm#pgfId-1105834}Add the following:

``` programlisting
[[registry]]
Location = "docker.io"
blocked=true
```

[]{#05.htm#pgfId-1105838}Save the file. Examine the settings using
`podman`{.fm-code-in-text} `info`{.fm-code-in-text}:

``` programlisting
$ podman info
...
registries:
 Docker.io:
  Blocked: true
  Insecure: false
  Location: docker.io
  MirrorByDigestOnly: false
  Mirrors: null
  Prefix: docker.io
  search:
  - registry.fedoraproject.org
  - registry.access.redhat.com
  - docker.io
  - quay.io
```

[]{#05.htm#pgfId-1105854}Now, attempt to pull an image from docker.io:

``` programlisting
$ podman pull docker.io/ubuntu
Trying to pull docker.io/library/ubuntu:latest...
Error: initializing source docker:/ /ubuntu:latest: registry docker.io is blocked in /etc/containers/registries.conf or /home/dwalsh/.config/containers/registries.conf.d
```

[]{#05.htm#pgfId-1105858}This demonstrates that administrators have the
ability to block content from specific registries. Table 5.5 describes
the suboptions available for the `[[registry]]`{.fm-code-in-text} table
in the registries.conf file.

[]{#05.htm#pgfId-1105859}[Note]{.fm-callout-head} Copy the original
registries.conf to pull from docker.io for the rest of this book:

``` programlisting
$ sudo cp /etc/containers/registries.conf.orig/
➥ etc/containers/registries.conf
```

[]{#05.htm#pgfId-1108659}Table 5.5 `[[registry]]`{.fm-code-in-text}
table fields

+-----------------+-----------------------------------------------------+
| [               | []{#05.htm#pgfId-1108665}Description                |
| ]{#05.htm#pgfId |                                                     |
| -1108663}Fields |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.ht       | []{#05.htm#pgfId-1108669}Name of the                |
| m#pgfId-1108667 | registry/repository to apply the filters on         |
| }`location`{.fm |                                                     |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.         | []{#05.htm#pgfId-1108673}Select the specified       |
| htm#pgfId-11086 | configuration when attempting to pull an image      |
| 71}`prefix`{.fm | matched by the specific prefix.                     |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.ht       | []{#05.htm#pgfId-1108677}If true, unencrypted HTTP  |
| m#pgfId-1108675 | as well as TLS connections with untrusted           |
| }`insecure`{.fm | certificates are allowed.                           |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.h        | []{#05.htm#pgfId-1108681}If true, pulling images    |
| tm#pgfId-110867 | with matching names is forbidden.                   |
| 9}`blocked`{.fm |                                                     |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+

[]{#05.htm#pgfId-1105888}Some users work on systems that are fully
isolated from the internet but still need to use applications that rely
on images from the internet. An example of this situation is if you have
an application that expects to use registry.access.redhat.com/ubi8/
httpd-24:latest but has no access to registry.access.redhat.com on the
internet. You can download the image and put it onto an internal
registry and then configure registries.conf with a mirror registry. If
you configure an entry in registries.conf, it will look like this:

``` programlisting
[[registry]]
location="registry.access.redhat.com"
[[registry.mirror]]
location="mirror-1.com"
```

[]{#05.htm#pgfId-1105893}Then your users can use the
`podman`{.fm-code-in-text} `pull`{.fm-code-in-text} command:

``` programlisting
$ podman pull registry.access.redhat.com/ubi8/httpd-24:latest
```

[]{#05.htm#pgfId-1105895}Podman actually pulls
mirror-1.com/ubi8/httpd-24:latest, but users will not notice the
difference.

[]{#05.htm#pgfId-1105896}[Note]{.fm-callout-head} You have examined a
few of the registries.conf fields, but there are many more. Use the
`containers-registries.conf(5)`{.fm-code-in-text1} man
page[]{#05.htm#marker-1105897} to explore all of them:

``` programlisting
$ man containers-registries.conf
https:/ /github.com/containers/image/blob/main/docs/containers-registries.conf.5.md
```

[]{#05.htm#pgfId-1105901}Now that you know how to configure storage and
registries, it is time to look at how to configure all of the options
[]{#05.htm#marker-1105902}[]{#05.htm#marker-1105903}central
[]{#05.htm#marker-1105904}to
[]{#05.htm#marker-1105905}[]{#05.htm#marker-1105906}Podman.

## 5.3 Configuration files for engines 

[]{#05.htm#pgfId-1105911}Podman
[]{#05.htm#marker-1108748}[]{#05.htm#marker-1108749}and other container
engines use the github.com/containers/common library for handling the
default settings not related to container storage or container
registries. These configuration settings come from the containers.conf
file. Podman reads the files in table 5.6 if they
exist.[]{#05.htm#id_hipq25nyuwlt}

[]{#05.htm#pgfId-1108711}Table 5.6 containers.conf files read by both
rootful and rootless Podman

+-----------------------------------+-----------------------------------+
| []{#05.htm#pgfId-1108715}File     | []{                               |
|                                   | #05.htm#pgfId-1108717}Description |
+-----------------------------------+-----------------------------------+
| []{#05.htm#pgfId-1108719}/usr     | []{#05.htm#pgfId-1108721}Usually  |
| /share/containers/containers.conf | shipped with the distribution     |
|                                   | defaults                          |
+-----------------------------------+-----------------------------------+
| []{#05.htm#pgfId-110872           | []{#05.htm#pgfId-1108725}System   |
| 3}/etc/containers/containers.conf | administrator can use this file   |
|                                   | to set and modify different       |
|                                   | defaults.                         |
+-----------------------------------+-----------------------------------+
| []{#05.htm#pgfId-1108727}/etc/con | []{#05.htm#pgfId-1108729}Some     |
| tainers/containers.conf.d/\*.conf | package tools might drop          |
|                                   | additional default files into     |
|                                   | this directory, sorted            |
|                                   | numerically.                      |
+-----------------------------------+-----------------------------------+

[]{#05.htm#pgfId-1105933}When running in rootless mode, Podman also
reads the files in table 5.7 if they exist.[]{#05.htm#id_b48ggzrj4qk2}

[]{#05.htm#pgfId-1108802}Table 5.7 containers.conf files read by
rootless Podman

+-----------------------------------+-----------------------------------+
| []{#05.htm#pgfId-1108806}File     | []{                               |
|                                   | #05.htm#pgfId-1108808}Description |
+-----------------------------------+-----------------------------------+
| []{#05.htm#pgfId-1108810}\$HOME/. | []{#05.htm#pgfId-1108812}Users    |
| config/containers/containers.conf | can create this file to override  |
|                                   | system defaults.                  |
+-----------------------------------+-----------------------------------+
| []{#05.htm                        | []{#05.htm#pgfId-1108816}Users    |
| #pgfId-1108814}\$HOME/.config/con | can also drop files here if they  |
| tainers/containers.conf.d/\*.conf | want, and they will be sorted     |
|                                   | numerically.                      |
+-----------------------------------+-----------------------------------+

[]{#05.htm#pgfId-1105951}Unlike storage.conf and registries.conf,
containers.conf files are merged together, and they do not fully
override previous versions. Individual fields can override the same
field in the higher-level containers.conf file. Podman does not require
any containers.conf file to exist, since it has built-in defaults. Most
systems come with only the distribution default overrides in
/usr/share/containers/containers.conf.

[]{#05.htm#pgfId-1105953}[Note]{.fm-callout-head} Podman supports the
CONTAINERS_CONF environment variable[]{#05.htm#marker-1105952}, which
forces Podman to use the target of the \$CONTAINERS_CONF. All other
containers.conf files are ignored. This is useful for testing
environments or making sure no one has customized the Podman defaults.

[]{#05.htm#pgfId-1105954}containers.conf currently supports five
different tables, as shown in table 5.8. You need to be careful that you
are in the correct table when you modify
options.[]{#05.htm#id_ompsioa1yai3}

[]{#05.htm#pgfId-1108881}Table 5.8 Containers.conf tables

+-----------------+-----------------------------------------------------+
| []{#05.htm#pgfI | []{#05.htm#pgfId-1108887}Description                |
| d-1108885}Table |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.htm#pg   | []{#05.htm#pgfId-1108891}Configuration on running   |
| fId-1108889}`[c | individual containers. Examples are the namespaces  |
| ontainers]`{.fm | to stick containers in, whether or not SELinux is   |
| -code-in-text1} | enabled, and default environment variables for      |
|                 | containers.                                         |
+-----------------+-----------------------------------------------------+
| []{#05.ht       | []{#05.htm#pgfId-1108895}Default configurations for |
| m#pgfId-1108893 | Podman to use. Examples are the default logging     |
| }`[engine]`{.fm | system, paths for OCI runtimes to use, and the      |
| -code-in-text1} | location of conmon.                                 |
+-----------------+-----------------------------------------------------+
| []{#05.h        | []{#05.htm#pgfId-1108899}Remote connection data for |
| tm#pgfId-110889 | use with `podman`{.fm-code-in-text1}                |
| 7}`[service_des | `--remote`{.fm-code-in-text1}. Remote service is    |
| tinations]`{.fm | covered in chapter 9.                               |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.htm      | []{#05.htm#pgfId-1108903}Information about the      |
| #pgfId-1108901} | `secrets`{.fm-code-in-text1} plugin driver to use   |
| `[secrets]`{.fm | for containers                                      |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#05.htm      | []{#05.htm#pgfId-1108907}Special configuration for  |
| #pgfId-1108905} | network configuration, including the default        |
| `[network]`{.fm | network name, location of CNI plugins, and default  |
| -code-in-text1} | subnets                                             |
+-----------------+-----------------------------------------------------+

[]{#05.htm#pgfId-1105984}Many users of Podman want to change the default
ways it launches containers in an environment. I previously explained
how the HPC community wants to use Podman to run their workloads, but
they are very specific about the volumes that get added to containers,
which environment variables are added, and which namespaces are enabled.

[]{#05.htm#pgfId-1105985}Perhaps you want all of your containers to have
the same environment variables set. Let's try an example. Run
`podman`{.fm-code-in-text} to show the default environment in the ubi8
image.

``` programlisting
$ podman run --rm ubi8 printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
container=oci
HOME=/root
HOSTNAME=ba4acf180386
```

[]{#05.htm#pgfId-1105992}[Note]{.fm-callout-head} When using Podman in
remote mode, for example on a Mac or Windows box, most of the settings
of the containers.conf files are used from the Linux box on the server
side. A containers.conf file in the user's home directory is used for
storing connection data, which is covered in chapter 9. Mac and Windows
clients are covered in appendixes E and F.

[]{#05.htm#pgfId-1105993}Now create an env.conf file in the home
directory with the `env="[foo=bar]"`{.fm-code-in-text} set:

``` programlisting
$ mkdir -p $HOME/.config/containers/containers.conf.d
$ cat << _EOF > $HOME/.config/containers/containers.conf.d/env.conf
[containers]
env=[ "foo=bar" ]
_EOF
Run any container and you see the foo=bar environment set.
$ podman run --rm ubi8 printenv
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TERM=xterm
container=oci
foo=bar
HOME=/root
HOSTNAME=406fc182d44b
```

[]{#05.htm#pgfId-1106007}I use containers.conf when configuring Podman
to run within a container. Many users want to run Podman within a
container for CI/CD systems or for just testing out newer versions of
Podman than their distribution enables. Because lots of people were
having a hard time running Podman in a container, I decided to try to
create a default image, quay.io/podman/stable, to help them. While
creating that image, I realized several of the Podman defaults did not
work well when running it within a container, so I used containers.conf
to change those settings. You can see my containers.conf file at this
link: [http://mng.bz/o5DM](http://mng.bz/o5DM){.url}.

[]{#05.htm#pgfId-1106008}You can see the contains.conf by actually
running the image:

``` programlisting
$ podman run quay.io/podman/stable cat /etc/containers/containers.conf 
[containers]
netns="host"
userns="host"
ipcns="host"
utsns="host"
cgroupns="host"
cgroups="disabled"
log_driver = "k8s-file"
[engine]
cgroup_manager = "cgroupfs"
events_logger="file"
runtime="crun"
```

[]{#05.htm#pgfId-1106022}Here was what I was thinking while writing this
file. First, I decided that since Podman is running inside of a
container, I would disable all of the cgroups and namespaces other than
the mount and user namespace. If users set cgroups or configured
namespaces, then the container run by Podman in a container would follow
the parent Podman's rules:

``` programlisting
[containers]
netns="host"
userns="host"
ipcns="host"
utsns="host"
cgroupns="host"
cgroups="disabled"
```

[]{#05.htm#pgfId-1106030}The default `log_driver`{.fm-code-in-text},
event logger, and cgroup manager on many distributions is journald and
system, respectively, but inside of the container, systemd and journald
are not running, so the container engine needs to use the filesystem:

``` programlisting
[containers]
log_driver = "k8s-file"
[engine]
cgroup_manager = "cgroupfs"
events_logger="file"
```

[]{#05.htm#pgfId-1106037}Finally, use the OCI runtime
`crun`{.fm-code-in-text}[]{#05.htm#marker-1106036} rather than
`runc`{.fm-code-in-text}, mainly because `crun`{.fm-code-in-text} is a
lot smaller than `runc`{.fm-code-in-text}:

``` programlisting
[engine]
runtime="crun"
```

[]{#05.htm#pgfId-1106040}Now attempt to run a container within a
container. A trick needed to make this work is running the podman/stable
image with `--user`{.fm-code-in-text} `podman`{.fm-code-in-text}. This
causes the Podman inside the container to run in rootless mode. Since
the podman/stable image uses the `fuse-overlay`{.fm-code-in-text}
driver[]{#05.htm#marker-1106041} within the container, you also need to
add the /dev/fuse device:

``` programlisting
$ podman run --device /dev/fuse --user podman quay.io/podman/stable podman 
➥ run ubi8-micro echo hi
Resolved "ubi8" as an alias (/etc/containers/registries.conf.d/
➥ 000-shortnames.conf
Trying to pull registry.access.redhat.com/ubi8:latest...
Getting image source signatures
Copying blob sha256:5368f457acd16b337e2b150741f727c46f886c69eea
➥ 1a4d56d0114c88029ed87
...
hi
```

[]{#05.htm#pgfId-1106052}[Note]{.fm-callout-head} You examined a few of
the containers.conf fields, but there are many more. Use the
`container.conf(5)`{.fm-code-in-text1} man page to explore all of them:

``` programlisting
$ man containers.conf
https:/ /github.com/containers/common/blob/main/docs/containers.conf.5.md
```

[]{#05.htm#pgfId-1106056}Now you know more about configuration tools
specific to container tools like Podman. Next, you'll learn about some
other system configuration files Podman
[]{#05.htm#marker-1106057}[]{#05.htm#marker-1106058}needs.

## 5.4 System configuration files

[]{#05.htm#pgfId-1106063}When
[]{#05.htm#marker-1108995}[]{#05.htm#marker-1108996}you run rootless
Podman, you are using the /etc/subuid and /etc/subgid files to specify
the UID ranges for your containers. As I explained in section 3.1.2,
Podman reads the /etc/subuid and /etc/subgid files for UID and GID
ranges allocated for your user account. Podman then launches
/usr/bin/newuidmap and /usr/bin/newgidmap, which verifies the range of
UIDs and GIDs Podman specified are actually allocated to you. In certain
cases you need to modify these files to add UIDs. Tools like
`useradd`{.fm-code-in-text}[]{#05.htm#marker-1108998} automatically
update the /etc/subuid and /etc/subgid when you add new users to your
system. For example, when I installed my laptop,
`useradd`{.fm-code-in-text} set up my user account to use UID
`3267`{.fm-code-in-text} and added the mapping
`dwalsh:100000:65536`{.fm-code-in-text} to /etc/subuid and /etc/subgid.
Figure 5.2 shows what containers based on this mapping look like on my
system.

::: figure
![](images/OEBPS/Images/05-02.png){.calibre18}

[]{#05.htm#pgfId-1111835}Figure 5.2 User namespace mapping for
containers
:::

[]{#05.htm#pgfId-1106070}[Note]{.fm-callout-head} You want to keep the
ranges of UIDs unique for each user and ensure they are not overlapping
with any system UIDs. Podman and the system do not verify there is no
overlap. If two different users had the same UIDs in their range, the
processes in the containers would be allowed to attack each other from
the user namespace perspective. Verifying this is a manual process. The
`useradd`{.fm-code-in-text1} tool automatically selects unique ranges.

[]{#05.htm#pgfId-1106072}As the `subuid(5)`{.fm-code-in-text} and
`subgid(5)`{.fm-code-in-text} man pages explain, each line in
/etc/subuid and /etc/subgid contains a username and a range of
subordinate user IDs or GIDs, respectively, that the user is allowed to
use. The entry is specified with three fields delimited by colons. These
fields are the following:

-   []{#05.htm#pgfId-1106073 .calibre17}Login name or UID

-   []{#05.htm#pgfId-1106074 .calibre17}Numerical subordinate user ID or
    group ID

-   []{#05.htm#pgfId-1106075 .calibre17}Numerical subordinate user ID or
    group ID count

[]{#05.htm#pgfId-1106076}Newer versions of the operating system,
specifically the packages that ship /usr/bin/ newuidmap and
/usr/bin/newgidmap, are gaining the ability to share the contents of
these files via the network from an LDAP server. On Fedora, these
executables are shipped in the `shadow-utils`{.fm-code-in-text} package.
Versions 4.9 or later have this feature.

[]{#05.htm#pgfId-1106077}[Tip]{.fm-callout-head} Changes to /etc/subuid
and /etc/subgid may not be immediately reflected in the user's account.
This is a common problem for users who modify these files after they
have already run Podman. But remember: when Podman first runs, it
launches the `podman`{.fm-code-in-text1} `pause`{.fm-code-in-text1}
process[]{#05.htm#marker-1108199} in the user namespace, and then all
other containers join this Podman process's user namespace. To have a
new user namespace take effect, you must execute the
`podman`{.fm-code-in-text1} `system`{.fm-code-in-text1}
`migrate`{.fm-code-in-text1} command[]{#05.htm#marker-1108200}, which
stops the `podman`{.fm-code-in-text1} `pause`{.fm-code-in-text1}
[]{#05.htm#marker-1108201}[]{#05.htm#marker-1108202}process[]{#05.htm#marker-1108203}
and re-creates the user namespace.

## Summary 

-   []{#05.htm#pgfId-1106085 .calibre17}Podman has multiple
    configuration files based on the libraries it uses.

-   []{#05.htm#pgfId-1106086 .calibre17}Configuration files are shared
    between rootful and rootless environments.

-   []{#05.htm#pgfId-1106087 .calibre17}The storage.conf file is used to
    configure containers/storage, including the storage driver as well
    as the location where containers and their images are to be stored.

-   []{#05.htm#pgfId-1106088 .calibre17}The registries.conf and
    policy.json files are used to configure the container/ image
    library---primarily affecting access to container registries, short
    names, and mirror sights.

-   []{#05.htm#pgfId-1106089 .calibre17}The containers.conf file is used
    to configure all of the other defaults used within Podman.

-   []{#05.htm#pgfId-1106090 .calibre17}System configuration files
    /etc/subuid and /etc/subgid are used to configure the user namespace
    required for running rootless []{#05.htm#marker-1106091
    .calibre17}Podman.

[]{#06.htm}
