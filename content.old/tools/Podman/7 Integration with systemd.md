# 7 Integration with systemd 

[]{#07.htm#pgfId-1110287}This chapter []{#07.htm#marker-1113414}covers

-   []{#07.htm#pgfId-1110288 .calibre17}Running systemd within the
    container as the primary process
-   []{#07.htm#pgfId-1110289 .calibre17}Generating systemd unit files
    from existing containers
-   []{#07.htm#pgfId-1110290 .calibre17}Socket-activated containerized
    services
-   []{#07.htm#pgfId-1110292 .calibre17}Using
    `sd-notify`{.fm-code-in-text} containerized
    services[]{#07.htm#marker-1113419 .calibre17}
-   []{#07.htm#pgfId-1110293 .calibre17}The advantages of using journald
    as a logging driver and events backend
-   []{#07.htm#pgfId-1110294 .calibre17}Using Podman and systemd to
    manage containerized services' life cycles on edge devices

[]{#07.htm#pgfId-1110295}Systemd is the de facto init system for Linux.
Almost every distribution of Linux defaults to systemd as the first
process launched after the kernel, which then launches all of the
services, including the login sessions for the user. Podman embraces the
power of systemd and uses it for starting up lots of its services. When
starting containerized services at boot time, Podman encourages users to
use systemd unit files with Podman commands. Unit files are what systemd
calls its configuration files. Systemd supports a few different types of
unit files, including service files in which you can define a service,
which you would want systemd to manage. A SystemD.socket is another kind
of unit file systemd uses (see section 7.6). The systemd service unit
files are a way to share your containerized service with the world. As
you see in figure 7.1, Podman's []{#07.htm#id_Hlk115164025}fork/exec
model grants systemd the ability to track the processes within a
containerized service.

::: figure
![](images/OEBPS/Images/07-01.png){.calibre18}

[]{#07.htm#pgfId-1118970}Figure 7.1 Systemd executing a Podman container
:::

[]{#07.htm#pgfId-1110304}Systemd puts all the processes within a unit
file service (called a scope[]{#07.htm#marker-1110303}) into the same
cgroup hierarchy. It then uses the PID cgroup to keep track of all the
processes and uses this information to manage the service. Container
engines that use client-server methodology prevent systemd from keeping
track of the containerized processes.

[]{#07.htm#pgfId-1110305}Podman also takes advantage of other services,
as you will see in this chapter, to handle auto-restarting containers,
[]{#07.htm#id_Hlk115164229}auto-updating, and basic management of
containerized services. You will be exposed to many Podman and systemd
features in this chapter, but first you will run systemd within a Podman
container.

## 7.1 Running systemd within a container

[]{#07.htm#pgfId-1110311}When
[]{#07.htm#marker-1110309}[]{#07.htm#marker-1110310}containerization was
first becoming popular, many evangelists taught the concept of
microservices. A *microservice*[]{#07.htm#marker-1110312} is defined as
one specialized service within a container. This single service runs as
the initial PID (PID 1) within the containers and writes its logs
directly to `stdout`{.fm-code-in-text} and `stderr`{.fm-code-in-text}.
Kubernetes assumes microservices, and thus gathers logs from the
`stdin`{.fm-code-in-text}/`stderr`{.fm-code-in-text} of the containers
it runs. Figure 7.2 shows Podman running microservices.

::: figure
![](images/OEBPS/Images/07-02.png){.calibre18}

[]{#07.htm#pgfId-1119015}Figure 7.2 Podman running three microservices
:::

[]{#07.htm#pgfId-1110319}An alternative idea was to run systemd as the
initial PID within the container and then allow systemd to start one or
more services within the container. This school of thought argues that
containerized services are to be launched the same way they are launched
within a VM. Because service package designers (e.g., RPM and APT)
develop systemd unit files as a precise way of launching their services
within the OS, container developers should take advantage of these unit
files. This approach allows running multiple services within the same
container, taking advantage of local communications paths, and speeding
up the conversion of large []{#07.htm#id_Hlk115164558}multiservice
applications into a container and then, over time, breaking each service
into its own microservice.

[]{#07.htm#pgfId-1110321}A final huge advantage of systemd in a
container is that the init system handles the cleaning up of a zombie
process[]{#07.htm#marker-1110322}. In Linux, when a process exits, the
kernel sends the signal
`SIGCHLD`{.fm-code-in-text}[]{#07.htm#marker-1110323} to the parent
process, and the parent process is supposed to collect the exit status
of the exiting process. The kernel removes the process from the system
when the parent reads the exit status. If no parent process reads the
exit status, the exited process is left in the exited status and is
referred to as a *zombie process*[]{#07.htm#marker-1110324}. The init
system, systemd, reaps most processes on the system. In containers, the
initial process running within the container is supposed to reap these
processes. Sometimes container processes exit, and if PID1 does not reap
them, they just linger and never disappear.

[]{#07.htm#pgfId-1110327}[Note]{.fm-callout-head} The
`podman-run`{.fm-code-in-text1} command[]{#07.htm#marker-1110325}
supports an `–init`{.fm-code-in-text1} option[]{#07.htm#marker-1110326},
which will launch a tiny init program just to reap the zombie processes.

[]{#07.htm#pgfId-1110328}Podman was designed to support both
methods---microservices as well as multiservice containers. Figure 7.3
shows systemd running a multiservice application within a container.

::: figure
![](images/OEBPS/Images/07-03.png){.calibre18}

[]{#07.htm#pgfId-1119056}Figure 7.3 Podman running systemd in a
container with three services
:::

[]{#07.htm#pgfId-1110336}Podman examines the `cmd`{.fm-code-in-text}
option[]{#07.htm#marker-1116219} of a container and then launches
systemd for init or system. It then automatically launches the container
in systemd mode.

[]{#07.htm#pgfId-1110337}The following list shows all the commands that
trigger Podman to run in systemd mode:

-   []{#07.htm#pgfId-1110338 .calibre17}/sbin/init

-   []{#07.htm#pgfId-1110339 .calibre17}/usr/sbin/init

-   []{#07.htm#pgfId-1110340 .calibre17}/usr/local/sbin/init

-   []{#07.htm#pgfId-1110341 .calibre17}/\*/systemd (any path ending
    with the systemd command)

[]{#07.htm#pgfId-1110342}The registry.access.redhat.com/ubi8-init image
is an example of an image intended to run in systemd mode.

[]{#07.htm#pgfId-1110343}Pull down the ubi8-init image, and examine the
command:

``` programlisting
$ podman pull ubi8-init
Resolved "ubi8-init" as an alias (/etc/containers/registries.conf.d/
➥ 000-shortnames.conf)
Trying to pull registry.access.redhat.com/ubi8-init:latest...
...
8cb83279f877a4bf3412827bf71c53188c3983194bd4663a1fc1378360844463
$ podman inspect ubi8-init --format '{{ .Config.Cmd }}'
[/sbin/init]
```

[]{#07.htm#pgfId-1110352}Systemd requires the environment to be
configured in a certain way; otherwise, systemd attempts to correct the
environment. The next section explains how Podman satisfies systemd
requirements.

7.1.1 Containerized systemd requirements

[]{#07.htm#pgfId-1110358}Systemd
[]{#07.htm#marker-1110356}[]{#07.htm#marker-1110357}makes some
assumptions about the environment it starts in, like /run and /tmp need
to have tmpfs mounted on them. When the environment is incorrect,
systemd attempts to correct it by mounting tmpfs on /run and /tmp.
Mounting requires `CAP_SYS_ADMIN`{.fm-code-in-text} privilege within the
container, which is not allowed in unprivileged containers. Systemd then
blows up.

[]{#07.htm#pgfId-1110359}To fix this problem, after examining the entry
point and `CMD`{.fm-code-in-text} of a container image to see if they
are running systemd, Podman modifies the container environment to match
systemd expectations. When systemd sees the mounts, it skips them,
allowing systemd to run within a locked-down environment. Table 7.1
describes the requirements systemd needs and Podman provides to
successfully run within an unprivileged
[]{#07.htm#marker-1110360}[]{#07.htm#marker-1110361}container.

[]{#07.htm#pgfId-1114353}Table 7.1 Systemd requirements for running
within a nonprivileged container

+-----------------+-----------------------------------------------------+
| []              | []{#07.htm#pgfId-1114359}Description                |
| {#07.htm#pgfId- |                                                     |
| 1114357}Systemd |                                                     |
| expectations    |                                                     |
+-----------------+-----------------------------------------------------+
| []{#07.htm#pgf  | []{#07.htm#pgfId-1114363}Systemd requires /run to   |
| Id-1114361}/run | have a tmpfs mounted on it. If /run is not mounted  |
| on a tmpfs      | with a tmpfs, systemd will attempt to mount a tmpfs |
|                 | on /run. A default locked-down container is         |
|                 | prevented from mounting, so systemd will fail.      |
+-----------------+-----------------------------------------------------+
| []{#07.htm#pgf  | []{#07.htm#pgfId-1114367}Similarly to /run, systemd |
| Id-1114365}/tmp | will attempt to mount a tmpfs on /tmp, if there is  |
| on a tmpfs      | not already one mounted there.                      |
+-----------------+-----------------------------------------------------+
| []{#07.htm#p    | []{#07.htm#pgfId-1114371}Systemd within the         |
| gfId-1114369}/v | container expects to be able to write to            |
| ar/log/journald | /var/log/journald, so Podman mounts a tmpfs to make |
| as a tmpfs      | this possible.                                      |
+-----------------+-----------------------------------------------------+
| []{#07.htm      | []{#07.htm#pgfId-1114375}Systemd uses the fact that |
| #pgfId-1114373} | a `container`{.fm-code-in-text1} environment        |
| `container`{.fm | variable is set to change some of its default       |
| -code-in-text1} | behavior, making it run better within a container.  |
| environment     |                                                     |
| variable        |                                                     |
+-----------------+-----------------------------------------------------+
| []{#07.         | []{#07.htm#pgfId-1114379}Unlike most processes on a |
| htm#pgfId-11143 | system, systemd ignores                             |
| 77}`STOPSIGNAL= | `SIGTERM`{.fm-code-in-text1} and will only cleanly  |
| SIGRTMIN+3`{.fm | exit with it when it receives the signal            |
| -code-in-text1} | `SIGRTMIN+3 (37)`{.fm-code-in-text1}.               |
+-----------------+-----------------------------------------------------+

### 7.1.2 Podman container in systemd mode

[]{#07.htm#pgfId-1110395}You
[]{#07.htm#marker-1116574}[]{#07.htm#marker-1116575}[]{#07.htm#marker-1116576}can
examine the environment of a systemd-based container with the
`--systemd =always`{.fm-code-in-text} flag[]{#07.htm#marker-1116578}.
First, launch a container with systemd mode enabled with the
`--systemd=always`{.fm-code-in-text} flag. This option runs the
container in systemd mode even when not running systemd, making it
easier to debug the environment. You can
`exec systemd`{.fm-code-in-text} at this point and start it as PID1:

``` programlisting
$ podman create –rm –name SystemD -ti –systemd=always ubi8-init sh
774a50204204768edd73f178b6afdf975cf9353e3b90af9df77273d639f60ac3
```

[]{#07.htm#pgfId-1110399}Use `podman`{.fm-code-in-text}
`inspect`{.fm-code-in-text} to examine the
`StopSignal`{.fm-code-in-text} for the container; Podman set it to
`37`{.fm-code-in-text} `(SIGRTMIN+3)`{.fm-code-in-text}:

``` programlisting
$ podman inspect SystemD --format '{{ .Config.StopSignal}}'
37
```

[]{#07.htm#pgfId-1110402}Now, start up the container, and look at the
mounts for /run and /tmp; you will see that both are mounted with a
tmpfs. Finally, check to see if the container environment variable is
set:

``` programlisting
$ podman start --attach SystemD
 mount | grep -e /tmp -e /run | head -2
tmpfs on /tmp type tmpfs 
➥ (rw,nosuid,nodev,relatime,context="system_u:object_r:container_file_t:s0:
➥ c37,c965",uid=3267,gid=3267,inode64)
tmpfs on /run type tmpfs 
➥ (rw,nosuid,nodev,relatime,context="system_u:object_r:container_file_t:s
➥ 0:c37,c965",uid=3267,gid=3267,inode64)
# printenv container
Oci
```

[]{#07.htm#pgfId-1110413}If you just run a container based on
`ubi8-init`{.fm-code-in-text}, you will see systemd launched:

``` programlisting
$ podman run -ti ubi8-init
SystemD 239 (239-45.el8_4.3) running in system mode. (+PAM +AUDIT +SELINUX 
➥ +IMA -APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS 
➥ +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD +IDN2 -IDN +PCRE2 
➥ default-hierarchy=legacy)
Detected virtualization container-other.
Detected architecture x86-64.
Welcome to Red Hat Enterprise Linux 8.4 (Ootpa)!
Set hostname to <26bbf9077219>.
Initializing machine ID from random generator.
Failed to read AF_UNIX datagram queue length, ignoring: 
➥ No such file or directory
[  OK  ] Listening on initctl Compatibility Named Pipe.
[  OK  ] Reached target Swap.
[  OK  ] Listening on Journal Socket (/dev/log).
[  OK  ] Listening on Journal Socket.
...
```

[]{#07.htm#pgfId-1110431}Here you can notice that systemd ignores
`SIGTERM`{.fm-code-in-text} by pressing Ctrl-C. So to stop this
container you need to go to a different terminal and execute

``` programlisting
# podman stop -l
```

[]{#07.htm#pgfId-1110433}This causes Podman to send the proper
`STOPSIGNAL`{.fm-code-in-text} `(SIGRTMIN+3)`{.fm-code-in-text} to
systemd in the container. Systemd will shut down instantly when it
receives this signal.

[]{#07.htm#pgfId-1110434}Now that you understand what systemd requires,
it is time to create a service systemd will run. In the following
section, you will build a systemd-based Apache service that will run
with systemd within the
[]{#07.htm#marker-1110435}[]{#07.htm#marker-1110436}[]{#07.htm#marker-1110437}container.

### 7.1.3 Running an Apache service within a systemd container

[]{#07.htm#pgfId-1110443}In
[]{#07.htm#marker-1110440}[]{#07.htm#marker-1110441}[]{#07.htm#marker-1110442}this
section, you will create a Containerfile that uses ubi8-init as the base
image and then install Apache `httpd`{.fm-code-in-text}. Finally, you
will enable this service and set up the Apache script we have been
working with.

[]{#07.htm#pgfId-1110444}Create a Containerfile:

``` programlisting
$ cat << _EOF >  /tmp/Containerfile
FROM ubi8-init
RUN dnf -y install httpd; dnf -y clean all
RUN systemctl enable httpd.service
_EOF
```

[]{#07.htm#pgfId-1110450}Recall that the `FROM`{.fm-code-in-text}
`ubi8-init`{.fm-code-in-text} line will tell Podman to use the ubi8-init
image as the base image for your new image:

``` programlisting
FROM ubi8-init
RUN dnf -y install httpd; dnf -y clean all
RUN systemctl enable httpd.service
```

[]{#07.htm#pgfId-1110454}The `RUN`{.fm-code-in-text}
`dnf`{.fm-code-in-text} `-y`{.fm-code-in-text}
`install`{.fm-code-in-text} `httpd;`{.fm-code-in-text}
`dnf`{.fm-code-in-text} `-y`{.fm-code-in-text} `clean`{.fm-code-in-text}
`all`{.fm-code-in-text} line tells Podman to run a container that
executes the `dnf`{.fm-code-in-text} command[]{#07.htm#marker-1110455}
and install the `httpd`{.fm-code-in-text} package on top of the
`ubi8-init`{.fm-code-in-text} image. The second `dnf`{.fm-code-in-text}
command removes excess files and logs `dnf`{.fm-code-in-text} created
while installing, as there is no reason to include these in the image:

``` programlisting
FROM ubi8-init
RUN dnf -y install httpd; dnf -y clean all
RUN systemctl enable httpd.service
```

[]{#07.htm#pgfId-1110462}The final `RUN`{.fm-code-in-text}
`systemctl`{.fm-code-in-text} `enable`{.fm-code-in-text}
`httpd.service`{.fm-code-in-text} command[]{#07.htm#marker-1110461}
tells Podman to launch another build container and execute the
`systemctl`{.fm-code-in-text} command[]{#07.htm#marker-1110463} to
enable the `httpd`{.fm-code-in-text} `.service`{.fm-code-in-text}. When
systemd runs on a container created from the newly created image, the
`httpd`{.fm-code-in-text} service[]{#07.htm#marker-1110464} will be
started:

``` programlisting
FROM ubi8-init
RUN dnf -y install httpd; dnf -y clean all
RUN systemctl enable httpd.service
```

[]{#07.htm#pgfId-1110468}Now build the image using
`podman`{.fm-code-in-text} `build`{.fm-code-in-text}, and name the image
`my-systemd`{.fm-code-in-text}:

``` programlisting
$ podman build -t my-systemd /tmp
STEP 1/3: FROM ubi8-init
STEP 2/3: RUN dnf -y install httpd; dnf -y clean all
Updating Subscription Management repositories.
Unable to read consumer identity
...
COMMIT my-systemd
--> 104fa99d9a2
Successfully tagged localhost/my-systemd:latest
104fa99d9a2138404039cf15b470ab04784cdaab2226f29bd8343f8e24ec60e2
```

[]{#07.htm#pgfId-1110479}Now run a container on this systemd-based
container image with a volume mounted from the host. Since the default
Apache package listens on port `80`{.fm-code-in-text}, use
`--p`{.fm-code-in-text} `8080:80`{.fm-code-in-text}, which, as you
learned, maps port `8080`{.fm-code-in-text} to port
`80`{.fm-code-in-text} within the container. Use an html folder with
index.html from section 3.1:

``` programlisting
$ podman run -d --rm -p 8080:80 -v ./html:/var/www/html:Z my-systemd
71f1678084390925b7488f68ab58cd55e16009d69b717045b8ed5ef14e8599ce
```

[]{#07.htm#pgfId-1110482}You volume mounted `(-v`{.fm-code-in-text}
`./html/:/var/www/html:Z)`{.fm-code-in-text} in the ./html directory,
with the goodbye world index.html file:

``` programlisting
$ podman run -d --rm -p 8080:80 -v ./html:/var/www/html:Z my-systemd
```

[]{#07.htm#pgfId-1110484}Launch a web browser to check whether the
containerized service is working (as seen in figure 7.4):

``` programlisting
$ web-browser localhost:8080
```

::: figure
![](images/OEBPS/Images/07-04.png){.calibre18}

[]{#07.htm#pgfId-1119097}Figure 7.4 Web browser window showing
system-based container image running your content
:::

[]{#07.htm#pgfId-1110492}Notice that you did not need to specially
handle the HTTPD server processes when designing the image; your
container is running HTTPD the same way a VM would. If you need to
enable another service within the image, you can easily do this by
installing the package and enabling its unit file.

[]{#07.htm#pgfId-1110494}To see one of the shortcomings of this setup,
you can run the `podman`{.fm-code-in-text} `logs`{.fm-code-in-text}
command[]{#07.htm#marker-1110493}:

``` programlisting
$ podman logs 71f1678084
```

[]{#07.htm#pgfId-1110496}There is no output. Since systemd is running at
the PID1 of the container, it is not writing any output to the logs. You
need to exec into the container and use `journalctl`{.fm-code-in-text}
or read the `httpd`{.fm-code-in-text} logs in /var/log/httpd/error_log
to see if there were any problems. Now that you have seen how to use
systemd within a container, it is time to see how you can use systemd
and Podman to take advantage of advanced
[]{#07.htm#marker-1110497}[]{#07.htm#marker-1110498}[]{#07.htm#marker-1110499}systemd
[]{#07.htm#marker-1110500}[]{#07.htm#marker-1110501}features.

## 7.2 Journald for logging and events

[]{#07.htm#pgfId-1110506}The
[]{#07.htm#marker-1110504}[]{#07.htm#marker-1110505}systemd journal
(journald) is the modern logging system on Linux. It is a system service
that collects and stores logging data. A big advantage of using journald
is that records are permanently stored, and log rotation is built in.
Podman uses journald by default for storing its logging data.

### []{#07.htm#pgfId-1110508}7.2.1 Log driver {#07.htm#heading_id_8 .fm-head1}

[]{#07.htm#pgfId-1110512}Podman
[]{#07.htm#marker-1110509}[]{#07.htm#marker-1110510}[]{#07.htm#marker-1110511}defaults
to using journald as the log driver on systems running with systemd as
the init system. If you run Podman in a container without systemd
running, it falls back to using the file driver. One consideration when
picking a log driver is whether the log data persists when the container
is removed.

[]{#07.htm#pgfId-1110513}A second concern is how large the log file
grows. The log records all `stdout`{.fm-code-in-text} and
`stderr`{.fm-code-in-text} within the container. Containers running for
a very long time can create a lot of log content. Only the journald
driver has log rotation built into it, provided by systemd. If you use
the k8s-file driver there is a risk your system could run out of space.
Table 7.2 shows the available log drivers and whether the log data
persists and the system supports log rotation.

[]{#07.htm#pgfId-1114456}Table 7.2 Log driver options

+-----------------+-----------------+-----------------+-----------------+
| []              | []{#07          | []              | []{#07.htm#pg   |
| {#07.htm#pgfId- | .htm#pgfId-1114 | {#07.htm#pgfId- | fId-1114470}Log |
| 1114464}Library | 466}Description | 1114468}Persist | rotation        |
|                 |                 | logs after      |                 |
|                 |                 | container       |                 |
|                 |                 | removal         |                 |
+-----------------+-----------------+-----------------+-----------------+
| []{             | []{#07.htm#pg   | []{#07          | []{#07          |
| #07.htm#pgfId-1 | fId-1114474}Use | .htm#pgfId-1114 | .htm#pgfId-1114 |
| 114472}Journald | systemd journal | 476}[✔]{.segoe} | 478}[✔]{.segoe} |
|                 | to store        |                 |                 |
|                 | logging         |                 |                 |
|                 | information     |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| []{             | []{#07.htm#pgfI | []{#07          | []{#07          |
| #07.htm#pgfId-1 | d-1114482}Store | .htm#pgfId-1114 | .htm#pgfId-1114 |
| 114480}k8s-file | logging data in | 484}[✘]{.segoe} | 486}[✘]{.segoe} |
|                 | Kubernetes      |                 |                 |
|                 | format flat     |                 |                 |
|                 | file            |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| []{#07.htm#pgf  | []{#07.htm#p    | []{#07          | []{#07          |
| Id-1114488}None | gfId-1114490}Do | .htm#pgfId-1114 | .htm#pgfId-1114 |
|                 | not store any   | 492}[✘]{.segoe} | 494}[✘]{.segoe} |
|                 | logging         |                 |                 |
|                 | information     |                 |                 |
+-----------------+-----------------+-----------------+-----------------+

[]{#07.htm#pgfId-1110550}While I recommend you use journald for the log
driver, some rootless users are not allowed to use journald, depending
on their system configuration. In other cases, like running Podman
within a container, journald is not available.

[]{#07.htm#pgfId-1110551}You can see the default log driver on your
system by using the following command:

``` programlisting
$ podman info --format '{{ .Host.LogDriver }}'
k8s-file
```

[]{#07.htm#pgfId-1110554}For some reason, the system settings on your
host were set to log to k8s-file. It is simple to override the default
log driver for your system using containers.conf. Create a
log_driver.conf file in the home directory,
\$HOME/.config/containers/containers .conf.d, with the
`log_driver`{.fm-code-in-text} option[]{#07.htm#marker-1110555} set:

``` programlisting
$ mkdir -p $HOME/.config/containers/containers.conf.d
$ cat > $HOME/.config/containers/containers.conf.d/log_driver.conf << _EOF
[containers]
log_driver="journald"
_EOF
$ podman info --format '{{ .Host.LogDriver }}'
journald
```

[]{#07.htm#pgfId-1110563}Great. Next, you will see the benefits of the
journald log driver by launching a container with the
`--rm`{.fm-code-in-text} option to remove the container when it exits:

``` programlisting
$ podman run --rm --name test2 ubi8 echo "Check if logs persist"
Check if logs persist
```

[]{#07.htm#pgfId-1110566}Check that the journal keeps a record of the
container being launched:

``` programlisting
$ journalctl -b | grep "Check if logs persist"
Nov 10 06:19:54 fedora conmon[657915]: Check if logs persist
```

[]{#07.htm#pgfId-1110569}If you had launched with the
`k8s_file`{.fm-code-in-text} option, Podman would have removed the log
file when the container was removed. No log entry would be left behind.
Like logs, Podman supports using the systemd journal to store
[]{#07.htm#marker-1110570}[]{#07.htm#marker-1110571}[]{#07.htm#marker-1110572}events.

### []{#07.htm#pgfId-1110574}7.2.2 Events {#07.htm#heading_id_9 .fm-head1}

[]{#07.htm#pgfId-1110578}Podman
[]{#07.htm#marker-1110575}[]{#07.htm#marker-1110576}[]{#07.htm#marker-1110577}events
record different steps in the container life cycle; for example, you can
see the start event of the last container you ran:

``` programlisting
$ podman events --filter event=start --since 1h
2021-11-10 06:35:06.780429582 -0500 EST container start 
➥ ecf04c4802bb120f34533560fbfc19ab023bcce63d48945ab0e8ff06cc6eeda1
...
```

[]{#07.htm#pgfId-1110583}Examine the default events logger with the
Podman info command:

``` programlisting
$ podman info --format '{{ .Host.EventLogger }}'
journald
```

[]{#07.htm#pgfId-1110588}You can modify the events logger with the
`events_logger`{.fm-code-in-text} option[]{#07.htm#marker-1110587} in
containers.conf similarly to how you did for the
`log_driver`{.fm-code-in-text}. Table 7.3 shows the available events
logging options.

[]{#07.htm#pgfId-1114771}Table 7.3 Events logger options

+-----------------+-----------------+-----------------+-----------------+
| []              | []{#07          | []              | []{#07.htm#pg   |
| {#07.htm#pgfId- | .htm#pgfId-1114 | {#07.htm#pgfId- | fId-1114785}Log |
| 1114779}Library | 781}Description | 1114783}Persist | rotation        |
|                 |                 | log data on     |                 |
|                 |                 | reboot          |                 |
+-----------------+-----------------+-----------------+-----------------+
| []{             | []{#07.htm#pg   | []{#07          | []{#07          |
| #07.htm#pgfId-1 | fId-1114789}The | .htm#pgfId-1114 | .htm#pgfId-1114 |
| 114787}Journald | systemd journal | 791}[✔]{.segoe} | 793}[✔]{.segoe} |
|                 | will record all |                 |                 |
|                 | events.         |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| []{#07.htm#pgf  | []{#07.htm#pgfI | []{#07          | []{#07          |
| Id-1114795}File | d-1114797}Store | .htm#pgfId-1114 | .htm#pgfId-1114 |
|                 | events in a     | 799}[✘]{.segoe} | 801}[✘]{.segoe} |
|                 | file, usually   |                 |                 |
|                 | on /run.        |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| []{#07.htm#pgf  | []{#07.htm#p    | []{#07          | []{#07          |
| Id-1114803}None | gfId-1114805}Do | .htm#pgfId-1114 | .htm#pgfId-1114 |
|                 | not store any   | 807}[✘]{.segoe} | 809}[✘]{.segoe} |
|                 | events          |                 |                 |
|                 | information.    |                 |                 |
+-----------------+-----------------+-----------------+-----------------+

[]{#07.htm#pgfId-1110625}If your system uses the file event logger, the
events backend file is stored on `$XDG_RUNTIME_DIR`{.fm-code-in-text}
for rootless users, which is on a tmpfs by default. The events backend
file grows continuously, until you reboot the system when using the file
driver. This could cause failures to run containers or the system to run
out of space, since the events backend does not roll over unless you are
using journald. Also, when you reboot, the events log is lost. Switching
to journald preserves the events and handles rotation of the events log.
I recommend you keep the log driver and the events driver the same
values, either as journald, a flat file, or none, if you don't need the
events and logs.

[]{#07.htm#pgfId-1110626}You have examined using systemd within Podman
as well as journald to manage log files and events. Now you will look at
how to set up your system to automatically run a container when the
system comes up
[]{#07.htm#marker-1110627}[]{#07.htm#marker-1110628}[]{#07.htm#marker-1110629}using
[]{#07.htm#marker-1110630}[]{#07.htm#marker-1110631}systemd.

## 7.3 Starting containers at boot

[]{#07.htm#pgfId-1110636}As
[]{#07.htm#marker-1110634}[]{#07.htm#marker-1110635}you learned in
chapter 1, Podman does not run as a daemon, meaning you cannot rely on a
daemon to automatically start containers at boot time. Often you will
need to run containerized services via systemd. Systemd can be
configured to install, run, and manage containerized applications. Many
applications are shipped as container images and will include systemd
service unit files for launching. There are many features provided by
systemd to improve the way containerized services run on your system.

## 7.3.1 Restarting containers

[]{#07.htm#pgfId-1110642}Podman
[]{#07.htm#marker-1110639}[]{#07.htm#marker-1110640}[]{#07.htm#marker-1110641}relies
on systemd to start containerized services by launching Podman within
systemd unit files. The `podman`{.fm-code-in-text}
`run`{.fm-code-in-text} command[]{#07.htm#marker-1110643} allows you to
choose whether to restart a container
(`--restart`{.fm-code-in-text}[]{#07.htm#marker-1110644}) if it is not
stopped by a user---for example, if the container crashes or the system
reboots. Table 7.4 shows the restart policies available to Podman.

[]{#07.htm#pgfId-1110645}One simple way systemd helps is by starting
containers with a restart policy of `always`{.fm-code-in-text}. If you
set the `always`{.fm-code-in-text} option[]{#07.htm#marker-1110646} and
the system reboots, Podman uses two systemd services to automatically
restart containers marked with `--restart=always`{.fm-code-in-text}. One
service handles rootful containers, and the other handles all rootless
containers on the system.

[]{#07.htm#pgfId-1115012}Table 7.4 Restart policy

+-----------------+-----------------------------------+-----------------+
| [               | []{                               | []              |
| ]{#07.htm#pgfId | #07.htm#pgfId-1115020}Description | {#07.htm#pgfId- |
| -1115018}Option |                                   | 1115022}Restart |
|                 |                                   | on boot         |
+-----------------+-----------------------------------+-----------------+
| []{             | []{#07.htm#pgfId-1115026}Do not   | []{#07          |
| #07.htm#pgfId-1 | restart containers on exit.       | .htm#pgfId-1115 |
| 115024}`no`{.fm |                                   | 028}[✘]{.segoe} |
| -code-in-text1} |                                   |                 |
+-----------------+-----------------------------------+-----------------+
| []{#07.htm      | []{#07.htm#pgfId-1115032}Restart  | []{#07          |
| #pgfId-1115030} | containers when they exit with a  | .htm#pgfId-1115 |
| `on-failure[:ma | nonzero exit code, retrying       | 034}[✘]{.segoe} |
| x_retries]`{.fm | indefinitely or until the         |                 |
| -code-in-text1} | optional                          |                 |
|                 | `max_retries`{.fm-code-in-text1}  |                 |
|                 | count is hit.                     |                 |
+-----------------+-----------------------------------+-----------------+
| []{#07.         | []{#07.htm#pgfId-1115038}Restart  | []{#07          |
| htm#pgfId-11150 | containers when they exit,        | .htm#pgfId-1115 |
| 36}`always`{.fm | regardless of status, retrying    | 040}[✔]{.segoe} |
| -code-in-text1} | indefinitely.                     |                 |
| or              |                                   |                 |
| `unle           |                                   |                 |
| ss-stopped`{.fm |                                   |                 |
| -code-in-text1} |                                   |                 |
+-----------------+-----------------------------------+-----------------+

[]{#07.htm#pgfId-1110676}When your system boots up, systemd runs the
following Podman command to start any containers with restart policy set
to
[]{#07.htm#marker-1114978}[]{#07.htm#marker-1114979}[]{#07.htm#marker-1114980}`always`{.fm-code-in-text}:

``` programlisting
/usr/bin/podman start --all --filter restart-policy=always
```

[]{#07.htm#pgfId-1110681}[Note]{.fm-callout-head} Podman ships with two
systemd service files used to restart services---one for rootful and one
for rootless:\
\
[]{#07.htm#pgfId-1110682}/usr/lib/systemd/system/podman-restart.service\
[]{#07.htm#pgfId-1110683}/usr/lib/systemd/user/podman-restart.service\
\
[]{#07.htm#pgfId-1110684}The `--restart=always`{.fm-code-in-text1} works
great, but it requires you to create a container on the system and will
restart containers even if they fail. Systemd was designed to run
services; you will see in the next section that you can easily create a
service unit file with Podman to run your containerized service.

## 7.3.2 Podman containers as systemd services

[]{#07.htm#pgfId-1110690}As
[]{#07.htm#marker-1110687}[]{#07.htm#marker-1110688}[]{#07.htm#marker-1110689}you
have seen, systemd uses unit files to specify how to run a service.
Figure 7.5 shows how systemd works with Podman to launch a container.

::: figure
![](images/OEBPS/Images/07-05.png){.calibre18}

[]{#07.htm#pgfId-1119135}Figure 7.5 Podman fork/exec architecture is
ideal for systemd service management.
:::

[]{#07.htm#pgfId-1110697}In figure 7.5, I point out that systemd is able
to monitor all the processes running within the systemd unit file. This
allows it to easily start and stop the processes. The
`conmon`{.fm-code-in-text} process[]{#07.htm#marker-1110698} is also
running within the systemd service monitoring the container processes.
`conmon`{.fm-code-in-text} still notices when the container exits, saves
its exit code, and cleanly shuts down the container environment. Systemd
does not know about the container; it only knows about the processes
running within the unit file, including the container processes.

[]{#07.htm#pgfId-1110699}Systemd unit files have many different ways to
run and launch processes, and Podman has many different options for
running containers. Configuring the unit files can be very complex. Many
users have written unit files to run containers, but several have
stumbled over problems when doing so. The most common problem is running
the `podman`{.fm-code-in-text} `run`{.fm-code-in-text}
`--detach`{.fm-code-in-text} command[]{#07.htm#marker-1110700} within a
unit file. When the Podman command detaches and exits, systemd assumes
the service is complete and takes it down, even though
`conmon`{.fm-code-in-text} and the container are still running. One of
the most common questions I hear from users is the following: "How
should I run my container within a systemd unit file?"

[]{#07.htm#pgfId-1110701}Podman has a feature to generate unit files
with the best defaults. First, re-create the container from
`myimage`{.fm-code-in-text}, and then use `podman`{.fm-code-in-text}
`systemd`{.fm-code-in-text} `generate`{.fm-code-in-text} to create a
systemd service unit file to manage your container.

[]{#07.htm#pgfId-1110702}Create a container based on the image you
created in chapter 2:

``` programlisting
$ podman create -p 8080:8080 --name myapp quay.io/rhatdan/myimage  
...
8879112805e976b4b6d97c07c9426bdde22ee4ffc7ba4daa59965ae25aa08331
```

[]{#07.htm#pgfId-1110706}Now use Podman to generate a unit file off of
this container:

``` programlisting
$ mkdir -p $HOME/.config/systemd/user
$ podman generate systemd myapp > $HOME/.config/systemd/user/myapp.service
```

[]{#07.htm#pgfId-1110709}Notice in the myapp.service script that Podman
created an `ExecStart`{.fm-code-in-text} field. On service start,
systemd will execute the `ExecStart`{.fm-code-in-text}
command[]{#07.htm#marker-1110710}, which simply starts the container you
created:

``` programlisting
ExecStart=/usr/bin/podman start 8879112805…
```

[]{#07.htm#pgfId-1110713}On service stop, systemd executes the
`ExecStop`{.fm-code-in-text} command[]{#07.htm#marker-1110712} added to
the unit file:

```bash
ExecStop=/usr/bin/podman stop -t 10 8879112805...
Let's take a look at the generated service file: 
$ cat $HOME/.config/systemd/user/myapp.service```bash
# container-8879112805e976b4b6d97c07c9426bdde22ee4ffc7ba4daa59965ae25aa08331.service
# autogenerated by Podman 3.4.1
# Wed Nov 10 08:23:06 EST 2021 
[Unit]
Description=Podman container-8879112805...service
Documentation=man:podman-generate-SystemD(1)
Wants=network-online.target
After=network-online.target
RequiresMountsFor=/run/user/3267/containers 
[Service]
Environment=PODMAN_SYSTEMD_UNIT=%n
Restart=on-failure
TimeoutStopSec=70
ExecStart=/usr/bin/podman start 8879112805...
ExecStop=/usr/bin/podman stop -t 10 8879112805...
ExecStopPost=/usr/bin/podman stop -t 10 8879112805...
PIDFile=/run/user/3267/containers/overlay-containers/8879112805.../userdata/conmon.pid
Type=forking
[Install]
WantedBy=multi-user.target default.target
```

[]{#07.htm#pgfId-1110737}To make this all work, you need to tell systemd
to reload its database, so it will notice changes in the unit files:

``` programlisting
$ systemctl --user daemon-reload
```

[]{#07.htm#pgfId-1110739}Start the service with the following command:

``` programlisting
$ systemctl --user start myapp
```

[]{#07.htm#pgfId-1110741}Check to see that the service is running:

```bash
$ systemctl --user status myapp
• myapp.service - Podman container-8879112805....service
   Loaded: loaded (/home/dwalsh/.config/SystemD/user/myapp.service; 
➥ disabled; vendor preset: disabled)
   Active: active (running) since Thu 2021-11-11 07:19:08 EST; 3min 9s ago
...
$ podman ps
CONTAINER ID  IMAGE                             COMMAND           
➥ CREATED     STATUS          PORTS         NAMES
8879112805e9  quay.io/rhatdan/myimage:latest    /usr/bin/run-http...  
➥ 23 hours ago  Up 5 minutes ago  0.0.0.0:8080->8080/tcp  myapp
```

[]{#07.htm#pgfId-1110753}Now you can run the web browser against
localhost port `8080`{.fm-code-in-text} to see it is running (see figure
7.6):

``` programlisting
$ web-browser localhost:8080
```

::: figure
![](images/OEBPS/Images/07-06.png){.calibre18}

[]{#07.htm#pgfId-1119176}Figure 7.6 Web browser window connecting
`myapp`{.fm-code-in-text}
:::

[]{#07.htm#pgfId-1110761}To shut down the service, execute

``` programlisting
$ systemctl --user stop myapp
```

[]{#07.htm#pgfId-1110763}The ability to generate systemd service files
offers a lot of flexibility to users, and it intentionally blurs the
difference between a container and any other program or service on the
host.

[]{#07.htm#pgfId-1110764}One problem with this unit file is that it's
specific to the container you created. You need to first create the
container and generate specific service files. You are not able to hand
the unit file to another user and have them run your service on their
machine. Luckily, Podman has support for creating a more portable
systemd unit file:
[]{#07.htm#marker-1110765}[]{#07.htm#marker-1110766}[]{#07.htm#marker-1110767}`podman`{.fm-code-in-text}
`generate`{.fm-code-in-text} `systemd`{.fm-code-in-text}
`--new.`{.fm-code-in-text}

### 7.3.3 Distributing systemd unit files to manage Podman containers

[]{#07.htm#pgfId-1110773}As
[]{#07.htm#marker-1110770}[]{#07.htm#marker-1110771}[]{#07.htm#marker-1110772}shown
previously, the `podman`{.fm-code-in-text} `generate`{.fm-code-in-text}
`systemd`{.fm-code-in-text} `command`{.fm-code-in-text} generated a unit
file, which started and stopped an existing container. The
`--new`{.fm-code-in-text} flag[]{#07.htm#marker-1110774} instructs
Podman to generate units that run, stop, and remove containers. Try it
out in the same container:

``` programlisting
$ podman generate systemd --new myapp > $HOME/.config/systemd/user/
➥ myapp-new.service
```

[]{#07.htm#pgfId-1110778}Notice that with the `--new`{.fm-code-in-text}
option,[]{#07.htm#marker-1116820} Podman creates a slightly different
unit file. Examine the following `ExecStart`{.fm-code-in-text}
command[]{#07.htm#marker-1116822}, and you will see the original
`podman`{.fm-code-in-text} `create`{.fm-code-in-text}
`-p`{.fm-code-in-text} `8080:8080`{.fm-code-in-text}
`--name`{.fm-code-in-text} `myapp`{.fm-code-in-text}
`quay.io/rhatdan/myimage`{.fm-code-in-text} command you used to create
the container has been changed to use the `podman`{.fm-code-in-text}
`run`{.fm-code-in-text} command[]{#07.htm#marker-1116823}. Also notice
that Podman added additional options to make running under systemd
easier `(--cidfile =%t/%n.ctr-id`{.fm-code-in-text}
`--cgroups=no-conmon`{.fm-code-in-text} `--rm`{.fm-code-in-text}
`--sdnotify=conmon`{.fm-code-in-text} `-d`{.fm-code-in-text}
`--replace).`{.fm-code-in-text}

[]{#07.htm#pgfId-1110782}Podman now adds the
`ExecStop`{.fm-code-in-text} command[]{#07.htm#marker-1116824}
(`/usr/bin/podman`{.fm-code-in-text} `stop`{.fm-code-in-text}
`--ignore`{.fm-code-in-text}
`--cidfile=%t/%n.ctr-id`{.fm-code-in-text}), which tells systemd how to
stop the container when someone executes `systemctl`{.fm-code-in-text}
`stop`{.fm-code-in-text} or the system shuts down.

[]{#07.htm#pgfId-1110784}Finally, Podman adds an
`ExecStopPost`{.fm-code-in-text} command[]{#07.htm#marker-1116811}
(`/usr/bin/podman`{.fm-code-in-text} `rm`{.fm-code-in-text}
`-f`{.fm-code-in-text} `--ignore`{.fm-code-in-text}
`--cidfile=%t/%n.ctr-idType=notify`{.fm-code-in-text}), which systemd
executes once the `ExecStop`{.fm-code-in-text}
command[]{#07.htm#marker-1116813} completes. The Podman command removes
the container from the system:

```bash
$ cat $HOME/.config/systemd/user/myapp-new.service
# container-8879112805....service
# autogenerated by Podman 3.4.1
# Thu Nov 11 07:40:34 EST 2021
[Unit]
Description=Podman container-8879112805...service
Documentation=man:podman-generate-SystemD(1)
Wants=network-online.target
After=network-online.target
RequiresMountsFor=%t/containers
[Service]
Environment=PODMAN_SystemD_UNIT=%n
Restart=on-failure
TimeoutStopSec=70
ExecStartPre=/bin/rm -f %t/%n.ctr-id
ExecStart=/usr/bin/podman run --cidfile=%t/%n.ctr-id --cgroups=no-conmon –
➥ rm --sdnotify=conmon -d --replace -p 8080:8080 --name myapp 
➥ quay.io/rhatdan/myimage
ExecStop=/usr/bin/podman stop --ignore --cidfile=%t/%n.ctr-id
ExecStopPost=/usr/bin/podman rm -f --ignore --cidfile=%t/%n.ctr-idType=notify
NotifyAccess=all
[Install]
WantedBy=multi-user.target default.target
```

[]{#07.htm#pgfId-1110809}You can remove the container and the image from
your system, and when you tell `systemctl`{.fm-code-in-text} to start
the service, Podman will pull the image and create a new container. This
means the myapp-new.service unit file can be shared with a different
user, and when they run the service, Podman will likewise pull the image
and run the container on their systems, without them ever creating the
container in the first place. Table 7.5 shows the different commands
added to the unit file based on whether you used the
`--new`{.fm-code-in-text} flag[]{#07.htm#marker-1110810}.

[]{#07.htm#pgfId-1114037}Table 7.5 Differences between unit files

+-----------------+-----------------------------------------------------+
| [               | []{#07.htm#pgfId-1114043}Commands                   |
| ]{#07.htm#pgfId |                                                     |
| -1114041}Option |                                                     |
+-----------------+-----------------------------------------------------+
| []{#07.htm#pgf  | []{#07.htm#pg                                       |
| Id-1114045}With | fId-1114047}`ExecStart=/usr/bin/podman run ...--cid |
| `--new`         | file=%t/%n.ctr-id --cgroups=no-`{.fm-code-in-text1} |
| {.fm-code-in-te |                                                     |
| xt1}[]{#07.htm# | [➥]{.fm-code-continuation-arrow1}                   |
| marker-1114052} | `conmon --rm --sdnotify=conmon -                    |
|                 | d --replace -p 8080:8080 --name`{.fm-code-in-text1} |
|                 |                                                     |
|                 | [➥]{.fm-code-continuation-arrow1}                   |
|                 | `myapp quay.io/rhatdan/myimage`{.fm-code-in-text1}  |
|                 |                                                     |
|                 | `ExecStop=/usr/bin/podman stop                      |
|                 | --ignore --cidfile=%t/%n.ctr-id`{.fm-code-in-text1} |
|                 |                                                     |
|                 | `ExecStopPost=/usr/bin/podman                       |
|                 |  rm -f --ignore --cidfile=%t/%n`{.fm-code-in-text1} |
|                 |                                                     |
|                 | [➥]{.fm-code-continuation-arrow1}                   |
|                 | `.ctr-idType=notify`{.fm-code-in-text1}             |
+-----------------+-----------------------------------------------------+
| []              | []{#07.htm#pgfId-1114051}`ExecStart=/usr            |
| {#07.htm#pgfId- | /bin/podman start 8879112805...`{.fm-code-in-text1} |
| 1114049}Without |                                                     |
| `--new`{.fm     | `ExecStop=/usr/bin/                                 |
| -code-in-text1} | podman stop -t 10 8879112805...`{.fm-code-in-text1} |
|                 |                                                     |
|                 | `ExecStopPost=/usr/bin/                             |
|                 | podman stop -t 10 8879112805...`{.fm-code-in-text1} |
+-----------------+-----------------------------------------------------+

[]{#07.htm#pgfId-1110837}Once you have your containerized service
running on many machines, you need to think about maintaining it. Podman
has a way to do this without human intervention:
[]{#07.htm#marker-1110838}[]{#07.htm#marker-1110839}[]{#07.htm#marker-1110840}auto-update.

### 7.3.4 Automatically updating Podman containers

[]{#07.htm#pgfId-1110846}In
[]{#07.htm#marker-1110843}[]{#07.htm#marker-1110844}[]{#07.htm#marker-1110845}chapter
2, we talked about container images aging like stinky cheese. When the
container image gets updated with new software or vulnerability fixes,
you need to reach out to these machines, pull the updated images, and
re-create the containerized services. It is much less labor intensive
when machines manage their own updates.

[]{#07.htm#pgfId-1110847}Imagine you configure a service to run on a
container image on hundreds of nodes. A few months later, you add new
features to the application in the image or, more importantly, a new CVE
is found. Now you need to update the image and then recreate the service
on all of the nodes.

[]{#07.htm#pgfId-1110848}Podman automates this process with auto-update;
each node watches for new images to appear in a container registry. When
the image shows up, the node pulls down the image and re-creates the
container. No human interaction is involved.

[]{#07.htm#pgfId-1110849}Podman auto-update enables you to use Podman in
edge use cases, update workloads once they are connected to the network,
and roll back failures to a known good state. In addition, running
containers is essential for implementing edge computing in remote data
centers or on internet-of-things (IoT) devices. Auto-updates enable you
to use Podman in edge use cases, update workloads once they are
connected to the network, and reduce maintenance costs.

[]{#07.htm#pgfId-1110850}To implement this behavior, Podman requires
containers to have a special label, `--label`{.fm-code-in-text}
`"io.containers.autoupdate=registry"`{.fm-code-in-text}, and the
container must be run in a systemd unit generated by
`podman`{.fm-code-in-text} `generate`{.fm-code-in-text}
`systemd`{.fm-code-in-text} `--new`{.fm-code-in-text}. Table 7.6
describes the auto-update modes available.

[]{#07.htm#pgfId-1115092}Table 7.6 Auto-update modes

+-----------------+-----------------------------------------------------+
| []{#07.ht       | []{#07.htm#pgfId-1115098}Description                |
| m#pgfId-1115096 |                                                     |
| }`io.containers |                                                     |
| .autoupdate`{.f |                                                     |
| m-code-in-text} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#07.ht       | []{#07.htm#pgfId-1115102}Podman connects to the     |
| m#pgfId-1115100 | container registry and checks if a different image  |
| }`registry`{.fm | than the one used to create the container is        |
| -code-in-text1} | available; if there is one, Podman will update the  |
|                 | container.                                          |
+-----------------+-----------------------------------------------------+
| []{#07          | []{#07.htm#pgfId-1115106}Podman connects to the     |
| .htm#pgfId-1115 | container registry but compares local images to the |
| 104}`local`{.fm | one the container was created with; if they are     |
| -code-in-text1} | different, Podman updates the container.            |
+-----------------+-----------------------------------------------------+

[]{#07.htm#pgfId-1110868}First, stop the systemd service if it is
running, and remove the existing `myapp`{.fm-code-in-text}
container[]{#07.htm#marker-1110867}:

``` programlisting
$ systemctl --user stop myapp-new
$ podman rm myapp --force -t 0
```

[]{#07.htm#pgfId-1110871}Re-create the myapp container with the special
label `"io.containers.autoupdate=registry"`{.fm-code-in-text}:

``` programlisting
$ podman create --label "io.containers.autoupdate=registry" -p 8080:8080 
➥ --name myapp quay.io/rhatdan/myimage 
397ad15601868eb6fd77fe0b67136869cde9e0ffad90ee5095a19de5bb4b999e
```

[]{#07.htm#pgfId-1110875}Re-create the systemd unit file with the
`--new`{.fm-code-in-text} option:

``` programlisting
$ podman generate systemd myapp --new > $HOME/.config/systemd/user/
➥ myapp-new.service
```

[]{#07.htm#pgfId-1110878}Tell systemd the unit file changed by executing
`daemon-reload`{.fm-code-in-text}, and start the service:

``` programlisting
$ systemctl --user daemon-reload
$ systemctl --user start myapp-new
```

[]{#07.htm#pgfId-1110882}The `myapp-new`{.fm-code-in-text}
service[]{#07.htm#marker-1110881} is now ready to be automatically
updated. When you execute the `podman`{.fm-code-in-text}
`auto-update`{.fm-code-in-text} command[]{#07.htm#marker-1110883},
Podman examines running containers for the
`io.containers.autoupdate`{.fm-code-in-text}
label[]{#07.htm#marker-1110884} set to `image`{.fm-code-in-text}. For
each container with that label, Podman reaches out to the container
registry and checks if the image has changed since the container was
created. If the image has changed, Podman restarts the corresponding
systemd unit. Recall that on a systemd restart, the following steps
happen:

1.  []{#07.htm#pgfId-1110886 .calibre17}Systemd stops the service by
    executing the `podman`{.fm-code-in-text} `stop`{.fm-code-in-text}
    command[]{#07.htm#marker-1110885 .calibre17}:

    ``` programlisting
    ExecStop=/usr/bin/podman stop --ignore --cidfile=%t/%n.ctr-id
    ```

2.  []{#07.htm#pgfId-1110889 .calibre17}Systemd executes the
    `ExecStopPost`{.fm-code-in-text} script[]{#07.htm#marker-1110888
    .calibre17}. Once the container stops, this script removes the
    container with `podman`{.fm-code-in-text} `rm`{.fm-code-in-text}:

    ``` programlisting
    ExecStopPost=/usr/bin/podman rm -f --ignore --cidfile=%t/
    ➥ %n.ctr-idType=notify
    ```

3.  []{#07.htm#pgfId-1110893 .calibre17}Systemd restarts the services
    with the `podman`{.fm-code-in-text} `run`{.fm-code-in-text}
    command[]{#07.htm#marker-1110892 .calibre17}, including the
    `--label`{.fm-code-in-text}
    `"io.containers.autoupdate=registry"`{.fm-code-in-text} option:

    ``` programlisting
    ExecStart=/usr/bin/podman run --cidfile=%t/%n.ctr-id --cgroups=no-conmon --rm 
    ➥ --sdnotify=conmon -d --replace --label 
    ➥ io.containers.autoupdate=registry -p 8080:8080 
    ➥ --name myapp quay.io/rhatdan/myimage
    ```

[]{#07.htm#pgfId-1110899}The `podman`{.fm-code-in-text}
`run`{.fm-code-in-text} command[]{#07.htm#marker-1110898} in the third
step will reach out to the registry and pull down the updated container
image and re-create the containerized application on it. The container,
its environment, and all dependencies are restarted.

[]{#07.htm#pgfId-1110900}You can test this by changing your image,
pushing it to a registry, and then running the
`podman`{.fm-code-in-text} `auto-update`{.fm-code-in-text}
command[]{#07.htm#marker-1110901} as follows:

``` programlisting
$ podman exec -i myapp bash -c 'cat > /var/www/html/index.html' << _EOF
<html>
 <head>
 </head>
 <body>
  <h1>Welcome to the new Hello World<h1>
 </body>
</html>
_EOF
```

[]{#07.htm#pgfId-1110911}Now commit the image as
`myimage-new`{.fm-code-in-text}, and push it to the registry with the
original name: `myimage`{.fm-code-in-text}. Finally, remove the image
from the local store to simulate that the image was never on your
system:

``` programlisting
$ podman commit myapp quay.io/rhatdan/myimage-new
...
226ec055eef82ac185c53a26de9e98da4e6403640e72c7461a711edcbcaa2422
$ podman push quay.io/rhatdan/myimage-new quay.io/rhatdan/myimage
...
$ podman rmi quay.io/rhatdan/myimage-new
```

[]{#07.htm#pgfId-1110918}Once the new image is at the registry, and you
have removed it from local storage, you can run
`podman`{.fm-code-in-text} `auto-update`{.fm-code-in-text}, which
notices the new image and restarts the service. This triggers Podman to
pull the new image and re-create the containerized service:

``` programlisting
$ podman auto-update
Trying to pull quay.io/rhatdan/myimage...
Getting image source signatures
Copying blob ecfb9899f4ce done   
Copying config 37e5619f4a done   
Writing manifest to image destination
Storing signatures
UNIT             CONTAINER           IMAGE                
➥ POLICY    UPDATED
myapp-new.service  c8888d1319c4 (myapp)  quay.io/rhatdan/myimage  registry
➥ true
```

[]{#07.htm#pgfId-1110930}Your application has been updated to the latest
version of the image.

[]{#07.htm#pgfId-1110932}Some notable `podman`{.fm-code-in-text}
`auto-update`{.fm-code-in-text} options[]{#07.htm#marker-1110931}
include the following:

-   []{#07.htm#pgfId-1110934
    .calibre17}`--dry-run`{.fm-code-in-text}[]{#07.htm#marker-1110933
    .calibre17}---This option is useful to see if any containers need to
    be updated, without actually updating them.

-   []{#07.htm#pgfId-1110936
    .calibre17}`--roll-back`{.fm-code-in-text}[]{#07.htm#marker-1110935
    .calibre17}---This option tells Podman to roll back to the previous
    image if the update fails, as covered in the next section.

[]{#07.htm#pgfId-1110938}[]{#07.htm#id_9a7jioh1skx9}systemd timers
trigger Podman updates

[]{#07.htm#pgfId-1110940}Podman []{#07.htm#marker-1110939}ships with two
auto-update systemd timer units and two auto-update service units---one
each for rootful containers and rootless containers. The timer units
triggered by systemd once per day are the following:

-   []{#07.htm#pgfId-1110941
    .calibre17}/usr/lib/systemd/system/podman-auto-update.timer

-   []{#07.htm#pgfId-1110942
    .calibre17}/usr/lib/systemd/user/podman-auto-update.timer

[]{#07.htm#pgfId-1110943}The timer units tell systemd to execute the
appropriate auto-update service unit file:

-   []{#07.htm#pgfId-1110944
    .calibre17}/usr/lib/systemd/system/podman-auto-update.service

-   []{#07.htm#pgfId-1110945
    .calibre17}/usr/lib/systemd/user/podman-auto-update.service

[]{#07.htm#pgfId-1110946}With this feature, systemd will launch Podman,
which looks for containers with the
`"io.containers.autoupdate=registry"`{.fm-code-in-text} label, like you
created last section. Once Podman finds a container with the label, it
checks if the container's image has been updated on the registry. If the
image has changed, Podman starts the update process. This means you can
run your systems unattended, and they are updated within 24 hours with
the newest version of the container image every time you push an updated
image to a registry. If you share the unit file you generated with
others, then they also get the auto-updates.

[]{#07.htm#pgfId-1110947}A big concern with auto-update is what happens
if the update is broken. In that case, you will have hundreds of nodes
updated to a broken service. Systemd has a feature called
`sd-notify`{.fm-code-in-text}[]{#07.htm#marker-1117747}, which allows a
service to say its initialization is complete and it is ready to be used
[]{#07.htm#marker-1117748}as
[]{#07.htm#marker-1117749}[]{#07.htm#marker-1117750}[]{#07.htm#marker-1117751}a
[]{#07.htm#marker-1117752}[]{#07.htm#marker-1117753}service.

[]{#07.htm#pgfId-1110955}[Note]{.fm-callout-head} Some of this section
is based on previously written blogs copied and rewritten from the "How
to use auto-updates and rollbacks in Podman" blog
([http://mng.bz/neDK](http://mng.bz/neDK){.url}), written by myself and
coworkers Valentin Rothberg and Preethi Thomas.

## 7.4 Running containers in notify unit files 

[]{#07.htm#pgfId-1110961}Unit
[]{#07.htm#marker-1110958}[]{#07.htm#marker-1110959}[]{#07.htm#marker-1110960}file
services can specify that they wait to start until other services are up
and running. For example, you can have a website that relies on a
database to be running before the web service accepts connections.
Systemd usually considers a service started after it launches the
primary process of the service. However, many services take time to
initialize and can't accept connections right away. The database in the
previous example might take minutes before it is ready for the web
service to start receiving connections.

[]{#07.htm#pgfId-1110964}Systemd defines a special service type called
`notify`{.fm-code-in-text}[]{#07.htm#marker-1110962} (or
`sd-notify`{.fm-code-in-text}[]{#07.htm#marker-1110963}) that allows the
service process to notify systemd when it is actually fully up and
running. Systemd starts the web service only when systemd is notified
that the database is ready.

[]{#07.htm#pgfId-1110965}Systemd tells a service that it needs to be
notified that the service is ready by passing the
`NOTIFY_SOCKET`{.fm-code-in-text} environment
variable[]{#07.htm#marker-1110966} pointing to the systemd socket to be
notified. By default, systemd listens on the /run/SystemD/notify socket.
When Podman executes within a `NOTIFY`{.fm-code-in-text} unit file, it
needs to volume mount the socket into the container and pass down the
environment variable into the container (figure 7.7).

::: figure
![](images/OEBPS/Images/07-07.png){.calibre18}

[]{#07.htm#pgfId-1119214}Figure 7.7 Containerized
`sd_notify`{.fm-code-in-text} systemd service launched by Podman
:::

[]{#07.htm#pgfId-1110974}If the service does not notify systemd within
the specified time, systemd will mark the service as failed. Podman
auto-update checks if the new service is fully up and running, and if
the check fails, Podman can automatically roll back to the previous
container---again, without human
[]{#07.htm#marker-1110975}[]{#07.htm#marker-1110976}[]{#07.htm#marker-1110977}intervention.

## 7.5 Rolling back failed containers after update

[]{#07.htm#marker-1110994}[]{#07.htm#pgfId-1110983}If
[]{#07.htm#marker-1110979}[]{#07.htm#marker-1110980}[]{#07.htm#marker-1110981}[]{#07.htm#marker-1110982}your
defined service supports `sd-notify`{.fm-code-in-text} and writes to the
notify socket within the time limit, the `podman`{.fm-code-in-text}
`auto-update`{.fm-code-in-text} command[]{#07.htm#marker-1110984} will
succeed. However, if it fails, Podman will remove the new container and
retag the original image. Finally, it will create the container on the
previous image, and your service will come back up in the previous
state. You could even set up your system-based containerized service to
notify your logging system that the update failed. The rollback gives
you time to figure out what went wrong and ship a new image, triggering
the auto-update again. As you can see, systemd can be used as the
container orchestrator of a single system.

[]{#07.htm#pgfId-1110985} You have now discovered a few nice features
systemd provides for running containers without human intervention. One
additional feature Podman can take advantage of is socket activation,
which allows you to specify a container within a unit file that will not
be running until the first packet comes to its
[]{#07.htm#marker-1118701}[]{#07.htm#marker-1118702}[]{#07.htm#marker-1118703}[]{#07.htm#marker-1118704}socket.

## 7.6 Socket-activated Podman containers

[]{#07.htm#pgfId-1110995}When
[]{#07.htm#marker-1118707}[]{#07.htm#marker-1118708}[]{#07.htm#marker-1118709}systemd
was first introduced, it was lauded for speeding up the boot of a
system. Before systemd, each service started sequentially, and services
that relied on different services to be run needed to wait. To speed up
the boot and get better with resource allocation, systemd uses
*socket-activated services*[]{#07.htm#marker-1118711}. When you set up a
socket-activated service, systemd sets up listening IP or UNIX domain
sockets on behalf of your service, without starting the service (figure
7.8).

::: figure
![](images/OEBPS/Images/07-08.png){.calibre18}

[]{#07.htm#pgfId-1119252}Figure 7.8 Systemd listening on a socket for a
socket-activated container
:::

[]{#07.htm#pgfId-1111003}When a connection to the socket arrives,
systemd activates the service and hands the connection to it.
Afterwards, the service handles connections. The service can at some
point in the future idle itself by exiting. If a new connection comes
in, systemd accepts the new connection and starts the service again.

[]{#07.htm#pgfId-1111004}Socket activation allows systemd to indicate
that a service started instantly, without actually starting or waiting
for the service to start, speeding up the boot process. Socket
activation allows systemd to run more services on the system, since many
services are idle and not using system resources. Basically, your
services can be stopped and only run when they are actually needed and
not sit idle, waiting for another connection. With containerized
services, the main process of the service is Podman, and it needs to
pass the connection down to the service running within the container
(figure 7.9).

::: figure
![](images/OEBPS/Images/07-09.png){.calibre18}

[]{#07.htm#pgfId-1119290}Figure 7.9 When a connection to the socket
systemd is listening on arrives, systemd activates Podman, which
launches the container, passing the socket down to the container.
:::

[]{#07.htm#pgfId-1111011}Shut down the myapp.service, and create the
myapp.socket:

``` programlisting
$ systemctl --user stop myapp.service
$ cat > $HOME/.config/systemd/user/myapp.socket <<_EOF
[Unit]
Description=myapp socket service
PartOf=myapp.service
[Socket]
ListenStream=127.0.0.1:8080
[Install]
WantedBy=sockets.target
_EOF
```

[]{#07.htm#pgfId-1111022}Now, enable the socket, and make sure no
containers are running:

``` programlisting
$ systemctl --user enable --now myapp.socket
$ podman ps
CONTAINER ID  IMAGE     COMMAND   CREATED   STATUS    
➥ PORTS     NAMES
```

[]{#07.htm#pgfId-1111027}Connect a web browser to the socket (see figure
7.10):

``` programlisting
$ web-browser localhost:8080
```

::: figure
![](images/OEBPS/Images/07-10.png){.calibre18}

[]{#07.htm#pgfId-1119328}Figure 7.10 A web browser window connecting to
the `ubi8/httpd-24`{.fm-code-in-text} container running in Podman with
updated Hello World HTML.
:::

[]{#07.htm#pgfId-1111035}Notice that podman.socket started the
podman.service, which created a container to handle the connection:

``` programlisting
$ podman ps
CONTAINER ID  IMAGE                         COMMAND             CREATED
➥ STATUS          PORTS                 NAMES
69c34949d632  quay.io/rhatdan/myimage:latest  /usr/bin/run-http...  
➥ 2 minutes ago  Up 2 minutes ago  0.0.0.0:8080->8080/tcp  myapp
```

[]{#07.htm#pgfId-1111041}Now if you stop the service, not only will the
container be stopped, but it will be removed:

``` programlisting
$ systemctl --user stop myapp.service
$ podman ps -a
CONTAINER ID  IMAGE     COMMAND   CREATED   STATUS    
➥ PORTS     NAMES
```

[]{#07.htm#pgfId-1111046}Socket activation allows you to run the service
only when needed, saving system resources. Later, you can take the
service down, knowing that if a new connection comes in, systemd and
Podman will handle
[]{#07.htm#marker-1111047}[]{#07.htm#marker-1111048}[]{#07.htm#marker-1111049}it.

## Summary

-   []{#07.htm#pgfId-1111052 .calibre17}Podman enables running systemd
    as the primary process within a container.

-   []{#07.htm#pgfId-1111053 .calibre17}Journald is recommended for
    Podman logs and events.

-   []{#07.htm#pgfId-1111054 .calibre17}Systemd can be used to start and
    restart containers at boot time.

-   []{#07.htm#pgfId-1111055 .calibre17}Podman auto-update is used to
    manage the life cycle of a container and its image.

-   []{#07.htm#pgfId-1111056 .calibre17}Socket-activated systemd
    services can be used with Podman-based containers.

-   []{#07.htm#pgfId-1111057 .calibre17}The `podman`{.fm-code-in-text}
    `generate`{.fm-code-in-text} `systemd`{.fm-code-in-text} command
    makes it easy to generate systemd service files for running your
    []{#07.htm#marker-1111058 .calibre17}containers.

[]{#08.htm}
