# 1 Intro

Podman and Docker feature comparison

+-----------------+------+------+--------------------------------------+
| Feature         | Po   | Do   | Description                          |
|                 | dman | cker |                                      |
+=================+======+======+======================================+
| Supports all    | [    | [    | Both pull and run container images   |
| OCI and Docker  | ✔]{. | ✔]{. | from container registries (i.e.,     |
| images          | camb | camb | quay.io and docker.io)               |
|                 | ria} | ria} |                                      |
+-----------------+------+------+--------------------------------------+
| Launches OCI    | [    | [    | Launch containers using runc, crun,  |
| container       | ✔]{. | ✔]{. | Kata, gVisor, and OCI container      |
| engines         | camb | camb | engines                              |
|                 | ria} | ria} |                                      |
+-----------------+------+------+--------------------------------------+
| Simple          | [    | [    | Podman and Docker share the same     |
| command-line    | ✔]{. | ✔]{. | CLI.                                 |
| interface       | camb | camb |                                      |
|                 | ria} | ria} |                                      |
+-----------------+------+------+--------------------------------------+
| Integration     | [    | [    | Podman supports running systemd      |
| with systemd    | ✔]{. | ✘]{. | inside of the container as well as   |
|                 | camb | camb | many systemd features.               |
|                 | ria} | ria} |                                      |
+-----------------+------+------+--------------------------------------+
| Fork/exec model | [    | [    | The container is a direct descendant |
|                 | ✔]{. | ✘]{. | of the podman command.               |
|                 | camb | camb |                                      |
|                 | ria} | ria} |                                      |
+-----------------+------+------+--------------------------------------+
| Fully support   | [    | [    | Only Podman supports running         |
| user namespace  | ✔]{. | ✘]{. | containers in separate user          |
|                 | camb | camb | namespaces.                          |
|                 | ria} | ria} |                                      |
+-----------------+------+------+--------------------------------------+
| Client--server  | [    | [    | Docker is a RESTful API daemon.      |
| model           | ✔]{. | ✔]{. | Podman supports RESTful API via a    |
|                 | camb | camb | systemd socket=activated service.    |
|                 | ria} | ria} |                                      |
+-----------------+------+------+--------------------------------------+
| Supports        | [    | [    | compose scripts work against both    |
| dockercompose   | ✔]{. | ✔]{. | restful APIs. Podman[']{.calibre6}s  |
|                 | camb | camb | works in rootless mode.              |
|                 | ria} | ria} |                                      |
+-----------------+------+------+--------------------------------------+
| Supports        | [    | [    | docker-py python bindings work       |
| docker-py       | ✔]{. | ✔]{. | against both restful APIs. Podman's  |
|                 | camb | camb | works in rootless mode. Podman also  |
|                 | ria} | ria} | supports podman-py for running       |
|                 |      |      | advanced features.                   |
+-----------------+------+------+--------------------------------------+
| Daemonless      | [    | [    | The podman command runs like a       |
|                 | ✔]{. | ✘]{. | traditional command-line tool, while |
|                 | camb | camb | Docker requires multiple             |
|                 | ria} | ria} | root-running daemons.                |
+-----------------+------+------+--------------------------------------+
| Supports        | [    | [    | Podman supports running multiple     |
| Kubernetes-like | ✔]{. | ✘]{. | containers within the same pod.      |
| pods            | camb | camb |                                      |
|                 | ria} | ria} |                                      |
+-----------------+------+------+--------------------------------------+
| Supports        | [    | [    | Podman can launch containers and     |
| Kubernetes yaml | ✔]{. | ✘]{. | pods based on Kubernetes yaml. It    |
|                 | camb | camb | can also generate Kuberenetes.yaml   |
|                 | ria} | ria} | from running containers.             |
+-----------------+------+------+--------------------------------------+
| Supports Docker | [    | [    | Podman believes the future for       |
| swarm           | ✘]{. | ✔]{. | orchestrated multi-node containers   |
|                 | camb | camb | is Kubernetes and does not plan on   |
|                 | ria} | ria} | implementing Swarm.                  |
+-----------------+------+------+--------------------------------------+
| Customizable    | [    | [    | Podman allows you to configure       |
| registries      | ✔]{. | ✘]{. | registries for short name expansion. |
|                 | camb | camb | Docker is hard coded to docker.io    |
|                 | ria} | ria} | when you specify a short name.       |
+-----------------+------+------+--------------------------------------+
| Customizable    | [    | [    | Podman supports fully customizing    |
| defaults        | ✔]{. | ✘]{. | all of its defaults including        |
|                 | camb | camb | security, namespaces, volumes, and   |
|                 | ria} | ria} | more.                                |
+-----------------+------+------+--------------------------------------+
| Mac OS support  | [    | [    | Podman and Docker support running    |
|                 | ✔]{. | ✔]{. | containers on a Mac via a VM running |
|                 | camb | camb | Linux.                               |
|                 | ria} | ria} |                                      |
+-----------------+------+------+--------------------------------------+
| Windows support | [    | [    | Podman and Docker support running    |
|                 | ✔]{. | ✔]{. | containers on a Windows WSL2 or a VM |
|                 | camb | camb | running Linux.                       |
|                 | ria} | ria} |                                      |
+-----------------+------+------+--------------------------------------+
| Linux support   | [    | [    | Podman and Docker are supported on   |
|                 | ✔]{. | ✔]{. | all major Linux distributions.       |
|                 | camb | camb |                                      |
|                 | ria} | ria} |                                      |
+-----------------+------+------+--------------------------------------+




 

 

## Podman in Action

Secure, rootless containers for Kubernetes, microservices, and more

 
 
Part 1.
Foundations

  1 Podman: A next-generation container engine

  2 Command line

  3 Volumes

  4 Pods

Part 2.
Design

  5 Customization and configuration files

  6 Rootless containers

Part 3. Advanced topics

  7 Integration with systemd

  8 Working with Kubernetes

  9 Podman as a service

Part 4. Container security

10 Security container isolation

11 Additional security considerations

  

Appendix A. Podman-related container tools

Appendix B. OCI runtimes

Appendix C. Getting Podman

Appendix D. Contributing to Podman

Appendix E. Podman on macOS

Appendix F. Podman on Windows

 <span style="background:#40a9ff">Working with a root-running daemon and then adding more and more</span>
<span style="background:#40a9ff">daemons felt like the wrong approach</span>. Instead, I felt <span style="background:#40a9ff">we could use</span>
<span style="background:#40a9ff">low-level operating systems concepts to create a tool that ran the same</span>
<span style="background:#40a9ff">containerized applications in the same manner but with more security and</span>



## Part 1. Foundations

in part 1 of the book,
I introduce you to <span style="background:#40a9ff">several ways you can use Podman from the command</span>
<span style="background:#40a9ff">line</span>. In chapter 2 you learn <span style="background:#40a9ff">how to create and work with containers and</span>
<span style="background:#40a9ff">how containers work with images. </span>You also learn the <span style="background:#40a9ff">difference between a</span>
<span style="background:#40a9ff">container and an image, how to save a container into an image, and then</span>
<span style="background:#40a9ff">to push the image to a registry</span>,<span style="background:#40a9ff"> so it can be shared with other users.</span>

In chapter 3 I introduce the concept of a
*volume*. <span style="background:#40a9ff">Volumes are the mechanisms most users of your containerized</span>
<span style="background:#40a9ff">applications use to store their data and keep it isolated from the</span>
<span style="background:#40a9ff">application.</span> The first two chapters really concentrate on the use of
containers and images, which is very similar to the way containers work
in Docker.

Chapter 4 adds the concept of *pods*, similar
to Kubernetes Pods, a feature Docker does not support. <span style="background:#40a9ff">Pods allow you to</span>
<span style="background:#40a9ff">share one or more containers within the same resource, namespaces, and</span>
<span style="background:#40a9ff">security constraints. </span>Pods <span style="background:#40a9ff">can allow you to write more complex</span>
<span style="background:#40a9ff">applications and manage them as a single</span>
<span style="background:#40a9ff">entity</span>.[]{#p1.htm#id_u6n877dddx1z}[]{#p1.htm#id_7dubk56duarh}[]{#p1.htm#id_j5ogsbez02wl}

[]{#01.htm}

## Podman: A next-generation container engine 

What Podman is
The advantages of using Podman over Docker
Examples of using Podman

Starting this book is difficult because so many
people come to it with different expectations and experiences. You
likely have some experience with containers, Docker, or Kubernetes---or
at least are interested in learning more about Podman because you've
heard about it. If you've used or evaluated Docker, you'll find that
Podman works the same as Docker in most cases, but it solves some
problems inherent in Docker; most significantly, <span style="background:#40a9ff">Podman offers enhanced</span>
<span style="background:#40a9ff">security and the ability to run commands with non-root privileges.</span> This
means you can manage containers with Podman without root access or
privileges. Because of Podman's design, it can run with much better
security than Docker by default.

[]{#01.htm#pgfId-1031529}In addition to being open source (and therefore
free), Podman's commands, run from the command-line interface (CLI), are
quite similar to Docker's. This book shows how you can use Podman as a
local container engine to launch containers on a single node, either
locally or through a remote REST API. You'll also learn how to find,
run, and build containers using Podman with open source tools such as
Buildah and Skopeo.

## About all these terms

[]{#01.htm#pgfId-1031533}Before []{#01.htm#marker-1031532}you go
further, I think it is important to define the terminology that will be
used throughout this book. In the container world, <span style="background:#40a9ff">terms like *container</span>
<span style="background:#40a9ff">orchestrator*[]{#01.htm#marker-1031534}, *container</span>
<span style="background:#40a9ff">engine*[]{#01.htm#marker-1031535}, and *container</span>
<span style="background:#40a9ff">runtime*[]{#01.htm#marker-1031536} are often used interchangeably</span>, which
commonly leads to confusion. The following list is a summary of what
each of these terms refers to in the context of this text:

-   <span style="background:#40a9ff">Container orchestrators </span>
	- <span style="background:#40a9ff">Software projects and products that orchestrate containers onto multiple</span>
<span style="background:#40a9ff">    different machines or nodes</span>. These orchestrators <span style="background:#40a9ff">communicate with</span>
<span style="background:#40a9ff">    container engines to run containers</span>. <span style="background:#40a9ff">The primary container</span>
<span style="background:#40a9ff">    orchestrator is Kubernetes</span>, <span style="background:#40a9ff">which was originally designed to talk to</span>
<span style="background:#40a9ff">    the Docker daemon container engine, but using Docker is becoming</span>
<span style="background:#40a9ff">    obsolete because Kubernetes primarily uses CRI-O or</span>
    <span style="background:#40a9ff">containerd as its container</span>
<span style="background:#40a9ff">    engine</span>. <span style="background:#40a9ff">CRI-O and containerd are purpose built for running</span>
<span style="background:#40a9ff">    orchestrated Kubernetes containers</span> (CRI-O is covered in appendix
    A).[]{#01.htm#id_Hlk113268570 .calibre17} Docker Swarm
    an[]{#01.htm#id_Hlk113268625 .calibre17}d Apache Mesos are other
    examples of container orchestrators.

-  *Container engines*
	- Primarily used for configuring containerized applications to run on a single local
    node. 
    - can be launched directly by users, administrators, and
    developers. They 
    - can also be launched out of systemd unit files at
    boot as well as launched by container orchestrators like Kubernetes.
    
    - CRI-O and containerd are container engines
    used by Kubernetes to manage containers locally. 
	    - They really are not intended to be used directly by users. 
	    - Docker and Podman are the primary container engines used by users to develop, manage, and run containerized applications on a single machine. 
	    - Podman is seldom used to launch containers for Kubernetes; therefore, Kubernetes is
    not generally covered in this book. 
    - Buildah is another container engine, although it is only used for building container images.

- Open Container Initiative (OCI) container runtimes
	- Configure different parts of the Linux kernel and then, finally, launch the containerized application. 
	- The two most commonly used container runtimes are `runc` and `crun`.
	- Kata and gVisor are other examples of container runtimes. See appendix B to understand the differences between the OCI container runtimes.

Categories these open source container projects fit.

![](images/01-01.png)

*Podman* is short for *Pod Manager*

*pod* (concept popularized by the Kubernetes project)
- One or more containers sharing the same namespaces and `cgroups`(resource
constraints. 

Podman runs individual containers as well as pods.

The Podman logo in figure 1.2 is a group of Selkies, the Irish concept
of a mermaid. Groups of Selkies are called pods.

![](images/01-02.png)

The Podman project describes Podman as "a daemonless container engine for developing, managing, and running OCI Containers on your Linux System. 

Containers can either be run as root or in rootless mode"

Podman is often summarized with the simple line *alias Docker = Podman* because
Podman does almost everything that Docker can do with the same command
line as Docker. 

The Open Container Initiative (OCI) is a standards body with the primary goal of creating open industry standards regarding container formats and runtimes. For more information, see
[https://opencontainers.org](https://opencontainers.org){.url}.

The Podman upstream project resides at github.com in the Containers project,
([https://github.com/containers/podman](https://github.com/containers/podman){.url}) shown in figure 1.3, along with other container libraries and container management tools like Buildah and Skopeo. (See appendix A for a description of some of these tools.)
![](images/01-03.png)

Podman runs images with the newer OCI format, described in section 1.1.2, as well as the
legacy Docker (v2 and v1) format images. 

Podman runs any image available at container registries, like docker.io and quay.io, as well as the
hundreds of other container registries. 

Podman pulls these images to a Linux host and launches them in the same way as Docker and Kubernetes.

Podman supports all OCI runtimes, including `runc`, `crun`, `kata`, and `gvisord` (appendix B), just
like Docker.

One of Podman's primary use cases is running containerized applications on single-node environments, such as edge devices. 

Podman and systemd allow you to manage the entire life cycle of the application on nodes without human intervention.

Application developers are also an intended audience for this book. 

Podman is a great tool for developers looking to containerize their applications in a secure manner. 

Podman allows developers to create Linux containers on all Linux distributions. 

In addition, Podman is available on the Mac and Windows platforms, where it can communicate with the Podman service running within a VM or on a Linux box available on the network. 

*Podman in Action* shows you how to work with containers, build container images, and then convert their
containerized applications into either single-node services to run on edge devices or into Kubernetes-based microservices.

## 1.2 A brief overview of containers

*Containers* are groups of processes running on a Linux system that are isolated from each other.

Containers make sure one group of processes does not interfere with other processes on the system. 

Rogue processes can't dominate system resources, which might prevent other processes from performing their
task. 

Hostile containers are also prevented from attacking other containers, stealing data, or causing denial of service attacks. 

A final goal of containers is allowing applications to be installed with their own versions of shared libraries that do not conflict with applications requiring different versions of the same libraries. Instead they allow applications to live in a virtualized environment, giving the impression that they own the entire system.

Containers are isolated via the following:

**Resource constraints (cgroups)**
	- The cgroup man page defines cgroups as the following: "Control groups, usually referred to as cgroups, are a Linux kernel feature which allow processes to be organized into hierarchical groups whose usage of various types of resources can then be limited and monitored."

Examples of resources controlled by cgroups include the following:

- The amount of memory a group of processes can use.
- The amount of CPU processes can use.
-  The amount of network resources a process can use.

The basic idea of cgroups is preventing one group of processes from dominating certain system resources in such a way that another group of processes can't make progress on the system.

*Security constraints*
- Containers are isolated from each other using many security tools available in the kernel. 
- The goal is blocking privilege escalation and preventing a rogue group of processes from committing hostile acts against the system, including the following examples:

-   Dropped Linux capabilities limit the power of root.
    -   SELinux controls access to the filesystem.
    -  There is read-only access to kernel filesystems.
    -  Seccomp limits the system calls available in the kernel.
    -  A user namespace to map one group of UIDs in the host to another allows access to limited root environments.

Further information:

**Linux  capabilities**
- Linux capabilities subdivide the power of root into distinct capabilities.  
- The capabilities man page is  a good overview of the available capabilities [https //bit.ly/3A3Ppeg]     

**SELinux**
- Security-Enhanced Linux (SELinux) is a Linux  kernel mechanism that labels every process and every filesystem object on the system. 
- A SELinux policy defines the rules on how labeled processes interact with label objects. The Linux kernel enforces the rules. 
- SELinux Coloring Book: https://github.com/mairin/selinux-coloring-book/blob/master/Print-Ready/Web.pdf
- SELinux notebook: https://github.com/SELinuxProject/selinux-notebook

**seccomp** 
- seccomp is a Linux kernel mechanism that limits the number of syscalls to a group of processes on the system. You can remove potentially dangerous syscalls from being called by the processes.
- The seccomp man page is a good source of additional information on seccomp: https://man7.org/linux/man-pages/man2/seccomp.2.html

**User namespaces**
- The user namespace allows you to have Linux capabilities within the group of UIDs and GIDs assigned to the namespace, without having root capabilities on the host. 


*Virtualization technologies (namespaces*)
- The Linux kernel employs a concept called *namespaces*, which creates virtualized environments, where one set of processes sees one set of resources, while another set of processes sees a different set of resources. 
- These virtualized environments eliminate processes' views into the rest of the system, giving them the feel of a virtual machine (VM) without the overhead. 

- Examples of namespaces:
    - *Network namespace*
	    - Eliminates the access to the host network but gives access to virtual network devices.
    - *Mount namespace*
	    - Eliminates the view of all the filesystem, except the containers filesystem.
    - *PID namespace*
	    - Eliminates the view of other processes on the system; container processes only see the processes within the container.

These container technologies have existed in the Linux kernel for many years. Security tools for isolating processes
started in Unix back in the 1970s, and SELinux started in 2001. Namespaces were introduced around 2004, and cgroups were introduced around 2006.

Windows container images exist, but this book concentrates on Linux-based containers. Even
when running Podman on Windows, you are still working with Linux containers. 

## 1.2.1 Container images: A new way to ship software

Containers really didn't take off until the Docker project introduced the concept of the container
image and container registry. Basically, they created a new way to ship
software.

Traditionally, installing multiple software applications on a Linux system has led to a problem of dependency
management. Before containers, you packaged software using package managers like RPM and Debian packages. 

These packages are installed on a host and share the content on the host, including shared libraries. When
developers test their code, everything might work fine when run on the host machine. 

The quality engineering team then might test the software on a different machine with different packages and see failures. Both
teams would need to work together to generate the proper requirements.

Finally, the software is shipped to customers, who have many different configurations and software installed, leading to further breakage of the application.

Container images solve the dependency management problem by bundling all the software needed to run your
application together into a unit. You ship all the libraries, executables, and configuration files together. The software is isolated
from the host via container technology. Usually the only part of the host system that your application interacts with is the host kernel.

The developer, quality engineers, and customer all run the exact same containerized environment along with the application. This helps guarantee consistency and limits the number of bugs caused by misconfiguration.

Containers are often compared to VMs in that they both can run multiple isolated applications on a single node. When
using VMs, you need to manage the entire VM operating system as well as the isolated application. 

You need to manage the life cycle of the different kernel, init system, logging, security updates, backups, and
so on. 

The system also has to deal with the overhead of the entire running operation system, not just the application. In the container world, all you run is the containerized application---there is no overhead and no additional OS management. 

Figure 1.4 shows three applications running in three different VMs.

![](images/01-04.png)

With VMs you end up needing to manage four operations systems, whereas with containers the three applications run with just their required user spaces. You end up managing just one operating system, as shown in figure 1.5.

![](images/01-05.png)

### 1.2.2 Container images lead to microservices

Packing applications inside of container images allows the installation of multiple applications with conflicting requirements on the same host. For example, one application might require a different version of the C library than another, which prevents them from being installed at the same time. 

Figure 1.6 shows a traditional application running within an operating system without use of containers. (Traditional LAMP stack (Linux, Apache, MariaDB, and PHP/PERL application))

![](images/01-06.png)

Containers can have the correct C library within their container image, with each image potentially having
different versions of the library specific to the container's application. You can run applications from totally different
distributions.

Containers make it easy to run multiple instances of the same application, as shown in figure 1.7. Container images encourage the packaging of a single service or application into a single container. Containers allow you to easily wire multiple applications together via the network. (LAMP stack packaged individually into microservice containers. As containers communicate via the network, they can be easily moved to other VMs, making reuse much easier.)

![](images/01-07.png)

Instead of designing monolithic applications in which you have a web frontend, a load balancer, and a database, you can build three different container images and then wire them together to build microservices. 

Microservices allow you and other users to experiment with running multiple databases and web frontends, then
orchestrate them together. 

Containerized microservices make the sharing and reuse of software possible.

### 1.2.3 Container image format

Container image consists of three components:
-  A directory tree containing all the software required to run your application.
-  A JSON file that describes the contents of the rootfs.
-  Another JSON file called a manifest list that links multiple images together to support different architectures.

The directory tree is called a *rootfs* (root filesystem). The software is laid out like it was the root (/) of a Linux system.

The executable to be run within the rootfs, the working directory, the environment variables to be used, the maintainer of the executable, and other labels to help identify the content of the image are defined in the first JSON file.

You can see this JSON file using the `podman inspect` command:
```bash
$ podman inspect docker:/ /registry.access.redhat.com/ubi8
{
...
  "created": "2022-01-27T16:00:30.397689Z",      ❶
  "architecture": "amd64",                       ❷
  "os": "linux",                                 ❸
  "config": {
         "Env": [                                ❹
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
            "container=oci"
         ],
         "Cmd": [                                ❺
                   "/bin/bash"
         ],
         "Labels": {                             ❻
                     "architecture": "x86_64",
                     "build-date": "2022-01-27T15:59:52.415605",
       ...
}
```

[❶] Date the image was created
[❷] Architecture for this image
[❸] Operating system for this image
[❹] Environment variables that the developer of the image wants to be set within the container
[❺] Default command to be executed when the container starts.
[❻] Labels to help describe the contents of the image. 

These fields can be free-form and do not affect the way images are run but can be used to search for and describe
the image.

The second JSON file, the manifest list, allows users on an arm64 machine to pull an image with the same name as they would if they were on an arm64 machine. 

Podman pulls the image based on the default architecture of the machine, using this manifest list. 

`skopeo` is a tool that uses the same underlying libraries as Podman and is available at [github.com/containers/skopeo](http://github.com/containers/skopeo){.url}. 

Skopeo provides lower-level output examining the structures of a container image. 

In the following example, use the `skopeo` command with the `--raw` option to examine the registry.access.redhat.com/ ubi8 image manifest specification:

```bash
$ skopeo inspect --raw docker:/ /registry.access.redhat.com/ubi8
{
    "manifests": [
      {
              "digest": "sha256:cbc1e8cea
➥ 8c78cfa1490c4f01b2be59d43ddbb
➥ ad6987d938def1960f64bcd02c",                                                   ❶
              "mediaType": "application/vnd.docker.distribution.manifest.v2+json",❷
              "platform": {
              "architecture": "amd64",                                            ❸
              "os": "linux"                                                       ❹
              },
              "size": 737
      },
      {
              "digest":                                                           ❺
➥ "sha256:f52d79a9d0a3c23e6ac4c3c8f2ed8d6337ea47f4e2dfd46201756160ca193308",
              "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
              "platform": {
              "architecture": "arm64", 
              "os": "linux"
              },
              "size": 737
      },
...
}
```

[❶] Digest of the exact image pulled when the architecture and OS match
[❷] mediaType describes the type of the image, OCI, Docker, and so on.
[❸] The architecture of this image digest: amd64
[❹] The OS of this image digest: Linux
[❺] This stanza points to a different image for a different architecture: arm64.

Images use the Linux tar utility to pack the rootfs and the JSON files together. 

These images are then stored on web servers called container registries (e.g., docker.io, quay.io, and Artifactory). 

Container engines like Podman can copy these images to a host and unpack them onto the filesystem. Then
the engine merges the image's JSON file, the engine's built-in defaults, and the user's input to create a new container OCI runtime specification JSON file. The JSON file describes how to run the containerized
application.

In the last step, the container engine launches a small program called a container runtime (e.g.,
`runc`, `crun`, `kata`, or `givisord`. 

The container runtime reads the container's JSON and instruments, kernel cgroups, security constraints, and namespaces before finally launching the primary process of the container.

### 1.2.4 Container standards

The OCI standards body defined the standard formats for storing and defining container images.

They also defined the standard for container engines running containers.

The OCI created the OCI Image Format, which standardizes the format of the container images and the images' JSON
file. 

They also created the OCI Runtime Specification, which standardized the container's JSON file to be used by OCI runtimes. The OCI standards allow other container engines, like Podman, to follow the standards and be able to work with all the images stored at container registries and to run them in the exact same way as all other container engines, including Docker (see figure 1.7).

## 1.3 Why use Podman when you have Docker?

I often get asked the question, "Why do you need Podman when you already have Docker?" Well one reason is that *open source is all about choice*. Operating systems have more than one editor, more than one shell, more than one filesystem, and more than one internet web browser. I believe that Podman's design is fundamentally better than Docker's and offers features that advance the security and use of containers.

## 1.3.1 Why have only one way to run containers?

One of Podman's advantages was that it was created long after Docker existed. Podman developers looked at ways to improve on Docker's design from a totally different perspective. Because Docker was written as open source, Podman shares some of the code and takes advantage of new standards, like the Open Container Initiative. Podman works with the open source community
to concentrate on developing new features.

In the rest of this section, I cover some of these improvements. Table 1.2 describes and compares features available
in Podman and Docker.

Table 1.2 Podman and Docker feature comparison

Supports all OCI and Docker images (Both)
- Pulls and runs container images from container registries (i.e., quay.io and docker.io). See chapter 2.

Launches OCI container engines (both)
- Launches `runc`, `crun`, Kata, gVisor, and OCI container engines. See appendix B.

Simple command-line interface (both)
- Podman and Docker share the same CLI. See chapter 2.

Integration with systemd (podman only)
- Podman supports running systemd inside the container as well as many systemd features. See chapter 7.

Fork/exec model (podman only)
- The container is a child of the command.

Fully supports user namespace (podman only)
- Only Podman supports running containers in separate user namespaces. See chapter 6.

Client-server model (both)
- Docker is a REST API daemon. Podman supports REST APIs via a systemd socket-activated service. See chapter 9.

Supports `docker-compose` (both)
- Compose scripts work against both REST APIs. Podman works in rootless mode. See chapter 9.

Supports docker-py (both)
- Docker-py Python bindings work against both REST APIs. Podman works in rootless mode. Podman also supports podman-py for running advanced features. See chapter 9.

Daemonless (podman only)
- The Podman command runs like a traditional command-line tool, while Docker requires multiple root-running daemons.

Supports Kubernetes-like pods (Podman only)
- Podman supports running multiple containers within the same pod. See chapter 4.

Supports Kubernetes YAML (Podman only)
- Podman can launch containers and pods based on Kubernetes YAML. It can also generate Kubernetes YAML from running containers. See chapter 8.

Supports Docker Swarm (Docker only)
- Podman believes the future for orchestrated multinode containers is Kubernetes and does not plan on implementing swarm.

Customizable registries (Podman only)
- Podman allows you to configure registries for short-name expansion. Docker is hardcoded to docker.io when you specify a short name. See chapter 5.

Customizable defaults (Podman only)
- Podman supports fully customizing all of its defaults, including security, namespaces, and volumes. See chapter 5.

macOS support (both)
- Podman and Docker support running containers on a Mac via a VM running Linux. See appendix E.

Windows support (both)
- Podman and Docker support running containers on a Windows WSL 2 or a VM running Linux. See appendix F.

Linux support (both)
- Podman and Docker are supported on all major Linux distributions. See appendix C.
### 1.3.2 Rootless containers

Probably the most significant feature of Podman is its ability to run in rootless mode. In many situations, you do not want to give full root access to your users, but users and developers still need to run containers and build container images. 

Requiring root access prevents lots of security-conscious companies from widespread adoption of Docker. Podman, on the other hand, can run containers with no additional security features in Linux other than a standard login account.

You can run the Docker client as a normal user by adding the user to the Docker user group (/etc/group), but I believe
granting this access is one of the most dangerous things you can do on a Linux machine. Access to the docker.sock allows you to gain full root access on the host by running the following command. In the command, you are mounting the entire host operating system / on the /host directory within the container. The `--privileged` flag turns off all container security, and then you `chroot` to /host. After the `chroot`, you are in a root shell at / of the operating system, with full root privileges:

```bash
$ docker run -ti --name hacker --privileged -v /:/host ubi8  chroot /host
#
```

At this point, you have full root privileges on the machine, and you can do whatever you want. When you are done hacking
the machine, you can simply execute the `docker` `rm` command to remove the container and all records of what you did:

```bash
$ docker rm hacker
```

When Docker is configured with default file logging, all records of your launching the container are erased. I believe this is far worse than setting up `sudo` without root, in that at least with `sudo`, you have the chance to see that `sudo` was run in your log
files.

With Podman, the processes running on the system are always owned by the user and have no capabilities greater than a normal user. Even if you break out of the container, the process is still running as your UID, and all actions on the system are recorded in
the audit logs. 

Users of Podman cannot simply remove the container and cover up their tracks. See chapter 6 for more information.

Docker now has the ability to run rootless similarly to Podman, but almost no one runs it
that way. Starting up multiple services in your home directory just to launch a single container has not caught on.

### 1.3.3 Fork/exec model

Docker is built as a REST API server. Fundamentally, Docker is a client-server architecture including multiple daemons. When a user executes the Docker client, they execute a command-line tool that connects to the Docker daemon. 

The Docker daemon then pulls images to its storage and then connects to the containerd daemon, which finally executes an OCI runtime that creates the container. The Docker daemon, then, is a communication platform that communicates reads and writes of `stdin`, `stdout`, and `stderr` from the initial process (PID1) created in the container. 

The daemon relays all of the output back to the Docker client. Users imagine the container's processes are just children of the current session, but there is a lot of communication going on behind the scenes. Figure 1.8 shows the Docker
client-server architecture.

![](images/01-08.png)

Figure 1.8 Docker client-server architecture. The container is a direct descendant of containerd, not the Docker
client. The kernel sees no relationship between the client program and the container.

The bottom line is the Docker client communicates with the Docker daemon, which then communicates with the containerd daemon, which finally launches an OCI runtime like `runc` to launch PID1 of the container. 

There is a lot of complexity involved in running containers in this way. Over the years, failures in any of the Daemons have led to
all containers shutting down, and it is often difficult to diagnose what happened. The core Podman engineering team comes from an operating system background grounded in the Unix philosophy.

Unix and C were designed with the fork/exec model of computing. Basically, when you execute a new program, a parent
program like the Bash shell forks a new process and then executes the new program as a child of the old program. 

The Podman engineering team thought they could make containers simpler by building a tool that pulls
container images from a container registry, configures container storage, and then launches an OCI runtime, which starts the container as a child of your container engine.

In the Unix operating system, processes can share content via the filesystem and inter-process communication (IPC)
mechanisms. These features of the operating system enable multiple container engines to share storage without requiring a daemon to be running to control access and share content. The engines do not need to communicate together aside from using locking mechanisms provided by the operating system's filesystems. 

Future chapters examine the advantages and disadvantages of this mechanism. 

Figure 1.9 shows the Podman architecture and communication flow.

![](images/01-09.png)

Figure 1.9 Podman fork/exec architecture. The user launches Podman, which executes the OCI runtime, which then launches the container. The container is a direct descendant of Podman.

### 1.3.4 Podman is daemonless

Podman is fundamentally different from Docker because it is daemonless. Podman can run all of
the same container images as Docker and launch containers with the same container runtimes. However, Podman does this without having multiple continuously root-running daemons.

Imagine you have a web service that you want to run at boot time. The web service is packaged in a container, so you need a container engine. In the Docker case, you need to set it up to be
running on your machine with each of the daemons running and accepting connections. Next, launch the Docker client to start the web service.

Now you have your containerized application running as well as all of the Docker daemons. 

In the Podman case, use the Podman command to launch your container, and Podman will go away. 

Your container will continue to run without the overhead of running the multiple daemons. Less overhead is incredibly popular on low-end machines like IOT devices and edge servers.

## 1.3.5 User-friendly command line

One of the great features of Docker is the simple command-line interface. There have been
other container command lines like `RKT`, `lxc`, and `lxcd`, but they have their own command-line interfaces. 

The Podman team realized early on that it wouldn't gain market share if Podman had its own command-line interface. Docker was the dominant tool, and almost everyone who had
played with containers had done it with its CLI. In addition, if you were to search how to do something with a container online, invariably you would get an example using the Docker command line. 

Right from the start, Podman had to match the Docker command line. A motto for
replacing Docker with Podman was quickly developed:
`alias` `Docker` = `Podman`.

With this command, you can continue to type in your Docker commands, but Podman runs your
containers. If the Podman command line differs from Docker, it is considered a bug in Podman, and users demand Podman to be fixed to make the tools match. 

There are a few commands, such as Docker Swarm, that Podman doesn't support, but for the most part, Podman is a complete replacement for the Docker CLI.

Many distributions supply a package called `podman-docker`, which
changes the alias from *docker* to *podman* and links the man page. The alias means when you type `docker ps`, the `podmanps` command runs. 

If you execute `man docker ps`, the Podman `ps` man pages show up. Figure 1.10 is a twitter message from a Podman user who aliased the `docker` command to `podman`, and was surprised to remember he had been using Podman for two months while thinking he was using Docker.

![](images/01-10.png)

### 1.3.6 Support for REST API

Podman can be run as a socket-activated REST API service. This allows remote clients to manage
and launch Podman containers. 

Podman supports the Docker API as well as the Podman API for advanced Podman features. Through the use of the Docker API, Podman supports `docker-compose` and other users of the docker-py Python bindings. This means that even if you built your infrastructure around using the Docker
socket for launching containers, you can simply replace Docker with the Podman service and continue to use your existing scripts and tools.

The Podman REST API also allows remote Podman clients on Mac, Windows, and Linux systems to interact with Podman containers on a Linux machine. 

### 1.3.7 Integration with systemd

Systemd is the fundamental init system in the operating systems. The init process on a Linux system
is the first process that is started by the kernel on boot. Therefore, the init system is the ancestor of all processes and can monitor them all. Podman wants to fully integrate the running of containers with the init system. Users want to use systemd to start and stop containers at boot time. Containers should do the following:

-   Support systemd within a container.
-   Support socket activation.
-   Support systemd notifications that a containerized application is fully activated.
-   Allow systemd to fully manage the cgroups and lifespan of a containerized application.

Basically, containers work as services in systemd unit files. Many developers want to run systemd within a container to run multiple system-defined services within a container.

However, the upstream Docker community disagrees with this and has denied all pull requests that attempt to integrate systemd into Docker. They believe Docker should manage the life cycle of the container, and they do not want to accommodate users who want to run systemd in a container.

The upstream Docker community believes the Docker daemon, as opposed to systemd, should be the controller of processes, it should manage the life cycle of containers, and it should
start and stop them at boot time. The problem is there are many more features in systemd than in Docker, including startup ordering, socket activation, service ready notifications, and so on. Figure 1.11 is an actual badge of a Docker employee at the first DockerCon, illustrating their hostility towards systemd.

![](images/01-11.png)

Figure 1.11 Docker employee badge at DockerCon

When Podman was designed, the developers wanted to make sure it fully integrated with systemd. 

When you run systemd inside a container, Podman sets up the container the way systemd expects
and allows it to simply run as PID1 of the container with limited privileges. Podman allows you to run services within the container the same way they run on a system or in a VM: via systemd unit files. 

Podman supports socket activation, service notifications, and many other systemd unit file features. 

Podman makes it simple to generate systemd unit files with best practices for running containers within a systemd service. For more information, see chapter 7 on systemd integration.

The Containers project ([https://github.com/containers](https://github.com/containers){.url}) where Podman, container libraries, and other container management tools reside, wants to embrace all features of the operating system and fully integrate it. Chapter 7 explains Podman integration with systemd.

### 1.3.8 Pods

One advantage of Podman  is described in its name. As mentioned earlier, *Podman* is actually short for *Pod Manager*. As the official Kubernetes documentation puts it, "A pod (as in a pod of seals, hence the logo, or pea pod) is a group of one or more containers, with shared storage/network resources, and a specification for how to run the containers." 

Podman works with either a single container at a time, like Docker, or it can manage groups of containers together in a pod. 

One of the design goals of containers is to separate services into single containers: microservices. 

Then you combine containers together to build larger services. Pods allow you to group multiple services together to form a larger service managed as a single entity. One of the goals of Podman is allowing you to experiment with pods. Figure 1.12 shows two pods running on a system, each pod
containing three containers.

![](images/01-12.png)

Two pods running on a host. Each pod runs two different containers along with the infra container.

`podman generate kube` command
- Allows you to generate Kubernetes YAML files from running containers and pods. S

`podman play kube` command
- Allows you to play Kubernetes YAML files and generate pods and containers on your host. 

I suggest using Podman for running pods and containers on a single host and using Kubernetes to take your pods and containers and run them on multiple machines and all through your infrastructure. 

Other projects, like kind ([https://kind.sigs.k8s.io/docs/user/rootless](https://kind.sigs.k8s.io/docs/user/rootless){.url}), are experimenting with running pods with Podman under the guidance of Kubernetes.

### 1.3.9 Customizable registries

Container engines like Podman support the concept of pulling images using short names, such as ubi8, without specifying the registry in which they reside: [registry.access.redhat.com](http://registry.access.redhat.com)

Complete image names include the name of the container registry they
were pulled from: [registry.access.redhat.com/library/ubi8:latest](http://registry.access.redhat.com/library/ubi8:latest)

Table 1.3 shows the components of the image name broken out.

Table 1.3 Short name to container image name table:

| Name          | Registry                   | Repo    | Name | Tag      |
| ------------- | -------------------------- | ------- | ---- | -------- |
| Short name    |                            |         | ubi8 |          |
| Complete name | registry.access.redhat.com | library | ubi8 | `latest` |

Docker is hardcoded to always pull from [https://docker.io](https://docker.io) when using a short name. If
you want to pull an image from a different container registry, you must fully specify the image. 

In the following example, I attempt to pull ubi8/httpd-24, and it fails because the container image is not on docker.io. The image is on [registry.access.redhat.com](http://registry.access.redhat.com):

```bash
# docker pull ubi8/httpd-24
Using default tag: latest
Error response from daemon: pull access denied for ubi8/httpd-24, 
repository does not exist or may require 'docker login': denied: requested 
access to the resource is denied
```

So if I want to use ubi8/httpd-24, I am forced to type the entire name, including the registry:

```bash
# docker pull registry.access.redhat.com/ubi8/httpd-24
```

The Docker engine gives docker.io an advantage over other container registries as the preferred registry. Podman was designed to allow you to specify multiple registries, like what you can
do with `dnf`, `yum`, and `apt` tools for installing packages. You can even remove docker.io from the list. If you attempt to pull ubi8/httpd-24 with Podman, Podman presents you with a list of registries to choose from:

``` programlisting
$ podman pull ubi8/httpd-24
? Please select an image:  
    registry.fedoraproject.org/ubi8/httpd-24:latest
       ▸ registry.access.redhat.com/ubi8/httpd-24:latest
    docker.io/ubi8/httpd-24:latest
    quay.io/ubi8/httpd-24:latest
```

Once you make your decision, Podman records the short-name alias and no longer prompts and uses the previously selected registry. Podman supports lots of other features, like blocking registries, only pulling signed images, setting up image mirrors, and specifying hardcoded short names, so specific short names map directly to the long names.

### 1.3.10 Multiple transports

Podman supports many different container image sources and targets called *transports* (see table 1.4). Podman can pull images from container registries and from local containers storage but also supports images stored in OCI format, OCI TAR format, legacy Docker TAR format, directory format, and images directly from the Docker daemon. 

Podman commands can easily run images from each of the formats.

### Podman-supported transports

Container registry(`docker`)
- References a container image stored in a remote container image registry website. Registries store and share container images (e.g., docker.io and quay.io).

`oci`
- References a container image compliant with OCI layout specifications. The manifest and layer tarballs are located in the local directory as individual files.

`dir`
- References a container image compliant with the Docker image layout, similar to the `oci` transport but storing the files using the legacy `docker` format.

`docker-archive`
- References a container image in a Docker image layout that is packed into a TAR archive.

`oci-archive`
- References a container image compliant with OCI layout specifications that is packed into a TAR archive.

`docker-daemon`
- References an image stored in the Docker daemon’s internal storage.

`container-storage`
- References a container image located in a local storage. Podman defaults to using container storage for local images.

## 1.3.11 Complete customizability

Container engines tend to have lots of built-in constants, like the namespaces they run with, whether or not SELinux is enabled, and which capabilities containers run with.

With Docker, most of these values are hardcoded and cannot be changed by default. Podman, on the other hand, has a very customizable configuration.

Podman has its built-in defaults but defines three locations for its configuration files to be stored:
-   */usr/share/containers/containers.conf* 
	- Where a distribution can define the changes the distribution likes to use

-   */etc/containers/containers.conf*
	- Where they can set up system overrides

-   *\$HOME/.config/containers/containers.conf*
	- Can be specified only in rootless mode

The configuration files allow you to configure Podman to run the way you want by default. You can even run with more
security by default if you choose.

### 1.3.12 User-namespace support

Podman is fully integrated with the user namespace. Rootless mode relies on user namespaces, which
allows for multiple UIDs to be assigned to a user. User namespaces provide isolation between users on a system, so you can have multiple rootless users running containers with multiple user IDs, all isolated
from each other.

A user namespace can be used to isolate containers from each other. Podman makes it simple to launch multiple
containers, each with a unique user namespace. The kernel then isolates the processes from host users as well as each other based on UID separation.

Docker only supports running containers in a single, separate, user namespace, meaning all containers run within the
same user namespace. Root in one container is the same as root in another container. It does not support running each container in a different user namespace, which means containers attack each other from a user-namespace perspective. Even though Docker supports this mode, almost no one runs containers with Docker in a separate user namespace.

## 1.4 When not to use Podman

Like Docker, Podman is not a container orchestrator. Podman is a tool for running container
workloads on a single host in either rootless or rootful mode.

Higher-level tools are required if you want to orchestrate running containers on multiple machines.

I believe the best tool for doing this now is Kubernetes. Kubernetes won the container orchestrator war when it comes
to mind share. Docker has an orchestrator called Swarm, which had some popularity, but it now seems to be out of favor. Because the Podman team believes Kubernetes is the way to go for containers on multiple machines, Podman does not support Swarm functionality. 

Podman has been used for different orchestrators and is used for grid/HPC computing, and open source developers have even added it to Kubernetes frontends.

Summary

- Containers technology has been around for many years, but the introduction of container images and container registries allows developers a better way to ship software.

- Podman is an excellent container engine, suitable for almost all of your single-node container projects. It is useful for developing, building, and running containerized applications.

- Podman is as simple to use as Docker, with the exact same command-line interface.

- Podman supports a REST API, which allows remote tools and languages, including
    `docker-compose`, to work with Podman containers.

- Unlike Docker, Podman includes such notable features as user-namespace support, multiple transports, customizable registries, integration with systems, the fork/exec model, and out-of-the-box rootless mode.

- Podman is a more secure way to run containers.

------------------------------------------------------------------------

Other container engines include Buildah, CRI-O, containerd, and many others.


