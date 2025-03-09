# Appendix B. OCI runtimes

[]{#B.htm#pgfId-1282816}This appendix []{#B.htm#marker-1282815}describes
the primary OCI runtimes used with container engines like Podman. As
discussed in chapter 1, the OCI runtime
([https://opencontainers.org](https://opencontainers.org/){.url}) is the
executable launched by container engines, including Podman, used to
configure the Linux kernel and subsystems to run the kernel; its last
step is launching the container. The OCI runtime reads the OCI runtime
specification JSON file and then configures the namespaces, security
controls, and cgroups and eventually starts the container process
(figure B.1)[]{#B.htm#marker-1285375}.

::: figure
![](images/OEBPS/Images/B-01.png){.calibre18}

Figure B.1 Podman executes the OCI runtime to launch the container.
:::

[]{#B.htm#pgfId-1282825}In this appendix, you'll learn the four main OCI
runtimes in use. The `--runtime`{.fm-code-in-text}
option[]{#B.htm#marker-1282824} allows you to switch between different
OCI runtimes. In the next example, you will run the same container
command twice, each time with a different runtime. In the first command,
you run the container with a runtime, `crun`{.fm-code-in-text}, defined
in the containers.conf, so you don't need to specify the path to the
runtime.

[]{#B.htm#pgfId-1282826}Listing B.1 Podman running with the alternate
OCI runtime `crun`{.fm-code-in-text}

``` programlisting
$ podman --runtime crun run --rm ubi8 echo hi     ❶
hi
```

[]{#B.htm#pgfId-1285577}[❶]{.fm-combinumeral} The \--runtime option
tells Podman to use the crun OCI runtime, rather than the default.

[]{#B.htm#pgfId-1282830}The default runtime is defined under the
`[containers]`{.fm-code-in-text} table in the containers.conf file on
the Linux machine.

[]{#B.htm#pgfId-1282831}Listing B.2 Modifying the default OCI runtime

``` programlisting
$ grep -iA 3 "Default OCI Runtime" /usr/share/containers/containers.conf
# Default OCI runtime
#
#runtime = "crun"      ❶
```

[]{#B.htm#pgfId-1285510}[❶]{.fm-combinumeral} Podman defaults to crun on
most systems; on some older distributions, like Red Hat Enterprise
Linux, Podman defaults to runc.

[]{#B.htm#pgfId-1282839}In the second example, you use the full path of
the OCI runtime, /usr/bin/runc:

``` programlisting
$ podman --runtime /usr/bin/runc run –rm ubi8 echo hi
hi
```

[]{#B.htm#pgfId-1282842}If you want to permanently change the default
OCI runtime, you can set the runtime option in the
`[engine]`{.fm-code-in-text} table in the containers.conf file in your
home directory:

``` programlisting
$ cat > ~/.config/containers/containers.conf << EOF
[engine]
runtime="runc"
EOF
$ podman --help | grep -- runc
   --runtime string Path to the OCI-compatible binary used to run containers. (default "runc")`
```

[]{#B.htm#pgfId-1282850}[Note]{.fm-callout-head} The
`--runtime`{.fm-code-in-text1} option[]{#B.htm#marker-1282849} is only
available on Linux. `podman`{.fm-code-in-text1}
`--remote`{.fm-code-in-text1}, and therefore Podman, on Mac and Windows,
does not support the `--runtime`{.fm-code-in-text1} option, so you need
to set the containers.conf file on the server side.

[]{#B.htm#pgfId-1282851}See the `podman(1)`{.fm-code-in-text} man page
for more information: `man`{.fm-code-in-text}
`podman.`{.fm-code-in-text}

[]{#B.htm#pgfId-1282852}OCI runtimes are continuously being developed
and experimented with. You can expect innovation to happen in this space
going forward. The first container runtime developed, and the de facto
standard, is `runc`{.fm-code-in-text}.

## []{#B.htm#pgfId-1282853}B.1 runc {#B.htm#heading_id_3 .fm-head}

[]{#B.htm#pgfId-1282857}`runc`{.fm-code-in-text}
[]{#B.htm#marker-1282854}[]{#B.htm#marker-1282855}is the original OCI
runtime
([https://github.com/opencontainers/runc](https://github.com/opencontainers/runc){.url}).
When the OCI originally formed, Docker donated `runc`{.fm-code-in-text}
to the OCI to serve as the default implementation of an OCI runtime. The
OCI continues to support and develop `runc`{.fm-code-in-text}. It is
written in Golang and also includes the libcontainer library, which is
used in many container engines and Kubernetes.

[]{#B.htm#pgfId-1282859}The `runc`{.fm-code-in-text} website states that
`runc`{.fm-code-in-text}, and all of the OCI runtimes, is a low-level
tool not designed to be used directly by the end user. It is recommended
to be launched by container engines like Podman or Docker.

[]{#B.htm#pgfId-1282860}Recall that the container engine's job is
pulling the container images to the host, configuring and mounting the
root filesystem (rootfs[]{#B.htm#marker-1282861}) to be used within the
container, and, finally, writing the OCI runtime JSON file before
launching the OCI runtime.

[]{#B.htm#pgfId-1282862}The OCI runtime specification describes only the
content of the JSON file used by OCI runtimes. Because every OCI engine
supports the `runc`{.fm-code-in-text} command line, the other OCI
runtimes adopted the same CLI commands and options. This makes it easier
for one runtime to replace another when launched by the container
engine. Table B.1 shows the commands supported by
`runc`{.fm-code-in-text} and therefore all OCI runtimes.

[]{#B.htm#marker-1282957}[]{#B.htm#pgfId-1284151}Table B.1
`runc`{.fm-code-in-text} commands

+-----------------+-----------------------------------------------------+
| [               | []{#B.htm#pgfId-1284157}Description                 |
| ]{#B.htm#pgfId- |                                                     |
| 1284155}Command |                                                     |
+-----------------+-----------------------------------------------------+
| []{#B           | []{#B.htm#pgfId-1284161}Checkpoints a running       |
| .htm#pgfId-1284 | container                                           |
| 159}`checkpoint |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284222} |                                                     |
+-----------------+-----------------------------------------------------+
| [               | []{#B.htm#pgfId-1284165}Creates a container         |
| ]{#B.htm#pgfId- |                                                     |
| 1284163}`create |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284223} |                                                     |
+-----------------+-----------------------------------------------------+
| [               | []{#B.htm#pgfId-1284169}Deletes any resources held  |
| ]{#B.htm#pgfId- | by the container often used with detached           |
| 1284167}`delete | containers                                          |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284224} |                                                     |
+-----------------+-----------------------------------------------------+
| [               | []{#B.htm#pgfId-1284173}Displays container events,  |
| ]{#B.htm#pgfId- | such as OOM notifications, CPU, memory, and IO      |
| 1284171}`events | usage statistics                                    |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284225} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#B.htm#pgfI  | []{#B.htm#pgfId-1284177}Initializes the namespaces  |
| d-1284175}`init | and launches the process                            |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284226} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#B.htm#pgfI  | []{#B.htm#pgfId-1284181}Sends the specified signal  |
| d-1284179}`kill | (default: `SIGTERM`{.fm-code-in-text1}) to the      |
| `{.fm-code-in-t | container's init process                            |
| ext1}[]{#B.htm# |                                                     |
| marker-1284227} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#B.htm#pgfI  | []{#B.htm#pgfId-1284185}Lists containers started by |
| d-1284183}`List | runc with the given root                            |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284228} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#B.htm#pgfId | []{#B.htm#pgfId-1284189}Suspends all processes      |
| -1284187}`pause | inside the container                                |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284229} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#B.htm#pg    | []{#B.htm#pgfId-1284193}Displays the processes      |
| fId-1284191}`ps | running inside a container                          |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284230} |                                                     |
+-----------------+-----------------------------------------------------+
| []              | []{#B.htm#pgfId-1284197}Restores a container from a |
| {#B.htm#pgfId-1 | previous checkpoint                                 |
| 284195}`restore |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284231} |                                                     |
+-----------------+-----------------------------------------------------+
| [               | []{#B.htm#pgfId-1284201}Resumes all processes that  |
| ]{#B.htm#pgfId- | have been previously paused                         |
| 1284199}`resume |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284232} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#B.htm#pgf   | []{#B.htm#pgfId-1284205}Creates and runs a          |
| Id-1284203}`run | container                                           |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284233} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#B.htm#pgfI  | []{#B.htm#pgfId-1284209}Creates a new specification |
| d-1284207}`spec | file                                                |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284234} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#B.htm#pgfId | []{#B.htm#pgfId-1284213}Executes the user-defined   |
| -1284211}`start | process in a created container                      |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284235} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#B.htm#pgfId | []{#B.htm#pgfId-1284217}Outputs the state of a      |
| -1284215}`state | container                                           |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284236} |                                                     |
+-----------------+-----------------------------------------------------+
| [               | []{#B.htm#pgfId-1284221}Updates container resource  |
| ]{#B.htm#pgfId- | constraints                                         |
| 1284219}`update |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#B.htm# |                                                     |
| marker-1284237} |                                                     |
+-----------------+-----------------------------------------------------+

[]{#B.htm#pgfId-1282951}`runc`{.fm-code-in-text} continues to be
developed and has a very active community. The problem with
`runc`{.fm-code-in-text} is that it is written in Golang. Golang was not
designed to be a small, often-executed application that needs to start
quickly and fork/exec a command and exit quickly. Fork/exec is a heavy
operation in Golang, and although `runc`{.fm-code-in-text} attempts to
work around this, it ultimately sacrifices a bit of performance. The *a
bit* can accumulate over time though, so `crun`{.fm-code-in-text}
performs much better at
[]{#B.htm#marker-1285720}[]{#B.htm#marker-1285721}scale.

## []{#B.htm#pgfId-1282955}B.2 crun {#B.htm#heading_id_4 .fm-head}

[]{#B.htm#pgfId-1282958}`runc`{.fm-code-in-text},
[]{#B.htm#marker-1285723}[]{#B.htm#marker-1285724}being written in
Golang, is a very heavy executable---12 megabytes in size. Golang is a
great language, but it doesn't take advantage of shared libraries.
Golang executables take up considerably more memory because of this. The
size of `runc`{.fm-code-in-text} causes it to be somewhat slower loading
during container start. Another problem with Golang is that it does not
support the fork/exec model all that well; it is slower than fork/exec
in other languages (e.g., C). This lack of speed is more important when
you are starting and stopping hundreds or thousands of containers---for
example, on a Kubernetes cluster. Container engines like Podman, also
written in Go, generally run for a much longer time, so the startup time
is not as important. OCI runtimes like `runc`{.fm-code-in-text} execute
for a very short time and exit quickly.

[]{#B.htm#pgfId-1282959}Giuseppe Scrivano, a contributor to
`runc`{.fm-code-in-text} and Podman, understood these deficiencies in
`runc`{.fm-code-in-text} and wanted to write a compatible OCI runtime in
the C language. He created a very lightweight OCI runtime called
`crun`{.fm-code-in-text}.

[]{#B.htm#pgfId-1282960}`crun`{.fm-code-in-text} describes itself as "*a
fast and lightweight OCI runtime.*"
([https://github.com/containers/crun](https://github.com/containers/crun){.url})
It supports all of the same commands and options as
`runc`{.fm-code-in-text}, and the `crun`{.fm-code-in-text} executable is
many times smaller than `runc`{.fm-code-in-text}. Execute the
`du`{.fm-code-in-text} `-s`{.fm-code-in-text}
command[]{#B.htm#marker-1284956} to compare sizes:

``` programlisting
$ du -s /usr/bin/runc /usr/bin/crun
14640    /usr/bin/runc
392    /usr/bin/crun
```

[]{#B.htm#pgfId-1282965}`crun`{.fm-code-in-text}, being written in C,
supports fork and exec much better than `Golang`{.fm-code-in-text} and,
therefore, is much quicker when launching a container.

[]{#B.htm#pgfId-1282966}This also makes it plug in easily to other
libraries on the system, and there is some experimentation on using
`crun`{.fm-code-in-text} as a library for processing the OCI runtime
JSON file and launching different types of containers (e.g., WASM and
Windows containers on Linux). `crun`{.fm-code-in-text} also has
potential for launching KVM-separated containers based on libkrun.

[]{#B.htm#pgfId-1282967}`crun`{.fm-code-in-text} is now the default OCI
runtime used by Podman in Fedora and in Red Hat Enterprise Linux 9.
`runc`{.fm-code-in-text} continues to be supported and is the default
OCI runtime in Red Hat Enterprise Linux 8.

[]{#B.htm#pgfId-1282969}`crun`{.fm-code-in-text} and
`runc`{.fm-code-in-text} are the two primary OCI runtimes for managing
traditional containers that use namespace separation. Both these
projects work fairly closely together. When bugs or problems are found
in either OCI runtime, they are quickly fixed in both. See the
`crun(1)`{.fm-code-in-text} man page for more information:
[]{#B.htm#marker-1282970}[]{#B.htm#marker-1282971}`man`{.fm-code-in-text}
`crun`{.fm-code-in-text}.

## []{#B.htm#pgfId-1282973}B.3 Kata {#B.htm#heading_id_5 .fm-head}

::: figure
![](images/OEBPS/Images/B-01-UN01.png){.calibre18}
:::

[]{#B.htm#pgfId-1282980}OCI
[]{#B.htm#marker-1283889}[]{#B.htm#marker-1283890}runtimes are also
written to use VM separation, with the primary example of this being
Kata Containers. The Kata Container project
([https://katacontainers.io](https://katacontainers.io){.url})
advertises itself as the following: "*The speed of containers, the
security of VMs. Kata Containers is an open source container runtime,
building lightweight virtual machines that seamlessly plug into the
container's ecosystem."*

::: figure
![](images/OEBPS/Images/B-02.png){.calibre18}

Figure B.2 Kata containers launches a lightweight VM, which only runs
the container.
:::

[]{#B.htm#pgfId-1282987}Kata containers use VM technology for launching
each container, which is very different from launching a VM and running
Podman within it. A standard VM has an init system, which launches all
sorts of services, like logging systems, cron, and more. On the other
hand, a Kata container launches a micro OS, which runs only the
container and its support services (figure B.2). As its only purpose is
launching the container, when the container exits, this VM goes away.

[]{#B.htm#pgfId-1282988}I believe running containers within
VM/hypervisor separation gives you better security separation than
traditional container separation, where containers communicate directly
with the host kernel. A VM-separated container has to first break out of
containment inside of the VM, then find a way to break out of the
hypervisor---only then to face attacking the host kernel.

[]{#B.htm#pgfId-1282989}While VM-separated containers are more secure,
this does come with some downsides. There is a decent amount of overhead
in starting a Kata container, configuring the hypervisor, launching the
kernel and other processes within the VM, and then finally the
container. The VM's memory, CPU, and so on have to be preallocated and
are difficult to change. Running Kata within a VM in the cloud is often
not allowed, or is at least more expensive, because most of the cloud
vendors frown on nested virtualization.

[]{#B.htm#pgfId-1282990}Finally, and most importantly, VM-separated
containers by their very nature have difficulties sharing content with
other containers and the host operating system. The biggest problem is
with volumes.

[]{#B.htm#pgfId-1282991}While sharing content with the host machine in
traditional containers is just a bind mount, in VM-separate containers,
bind mounts do not work. Since the processes on the host and in the
container are running with two different kernels, you need a network
protocol to share content. Kata containers originally used NFS and Plan
9 networked filesystems. Reading/writing data over these networked
filesystems is considerably slower than native filesystem reads and
writes you get with a bind mount.

[]{#B.htm#pgfId-1282992}Virtiofs is a new filesystem that has the
properties of a network filesystem but allows VMs to access files on the
host. It is able to show major improvements in speed over the
network-based filesystems, while still remaining under heavy
development.

[]{#B.htm#pgfId-1282993}Kata containers have two ways to be launched.
Kata traditionally has an OCI command line,
`kata-runtime`{.fm-code-in-text}, based on the `runc`{.fm-code-in-text}
command[]{#B.htm#marker-1282994} supported by Podman. You can see the
paths defined in containers.conf, on the Linux machine, by searching for
`#kata`{.fm-code-in-text}:

``` programlisting
$ grep -A 9 '^#kata' /usr/share/containers/containers.conf
#kata = [
```bash
#  "/usr/bin/kata-runtime",
#  "/usr/sbin/kata-runtime",
#  "/usr/local/bin/kata-runtime",
#  "/usr/local/sbin/kata-runtime",
#  "/sbin/kata-runtime",
#  "/bin/kata-runtime",
#  "/usr/bin/kata-qemu",
#  "/usr/bin/kata-fc",
#]
```

[]{#B.htm#pgfId-1283006}The bottom line on Kata containers is that you
get better security with a performance overhead. You can choose between
these OCI runtimes with your workload's needs in
[]{#B.htm#marker-1283007}[]{#B.htm#marker-1283008}mind.

## []{#B.htm#pgfId-1283010}B.4 gVisor {#B.htm#heading_id_6 .fm-head}

::: figure
![](images/OEBPS/Images/B-02-UN02.png){.calibre18}
:::

[]{#B.htm#pgfId-1283017}The
[]{#B.htm#marker-1283015}[]{#B.htm#marker-1283016}last OCI runtime I
cover in this appendix is gVisor
([https://gvisor.dev/](https://gvisor.dev/){.url}). The gVisor website
advertises itself as "an application kernel for containers that provides
efficient defense-in-depth anywhere."

[]{#B.htm#pgfId-1283019}gVisor includes an OCI runtime called
`runsc`{.fm-code-in-text}[]{#B.htm#marker-1283018} and works with Podman
and other container engines. The gVisor project calls itself an
application kernel, written in Golang, that implements a substantial
portion of the Linux system call interface. It provides an additional
layer of isolation between running applications and the host operating
system. Google engineering wrote the original versions of gVisor and
claims that the bulk of the containers Google Cloud run use the gVisor
OCI runtime.

[]{#B.htm#pgfId-1283020}gVisor is somewhat similar to VM-isolated
containers in that gVisor intercepts almost all system calls from within
the container and then processes them. gVisor describes itself as an
application kernel for containers written in Golang, limiting the access
to the host kernel. At the same time, it does not have the same problem
of a nested virtualization as Kata.

[]{#B.htm#pgfId-1283021}However, gVisor introduces a performance penalty
with additional CPU cycles and higher memory usage. This may introduce
increased latency, reduced throughput, or both. gVisor is also an
independent implementation of the system call surface, meaning many of
the subsystems or specific calls are not as optimized as more
[]{#B.htm#marker-1283022}[]{#B.htm#marker-1283023}mature
[]{#B.htm#marker-1283024}implementations[]{#B.htm#marker-1285384}.

[]{#C.htm}
