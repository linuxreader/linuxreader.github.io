# 3 Volumes

[]{#03.htm#pgfId-1048583}This chapter []{#03.htm#marker-1051546}covers

-   []{#03.htm#pgfId-1048584 .calibre17}Using volumes to isolate data
    from the containerized application
-   []{#03.htm#pgfId-1048585 .calibre17}Sharing content from your host
    into containers via volumes
-   []{#03.htm#pgfId-1048587 .calibre17}Using volumes with the user
    namespace and []{#03.htm#id_Hlk114568025 .calibre17}SELinux
-   []{#03.htm#pgfId-1048588 .calibre17}Embedding volumes into container
    images
-   []{#03.htm#pgfId-1048590 .calibre17}Exploring different types of
    volumes and the []{#03.htm#id_Hlk114568236 .calibre17}volume
    commands

[]{#03.htm#pgfId-1048591}Up until now, the containers you have been
working with include all their content within the container image. As I
described in chapter 1, the only thing required to be shared with
traditional containers is the Linux kernel. There are several reasons
you need to isolate application data from the application, including the
following:

-   []{#03.htm#pgfId-1048592 .calibre17}Avoiding embedding actual data
    for applications such as databases.

-   []{#03.htm#pgfId-1048593 .calibre17}Using the same container image
    to run multiple environments.

-   []{#03.htm#pgfId-1048594 .calibre17}Reducing overhead and improving
    storage read/write performance, since volumes write directly to the
    filesystem, while containers use the overlay or
    []{#03.htm#id_Hlk114568472 .calibre17}fuse-overlayfs filesystem to
    mount their layers. *Overlay* is a layered filesystem, meaning the
    kernel needs to copy the previous layer entirely to create a new
    layer, and fuse-overlayfs switches each read and write from kernel
    space to user space and back. All of this creates quite an overhead.

-   []{#03.htm#pgfId-1048596 .calibre17}Sharing content available via
    network storage.

[]{#03.htm#pgfId-1048597}[Note]{.fm-callout-head}
`bind`{.fm-code-in-text1} mounts remount parts of the file hierarchy in
a different location on the filesystem. The files and directories in the
`bind`{.fm-code-in-text1} mount are the same as the original (see the
`mount`{.fm-code-in-text1} command[]{#03.htm#marker-1048598} man page
for an explanation of `bind`{.fm-code-in-text1} mounts). A
`bind`{.fm-code-in-text1} mount allows the same content to be accessible
in two places, without any additional overhead. It is important to
understand that `bind`{.fm-code-in-text1} does not copy the data or
create new data.

[]{#03.htm#pgfId-1048599}Supporting volumes also adds complexity,
especially concerning security. A lot of the security features of
containers prevent the container processes from gaining access to the
filesystem outside the container image. In this chapter, you will
discover the ways Podman allows you to work around these obstacles.

## []{#03.htm#pgfId-1048601}3.1 Using volumes with containers {#03.htm#heading_id_3 .fm-head}

[]{#03.htm#pgfId-1048603}Let's []{#03.htm#marker-1048602}go back to your
containerized application. Up until now, you have simply embedded the
web application data into your container filesystem directly. Recall
that in section 2.1.8, you used the `podman`{.fm-code-in-text}
`exec`{.fm-code-in-text} command[]{#03.htm#marker-1048604} to modify the
Hello World index.html data within the container:

``` programlisting
$ podman exec -i myapp bash -c 'cat > /var/www/html/index.html' << _EOF
<html>
 <head>
 </head>
 <body>
 <h1>Hello World</h1>
 </body>
</html>
_EOF 
```

[]{#03.htm#pgfId-1048614}You have made the containerized image more
flexible by allowing users to supply their own content for the web
service or perhaps to update the web service on the fly. At the same
time, while this method is possible, it is error prone and not scalable;
it is where volumes come in handy.

[]{#03.htm#pgfId-1048616}Podman allows you to mount host filesystem
content into containers using the `podman`{.fm-code-in-text}
`run`{.fm-code-in-text} command[]{#03.htm#marker-1048615} via the
`--volume`{.fm-code-in-text} `(-v)`{.fm-code-in-text}
option[]{#03.htm#marker-1048617}.

[]{#03.htm#pgfId-1048619}The `--volume`{.fm-code-in-text}
`HOST-DIR:CONTAINER-DIR`{.fm-code-in-text}
option[]{#03.htm#marker-1048618} tells Podman to
`bind`{.fm-code-in-text} mount `HOST-DIR`{.fm-code-in-text} in the host
to `CONTAINER-DIR`{.fm-code-in-text} in the container. Podman supports
other kinds of volumes as well, but in this section, I will focus on
`bind`{.fm-code-in-text} mount volumes.

[]{#03.htm#pgfId-1048620}It is possible to mount both files or
directories in a single option. Changes of the content on the host will
be seen inside the container. Similarly, if container processes change
the content inside the container, the changes will be seen on the host.

[]{#03.htm#marker-1051591}[]{#03.htm#pgfId-1048621}Let's look at an
example. Create a directory, html, in your home directory, and then
create a new html/index.html file in it:

``` programlisting
$ mkdir html
$ cat > html/index.html << _EOF
<html>
 <head>
 </head>
 <body>
 <h1>Goodbye World</h1>
 </body>
</html>
_EOF
```

[]{#03.htm#pgfId-1048632}Now launch a container with the option
`-v`{.fm-code-in-text} `./html:/var/www/html`{.fm-code-in-text}:

``` programlisting
$ podman run -d -v ./html:/var/www/html:ro,z -p 8080:8080
     quay.io/rhatdan/myimage
94c21a3d8fda740857abc571469aaaa181f4db27a464ceb6743c4a37fb875772
```

[]{#03.htm#pgfId-1048637}Notice the extra `:ro,z`{.fm-code-in-text}
fields in the `--volume`{.fm-code-in-text}
option[]{#03.htm#marker-1048635}. The `ro`{.fm-code-in-text}
option[]{#03.htm#marker-1048636} tells Podman to mount the volume in
read-only mode. The read-only mount means processes within the container
cannot modify any content under /var/www/html, while processes on the
host are still able to modify the content. Podman defaults all volume
mounts to read/write mode. The `z`{.fm-code-in-text} option tells Podman
to relabel the content to a shared label for use by SELinux (see section
3.1.2).

[]{#03.htm#pgfId-1048638}Now that you have launched the container, open
a web browser, and navigate to localhost:8080 to make sure the changes
have taken place (see figure 3.1).

``` programlisting
$ web-browser localhost:8080
```

::: figure
![](images/OEBPS/Images/03-01.png){.calibre18}

[]{#03.htm#pgfId-1054147}Figure 3.1 Web browser window connecting to the
`myimage`{.fm-code-in-text} Podman container[]{#03.htm#marker-1054148}
with volume mounted L
:::

[]{#03.htm#pgfId-1048647}Now you can shut down and remove the container
you just created. Removing the container does not affect the content at
all. The following command removes the latest
(`--latest`{.fm-code-in-text}) container, yours. The
`--force`{.fm-code-in-text} option[]{#03.htm#marker-1048648} tells
Podman to stop the container and then remove it:

``` programlisting
$ podman rm --latest --force
```

[]{#03.htm#pgfId-1048650}Finally, remove the content with this command:

``` programlisting
$ rm -rf html
```

[]{#03.htm#pgfId-1048653}[Note]{.fm-callout-head} The
`--latest`{.fm-code-in-text1} option[]{#03.htm#marker-1048652} is not
available on Mac and Windows. You must specify the container name or ID.
Remote mode is explained in chapter 9, and Podman on Mac and Windows is
explained in appendixes E and F.

### []{#03.htm#pgfId-1048655}3.1.1 Named volumes {#03.htm#heading_id_4 .fm-head1}

[]{#03.htm#pgfId-1048659}In
[]{#03.htm#marker-1048656}[]{#03.htm#marker-1048657}[]{#03.htm#marker-1048658}the
first volume example, you created a directory on disk and then mounted
it into the container. Similarly, you can take any existing file or
directory and mount it into a container, as long as you have read access
to it.

[]{#03.htm#pgfId-1048660}Another mechanism for persisting Podman
containers data is named `volume`{.fm-code-in-text}. You can create one
of these with the `podman`{.fm-code-in-text} `volume`{.fm-code-in-text}
`create`{.fm-code-in-text} command[]{#03.htm#marker-1048661}. In the
following example you will create a `volume`{.fm-code-in-text} named
`webdata`{.fm-code-in-text}[]{#03.htm#marker-1048662}:

``` programlisting
$ podman volume create webdata
webdata
```

[]{#03.htm#pgfId-1048665}Podman defaults to creating local-named
volumes, with storage allocated in the container storage directories.
You can inspect the volume and look for its mount point using the
following command:

``` programlisting
$ podman volume inspect webdata
[
  {
      "Name": "webdata",
      "Driver": "local",
      "Mountpoint":
➥ "/home/dwalsh/.local/share/containers/storage/volumes/webdata/_data",
      "CreatedAt": "2021-10-11T14:10:48.741367132-04:00",
      "Labels": {},
      "Scope": "local",
      "Options": {}
  }
]
```

[]{#03.htm#pgfId-1048679}Podman actually creates a directory in your
local container storage, /home/dwalsh/
.local/share/containers/storage/volumes/webdata/\_data, to store the
content of the volume. You can create content from the host in this
directory:

``` programlisting
$ cat > /home/dwalsh/.local/share/containers/storage/volumes/webdata/_
data/index.html << _EOL
<html>
 <head>
 </head>
 <body>
 <h1>Goodbye World</h1>
 </body>
</html>
_EOL
```

[]{#03.htm#marker-1051690}[]{#03.htm#pgfId-1048690}Now you can run the
`myimage`{.fm-code-in-text} application[]{#03.htm#marker-1048689} using
this volume:

``` programlisting
$ podman run -d -v webdata :/var/www/html:ro,z -p 8080:8080 quay.io/rhatdan/myimage
0c8eb612831f8fe22438d73d801e5bb664ec3b1d524c5c10759ee0049061cb6b
```

[]{#03.htm#pgfId-1048693}Now refresh the web browser to ensure the file
created in the host directory is displaying "Goodbye World" (see figure
3.2).

::: figure
![](images/OEBPS/Images/03-02.png){.calibre18}

[]{#03.htm#pgfId-1054186}Figure 3.2 Web browser window connecting to the
`myimage`{.fm-code-in-text} Podman container[]{#03.htm#marker-1054187}
with the named volume mounted
:::

[]{#03.htm#pgfId-1048701}Named volumes can be used for more than one
container at a time, and they will stay around even after the container
is removed. If you are done with the named volume and container, you can
first stop the container without waiting for the processes to finish:

``` programlisting
$ podman stop -t 0 0c8eb61283
```

[]{#03.htm#pgfId-1048705}Then remove the volume with the
`podman`{.fm-code-in-text} `volume`{.fm-code-in-text}
`rm`{.fm-code-in-text} command[]{#03.htm#marker-1048703}. Note the
`--force`{.fm-code-in-text} option[]{#03.htm#marker-1048704}, which
tells Podman to remove the volume and all containers that rely on the
volume:

``` programlisting
$ podman volume rm --force webdata 
```

[]{#03.htm#pgfId-1048708}Now you can make sure the volume is gone by
executing the `volume`{.fm-code-in-text} `list`{.fm-code-in-text}
command[]{#03.htm#marker-1048707}:

``` programlisting
$ podman volume list
```

[]{#03.htm#pgfId-1048711}If a named volume doesn\'t exist prior to
executing the `podman`{.fm-code-in-text} `run`{.fm-code-in-text}
command[]{#03.htm#marker-1048710}, it will be created automatically. In
the following example, you will specify `webdata1`{.fm-code-in-text} for
the name of the named volume, then list the volumes:

``` programlisting
$ podman run -d -v webdata1:/var/www/html:ro,z -p 8080:8080\ 
➥ quay.io/rhatdan/myimage
58ccaf37958496322e34cd933cd4dd5a61ab06c5ba678beb28fdc29cfb81f407
  
$ podman volume list
DRIVER   VOLUME NAME
local     webdata1
```

[]{#03.htm#pgfId-1048719}Of course, this volume is empty. Remove the
`webdata1`{.fm-code-in-text} volume and container:

``` programlisting
$ podman volume rm --force webdata1 
```

[]{#03.htm#pgfId-1048721}Podman also supports other types of volumes. It
uses the concept of volume plugins so third parties can provide volumes;
see the `podman-volume-create`{.fm-code-in-text} man
pages[]{#03.htm#marker-1048722} for more information.

[]{#03.htm#pgfId-1048724}Podman has other interesting volume features.
The `podman`{.fm-code-in-text} `volume`{.fm-code-in-text}
`export`{.fm-code-in-text} command[]{#03.htm#marker-1048723} exports all
the content of a volume into an external TAR archive. This archive can
be copied to other machines used to recreate the volume on another
machine with the `podman`{.fm-code-in-text} `volume`{.fm-code-in-text}
`import`{.fm-code-in-text} command[]{#03.htm#marker-1048725}. Now that
you understand the handling of volumes, it is time to dig deeper into
volume
[]{#03.htm#marker-1048726}[]{#03.htm#marker-1048727}[]{#03.htm#marker-1048728}options.

### []{#03.htm#pgfId-1048730}3.1.2 Volume mount options {#03.htm#heading_id_5 .fm-head1}

[]{#03.htm#pgfId-1048735}You
[]{#03.htm#marker-1048731}[]{#03.htm#marker-1048732}[]{#03.htm#marker-1048733}have
been using volume mount options throughout this chapter. The
`ro`{.fm-code-in-text} option[]{#03.htm#marker-1048734} tells Podman to
mount the read-only volume, and the lowercase `z`{.fm-code-in-text}
option[]{#03.htm#marker-1048736} tells Podman to relabel the content
with SELinux labels that will allow multiple containers to read and
write in the volume:

``` programlisting
$ podman run -d -v ./html:/var/www/html:ro,z -p 8080:8080 quay.io/rhatdan/myimage
```

[]{#03.htm#pgfId-1048738}Podman supports some other interesting volume
options.

[]{#03.htm#pgfId-1048740}The U volume option

[]{#03.htm#pgfId-1048744}Sometimes
[]{#03.htm#marker-1048741}[]{#03.htm#marker-1048742}[]{#03.htm#marker-1048743}when
you run a rootless container, you need a volume to be owned by the user
of the container. Imagine your application needs to allow the
[]{#03.htm#id_Hlk114570711}web server to write to the volume. In your
container, the Apache Web Server process (`httpd`{.fm-code-in-text}) is
run as the `apache`{.fm-code-in-text} (`UID==60`{.fm-code-in-text})
user. The html directory in your home directory is owned by your UID,
meaning it is owned by root inside the container. The kernel does not
allow a process running as `UID==60`{.fm-code-in-text} inside the
container to make changes to a directory owned by root. You must set the
ownership of the volume to `UID==60`{.fm-code-in-text}.

[]{#03.htm#pgfId-1048746}In rootless containers, the UIDs of the
container are offset by the user namespace. My user namespace mapping
looks like this:

``` programlisting
$ podman unshare cat /proc/self/uid_map
       0     3267      1
       1   100000    65536
```

[]{#03.htm#pgfId-1048750}The `UID==0`{.fm-code-in-text} inside the
container is my `UID`{.fm-code-in-text} `3267`{.fm-code-in-text}, and
`UID`{.fm-code-in-text} `1==100000`{.fm-code-in-text},
`UID`{.fm-code-in-text} `2==10000`{.fm-code-in-text} \...
`UID60==100059`{.fm-code-in-text}, meaning I need to set the ownership
of the html directory to `100059`{.fm-code-in-text}.

[]{#03.htm#pgfId-1048752}I can do this fairly simply, using the
`podman`{.fm-code-in-text} `unshare`{.fm-code-in-text}
command[]{#03.htm#marker-1048751}, as follows:

``` programlisting
$ podman unshare chown 60:60 ./html
```

[]{#03.htm#pgfId-1048754}Now everything works. One problem with this is
that I need to do some mental gymnastics to figure out which UID the
container will run with.

[]{#03.htm#pgfId-1048756}Many container images exist with the default
UID defined in them. The `mariadb`{.fm-code-in-text}
image[]{#03.htm#marker-1048755} is another example of this; it runs with
the `mysql`{.fm-code-in-text} user, `UID=999`{.fm-code-in-text}:

``` programlisting
$ podman run docker.io/mariadb grep mysql /etc/passwd
mysql:x:999:999::/home/mysql:/bin/sh
```

[]{#03.htm#pgfId-1048759}If you created a volume to be used for the
database, you need to figure out what `UID=999`{.fm-code-in-text} mapped
to within the user namespace. On my system this is
`UID=100998`{.fm-code-in-text}.

[]{#03.htm#pgfId-1048760}Podman supplies the `U`{.fm-code-in-text}
command option for this exact situation. The `U`{.fm-code-in-text}
option tells Podman to recursively change ownership
(`chown`{.fm-code-in-text}) the source volume to match the default UID
the container executes with.

[]{#03.htm#pgfId-1048761}Try it out by first creating the directory for
the database. Notice the directory in the home directory is owned by
your user:

``` programlisting
$ mkdir mariadb
$ ls -ld mariadb/
drwxrwxr-x. 1 dwalsh dwalsh 0 Oct 23 06:55 mariadb/
```

[]{#03.htm#pgfId-1048766}Now run the `mariadb`{.fm-code-in-text}
container[]{#03.htm#marker-1048765} with the `--user`{.fm-code-in-text}
`mysql`{.fm-code-in-text}, and `bind`{.fm-code-in-text} mount the
./mariadb directory to /var/lib/mariadb with the `:U`{.fm-code-in-text}
option. Notice that the directory is now owned by the
`mysql`{.fm-code-in-text} user[]{#03.htm#marker-1048767}:

``` programlisting
$ podman run --user mysql -v ./mariadb:/var/lib/mariadb:U \
➥ docker.io/mariadb ls -ld /var/lib/mariadb
drwxrwxr-x. 1 mysql mysql 0 Oct 23 10:55 /var/lib/mariadb
```

[]{#03.htm#pgfId-1048770}If you look at the mariadb directory on the
host again, you will see that it is now owned by `UID`{.fm-code-in-text}
`100998`{.fm-code-in-text} or whatever `UID`{.fm-code-in-text}
`999`{.fm-code-in-text} maps to within your user namespace:

``` programlisting
$ ls -ld mariadb/
drwxrwxr-x. 1 100998 100998 0 Oct 23 06:55 mariadb/
```

[]{#03.htm#pgfId-1048773}User namespace is not the only security
mechanism you need to work around with rootless containers. SELinux,
while great for container security, can cause some problems when working
with
[]{#03.htm#marker-1048774}[]{#03.htm#marker-1048775}[]{#03.htm#marker-1048776}volumes.

[]{#03.htm#pgfId-1048778}The SELinux volume options

[]{#03.htm#pgfId-1048782}In
[]{#03.htm#marker-1048779}[]{#03.htm#marker-1048780}[]{#03.htm#marker-1048781}my
opinion, SELinux is the best mechanism for protecting the filesystem
from hostile container processes. Over the years, several container
escapes have been thwarted by SELinux (see section 10.8 for more
information on SELinux).

[]{#03.htm#pgfId-1048783}As I explained previously, volumes leak files
from the OS into the container, and from an SELinux point of view, these
files and directories must be labeled correctly, or the kernel will
block access.

[]{#03.htm#pgfId-1048785}The lowercase `z`{.fm-code-in-text} command
option[]{#03.htm#marker-1048784} you have been using in this chapter
tells Podman to recursively relabel all content in the source directory
with a label that can be read and written by all containers from an
SELinux point of view. If the volume will not be used by more than one
container, relabeling with the lowercase `z`{.fm-code-in-text} option
isn't what you want. If a different hostile container escapes
confinement, it might be able to access this data and read/write it.
Podman provides an uppercase `Z`{.fm-code-in-text} option that tells
Podman to recursively relabel the content in such a way that only the
processes within the container can read/write the content.

[]{#03.htm#pgfId-1048786}In both previous cases, you relabeled the
content of the directory. Relabeling works great if the directory is
specified for use by containers. Sometimes you may want to use a
container to examine content in a system-specific directory---for
example, if you want to run a container that examines all the logs in
/var/log or examines all your home directories (/home/dwalsh).

[]{#03.htm#pgfId-1048787}[Note]{.fm-callout-head} Using this option on a
home directory can have disastrous effects on the system because it
recursively relabels all content in the directory as if the data was
private to a container. Other confined domains would be prevented from
using the mislabeled data.

[]{#03.htm#pgfId-1048788}For these cases, you need to disable SELinux
enforcement for container separation to allow the containers to use the
volume. Podman provides the command option
`--security-opt`{.fm-code-in-text}
`label=disable`{.fm-code-in-text}[]{#03.htm#marker-1048789} to disable
SELinux support for a single container, basically running the container
with an *unconfined* label[]{#03.htm#marker-1048790} from an SELinux
perspective:

``` programlisting
$ podman run --security-opt label=disable -v /home/dwalsh:/home/dwalsh -p\ 
➥ 8080:8080 quay.io/rhatdan/myimage
```

[]{#03.htm#pgfId-1052208}Table 3.1 lists and describes all of the mount
[]{#03.htm#marker-1053206}[]{#03.htm#marker-1053207}[]{#03.htm#marker-1053208}options
[]{#03.htm#marker-1053223}[]{#03.htm#marker-1053224}[]{#03.htm#marker-1053225}available
[]{#03.htm#marker-1053240}in []{#03.htm#marker-1053255}Podman.

[]{#03.htm#pgfId-1053090}Table 3.1 Volume mount options

+-----------------+-----------------------------------------------------+
| [               | []{#03.htm#pgfId-1053096}Description                |
| ]{#03.htm#pgfId |                                                     |
| -1053094}Volume |                                                     |
| option          |                                                     |
+-----------------+-----------------------------------------------------+
| []{#03          | []{#03.htm#pgfId-1053100}Prevent container          |
| .htm#pgfId-1053 | processes from using character or block devices on  |
| 098}`nodev`{.fm | the volume.                                         |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#03.         | []{#03.htm#pgfId-1053104}Prevent container          |
| htm#pgfId-10531 | processes from direct execution of any binaries on  |
| 02}`noexec`{.fm | the volume.                                         |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#03.         | []{#03.htm#pgfId-1053108}Prevent SUID applications  |
| htm#pgfId-10531 | from changing their privilege on the volume.        |
| 06}`nosuid`{.fm |                                                     |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+
| []              | []{#03.htm#pgfId-1053112}Mount the directory from   |
| {#03.htm#pgfId- | the host as a temporary storage using the overlay   |
| 1053110}`O`{.fm | filesystem. Modifications to the mount point are    |
| -code-in-text1} | destroyed when the container finishes executing.    |
|                 | This option is useful for sharing the package cache |
|                 | from the host into the container to allow speeding  |
|                 | up builds.                                          |
+-----------------+-----------------------------------------------------+
| []{#03.htm#     | []{#03.htm#pgfId-1053116}Specify mount propagation  |
| pgfId-1053114}` | mode. Mount propagation controls how changes to     |
| [r]shared|`{.fm | mounts are propagated across mount boundaries:      |
| -code-in-text1} |                                                     |
|                 | -   []                                              |
| `[r]slave|`{.fm | {#03.htm#pgfId-1053158}`private`{.fm-code-in-text1} |
| -code-in-text1} |     (default)---Any mounts done inside container    |
|                 |     will not be visible on host and vice versa.     |
| `[              |                                                     |
| r]private|`{.fm | -   []{#03.htm                                      |
| -code-in-text1} | #pgfId-1053140}`shared`{.fm-code-in-text1}---Mounts |
|                 |     done under that volume inside container will be |
| `[r]            |     visible on host and vice versa.                 |
| unbindable`{.fm |                                                     |
| -code-in-text1} | -   []{#03.ht                                       |
|                 | m#pgfId-1053143}`slave`{.fm-code-in-text1}---Mounts |
|                 |     done on host under that volume will be visible  |
|                 |     inside container but not the other way around.  |
|                 |                                                     |
|                 | -   []{#03.htm                                      |
|                 | #pgfId-1053146}`unbindable`{.fm-code-in-text1}---An |
|                 |     unbindable version of private mode.             |
|                 |                                                     |
|                 | []{#03.htm#pgfId-1053149}The prefix                 |
|                 | `r`{.fm-code-in-text1} stands for *recursive*,      |
|                 | meaning that any mounts underneath the mount point  |
|                 | will also be treated the same way.                  |
+-----------------+-----------------------------------------------------+
| []{#03          | []{#03.htm#pgfId-1053120}Mount a volume in          |
| .htm#pgfId-1053 | read-only (`ro`{.fm-code-in-text1}) or read-write   |
| 118}`rw|ro`{.fm | (`rw`{.fm-code-in-text1}) mode. By default,         |
| -code-in-text1} | read/write is implied.                              |
+-----------------+-----------------------------------------------------+
| []              | []{#03.htm#pgfId-1053124}Use the correct host UID   |
| {#03.htm#pgfId- | and GID based on the UID and GID within the         |
| 1053122}`U`{.fm | container. Use with caution because this will       |
| -code-in-text1} | modify the host filesystem.                         |
+-----------------+-----------------------------------------------------+
| []{#            | []{#03.htm#pgfId-1053128}Relabel file objects on    |
| 03.htm#pgfId-10 | the shared volumes. Choose the                      |
| 53126}`z|Z`{.fm | `z`{.fm-code-in-text1} option to label volume       |
| -code-in-text1} | content as shared among multiple containers. Choose |
|                 | the `Z`{.fm-code-in-text1} option to label content  |
|                 | as unshared and private.                            |
+-----------------+-----------------------------------------------------+

[]{#03.htm#pgfId-1052260}For more information, see the man pages for
`mount`{.fm-code-in-text} and `mount_namespaces(7)`{.fm-code-in-text}.

[]{#03.htm#pgfId-1052261}Most of the time, the simple
`--volume`{.fm-code-in-text} option is powerful enough for mounting
volumes into containers. Over time, the requests for new mount options
grew too complex, so a new option called `--mount`{.fm-code-in-text} was
added.

### []{#03.htm#pgfId-1052263}3.1.3 podman run - -mount command option {#03.htm#heading_id_6 .fm-head1}

[]{#03.htm#pgfId-1052264}The `podman`{.fm-code-in-text}
`run`{.fm-code-in-text} `--mount`{.fm-code-in-text} option is a much
closer option to the underlying Linux mount command. It allows you to
specify all of the mount options that the mount command understands;
Podman passes them down directly to the kernel.

[]{#03.htm#pgfId-1052265}The only mount types currently supported are
`bind`{.fm-code-in-text}, `volume`{.fm-code-in-text},
`image`{.fm-code-in-text}, `tmpfs`{.fm-code-in-text}, and
`devpts`{.fm-code-in-text}. (For more information, see the
`podman-mount(1)`{.fm-code-in-text} man page for more information.)

[]{#03.htm#pgfId-1052267}[]{#03.htm#id_y86zr7tjtxmy}Volumes and mounts
are excellent ways to keep data separate from the container image. In
most cases, the container image should be treated as read-only, and any
data that needs to be written or is not specific to the application
should be stored outside of the container image via volumes. In a lot of
cases, you will get much better performance keeping your data separate,
because reads and writes will not have the overhead of the copy-on-write
filesystem. These mounts also make it easier to use the same container
images with different data (table 3.2).

[]{#03.htm#pgfId-1052432}Table 3.2 Podman volume commands

+---------+------------------------+-----------------------------------+
| []{#    | []{#03                 | []{                               |
| 03.htm# | .htm#pgfId-1052440}Man | #03.htm#pgfId-1052442}Description |
| pgfId-1 | page                   |                                   |
| 052438} |                        |                                   |
| Command |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{     | []{                    | []{#03.htm#pgfId-1052448}Create a |
| #03.htm | #03.htm#pgfId-1052446} | new volume.                       |
| #pgfId- | `podman-volume-create( |                                   |
| 1052444 | 1)`{.fm-code-in-text1} |                                   |
| }`creat |                        |                                   |
| e`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{     | []{                    | []{#03.htm#pgfId-1052454}Check if |
| #03.htm | #03.htm#pgfId-1052452} | a volume exists.                  |
| #pgfId- | `podman-volume-exists( |                                   |
| 1052450 | 1)`{.fm-code-in-text1} |                                   |
| }`exist |                        |                                   |
| s`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{     | []{                    | []{#03.htm#pgfId-1052460}Export   |
| #03.htm | #03.htm#pgfId-1052458} | the contents of a volume into a   |
| #pgfId- | `podman-volume-export( | tar ball.                         |
| 1052456 | 1)`{.fm-code-in-text1} |                                   |
| }`expor |                        |                                   |
| t`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{     | []{                    | []{#03.htm#pgfId-1052466}Untar a  |
| #03.htm | #03.htm#pgfId-1052464} | tarball into a volume.            |
| #pgfId- | `podman-volume-import( |                                   |
| 1052462 | 1)`{.fm-code-in-text1} |                                   |
| }`impor |                        |                                   |
| t`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#    | []{#                   | []{#03.htm#pgfId-1052472}Display  |
| 03.htm# | 03.htm#pgfId-1052470}` | detailed information on a volume. |
| pgfId-1 | podman-volume-inspect( |                                   |
| 052468} | 1)`{.fm-code-in-text1} |                                   |
| `inspec |                        |                                   |
| t`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| [       | [                      | []{#03.htm#pgfId-1052478}List all |
| ]{#03.h | ]{#03.htm#pgfId-105247 | of the volumes.                   |
| tm#pgfI | 6}`podman-volume-list( |                                   |
| d-10524 | 1)`{.fm-code-in-text1} |                                   |
| 74}`lis |                        |                                   |
| t`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []      | []                     | []{#03.htm#pgfId-1052484}Remove   |
| {#03.ht | {#03.htm#pgfId-1052482 | all unused volumes.               |
| m#pgfId | }`podman-volume-prune( |                                   |
| -105248 | 1)`{.fm-code-in-text1} |                                   |
| 0}`prun |                        |                                   |
| e`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#03  | []{#03.htm#pgfId-1052  | []{#03.htm#pgfId-1052490}Remove   |
| .htm#pg | 488}`podman-volume-rm( | one or more volumes.              |
| fId-105 | 1)`{.fm-code-in-text1} |                                   |
| 2486}`r |                        |                                   |
| m`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+

## []{#03.htm#pgfId-1052328}Summary {#03.htm#heading_id_7 .fm-head}

-   []{#03.htm#pgfId-1052329 .calibre17}Volumes are useful for
    separating the data used by a container from the application inside
    an image.

-   []{#03.htm#pgfId-1052204 .calibre17}Volumes mount parts of the
    filesystem into a container\'s environment, which means security
    concerns like SELinux and user namespace need to be modified to
    allow access.

[]{#04.htm}
