# 11 Additional security considerations

[]{#11.htm#pgfId-1121654}This chapter []{#11.htm#marker-1124735}covers

-   []{#11.htm#pgfId-1121655 .calibre17}Securing running applications on
    different standalone servers, inside different VMs and containers
-   []{#11.htm#pgfId-1121656 .calibre17}Running a container via a
    service versus as a child of the container engine via fork and exec
-   []{#11.htm#pgfId-1121657 .calibre17}Linux security features used to
    keep containers isolated from each other
-   []{#11.htm#pgfId-1121658 .calibre17}Setting up container image trust
-   []{#11.htm#pgfId-1121659 .calibre17}Signing images and trusting them

[]{#11.htm#pgfId-1121660}In this chapter, I review and demonstrate some
additional security considerations when using Podman to run containers.
Some of the content was covered in other chapters, but I think it is
useful to concentrate on these features from a security perspective.

[]{#11.htm#pgfId-1121661}One of the most frequent problems I see with
people running containers is that when the container process is denied
some access, the user's first reaction is to run the container in
`--privileged`{.fm-code-in-text} mode, which turns off all security
separation for your container. Understanding how to deal with the
security features discussed in this chapter helps you avoid this.

## []{#11.htm#pgfId-1121663}11.1 Daemon versus the fork/exec model {#11.htm#heading_id_3 .fm-head}

[]{#11.htm#pgfId-1121667}Throughout
[]{#11.htm#marker-1121664}[]{#11.htm#marker-1121665}[]{#11.htm#marker-1121666}the
previous chapters, you have learned quite a bit about the problems of a
daemon like Docker versus the fork/exec model employed by Podman.

### []{#11.htm#pgfId-1121669}11.1.1 Access to the docker.sock {#11.htm#heading_id_4 .fm-head1}

[]{#11.htm#pgfId-1121674}Recall
[]{#11.htm#marker-1121670}[]{#11.htm#marker-1121671}[]{#11.htm#marker-1121672}[]{#11.htm#marker-1121673}that
Docker, by default, runs a daemon owned by the root user. This means any
user who has access to the daemon can launch processes with full root
access on the system. Docker recommends some users put their accounts
into the docker group in the /etc/group. On some distributions, this
allows you to access the /run/docker.sock without being root:

``` programlisting
# ls -l /run/docker.sock
srw-rw----. 1 root docker 0 Jun 13 14:54 /run/docker.sock
```

[]{#11.htm#pgfId-1121677}You can run a Docker container similarly to how
you have been running a Podman container:

``` programlisting
$ docker run registry.access.redhat.com/ubi8-micro echo hi
Unable to find image 'registry.access.redhat.com/ubi8-micro:latest' locally|
latest: Pulling from ubi8-micro
4f4fb700ef54: Pull complete
b6d5e0581b2f: Pull complete
Digest: sha256:a519ab06c0287085c352af0d2b84f2a2b257d2afb2e554b8d38a076cd6205b48
Status: Downloaded newer image for registry.access.redhat.com/
ubi8-micro:latest
hi
```

[]{#11.htm#pgfId-1121688}This excites a lot of users, until they
understand they can also launch a root shell on their system with a
simple Docker command:

``` programlisting
$ docker run -ti --name hack -v /:/host --privileged 
➥ registry.access.redhat.com/ubi8-micro chroot /host
# cat /etc/shadow
...
```

[]{#11.htm#pgfId-1121693}At this point, you have a fully privileged root
shell on the host system, in which you can hack the machine all you
want. Not only that, but Docker defaults all logging to being file
based. When you are done hacking the system, you can remove the log
files and all records of your activity:

``` programlisting
$ docker rm hack
hack
```

[]{#11.htm#pgfId-1121697}Using rootless Podman, you cannot do this,
since when you run the container, the container processes are run as
your user UID and only have access to the same files as any process in
your account. One way administrators figure out if they have been hacked
is by examining the logging system, including the audit
[]{#11.htm#marker-1121698}[]{#11.htm#marker-1121699}[]{#11.htm#marker-1121700}[]{#11.htm#marker-1121701}logs.

### []{#11.htm#pgfId-1121703}11.1.2 Auditing and logging {#11.htm#heading_id_5 .fm-head1}

[]{#11.htm#pgfId-1121710}One
[]{#11.htm#marker-1121704}[]{#11.htm#marker-1121705}[]{#11.htm#marker-1121706}[]{#11.htm#marker-1121707}[]{#11.htm#marker-1121708}key
feature of a Linux system is tracking what processes do when they are
running on a system. When you log in to a Linux system, your UID is
recorded by the kernel into the process data in /proc/self/loginuid. You
can see this data by executing the following command:

``` programlisting
$ cat /proc/self/loginuid
3267
```

[]{#11.htm#pgfId-1121713}All processes created by this first process
after login maintain this field. Even if you use a
`setuid`{.fm-code-in-text} program, like `su`{.fm-code-in-text} or
`sudo`{.fm-code-in-text}, your `loginuid`{.fm-code-in-text} stays the
same:

``` programlisting
$ sudo cat /proc/self/loginuid
3267
```

[]{#11.htm#pgfId-1121716}Even when you launch a container, the
`loginuid`{.fm-code-in-text} stays the same. In this next example, you
run a simple container in daemon mode that sleeps, then use
`podman`{.fm-code-in-text} `inspect`{.fm-code-in-text} to get the PID of
the sleep processes, and finally examine the
`loginuid`{.fm-code-in-text} of the containerized process:

``` programlisting
$ podman run -d ubi8-micro sleep 20
1c55b9cfa0cd20c36da4b606415e190a6c20cc868d3486981c7713d41ee9ea6a
$ podman inspect -l --format '{{ .State.Pid }}'
119394
$ cat /proc/119394/loginuid
3267
```

[]{#11.htm#pgfId-1121723}Notice the containerized process is still
running with your `loginuid`{.fm-code-in-text}. This shows that the
kernel can track which user launched a container process on the system
via this field, as long as the container engine uses the fork/exec
model. If you run this same test with Docker, you get very different
results:

``` programlisting
$ docker run -d registry.access.redhat.com/ubi8-micro sleep 20
df2302cf8c6385df2b86ccd3429166e0d8dd0c9f0d0139e98e6354809a04080e
$ docker inspect df2302cf8c6 --format '{{ .State.Pid }}'
120022
$ cat /proc/120022/loginuid
4294967295
```

[]{#11.htm#pgfId-1121730}Instead of showing your
`loginuid`{.fm-code-in-text}, you see `4294967295`{.fm-code-in-text},
which is 2^32^ -- 1. This is how the Linux kernel represents
`-1`{.fm-code-in-text}, the default `loginuid`{.fm-code-in-text} for all
processes started by the system, not by users who logged into the
system. The reason for this is that Docker uses a client-server model,
and the container process is a child of the Docker daemon as opposed to
the Docker client. Since the Docker daemon was launched by systemd when
the system booted up, all of its children processes have the
`-1`{.fm-code-in-text} `loginuid`{.fm-code-in-text}.

[]{#11.htm#pgfId-1121731}The kernel's audit subsystem records the
`loginuid`{.fm-code-in-text} of every process on the system when it
completes an auditable event. For example, when a user logs in and out
of a system, these events are logged. Modifying /etc/passwd and
/etc/shadow are also loggable events.

[]{#11.htm#pgfId-1121733}Following is the `USER_START`{.fm-code-in-text}
audit log entry[]{#11.htm#marker-1121732} for when I logged into my
system today. My UID `3267`{.fm-code-in-text} is recorded along with my
username:

``` programlisting
```bash
# ausearch -m USER_START
type=USER_START msg=audit(1651064687.963:315): pid=2579 uid=0 auid=3267 
➥ ses=3 subj=system_u:system_r:xdm_t:s0-s0:c0.c1023 msg='op=PAM:session_open 
➥ grantors=pam_selinux,pam_loginuid,pam_selinux,pam_keyinit,pam_namespace,
➥ pam_keyinit,pam_limits,pam_systemd,pam_unix,pam_gnome_keyring,pam_umask acct=
➥ "dwalsh" exe="/usr/libexec/gdm-session-worker" hostname=fedora addr=? 
➥ terminal=/dev/tty2 res=success'UID="root" AUID="dwalsh"
```

[]{#11.htm#pgfId-1121741}If you launched the container by using a Podman
command, then the audit subsystem records your UID in the audit logs. If
the container was launched via Docker, it records `-1`{.fm-code-in-text}
as the `loginuid`{.fm-code-in-text}. Imagine your system was hacked via
a container. You need to go back and examine which user launched the
container that hacked your system via the audit.log.

[]{#11.htm#pgfId-1121742}Let's show an example of this. First, become
root, and set up a watch on the /etc/passwd file using
`auditctl`{.fm-code-in-text}:

``` programlisting
# auditctl -w /etc/passwd -p wa -k passwd
```

[]{#11.htm#pgfId-1121745}Now run a `--privileged`{.fm-code-in-text}
container[]{#11.htm#marker-1121744} using Docker, which touches the
host's /etc/ passwd file:

``` programlisting
# docker run --privileged -v /:/host registry.access.redhat.com/ubi8-
➥ micro:latest touch /host/etc/passwd
```

[]{#11.htm#pgfId-1121748}This simulated what would happen if a Docker
container escaped confinement and was able to modify the host's
/etc/passwd file. Now examine the audit.log, where there should be a
record of the /etc/passwd modification. Notice that the audit log shows
`auid=unset`{.fm-code-in-text}. This is how the audit log represents the
`loginuid`{.fm-code-in-text} of the user that modified the /etc/passwd
file. As you can see, because no user launched the Docker daemon
directly, the audit log has no record of the user who launched the
container:

``` programlisting
# ausearch -k passwd -i
...
type=SYSCALL msg=audit(05/03/2022 08:24:52.885:464) : arch=x86_64 
➥ syscall=openat success=yes exit=3 a0=AT_FDCWD a1=0x7ffef7a9ef75 
➥ a2=O_WRONLY|O_CREAT|O_NOCTTY|O_NONBLOCK a3=0x1b6 items=2 ppid=6723 
➥ pid=6743 auid=unset uid=root gid=root euid=root suid=root fsuid=root 
➥ egid=root sgid=root fsgid=root tty=(none) ses=unset comm=touch 
➥ exe=/usr/bin/coreutils 
```

[]{#11.htm#pgfId-1121757}Now run the same command with Podman:

``` programlisting
# podman run --privileged -v /:/host registry.access.redhat.com/
➥ ubi8-micro:latest touch /host/etc/passwd
```

[]{#11.htm#pgfId-1121760}Examine the audit.log for the Podman container
that modifies the /etc/passwd file, and you see that
`auid=dwalsh`{.fm-code-in-text}. Because Podman follows the fork/exec
model and was launched by a user who logged into the system and had a
record in the `loginuid`{.fm-code-in-text}, the audit.log can record
which user launched a container that hacked the system:

``` programlisting
# ausearch -k passwd -i
...
type=SYSCALL msg=audit(05/03/2022 08:25:42.466:480) : arch=x86_64 
➥ syscall=openat success=no exit=EACCES(Permission denied) a0=AT_FDCWD 
➥ a1=0x7fff3d5aef59 a2=O_WRONLY|O_CREAT|O_NOCTTY|O_NONBLOCK a3=0x1b6 
➥ items=2 ppid=6978 pid=6986 auid=dwalsh uid=root gid=root euid=root 
➥ suid=root fsuid=root egid=root sgid=root fsgid=root tty=(none) ses=1 
➥ comm=touch exe=/usr/bin/coreutils 
➥ subj=system_u:system_r:container_t:s0:c484,c845 key=passwd
```

[]{#11.htm#pgfId-1121770}[Note]{.fm-callout-head} On current Fedora, the
audit subsystem is disabled. You can enable it by removing
`/etc/audit/rules.d/audit.rules`{.fm-code-in-text1} and regenerating the
audit rules with the `augenrules`{.fm-code-in-text1}
`--load`{.fm-code-in-text1} command[]{#11.htm#marker-1121771}.

[]{#11.htm#pgfId-1121772}This is one reason, back in 2014, I said access
to the docker.sock via non-root processes is more dangerous than giving
out the root process or sudo access, since both of those record the
`loginuid`{.fm-code-in-text}, meaning you can track what the user is
doing on your system. When you give access to the root running
docker.sock, you have no tracking data. Let's look into how you can
protect the kernel and the filesystem from processes running within a
container in the
[]{#11.htm#marker-1121773}[]{#11.htm#marker-1121774}[]{#11.htm#marker-1121775}[]{#11.htm#marker-1121776}[]{#11.htm#marker-1121777}next
[]{#11.htm#marker-1121778}[]{#11.htm#marker-1121779}[]{#11.htm#marker-1121780}section.

## []{#11.htm#pgfId-1121782}11.2 Podman secret handling {#11.htm#heading_id_6 .fm-head}

[]{#11.htm#pgfId-1121785}Often,
[]{#11.htm#marker-1121783}[]{#11.htm#marker-1121784}when running a
container, you need to provide a secret to the service running within
the container. An example of this is a database tool that requires an
administrator and password to control access. Another example is a
service that requires a password to reach another service.

[]{#11.htm#pgfId-1121786}Developers of these applications do not want to
hardcode the secret information into the image. The user of the
container application must provide the secret. You can just provide the
secret to the application via environment variables, but this means if
you commit the image, the secret gets committed to the image.

[]{#11.htm#pgfId-1121788}Podman provides a secret mechanism,
`podman`{.fm-code-in-text}
`secret`{.fm-code-in-text}[]{#11.htm#marker-1121787}, which allows you
to either add files or environment variables to a container without
these secrets getting saved when you commit the container to an image.
First, let's look at creating a secret.

[]{#11.htm#pgfId-1121789}Listing 11.1 Using secrets within a Podman
container

``` programlisting
$ echo "This is my secret" > /tmp/secret               ❶
$ podman secret create my_secret /tmp/secret           ❷
b5f27b90e9b3486fb5a78d1eb
$ podman run --rm --secret my_secret ubi8 cat 
/run/secrets/my_secret                                 ❸
This is my secret
```

[]{#11.htm#pgfId-1129575}[❶]{.fm-combinumeral} Add your secret data to a
file.

[]{#11.htm#pgfId-1129596}[❷]{.fm-combinumeral} Use podman secret create
to name a secret based on the file.

[]{#11.htm#pgfId-1129613}[❸]{.fm-combinumeral} Use the \--secret option
to leak the secret into the container.

[]{#11.htm#pgfId-1121799}You can also leak the secret into the container
as an environment variable by adding the `--secret`{.fm-code-in-text}
`my_secret,type=env`{.fm-code-in-text} flag[]{#11.htm#marker-1121800}:

``` programlisting
$ podman run --secret my_secret,type=env --name secret_ctr ubi8 bash 
➥ -c 'echo $my_secret'
This is my secret
```

[]{#11.htm#pgfId-1121804}If you were to commit this container to an
image, the secret would not be saved inside the image.

[]{#11.htm#pgfId-1121805}Listing 11.2 The secret is not saved when the
container is committed to the image.

``` programlisting
$ podman commit secret_ctr secret_img                       ❶
Getting image source signatures
Copying blob a9820c2af00a skipped: already exists 
Copying blob 3d5ecee9360e skipped: already exists 
Copying blob dc409efbefc4 done 
Copying config 501812299f done 
Writing manifest to image destination
Storing signatures
501812299f0c0cfbb032d144e6d2c2a41c5eadf229e7b76f6264ab74d9f6c069
$ podman image inspect secret_img --format 
➥ '{{ .Config.Env }}'                                      ❷
[TERM=xterm container=oci PATH=/usr/local/sbin:/usr/local/
➥ bin:/usr/sbin:/usr/bin:/sbin:/bin]
```

[]{#11.htm#pgfId-1129413}[❶]{.fm-combinumeral} Commit the secret_ctr
into the secret_img image.

[]{#11.htm#pgfId-1129434}[❷]{.fm-combinumeral} Inspect the image to view
the committed environment variables, and notice the my_secret
environment is not committed.

[]{#11.htm#pgfId-1121823}Table 11.1 lists all of the
`podman`{.fm-code-in-text} `secret`{.fm-code-in-text}
[]{#11.htm#marker-1121821}[]{#11.htm#marker-1121822}commands.

[]{#11.htm#pgfId-1125698}Table 11.1 `podman`{.fm-code-in-text}
`secret`{.fm-code-in-text} commands

+---------+------------------------+-----------------------------------+
| []{#    | []{#11                 | []{                               |
| 11.htm# | .htm#pgfId-1125706}Man | #11.htm#pgfId-1125708}Description |
| pgfId-1 | page                   |                                   |
| 125704} |                        |                                   |
| Command |                        |                                   |
+---------+------------------------+-----------------------------------+
| [       | []{                    | []{#11.htm#pgfId-1125714}Create a |
| ]{#11.h | #11.htm#pgfId-1125712} | new secret.                       |
| tm#pgfI | `podman-secret-create( |                                   |
| d-11257 | 1)`{.fm-code-in-text1} |                                   |
| 10}`cre |                        |                                   |
| ate`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#1 |                        |                                   |
| 1.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 125733} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []      | []{#                   | []{#11.htm#pgfId-1125720}Display  |
| {#11.ht | 11.htm#pgfId-1125718}` | detailed information on one or    |
| m#pgfId | podman-secret-inspect( | more secrets.                     |
| -112571 | 1)`{.fm-code-in-text1} |                                   |
| 6}`insp |                        |                                   |
| ect`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#1 |                        |                                   |
| 1.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 125734} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#    | []{#11.htm#pgfId-1125  | []{#11.htm#pgfId-1125726}List all |
| 11.htm# | 724}`podman-secret-ls( | available secrets.                |
| pgfId-1 | 1)`{.fm-code-in-text1} |                                   |
| 125722} |                        |                                   |
| `ls`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#1 |                        |                                   |
| 1.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 125735} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#    | []{#11.htm#pgfId-1125  | []{#11.htm#pgfId-1125732}Remove   |
| 11.htm# | 730}`podman-secret-rm( | one or more secrets.              |
| pgfId-1 | 1)`{.fm-code-in-text1} |                                   |
| 125728} |                        |                                   |
| `rm`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#1 |                        |                                   |
| 1.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 125736} |                        |                                   |
+---------+------------------------+-----------------------------------+

## []{#11.htm#pgfId-1121863}11.3 Podman image trust {#11.htm#heading_id_7 .fm-head}

[]{#11.htm#pgfId-1121866}In
[]{#11.htm#marker-1121864}[]{#11.htm#marker-1121865}many situations,
users of container images want to specify which container image
registries and images they trust. The `podman`{.fm-code-in-text}
`image`{.fm-code-in-text} `trust`{.fm-code-in-text}
command[]{#11.htm#marker-1121867} allows you to specify which registries
you trust. It also allows you to specify registries to block.

[]{#11.htm#pgfId-1121868}The location of the trusted registry is
determined by the transport and the registry host of the image. Using
this container image---docker://quay.io/podman/stable---as an example,
Docker is the transport, and quay.io is the registry host.

[]{#11.htm#pgfId-1121869}[Note]{.fm-callout-head} Podman image trust is
not available in remote mode, for example, on a Mac or Windows box. You
have to execute the commands documented here on a Linux box. If you are
using the Podman machine, use the `podman`{.fm-code-in-text1}
`machine`{.fm-code-in-text1} `ssh`{.fm-code-in-text1}
command[]{#11.htm#marker-1121870} to enter the VM. See appendixes E and
F for more information.

[]{#11.htm#pgfId-1121871}The trust policy is defined in
/etc/containers/policy.json, which describes a registry scope (registry
and/or repository) for the trust. The trust policy can use public keys
for signed images. The `podman`{.fm-code-in-text}
`image`{.fm-code-in-text} `trust`{.fm-code-in-text}
command[]{#11.htm#marker-1121872} must be run as root.

[]{#11.htm#pgfId-1121873}The scope of the trust is evaluated from the
most specific to the least specific. In other words, a policy may be
defined for an entire registry. Or it can be defined for a particular
repository in that registry or defined down to a specific signed image
inside the registry. In the following example, you reject pulls from
docker.io and then later specify only docker.io/library images are
allowed for pulling.

[]{#11.htm#pgfId-1121874}The following list includes valid scope values
that can be used in policy.json from most specific to the least
specific:

``` programlisting
docker.io/library/busybox:notlatest
docker.io/library/busybox
docker.io/library
docker.io
```

[]{#11.htm#pgfId-1121879}If no configuration is found for any of these
scopes, the default value (specified by using
`default`{.fm-code-in-text} instead of
`REGISTRY[/REPOSITORY]`{.fm-code-in-text}) is used, as shown in the
following listing. Table 11.2 describes the valid trust values used for
registries.

[]{#11.htm#pgfId-1121880}Listing 11.3 Telling Podman to not pull images
from a specific container registry

``` programlisting
$ sudo podman image trust set -t reject docker.io                     ❶
$ podman pull alpine                                                  ❷
Trying to pull docker.io/library/alpine:latest...
Error: Source image rejected: Running image docker://alpine:latest 
➥ is rejected by policy.
$ sudo podman image trust set -t accept 
➥ docker.io/library                                                  ❸
$ podman pull alpine                                                  ❹
Trying to pull docker.io/library/alpine:latest...
Getting image source signatures
Copying blob 59bf1c3509f3 skipped: already exists 
Copying config c059bfaa84 done 
Writing manifest to image destination
Storing signatures
C059bfaa849c4d8e4aecaeb3a10c2d9b3d85f5165c66ad3a4d937758128c4d18
$ podman pull bitnami/nginx                                           ❺
Resolving "bitnami/nginx" using unqualified-search registries 
➥ (/etc/containers/registries.conf.d/999-podman-machine.conf)
Trying to pull docker.io/bitnami/nginx:latest...
Error: Source image rejected: Running image docker://bitnami/nginx:latest 
➥ is rejected by policy.
```

[]{#11.htm#pgfId-1128975}[❶]{.fm-combinumeral} Use podman image trust to
reject all images from the docker.io container registry.

[]{#11.htm#pgfId-1129011}[❷]{.fm-combinumeral} Attempt to pull the
alpine image from the container registry, and see that Podman rejects
the image.

[]{#11.htm#pgfId-1129028}[❸]{.fm-combinumeral} Use Podman image trust to
set a more specific registry/repository for docker.io/library.

[]{#11.htm#pgfId-1129045}[❹]{.fm-combinumeral} Podman can pull the
docker.io/library/alpine image.

[]{#11.htm#pgfId-1128976}[❺]{.fm-combinumeral} Images pulled from the
rest of docker.io are rejected.

[]{#11.htm#pgfId-1125964}Table 11.2 The trust type tells container
engines like Podman which registries to trust.

+-----------------+-----------------------------------------------------+
| []{#11.htm#pgfI | []{#11.htm#pgfId-1125970}Description                |
| d-1125968}Types |                                                     |
+-----------------+-----------------------------------------------------+
| []{#11.         | []{#11.htm#pgfId-1125974}Images from the specified  |
| htm#pgfId-11259 | registry are allowed to be pulled.                  |
| 72}`accept`{.fm |                                                     |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#11.         | []{#11.htm#pgfId-1125978}Images from the specified  |
| htm#pgfId-11259 | registries are not allowed to be pulled.            |
| 76}`reject`{.fm |                                                     |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#11.         | []{#11.htm#pgfId-1125982}Images from the specified  |
| htm#pgfId-11259 | registries must be signed by the specified name.    |
| 80}`signBy`{.fm |                                                     |
| -code-in-text1} |                                                     |
+-----------------+-----------------------------------------------------+

[]{#11.htm#pgfId-1121928}If you examine the policy.json file, you see
the entries added by the `podman`{.fm-code-in-text}
`image`{.fm-code-in-text} `trust`{.fm-code-in-text} command:

``` programlisting
$ cat /etc/containers/policy.json
{
    "default": [
        {
            "type": "insecureAcceptAnything"
        }
    ],
    "transports": {
        "docker": {
            "docker.io": [
                {
                    "type": "reject"
                }
            ],
            "docker.io/library": [
                {
                    "type": "insecureAcceptAnything"
                }
            ]
 
...   
```

[]{#11.htm#pgfId-1121951}You can use the `podman`{.fm-code-in-text}
`image`{.fm-code-in-text} `trust`{.fm-code-in-text}
`show`{.fm-code-in-text} command[]{#11.htm#marker-1121950} to show the
current settings in an easier-to-view form:

``` programlisting
$ podman image trust show
all          default                     accept                         
repository   docker.io                   reject                         
repository   docker.io/library           accept                      
   
repository   registry.access.redhat.com  signed    security@redhat.com 
https://access.redhat.com/webassets/docker/content/sigstore
repository   registry.redhat.io          signed    
➥ security@redhat.com  https://registry.redhat.io/containers/sigstore
docker-daemon                            accept   
```

[]{#11.htm#pgfId-1121964}Through the
`accep`{.fm-code-in-text}t[]{#11.htm#marker-1121962} and
`reject`{.fm-code-in-text} flags,[]{#11.htm#marker-1121963} you can set
up which registries you trust and which you reject. If you want to lock
down where images on your production system come from, you can change
the `default`{.fm-code-in-text} policy[]{#11.htm#marker-1121965} for
your system to `reject`{.fm-code-in-text} images from any registry. All
images you want to allow need to come from a specific registry:

``` programlisting
$ sudo podman image trust set --type=reject default
$ podman image trust show
all          default                     reject                      
   
repository   docker.io                   reject                      
   
repository   docker.io/library           accept                      
   
repository   registry.access.redhat.com  signed    security@redhat.com 
https://access.redhat.com/webassets/docker/content/sigstore
repository   registry.redhat.io          signed    
➥ security@redhat.com  https://registry.redhat.io/containers/sigstore
docker-daemon                            accept   
```

[]{#11.htm#pgfId-1121979}With these settings on your system, Podman
accepts images from docker.io/library and signed images from
registry.redhat.io. All images from other registries are rejected.
Podman allows pulling of images directly from the
`docker-daemon`{.fm-code-in-text} as well.

[]{#11.htm#pgfId-1121980}Don't forget to restore the default
policy.json:

``` programlisting
$ sudo cp /tmp/policy.json /etc/containers/policy.json 
```

[]{#11.htm#pgfId-1121982}Podman supports using signed images from
container registries. Red Hat signs and ships its images. Let's look at
how you, too, can sign images.

### []{#11.htm#pgfId-1121984}11.3.1 Podman image signing {#11.htm#heading_id_8 .fm-head1}

[]{#11.htm#pgfId-1121986}One []{#11.htm#marker-1121985}way of signing
images is utilizing a GNU Privacy Guard
([https://gnupg.org](https://gnupg.org){.url}) key. Podman can sign
images before pushing them to remote registries, referred to as *simple
signing*. You can configure Podman and other container engines to
require images to be signed with a particular signature. All unsigned
images are rejected.

[]{#11.htm#pgfId-1121987}First, you need to create a GPG key pair or
select a premade pair. You can generate new GPG keys by running
`gpg`{.fm-code-in-text} `--full-gen-key`{.fm-code-in-text} and following
the interactive dialog. Refer to the following web page for a
description of creating keys:
[http://mng.bz/JV9V](http://mng.bz/JV9V){.url}.

[]{#11.htm#pgfId-1121988}Following is an example of creating a simple
key with default params. Make sure to use your own email address:

``` programlisting
$ gpg --batch --passphrase '' --quick-gen-key dwalsh@redhat.com default 
➥ default
```

[]{#11.htm#pgfId-1121991}Most container registries do not understand
image signing; they just provide the remote storage for the container
images. If you want to sign an image, you need to distribute the
signatures yourself, usually using a web server. You can configure
Podman and other container engines to retrieve signatures from this web
service.

[]{#11.htm#pgfId-1121992}In the following examples, you will create a
web service running on your local machine to demonstrate image signing.
Podman is able to push and sign the image in a single command. Podman
reads signature locations in the registries configuration file
/etc/containers/registries.d/default.yaml.

[]{#11.htm#pgfId-1121994}Examine the default.yaml file to find the
`sigstore-staging`{.fm-code-in-text} flag[]{#11.htm#marker-1121993} and
see the default location where Podman stores signatures:

``` programlisting
sigstore-staging: file:///var/lib/containers/sigstore
```

[]{#11.htm#pgfId-1121997}The `sigstore-staging`{.fm-code-in-text}
flag[]{#11.htm#marker-1121996} tells Podman to store signatures in the
/var/lib/containers/sigstore directory. When you want other users to use
these signatures to verify your images, you need to put these images
onto a web server. Now you are ready to test out simple signing, first
signing the ubi8 image and then setting up Podman to pull the image
using the signature to verify it.

[]{#11.htm#pgfId-1121999}Signing and pushing the image

[]{#11.htm#pgfId-1122003}Before
[]{#11.htm#marker-1122000}[]{#11.htm#marker-1122001}[]{#11.htm#marker-1122002}starting
this section, you should back up a couple of security files, so you can
restore them later:

``` programlisting
$ sudo cp /etc/containers/registries.d/default.yaml 
➥ /etc/containers/policy.json /tmp 
```

[]{#11.htm#pgfId-1122006}Let's pull an image from a registry and add a
signature, then push it back to the registry. Make sure to use your own
registry account, image, and previously created GPG key:

``` programlisting
$ sudo podman pull quay.io/rhatdan/myimage
Trying to pull quay.io/rhatdan/myimage:latest...
...
2c7e43d880382561ebae3fa06c7a1442d0da2912786d09ea9baaef87f73c29ae
$ podman login quay.io/rhatdan
Username: rhatdan
Password:
Login Succeeded!
$ sudo -E GNUPGHOME=$HOME/.gnupg \
    podman push --tls-verify=false --sign-by dwalsh@redhat.com 
➥ quay.io/rhatdan/myimage
...
Storing signatures
```

[]{#11.htm#pgfId-1122020}Look in the sigstore-staging directory
/var/lib/containers/sigstore for the repository name rhatdan. You will
see that there is a new signature available, created by the
`podman`{.fm-code-in-text} `push`{.fm-code-in-text}
command[]{#11.htm#marker-1122021}. Make sure to use your own registry
account name:

``` programlisting
$ sudo ls /var/lib/containers/sigstore/rhatdan/
'myimage@sha256=0460a9d13a806e124639b23e9d6ffa1e5773f7bef91469bee6ac88
➥ a4be213427'
```

[]{#11.htm#pgfId-1122025}Now that you have signed the image, you need to
set up a web server to provide the signature and configure Podman and
other container engines to use the signatures and signed
[]{#11.htm#marker-1122026}[]{#11.htm#marker-1122027}[]{#11.htm#marker-1122028}images.

[]{#11.htm#pgfId-1122030}Configuring Podman to pull signed images

[]{#11.htm#pgfId-1122033}When
[]{#11.htm#marker-1122031}[]{#11.htm#marker-1122032}configuring Podman
to use signatures to verify images, you need to configure the system to
retrieve the signatures. Usually, you share signatures on a web service.
You can do this by configuring the `sigstore`{.fm-code-in-text}
flag[]{#11.htm#marker-1122034} in the
/etc/containers/registries.d/default.yaml file to identify the website
that stores signatures. Podman downloads these signatures from this
website.

[]{#11.htm#pgfId-1122035}For this example, you will create a web service
running on localhost at port `8000`{.fm-code-in-text}. Add the
`sigstore:`{.fm-code-in-text} `http://localhost:8000`{.fm-code-in-text}
web server to the default.yaml file. This will tell Podman to retrieve
signatures from this web server when pulling images. Podman looks for a
signature based on the name of the image along with its digest:

``` programlisting
$ echo "  sigstore: http://localhost:8000" | sudo tee --append 
➥ /etc/containers/registries.d/default.yaml
```

[]{#11.htm#pgfId-1122038}For this example, start a new server using
`python3`{.fm-code-in-text} inside the local staging signature store
/var/lib/containers/sigstore:

``` programlisting
$ cd /var/lib/containers/sigstore && python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

[]{#11.htm#pgfId-1122041}In another window, remove
quay.io/rhatdan/myimage from local storage, since you want to pull with
the signatures:

``` programlisting
$ podman rmi quay.io/rhatdan/myimage
Untagged: quay.io/rhatdan/myimage:latest
Deleted: 2c7e43d880382561ebae3fa06c7a1442d0da2912786d09ea9baaef87f73c29ae
```

[]{#11.htm#pgfId-1122045}You need to set up image trust for the
quay.io/rhatdan repository and assign the publickey.gpg public key to
use when verifying images signed by dwalsh@redhat.com:

``` programlisting
$ sudo podman image trust set -f /tmp/publickey.gpg quay.io/rhatdan
```

[]{#11.htm#pgfId-1122047}The previous Podman command adds the following
stanza to the /etc/containers/ policy.json file:

``` programlisting
...
"transports": {
    "docker": {
        "quay.io/rhatdan": [
            {
                  "type": "signedBy",
                  "keyType": "GPGKeys",
                  "keyPath": "/tmp/publickey.gpg"
            }
        ],
...
```

[]{#11.htm#pgfId-1122059}You have not created the
`keyPath`{.fm-code-in-text} file /tmp/publickey.gpg yet. Create it using
the following GPG command:

``` programlisting
$ gpg --output /tmp/publickey.gpg --armor --export dwalsh@redhat.com
```

[]{#11.htm#pgfId-1122061}Now you can pull the signed image:

``` programlisting
$ podman pull quay.io/rhatdan/myimage
Trying to pull quay.io/rhatdan/myimage:latest...
...
Writing manifest to image destination
Storing signatures
2c7e43d880382561ebae3fa06c7a1442d0da2912786d09ea9baaef87f73c29ae
```

[]{#11.htm#pgfId-1122068}It worked! Still, you are not really sure if it
used signatures. Prove it to yourself by attempting to pull another
image from the repository, which you don't have signatures for, and it
will fail:

``` programlisting
$ podman pull quay.io/rhatdan/podman
Trying to pull quay.io/rhatdan/podman:latest...
Error: Source image rejected: A signature was required, 
➥ but no signature exists
```

[]{#11.htm#pgfId-1122073}Make sure to restore all settings back to
default:

``` programlisting
$ sudo cp /tmp/default.yaml /etc/containers/registries.d/default.yaml
$ sudo cp /tmp/policy.json /etc/containers/policy.json
```

[]{#11.htm#pgfId-1122076}Also, stop the localhost web server started in
another terminal. Table 11.3 describes the infrastructure you need to
set up to allow simple signing to be used within your
environment.[]{#11.htm#id_rersmg5h3akf}

[]{#11.htm#pgfId-1126058}Table 11.3 Infrastructure required for simple
signing

+-----------------+-----------------------------------------------------+
| []{#11.         | []{#11.htm#pgfId-1126064}Description                |
| htm#pgfId-11260 |                                                     |
| 62}Requirements |                                                     |
+-----------------+-----------------------------------------------------+
| []{#11.htm#pg   | []{#11.htm#pgfId-1126068}You need a GPG key pair,   |
| fId-1126066}GPG | where the private key is used on the service that   |
| private key     | signs the images.                                   |
+-----------------+-----------------------------------------------------+
| []{#            | []{#11.htm#pgfId-1126072}A web server has to run    |
| 11.htm#pgfId-11 | somewhere that has access to the signature storage. |
| 26070}Signature |                                                     |
| web server      |                                                     |
+-----------------+-----------------------------------------------------+

[]{#11.htm#pgfId-1122094}Once you have the infrastructure set up to use
simple signing, you will need to know the requirements of each client
that uses and verifies the signatures. Table 11.4 lists each
[]{#11.htm#marker-1126246}[]{#11.htm#marker-1126247}of[]{#11.htm#marker-1126248}
these
[]{#11.htm#marker-1126249}[]{#11.htm#marker-1126250}requirements.[]{#11.htm#id_gp7r9gjpnuax}

[]{#11.htm#pgfId-1126341}Table 11.4 Client configuration required for
simple signing

+-----------------+-----------------------------------------------------+
| []{#11.         | []{#11.htm#pgfId-1126347}Description                |
| htm#pgfId-11263 |                                                     |
| 45}Requirements |                                                     |
+-----------------+-----------------------------------------------------+
| []{#11.htm#pg   | []{#11.htm#pgfId-1126351}The public GPG key used    |
| fId-1126349}GPG | for signing must be present on any machine that     |
| public key      | pulls the signed images.                            |
|                 |                                                     |
| []{#11.htm#pgfI |                                                     |
| d-1126387}(/tmp |                                                     |
| /publickey.gpg) |                                                     |
+-----------------+-----------------------------------------------------+
| []{             | []{#11.htm#pgfId-1126355}The signature web server   |
| #11.htm#pgfId-1 | has to be configured as a sigstore in a             |
| 126353}Client's | /etc/containers/registries.d/\*.yaml file on all    |
| sigstore        | systems, which need to pull the signed images.      |
| configured      |                                                     |
+-----------------+-----------------------------------------------------+
| []{             | []{#11.htm#pgfId-1126359}Image trust has to be      |
| #11.htm#pgfId-1 | configured on every container engine system that    |
| 126357}Client's | uses the images.                                    |
| image trust     |                                                     |
| configured      |                                                     |
+-----------------+-----------------------------------------------------+

## []{#11.htm#pgfId-1122123}11.4 Podman image scanning {#11.htm#heading_id_9 .fm-head}

[]{#11.htm#pgfId-1122126}Podman
[]{#11.htm#marker-1122124}[]{#11.htm#marker-1122125}is not an image
scanner; it leaves this to other tools. But Podman does have a nice
feature that makes it easier for a scanner to scan an image. Podman can
directly mount an image that can be scanned. Scanners look at the
mounted content of an image without having to execute any of the code in
the image. Recall that you cannot mount containers or images in rootless
mode, without first entering the user namespaces. Execute the
`podman`{.fm-code-in-text} `image`{.fm-code-in-text}
`mount`{.fm-code-in-text} command[]{#11.htm#marker-1122127} to show the
error:

``` programlisting
$ podman image mount ubi8
Error: cannot run command "podman image mount" in rootless mode, must 
➥ execute `podman unshare` first
```

[]{#11.htm#pgfId-1122131}In this next example, you first use
`podman`{.fm-code-in-text} `unshare`{.fm-code-in-text} to enter the user
namespace, and then you mount the ubi8 image. Finally, change the
directory to the mount directory, and run a `find`{.fm-code-in-text}
command[]{#11.htm#marker-1122132} to locate all of the
`setuid`{.fm-code-in-text} binaries in the image. Notice that you use
tools from the host operating system to scan the image:

``` programlisting
$ podman unshare
```bash
# podman image mount
# mnt=$(podman image mount ubi8)
# echo $mnt
/home/dwalsh/.local/share/containers/storage/overlay/05ddfb76c5eb2146646c70
➥ e20db21a35dfec2215f130ce8bd04fce530142cfbd/merged
# cd $mnt
# /usr/bin/find . -user root -perm -4000
./usr/libexec/dbus-1/dbus-daemon-launch-helper
./usr/bin/chage
./usr/bin/mount
./usr/bin/umount
./usr/bin/newgrp
./usr/bin/gpasswd
./usr/bin/passwd
./usr/bin/su
./usr/sbin/userhelper
./usr/sbin/unix_chkpwd
./usr/sbin/pam_timestamp_check
```

[]{#11.htm#pgfId-1122152}Scanning an image with tools within the image
is not safe, since a hacker of the image can modify the scanning tools.
Podman makes it easy for scanners to do their jobs.

### []{#11.htm#pgfId-1122154}11.5.1 Read-only containers {#11.htm#heading_id_10 .fm-head1}

[]{#11.htm#pgfId-1122156}I []{#11.htm#marker-1122155}often talk about
containers in production versus containers in development. When a
containerized application is in development, it is useful to be able to
write to the container image and potentially commit that image later.
Although this is somewhat common, most people switch to using
Containerfiles when it comes to actually building images. The bottom
line is once developers hand off their software to quality engineering,
they expect content to be treated as read only.

[]{#11.htm#pgfId-1122157}When running a container in production, I
believe it makes sense to run the image in read-only mode. Imagine you
are running an application that gets hacked. The first thing the hacker
wants to do is to write the backdoor into the application; then, the
next time the container or application starts, the container has the
exploit in place. If the image was read only, the hacker is prevented
from leaving a backdoor in place and is forced to start the cycle from
the beginning.

[]{#11.htm#pgfId-1122159}The `--read-only`{.fm-code-in-text}
option[]{#11.htm#marker-1122158} prevents applications from writing
content to the image and forces applications to only write content to
either tmpfs filesystems or volumes added to the container. Sometimes
you might want to block the container from writing anywhere on your
system and only read or execute code within the container. Another
benefit of running containers in read-only mode is that you catch errors
where you did not know the container was writing to the image. Finally,
writing on top of a copy-on-write filesystem, like overlayfs, is almost
always slower than writing to a volume or a tmpfs:

``` programlisting
$ podman run --read-only ubi8 touch /foo
touch: cannot touch '/foo': Read-only file system
```

[]{#11.htm#pgfId-1122162}One problem with running in rootless mode is
that applications often expect to write to /run, /tmp, and /var/tmp.
Podman manages this by automatically mounting tmpfs filesystems at those
locations:

``` programlisting
$ podman run --read-only ubi8 touch /run/foo
```

[]{#11.htm#pgfId-1122164}Because some users believe allowing any places
for a containerized application to write, even on tmpfs mounts, is too
insecure, Podman added a `--read-only-tmpfs`{.fm-code-in-text}
option[]{#11.htm#marker-1122165}. The
`--read-only-tmpfs`{.fm-code-in-text} option[]{#11.htm#marker-1122166}
adds the /run, /tmp, and /var/tmp tmpfs when run in
`--read-only`{.fm-code-in-text} mode[]{#11.htm#marker-1122167}. If you
want to disable this, you can use the
[]{#11.htm#marker-1122168}`–-read-only-tmpfs=false`{.fm-code-in-text}[]{#11.htm#marker-1122169}[]{#11.htm#marker-1122170}
flag[]{#11.htm#marker-1122171}:

``` programlisting
$ podman run --read-only-tmpfs=false --read-only ubi8 touch /run/foo
touch: cannot touch '/run/foo': Read-only file system
```

## []{#11.htm#pgfId-1122175}11.5 Security in depth {#11.htm#heading_id_11 .fm-head}

[]{#11.htm#pgfId-1122178}In
[]{#11.htm#marker-1122176}[]{#11.htm#marker-1122177}the security field,
there is a common idea of *security in depth*. According to this notion,
multiple layers or tools should be used to safeguard assets. The classic
analogy for this is the security of an ancient castle, which would
usually be built high on a hill, have multiple walls, have moats, and
have even more security features. An attacker would need to break
through all of these layers to get to the ruler.

[]{#11.htm#pgfId-1122179}Container security works in much the same way.
Podman uses all of the security mechanisms provided by Linux, giving you
security in depth.

### []{#11.htm#pgfId-1122181}11.5.1 Podman uses all security mechanisms simultaneously {#11.htm#heading_id_12 .fm-head1}

[]{#11.htm#pgfId-1122184}Podman
[]{#11.htm#marker-1122182}[]{#11.htm#marker-1122183}containers can run
with all of the security mechanisms mentioned in this chapter. This
means a hacked container needs to find a way to escape read-only
filesystems, namespaces, dropped capabilities, SELinux, seccomp, and so
on to gain access to your system.

[]{#11.htm#pgfId-1122185}In certain cases, you might need to loosen some
security mechanisms to allow a container to run. Understanding how to
deal with the security features discussed in this chapter is always
better than just running your containers with the
`--privileged`{.fm-code-in-text} flag[]{#11.htm#marker-1122186}, which
turns off all of your defenses.

[]{#11.htm#pgfId-1122187}Podman shoots for a reasonable amount of
security wrapping for containers, but it needs to allow general-purpose
containers to succeed. Understanding your container application's
security requirements and the Podman security features allows you to
ratchet up the security wrapping of your containers. If you know your
container does not need to run as root, don't start it as root. If your
container does not need any Linux capabilities, drop them. Rootless
containers are better than rootful containers. Consider also running
containers in read-only mode or inside of separated user namespaces. You
have the ability to make your castle walls thicker around your
containerized applications by simply employing these
[]{#11.htm#marker-1122188}[]{#11.htm#marker-1122189}measures.

### []{#11.htm#pgfId-1122191}11.5.2 Where should you run your containers? {#11.htm#heading_id_13 .fm-head1}

[]{#11.htm#pgfId-1122194}I'll
[]{#11.htm#marker-1122192}[]{#11.htm#marker-1122193}leave you with one
final thought. At the beginning of this chapter, I talked about the
three pigs living in different types of shelters---standalone houses,
duplexes, and condominium buildings---each slightly less secure than the
last. Container security can do better than the pigs living in
individual housing units, in that the units can be stacked together.

[]{#11.htm#pgfId-1122195}Imagine you had two different containers: a web
frontend and a database with credit card data. If you wanted to make
sure they were separate, you could put them together on the system
inside containers or, better yet, put them in containers but put them
into separate VMs, and then, finally, put the VMs on separate machines.
You would be able to put your web frontend into a machine running a VM
inside of a container inside of your DMZ exposed to the internet. You
could do all this while putting your database inside of your private
network, without limited network access to your web frontends. The
possibilities are
[]{#11.htm#marker-1122196}[]{#11.htm#marker-1122197}nearly
[]{#11.htm#marker-1122198}[]{#11.htm#marker-1122199}endless.

## []{#11.htm#pgfId-1122200}Summary {#11.htm#heading_id_14 .fm-head}

-   []{#11.htm#pgfId-1122201 .calibre17}Container security has many
    different facets, including separation of running containers,
    trusting the images and registries, scanning the images, and so on.

-   []{#11.htm#pgfId-1122202 .calibre17}Defense in depth means your
    container tooling takes advantage of as many security mechanisms as
    possible. If one security mechanism fails, the others might still
    protect your system.

-   []{#11.htm#pgfId-1122203 .calibre17}Container security is all about
    protecting the Linux kernel and host filesystem from hostile
    container processes.

-   []{#11.htm#pgfId-1122204 .calibre17}Setting up and controlling the
    container images you run on your systems is critical. Do not allow
    your users to run random applications from the
    []{#11.htm#marker-1122205 .calibre17}internet.

[]{#A.htm}
