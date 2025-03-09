# Appendix A. Podman-related container tools 

[]{#A.htm#pgfId-1277656}[]{#A.htm#id_ypyilma7lhnr}This appendix
[]{#A.htm#marker-1277655}describes the three tools that use
containers/storage and containers/image libraries. These tools address
the following functionalities:

-   []{#A.htm#pgfId-1277657 .calibre17}Moving container images between
    different container registries and storage

-   []{#A.htm#pgfId-1277658 .calibre17}Building container images

-   []{#A.htm#pgfId-1277659 .calibre17}Testing, developing, and running
    containers in production on a single node

-   []{#A.htm#pgfId-1277660 .calibre17}Running containers in production
    at scale

[]{#A.htm#pgfId-1277661}As the original creator of Podman, I recognized
the need for specialized tools, each performing specific functionality
rather than a one-size-fits-all monolithic solution.

[]{#A.htm#pgfId-1277662}From a security perspective, each of these four
categories requires different security constraints. Containers running
in production need to be run in a more secure environment than ones
running in development and testing. Moving container images between
registries requires no privileged access to the host you are running the
command on---only remote access to the registries. You will have the
least secure system with a monolithic daemon. If my containers need more
access during builds, then in production, they get the same access as
during builds.

[]{#A.htm#pgfId-1277664}Another critical problem with a monolithic
daemon is that it prevents experimentation with the tools and doesn't
allow them to go their own way. One example of this is when we proposed
a change to the Docker daemon to allow users to pull different types of
OCI content off of container registries. This change was denied, as it
had little to do with Docker containers.

[]{#A.htm#pgfId-1277665}Similarly, when the monolithic daemon is
modified for one product, it can negatively affect features of another
one using that daemon. It could cause performance degradation or
complete breakage. This happened when Kubernetes was being developed,
since it relied on the Docker daemon as the container engine. But since
Docker is monolithic and being developed for many other projects, many
of its changes affected Kubernetes, leading to instability. It was
obvious that Kubernetes needed a dedicated container engine for its
workloads, and in December 2020 it was announced that Kubernetes will
eventually use the newly developed standard, the Container Runtime
Interface (CRI; see [http://mng.bz/yaDq](http://mng.bz/yaDq){.url}) to
improve interaction between orchestrators and different container
runtimes. I wrote a coloring book, *The Container
Commandos*[]{#A.htm#marker-1282235} (figure A.1;
[https://red.ht/3gfVlHF](https://red.ht/3gfVlHF){.url}), illustrated by
Máirín Duffy[]{#A.htm#marker-1282237} (@marin), describing the container
tools talked about in this appendix, based on superheroes.

::: figure
![](images/OEBPS/Images/A-01.png){.calibre18}

Figure A.1 The Container Coloring Book
([https://red.ht/3gfVlHF](https://red.ht/3gfVlHF){.url})
:::

[]{#A.htm#pgfId-1277678}Finally, sometimes there are conflicting
interests or release schedules in play. Having separate, independent
tools allows releases to be deployed independently from all the others
at their own pace to guarantee new features to their customers. Four
projects were created for the distinct functions described in table A.1.

[]{#A.htm#pgfId-1280253}Table A.1 Primary container tools based on
containers/storage and containers/image.

+-----------------+-----------------------------------------------------+
| []{#A.htm#pgf   | []{#A.htm#pgfId-1280259}Description                 |
| Id-1280257}Tool |                                                     |
+-----------------+-----------------------------------------------------+
| []{#A.htm#      | []{#A.htm#pgfId-1280263}Performs various operations |
| pgfId-1280261}S | on container images and image repositories          |
| kopeo[]{#A.htm# | ([https://github.com/containers/s                   |
| marker-1282252} | kopeo](https://github.com/containers/skopeo){.url}) |
+-----------------+-----------------------------------------------------+
| []{#A.htm#p     | []{#A.htm#pgfId-1280267}Facilitates a wide range of |
| gfId-1280265}Bu | operations on container images                      |
| ildah[]{#A.htm# | ([https://github.com/containers/bui                 |
| marker-1282258} | ldah](https://github.com/containers/buildah){.url}) |
+-----------------+-----------------------------------------------------+
| []{#A.htm#      | []{#A.htm#pgfId-1280271}All-in-one management tool  |
| pgfId-1280269}P | for pods, containers, and images                    |
| odman[]{#A.htm# | ([https://github.com/containers/p                   |
| marker-1282264} | odman](https://github.com/containers/podman){.url}) |
+-----------------+-----------------------------------------------------+
| []{#A.htm       | []{#A.htm#pgfId-1280275}OCI-based implementation of |
| #pgfId-1280273} | the Kubernetes Container Runtime Interface          |
| CRI-O[]{#A.htm# | ([https://github.com/                               |
| marker-1282270} | cri-o/cri-o](https://github.com/cri-o/cri-o){.url}) |
+-----------------+-----------------------------------------------------+

[]{#A.htm#pgfId-1277711}As you have already learned a great deal about
Podman, you know now why it is included in this list. Podman is an
excellent tool for understanding and developing containers as well as
pods and images. It encapsulates everything Docker CLI does but without
locking everything under one central daemon. Because Podman works
without a daemon and uses the operating system for sharing data, other
tools can work with the same data stores and libraries. The rest of this
appendix describes the rest of the tools, starting with Skopeo (figure
A.2).

::: figure
![](images/OEBPS/Images/A-02.png){.calibre18}

Figure A.2 Skopeo, Buildah, and Podman work together by sharing the same
containers/storage images and containers/image library for pulling and
pushing images.
:::

## []{#A.htm#pgfId-1277719}A.1 Skopeo {#A.htm#heading_id_3 .fm-head}

[]{#A.htm#pgfId-1277722}While
[]{#A.htm#marker-1280942}[]{#A.htm#marker-1280943}using container
engines like Docker or Podman, if you want to inspect a container image
in a registry, you are required to pull this image from the registry to
your local storage. Only then can you examine it. The problem is that
this image can be huge, and after inspecting it, you might realize it
wasn't what you expected, and you wasted time pulling it. Because the
protocol used to pull the image and inspect it is just a web protocol, a
simple tool, Skopeo, was created to pull the image's detailed
information and display it on the screen. *Skopeo* is the Greek word for
*remote viewing*.

::: figure
![](images/OEBPS/Images/A-UN01.png){.calibre18}
:::

[]{#A.htm#pgfId-1277728}Execute the following `skopeo`{.fm-code-in-text}
`inspect`{.fm-code-in-text} command[]{#A.htm#marker-1280173} to examine
an image's detailed information in JSON form:

``` programlisting
$ skopeo inspect docker:/ /quay.io/rhatdan/myimage
{
  "Name": "quay.io/rhatdan/myimage",
  "Digest":
"sha256:fe798c1576dc7b70d7de3b3ab7c72cd22300b061921f052279d88729708092d8",
  "RepoTags": [
      "Latest",
      "1.0"
  ],
...
```

[]{#A.htm#pgfId-1277739}Skopeo was extended to also copy images off of
registries. Eventually, Skopeo became the tool for copying images
between different types of storage (transports). These types of storage
became the transports defined in table A.2.

[]{#A.htm#pgfId-1280371}Table A.2 Podman-supported transports

+-----------------+-----------------------------------------------------+
| []{             | []{#A.htm#pgfId-1280377}Description                 |
| #A.htm#pgfId-12 |                                                     |
| 80375}Transport |                                                     |
+-----------------+-----------------------------------------------------+
| []{             | []{#A.htm#pgfId-1280381}This is the default         |
| #A.htm#pgfId-12 | transport. It references a container image stored   |
| 80379}Container | in a remote container image registry website.       |
| registry        | Registries store and share container images (e.g.,  |
| (`docker`       | docker.io and quay.io).                             |
| {.fm-code-in-te |                                                     |
| xt1}[]{#A.htm#m |                                                     |
| arker-1280406}) |                                                     |
+-----------------+-----------------------------------------------------+
| []{#A.htm#pgf   | []{#A.htm#pgfId-1280385}References a container      |
| Id-1280383}`oci | image; compliant with the Open Container Initiative |
| `{.fm-code-in-t | Format specification. The manifest and layer        |
| ext1}[]{#A.htm# | tarballs are located in the local directory as      |
| marker-1280407} | individual files.                                   |
+-----------------+-----------------------------------------------------+
| []{#A.htm#pgf   | []{#A.htm#pgfId-1280389}References a container      |
| Id-1280387}`dir | image; compliant with the Docker image layout. It   |
| `{.fm-code-in-t | is very similar to the `oci`{.fm-code-in-text1}     |
| ext1}[]{#A.htm# | transport but stores the files using the legacy     |
| marker-1280408} | Docker format. As a non-standardized format, it is  |
|                 | primarily useful for debugging or noninvasive       |
|                 | container inspection.                               |
+-----------------+-----------------------------------------------------+
| []{#A.htm       | []{#A.htm#pgfId-1280393}References a container      |
| #pgfId-1280391} | image in a Docker image layout, which is packed     |
| `docker-archive | into a TAR archive.                                 |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#A.htm# |                                                     |
| marker-1280409} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#A.          | []{#A.htm#pgfId-1280397}References an image         |
| htm#pgfId-12803 | compliant with the Open Container Initiative Format |
| 95}`oci-archive | specification, which is packed into a TAR archive.  |
| `{.fm-code-in-t | It is very similar to the                           |
| ext1}[]{#A.htm# | `docker-archive`{.fm-code-in-text1} transport but   |
| marker-1280410} | stores an image in OCI format.                      |
+-----------------+-----------------------------------------------------+
| []{#A.ht        | []{#A.htm#pgfId-1280401}References an image stored  |
| m#pgfId-1280399 | in the Docker daemon's internal storage. Since the  |
| }`docker-daemon | Docker daemon requires root privileges, Podman has  |
| `{.fm-code-in-t | to be run by the root user.                         |
| ext1}[]{#A.htm# |                                                     |
| marker-1280411} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#A.htm#pg    | []{#A.htm#pgfId-1280405}References an image located |
| fId-1280403}`co | in a local container storage. It is not a transport |
| ntainer-storage | but more of a mechanism for storing images. It can  |
| `{.fm-code-in-t | be used to convert other transports into            |
| ext1}[]{#A.htm# | `container-storage`{.fm-code-in-text1}. Podman      |
| marker-1280412} | defaults to using                                   |
|                 | `container-storage`{.fm-code-in-text1} for local    |
|                 | images.                                             |
+-----------------+-----------------------------------------------------+

[]{#A.htm#pgfId-1277784}Other container engines and tools wanted to use
the functionality developed in Skopeo to copy images, so Skopeo was
split in two: the command line, Skopeo, and the underlying library,
containers/image[]{#A.htm#marker-1277785}. Splitting functionality into
a separate library made it possible to build other container tools,
including Podman.

[]{#A.htm#pgfId-1277788}The `skopeo`{.fm-code-in-text}
`copy`{.fm-code-in-text} command[]{#A.htm#marker-1277787} is very
popular for copying images between different types of container storage.
One difference compared to Podman and Buildah, as you'll see in section
A.2, is that Skopeo forces users to specify the transport for the source
and destination. Podman and Buildah default to using the
`docker`{.fm-code-in-text}[]{#A.htm#marker-1277790} or
`containers-storage`{.fm-code-in-text}
transport[]{#A.htm#marker-1277791}, depending on the context and
command. In the following example, you will copy an image from a
container registry using the `docker`{.fm-code-in-text}
transport[]{#A.htm#marker-1277792} and store the image locally using the
`container-storage`{.fm-code-in-text} transport:

``` programlisting
$ skopeo copy docker:/ /quay.io/rhatdan/myimage containers-storage:quay.io/rhatdan/myimage
Getting image source signatures
Copying blob dfd8c625d022 done 
Copying blob 68e8857e6dcb done 
Copying blob e21480a19686 done 
Copying blob fbfcc23454c6 done 
Copying blob 3f412c5136dd done 
Copying config 2c7e43d880 done 
Writing manifest to image destination
Storing signatures
```

[]{#A.htm#pgfId-1277805}Another command many Skopeo users use is
`skopeo`{.fm-code-in-text}
`sync`{.fm-code-in-text}[]{#A.htm#marker-1277804}, which lets you
synchronize images between container registries and local storage.

[]{#A.htm#pgfId-1277806}Skopeo is mainly used for infrastructure
projects to help provision multiple container registries---for example,
copying images from a public registry to a private one. Table A.3
describes the most popular commands used with Skopeo. One of the first
tools to take advantage of the containers/image library was
[]{#A.htm#marker-1277807}[]{#A.htm#marker-1277808}Buildah.

[]{#A.htm#pgfId-1280501}Table A.3 Primary Skopeo commands and their
description

+-----------------+-----------------------------------------------------+
| [               | []{#A.htm#pgfId-1280507}Description                 |
| ]{#A.htm#pgfId- |                                                     |
| 1280505}Command |                                                     |
+-----------------+-----------------------------------------------------+
| []{#A.          | []{#A.htm#pgfId-1280511}Copy an image (manifest,    |
| htm#pgfId-12805 | filesystem layers, or signatures) from one location |
| 09}`skopeo copy | to another.                                         |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#A.htm# |                                                     |
| marker-1280540} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#A.          | []{#A.htm#pgfId-1280515}Mark the image name for     |
| htm#pgfId-12805 | later deletion by the registry's garbage collector. |
| 13}`skopeo`{.fm |                                                     |
| -code-in-text1} |                                                     |
| `delete         |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#A.htm# |                                                     |
| marker-1280541} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#A.          | []{#A.htm#pgfId-1280519}Return low-level            |
| htm#pgfId-12805 | information about an image name in a registry.      |
| 17}`skopeo`{.fm |                                                     |
| -code-in-text1} |                                                     |
| `inspect        |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#A.htm# |                                                     |
| marker-1280542} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#A.          | []{#A.htm#pgfId-1280523}List tags in the            |
| htm#pgfId-12805 | transport-specific image repository.                |
| 21}`skopeo`{.fm |                                                     |
| -code-in-text1} |                                                     |
| `list-tags      |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#A.htm# |                                                     |
| marker-1280543} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#A.          | []{#A.htm#pgfId-1280527}Log in to a container       |
| htm#pgfId-12805 | registry (the same as `podman`{.fm-code-in-text1}   |
| 25}`skopeo`{.fm | `login`{.fm-code-in-text1}).                        |
| -code-in-text1} |                                                     |
| `login          |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#A.htm# |                                                     |
| marker-1280544} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#A.          | []{#A.htm#pgfId-1280531}Log out of a container      |
| htm#pgfId-12805 | registry (the same as `podman`{.fm-code-in-text1}   |
| 29}`skopeo`{.fm | `logout`{.fm-code-in-text1}).                       |
| -code-in-text1} |                                                     |
| `logout         |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#A.htm# |                                                     |
| marker-1280545} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#A.          | []{#A.htm#pgfId-1280535}Compute a manifest digest   |
| htm#pgfId-12805 | for a manifest file, and write it to standard       |
| 33}`skopeo`{.fm | output.                                             |
| -code-in-text1} |                                                     |
| `manifest`{.fm  |                                                     |
| -code-in-text1} |                                                     |
| `digest         |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#A.htm# |                                                     |
| marker-1280546} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#A.          | []{#A.htm#pgfId-1280539}Synchronize images between  |
| htm#pgfId-12805 | container registries and local directories.         |
| 37}`skopeo`{.fm |                                                     |
| -code-in-text1} |                                                     |
| `sync           |                                                     |
| `{.fm-code-in-t |                                                     |
| ext1}[]{#A.htm# |                                                     |
| marker-1280547} |                                                     |
+-----------------+-----------------------------------------------------+

## []{#A.htm#pgfId-1277858}A.2 Buildah {#A.htm#heading_id_4 .fm-head}

[]{#A.htm#pgfId-1277861}As
[]{#A.htm#marker-1277859}[]{#A.htm#marker-1277860}you learned in section
1.1.2, creating a container image means creating a directory on disk and
adding content to it to make it look like the root,
`/`{.fm-code-in-text}, directory on a Linux machine, called a rootfs.
Originally, the only way to do this was with `docker`{.fm-code-in-text}
`build`{.fm-code-in-text}, using a Dockerfile. While Dockerfiles and
Containerfiles are excellent ways of creating recipes for your container
images, a low-level building block tool that allowed other ways to build
container images was needed---one that allowed breaking the image-build
process into individual commands, letting you use other more powerful
scripting tools and languages than Containerfile to build images. We
created a tool called Buildah[]{#A.htm#marker-1277862}
([https://buildah.io](https://buildah.io){.url}) to serve this purpose.

::: figure
![](images/OEBPS/Images/A-UN02.png){.calibre18}
:::

[]{#A.htm#pgfId-1277867}Buildah was designed to be that simple tool for
building container images. It's built on top of the container/storage
and container/image libraries, just like Podman and Skopeo. It has a lot
of functionality similar to Podman. You can pull images, push images,
commit images, and even run containers on images. What mainly
differentiates Podman from Buildah is the underlying concept of a
*container*[]{#A.htm#marker-1277868}. A Podman container is a long-lived
one, a *running* container[]{#A.htm#marker-1277869}, while a Buildah
container is just a temporary one, a *working* container, which will be
used to create an OCI image.

[]{#A.htm#pgfId-1277870}[Note]{.fm-callout-head} Buildah is a Linux-only
tool, not available on Mac or Windows. However, Podman embeds Buildah in
the `podman`{.fm-code-in-text1} `build`{.fm-code-in-text1}
command[]{#A.htm#marker-1277871}. Podman on Mac and Windows uses the
Buildah code on the server side, allowing those platforms to build using
Containerfiles and Dockerfiles. See appendixes E and F for more
information.

[]{#A.htm#marker-1282679}[]{#A.htm#pgfId-1277872}Buildah was designed to
take the steps defined in a Dockerfile and make them available at the
command line. Buildah wanted to simplify building a container image by
allowing you to use all of the tools available within the OS to populate
the image. You can add data to this directory via standard Linux tools,
like `cp`{.fm-code-in-text}[]{#A.htm#marker-1277873},
`make`{.fm-code-in-text}[]{#A.htm#marker-1277874},
`yum`{.fm-code-in-text}
`install`{.fm-code-in-text}[]{#A.htm#marker-1277875}, and so on. Then
commit the rootfs into a tarball, add some JSON to describe what the
creator of the image wanted the image to do, and finally, push this to a
container registry. Basically, Buildah breaks down the steps you learned
about in a Containerfile into individual commands you can execute from a
shell.

[]{#A.htm#pgfId-1277876}[Note]{.fm-callout-head} The name *Buildah* is a
play on the way I pronounce *builder*. If you ever heard me speak, you'd
notice I have a strong Boston accent. When the core team asked what I
wanted to call the tool, I said, "I don't care, just call it *builder*."
And they heard *Buildah*.

[]{#A.htm#pgfId-1277877}The first step when building a new container
image is pulling a base image. In a Containerfile, this is done with the
`FROM`{.fm-code-in-text} instruction[]{#A.htm#marker-1284237}.

### []{#A.htm#pgfId-1277883}A.2.1 Creating a working container from a base image {#A.htm#heading_id_5 .fm-head1}

[]{#A.htm#pgfId-1277888}The
[]{#A.htm#marker-1284239}[]{#A.htm#marker-1284240}[]{#A.htm#marker-1284241}first
command to look at is `buildah`{.fm-code-in-text}
`from`{.fm-code-in-text}[]{#A.htm#marker-1284242}. It is equivalent to
the Containerfile's `FROM`{.fm-code-in-text}
instruction[]{#A.htm#marker-1284244}. When executing
`buildah`{.fm-code-in-text} `from`{.fm-code-in-text}
`IMAGE`{.fm-code-in-text}, it pulls the specified image from the
container registry, saves it in a local container storage, and creates a
working container based on this image. As mentioned previously, this
container is similar to a Podman container, except it exists temporarily
only to become a container image. In the following example, a working
container is created based on an ubi8-init image.

[]{#A.htm#pgfId-1277890}Listing A.1 Buildah pulling an image and
creating a Buildah container

``` programlisting
$ buildah from ubi8-init
Resolved "ubi8-init" as an alias (/etc/containers/registries.conf.d/
➥ 000-shortnames.conf) 
Trying to pull registry.access.redhat.com/
➥ ubi8-init:latest...                        ❶
Getting image source signatures
Checking if image destination supports signatures
Copying blob adffa6963146 done 
Copying blob 29250971c1d2 done 
Copying blob 26f1167feaf7 done 
Copying config 4b85030f92 done 
Writing manifest to image destination
Storing signatures
ubi8-init-working-container                 ❷
```

[]{#A.htm#pgfId-1283956}[❶]{.fm-combinumeral} Pulls the image from the
container registry

[]{#A.htm#pgfId-1283957}[❷]{.fm-combinumeral} Outputs a new container
name

[]{#A.htm#pgfId-1277907}Notice that the `buildah`{.fm-code-in-text}
`from`{.fm-code-in-text} output looks the same as the
`podman`{.fm-code-in-text} `pull`{.fm-code-in-text} output, except for
the last line, which outputs the container name:
`ubi8-init-working-container`{.fm-code-in-text}. If you run the
`buildah`{.fm-code-in-text} `from`{.fm-code-in-text}
command[]{#A.htm#marker-1277908} again, you get a second container name:

``` programlisting
$ buildah from ubi8-init
ubi8-init-working-container-1
```

[]{#A.htm#pgfId-1277911}Buildah keeps track of its containers and
generates each one by incrementing a counter. Of course you can override
the container name with the `--name`{.fm-code-in-text}
option[]{#A.htm#marker-1277912}. Next, you will add content to this
container
[]{#A.htm#marker-1277913}[]{#A.htm#marker-1277914}[]{#A.htm#marker-1277915}image.

### []{#A.htm#pgfId-1277917}A.2.2 Adding data to a working container {#A.htm#heading_id_6 .fm-head1}

[]{#A.htm#pgfId-1277923}Buildah
[]{#A.htm#marker-1277918}[]{#A.htm#marker-1277919}[]{#A.htm#marker-1277920}has
two commands, `buildah`{.fm-code-in-text}
`copy`{.fm-code-in-text}[]{#A.htm#marker-1277921} and
`buildah`{.fm-code-in-text}
`add`{.fm-code-in-text}[]{#A.htm#marker-1277922}, for copying the
contents of a file, URL, or directory into the container's working
directory. They map to the same functionality as the Containerfile's
`COPY`{.fm-code-in-text}[]{#A.htm#marker-1277924} and
`ADD`{.fm-code-in-text} instructions[]{#A.htm#marker-1277925}.

[]{#A.htm#pgfId-1277926}[Note]{.fm-callout-head} It is somewhat
confusing to have two commands that do almost the same thing. In most
cases, I recommend you just use `buildah`{.fm-code-in-text1}
`copy`{.fm-code-in-text1} and `COPY`{.fm-code-in-text1} inside a
Containerfile. The main difference between the two is that
`COPY`{.fm-code-in-text1} only copies local files and directories off of
the host into the container image. The `add`{.fm-code-in-text1}
command[]{#A.htm#marker-1277927} supports the use of URLs to pull remote
content and insert it into your container. The `ADD`{.fm-code-in-text1}
command[]{#A.htm#marker-1277928} also supports taking TAR and ZIP files
and expanding them when copied into the container image.

[]{#A.htm#pgfId-1277930}The syntax of the `buildah`{.fm-code-in-text}
`copy`{.fm-code-in-text} command requires you to specify the name of the
container previously created by the `buildah`{.fm-code-in-text}
`from`{.fm-code-in-text} command, followed by the source and,
optionally, destination. If the destination is not provided, source data
will be copied into the container's working directory. The destination
directory will be created if it doesn't exist yet.

[]{#A.htm#pgfId-1277932}The following example copies the local
html/index.html file (created previously in section 3.1) into the
/var/lib/www/html directory in the container:

``` programlisting
$ buildah copy ubi8-init-working-container html/index.html 
➥ /var/lib/www/html/
```

[]{#A.htm#pgfId-1277935}If you would like to use more advanced tools
like package managers to add content to your containers, Buildah
supports running commands inside the
[]{#A.htm#marker-1277936}[]{#A.htm#marker-1277937}[]{#A.htm#marker-1277938}containers.

### []{#A.htm#pgfId-1277940}A.2.3 Running commands in a working container {#A.htm#heading_id_7 .fm-head1}

[]{#A.htm#pgfId-1277944}To
[]{#A.htm#marker-1277941}[]{#A.htm#marker-1277942}[]{#A.htm#marker-1277943}run
a command inside the working container, you need to execute
`buildah`{.fm-code-in-text} `run`{.fm-code-in-text}. Under the hood,
this command works exactly the same as the `RUN`{.fm-code-in-text}
instruction[]{#A.htm#marker-1277945}; it starts a new container on top
of the current one, executes a specified command, and commits the result
back to the working container. The syntax of `buildah`{.fm-code-in-text}
`run`{.fm-code-in-text} requires you to specify the name of the working
container followed by the command. In the following example, you install
the `httpd`{.fm-code-in-text} service[]{#A.htm#marker-1277946} within
the container:

``` programlisting
$ buildah run ubi8-init-working-container dnf -y install httpd
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered with an entitlement server. You can use 
➥ subscription-manager to register.
...
Complete!
```

[]{#A.htm#pgfId-1277955}To make sure you will have a running web server
once the running container is created, the next command enables the
Apache HTTP Server service:

``` programlisting
$ buildah run ubi8-init-working-container systemctl enable httpd.service
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → 
➥ /usr/lib/systemd/system/httpd.service.
```

[]{#A.htm#pgfId-1277962}Table A.4 shows the relationship between
Containerfile instructions and Buildah
[]{#A.htm#marker-1277959}[]{#A.htm#marker-1277960}[]{#A.htm#marker-1277961}commands.

[]{#A.htm#pgfId-1280628}Table A.4 Containerfile instructions mapped to
Buildah commands

+-----------------+-----------------+-----------------------------------+
| []{#A           | [               | []                                |
| .htm#pgfId-1280 | ]{#A.htm#pgfId- | {#A.htm#pgfId-1280638}Description |
| 634}Instruction | 1280636}Command |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#A.htm#pgf   | []{#A.h         | []{#A.htm#pgfId-1280644}Add the   |
| Id-1280640}`ADD | tm#pgfId-128064 | contents of a file, URL, or       |
| `{.fm-code-in-t | 2}`buildah`{.fm | directory to the container.       |
| ext1}[]{#A.htm# | -code-in-text1} |                                   |
| marker-1282708} | `add`{.fm       |                                   |
|                 | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#A.htm#pgfI  | []{#A.h         | []{#A.htm#pgfId-1280650}Copies    |
| d-1280646}`COPY | tm#pgfId-128064 | the contents of a file, URL, or   |
| `{.fm-code-in-t | 8}`buildah`{.fm | directory into a container's      |
| ext1}[]{#A.htm# | -code-in-text1} | working directory.                |
| marker-1282715} | `copy`{.fm      |                                   |
|                 | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#A.htm#pgfI  | []{#A.h         | []{#A.htm#pgfId-1280656}Creates a |
| d-1280652}`FROM | tm#pgfId-128065 | new working container, either     |
| `{.fm-code-in-t | 4}`buildah`{.fm | from scratch or using a specified |
| ext1}[]{#A.htm# | -code-in-text1} | image as a starting point.        |
| marker-1282722} | `from`{.fm      |                                   |
|                 | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#A.htm#pgf   | []{#A.h         | []{#A.htm#pgfId-1280662}Runs a    |
| Id-1280658}`RUN | tm#pgfId-128066 | command inside the container.     |
| `{.fm-code-in-t | 0}`buildah`{.fm |                                   |
| ext1}[]{#A.htm# | -code-in-text1} |                                   |
| marker-1282729} | `run`{.fm       |                                   |
|                 | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+

### []{#A.htm#pgfId-1278002}A.2.4 Adding content to a working container directly from the host {#A.htm#heading_id_8 .fm-head1}

[]{#A.htm#pgfId-1278006}Up
[]{#A.htm#marker-1278003}[]{#A.htm#marker-1278004}[]{#A.htm#marker-1278005}until
now, you've seen how Buildah can perform the same commands you execute
within a Containerfile, but one of Buildah's goals is exposing the
container image rootfs[]{#A.htm#marker-1278007} directly to the host.
This allows you to use commands available on your host machine to add
content to the container image, without requiring the commands to be
present inside the container image.

[]{#A.htm#pgfId-1278009}The `buildah`{.fm-code-in-text}
`mount`{.fm-code-in-text} command[]{#A.htm#marker-1278008} allows you to
mount a working container's root filesystem directly on your system and
then use tools like `cp`{.fm-code-in-text}[]{#A.htm#marker-1278010},
`make`{.fm-code-in-text}[]{#A.htm#marker-1278011},
`dnf`{.fm-code-in-text}[]{#A.htm#marker-1278012}, or even an editor to
manipulate the contents of the container's rootfs.

[]{#A.htm#pgfId-1278014}If you run Buildah as root, you can simply
execute the `buildah`{.fm-code-in-text} `mount`{.fm-code-in-text}
command. But in rootless mode, this isn't allowed. Recall from section
2.2.10, where you learned about the `podman`{.fm-code-in-text}
`mount`{.fm-code-in-text} command[]{#A.htm#marker-1278015}, that you
must first enter the user namespace. Similarly, the
`buildah`{.fm-code-in-text} `unshare`{.fm-code-in-text}
command[]{#A.htm#marker-1278016} creates a shell running in the user
namespace. Once you are in the user namespace, you can mount the
container. In the following example, using what you have learned so far,
you will use commands from your host's operating system
`grep`{.fm-code-in-text} to add content to the container:

``` programlisting
$ buildah unshare
```bash
# mnt=$(buildah mount ubi8-init-working-container)
# echo $mnt
/home/dwalsh/.local/share/containers/storage/overlay/133e1728eac26589b07984
➥ e3bdf31b5e318159940c866d9e0493a1d08e1d2f6a/merged
# grep dwalsh /etc/passwd >> $mnt/etc/passwd
# exit
```

[]{#A.htm#pgfId-1278024}Now you can check if your changes were actually
applied inside a working container:

``` programlisting
$ buildah run ubi8-init-working-container grep dwalsh /etc/passwd
dwalsh:x:3267:3267:Daniel J Walsh:/home/dwalsh:/bin/bash
```

[]{#A.htm#pgfId-1278027}After you are done populating the content of the
working container, it's time to specify other instructions from the
Containerfile. These will describe your intentions as the container
image
[]{#A.htm#marker-1278028}[]{#A.htm#marker-1278029}[]{#A.htm#marker-1278030}creator.

### []{#A.htm#pgfId-1278032}A.2.5 Configuring a working container {#A.htm#heading_id_9 .fm-head1}

[]{#A.htm#pgfId-1278035}You
[]{#A.htm#marker-1278033}[]{#A.htm#marker-1278034}probably noticed in
table A.3 that there are a lot of missing Containerfile instructions.
Containerfile instructions like
`LABEL`{.fm-code-in-text}[]{#A.htm#marker-1278036},
`EXPOSE`{.fm-code-in-text}[]{#A.htm#marker-1278037},
`WORKDIR`{.fm-code-in-text}[]{#A.htm#marker-1278038},
`CMD`{.fm-code-in-text}[]{#A.htm#marker-1278039}, and
`ENTRYPOINT`{.fm-code-in-text}[]{#A.htm#marker-1278040} are used to
populate the OCI image specification.

[]{#A.htm#pgfId-1278042}Now, using the `buildah`{.fm-code-in-text}
`config`{.fm-code-in-text} command[]{#A.htm#marker-1278041}, you can add
a port to expose (`EXPOSE`{.fm-code-in-text}) and mark a location inside
the container rootfs as a volume (`VOLUME`{.fm-code-in-text}), which
will be used as the website root directory:

``` programlisting
$ buildah config --port=80 --volume=/var/lib/www/html 
➥ ubi8-init-working-container
```

[]{#A.htm#pgfId-1278045}You can inspect the corresponding OCI image
specification fields using the `buildah`{.fm-code-in-text}
`inspect`{.fm-code-in-text} command[]{#A.htm#marker-1281981}:

``` programlisting
$ buildah inspect --format '{{ .OCIv1.Config.ExposedPorts }} {{ 
➥ .OCIv1.Config.Volumes }}' ubi8-init-working-container
map[80:{}] map[/var/lib/www/html:{}]
```

[]{#A.htm#pgfId-1278052}[]{#A.htm#id_zhze1r6dd4nl}Table A.4 shows the
relationship between Containerfile instructions and Buildah config
options. You can also refer to table A.5 for additional information on
these instructions.

[]{#A.htm#pgfId-1280793}Table A.5 Containerfile instructions mapped to
Buildah config options

+-----------------+-----------------+-----------------------------------+
| []{#A           | []{#A.htm#pgfId | []                                |
| .htm#pgfId-1280 | -1280801}Option | {#A.htm#pgfId-1280803}Description |
| 799}Instruction |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#A.htm#      | []{#A.ht        | []{#A.htm#pgfId-1280809}Sets      |
| pgfId-1280805}` | m#pgfId-1280807 | contact information of the image  |
| MAINTAINER`{.fm | }`--author`{.fm | author                            |
| -code-in-text1} | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#A.htm#pgf   | []{#A           | []{#A.htm#pgfId-1280815}Sets a    |
| Id-1280811}`CMD | .htm#pgfId-1280 | default command to run within a   |
| `{.fm-code-in-t | 813}`--cmd`{.fm | container                         |
| ext1}[]{#A.htm# | -code-in-text1} |                                   |
| marker-1280876} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#A           | []{#A.htm#pg    | []{#A.htm#pgfId-1280821}Sets a    |
| .htm#pgfId-1280 | fId-1280819}`-- | command for a container that will |
| 817}`ENTRYPOINT | entrypoint`{.fm | run as an executable              |
| `{.fm-code-in-t | -code-in-text1} |                                   |
| ext1}[]{#A.htm# |                 |                                   |
| marker-1280877} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#A.htm#pgf   | []{#A           | []{#A.htm#pgfId-1280827}Sets the  |
| Id-1280823}`ENV | .htm#pgfId-1280 | environment variable for all      |
| `{.fm-code-in-t | 825}`--env`{.fm | subsequent instructions           |
| ext1}[]{#A.htm# | -code-in-text1} |                                   |
| marker-1280878} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#A.          | []{#A.htm#pgf   | []{#A.htm#pgfId-1280833}Specifies |
| htm#pgfId-12808 | Id-1280831}`--h | a command to check if a container |
| 29}`HEALTHCHECK | ealthcheck`{.fm | is still running                  |
| `{.fm-code-in-t | -code-in-text1} |                                   |
| ext1}[]{#A.htm# |                 |                                   |
| marker-1280879} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#A.htm#pgfId | []{#A.h         | []{#A.htm#pgfId-1280839}Adds      |
| -1280835}`LABEL | tm#pgfId-128083 | key-value metadata                |
| `{.fm-code-in-t | 7}`--label`{.fm |                                   |
| ext1}[]{#A.htm# | -code-in-text1} |                                   |
| marker-1280880} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []              | []{#A.htm       | []{#A.htm#pgfId-1280845}Sets a    |
| {#A.htm#pgfId-1 | #pgfId-1280843} | command to be run when the image  |
| 280841}`ONBUILD | `--onbuild`{.fm | is used as the base for another   |
| `{.fm-code-in-t | -code-in-text1} | image                             |
| ext1}[]{#A.htm# |                 |                                   |
| marker-1280881} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| [               | []{#A.          | []{#A.htm#pgfId-1280851}Specifies |
| ]{#A.htm#pgfId- | htm#pgfId-12808 | a port that the container will    |
| 1280847}`EXPOSE | 49}`--port`{.fm | listen on at run time             |
| `{.fm-code-in-t | -code-in-text1} |                                   |
| ext1}[]{#A.htm# |                 |                                   |
| marker-1280882} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#A           | []{#A.htm#pgf   | []{#A.htm#pgfId-1280857}Sets the  |
| .htm#pgfId-1280 | Id-1280855}`--s | stop signal to be sent when the   |
| 853}`STOPSIGNAL | top-signal`{.fm | container is stopped              |
| `{.fm-code-in-t | -code-in-text1} |                                   |
| ext1}[]{#A.htm# |                 |                                   |
| marker-1280883} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#A.htm#pgfI  | []{#A.          | []{#A.htm#pgfId-1280863}Sets the  |
| d-1280859}`USER | htm#pgfId-12808 | user to be used when running the  |
| `{.fm-code-in-t | 61}`--user`{.fm | container and for all subsequent  |
| ext1}[]{#A.htm# | -code-in-text1} | `RUN`{.fm-code-in-text1},         |
| marker-1280884} |                 | `CMD`{.fm-code-in-text1}, and     |
|                 |                 | `ENTRYPOINT`{.fm-code-in-text1}   |
|                 |                 | instructions                      |
+-----------------+-----------------+-----------------------------------+
| [               | []{#A.ht        | []{#A.htm#pgfId-1280869}Adds a    |
| ]{#A.htm#pgfId- | m#pgfId-1280867 | mount point and marks it as a     |
| 1280865}`VOLUME | }`--volume`{.fm | volume for external data          |
| `{.fm-code-in-t | -code-in-text1} |                                   |
| ext1}[]{#A.htm# |                 |                                   |
| marker-1280885} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []              | []{#A.htm#pg    | []{#A.htm#pgfId-1280875}Sets the  |
| {#A.htm#pgfId-1 | fId-1280873}`-- | working directory for all         |
| 280871}`WORKDIR | workingdir`{.fm | subsequent                        |
| `{.fm-code-in-t | -code-in-text1} | `RUN`{.fm-code-in-text1},         |
| ext1}[]{#A.htm# |                 | `CMD`{.fm-code-in-text1},         |
| marker-1280886} |                 | `ENTRYPOINT`{.fm-code-in-text1},  |
|                 |                 | `COPY`{.fm-code-in-text1}, and    |
|                 |                 | `ADD`{.fm-code-in-text1}          |
|                 |                 | instructions                      |
+-----------------+-----------------+-----------------------------------+

[]{#A.htm#pgfId-1278146}Once you have finished adding content to the
Buildah container image and adding configuration to the OCI image
specification, you need to create an image from the working
[]{#A.htm#marker-1278147}[]{#A.htm#marker-1278148}container.

### []{#A.htm#pgfId-1278150}A.2.6 Creating an image from a working container {#A.htm#heading_id_10 .fm-head1}

[]{#A.htm#pgfId-1278154}The
[]{#A.htm#marker-1278151}[]{#A.htm#marker-1278152}[]{#A.htm#marker-1278153}working
container you've been building so far can be used to create the
OCI-compliant image using the `buildah`{.fm-code-in-text}
`commit`{.fm-code-in-text} command[]{#A.htm#marker-1278155}. This
command works in the same way as the `podman`{.fm-code-in-text}
`commit`{.fm-code-in-text} command[]{#A.htm#marker-1278156} you learned
about in section 2.1.9. Inputs for this command are the working
container name and an optional image tag; if a tag is not specified, the
image will have no name:

``` programlisting
$ buildah commit ubi8-init-working-container quay.io/rhatdan/myimage2
Getting image source signatures
Copying blob 352ba846236b skipped: already exists 
Copying blob 3ba8c926eef9 skipped: already exists 
Copying blob 421971707f97 skipped: already exists 
Copying blob 9ff25f020d5a done 
Copying config 5e47dbd9b7 done 
Writing manifest to image destination
Storing signatures
5e47dbd9b7b7a43dd29f3e8a477cce355e42c019bb63626c0a8feffae56fcbf9
```

[]{#A.htm#pgfId-1278167}You can see the image using
`buildah`{.fm-code-in-text} `images`{.fm-code-in-text}:

``` programlisting
$ buildah images
REPOSITORY                         TAG       IMAGE ID         CREATED    SIZE
quay.io/rhatdan/myimage2        latest   5e47dbd9b7b7   2 minutes ago  293 MB
registry.access.redhat
➥ .com/ubi8-init               latest   4b85030f924b     5 weeks ago  253 MB
```

[]{#A.htm#pgfId-1278174}Because Podman and Buildah share the same
container image storage, you can see the same images with
`podman`{.fm-code-in-text} `images`{.fm-code-in-text}:

``` programlisting
$ podman images
REPOSITORY                         TAG        IMAGE ID        CREATED    SIZE
quay.io/rhatdan/myimage2        latest    5e47dbd9b7b7  4 minutes ago  293 MB
registry.access.redhat
➥ .com/ubi8-init               latest    4b85030f924b    5 weeks ago  253 MB
```

[]{#A.htm#pgfId-1278185}You can even run a Podman container on the
[]{#A.htm#marker-1278182}[]{#A.htm#marker-1278183}[]{#A.htm#marker-1278184}image:

``` programlisting
$ podman run quay.io/rhatdan/myimage2 grep dwalsh /etc/passwd
dwalsh:x:3267:3267:Daniel J Walsh:/home/dwalsh:/bin/bash
```

### []{#A.htm#pgfId-1278189}A.2.7 Pushing an image to a container registry {#A.htm#heading_id_11 .fm-head1}

[]{#A.htm#pgfId-1278195}Similarly
[]{#A.htm#marker-1278190}[]{#A.htm#marker-1278191}[]{#A.htm#marker-1278192}to
Podman, Buildah has the `buildah`{.fm-code-in-text}
`login`{.fm-code-in-text}[]{#A.htm#marker-1278193} and
`buildah`{.fm-code-in-text} `push`{.fm-code-in-text}
commands[]{#A.htm#marker-1278194}, which allow you to push images to
container registries, as shown in the following example:

``` programlisting
$ buildah login quay.io
Username: rhatdan
Password:
Login Succeeded!
$ buildah push quay.io/rhatdan/myimage2
Getting image source signatures
Copying blob 3ba8c926eef9 done 
Copying blob 421971707f97 done 
Copying blob 9ff25f020d5a done 
Copying blob 352ba846236b done 
Copying config 5e47dbd9b7 done 
Writing manifest to image destination
Copying config 5e47dbd9b7 done 
Writing manifest to image destination
Storing signatures
```

[]{#A.htm#pgfId-1278196}[Note]{.fm-callout-head} You can also use
`podman`{.fm-code-in-text1} `login`{.fm-code-in-text1} and
`podman`{.fm-code-in-text1} `push`{.fm-code-in-text1} or even
`skopeo`{.fm-code-in-text1} `login`{.fm-code-in-text1} and
`skopeo`{.fm-code-in-text1} `copy`{.fm-code-in-text1} to accomplish the
same task.

[]{#A.htm#pgfId-1278212}Congratulations! You have successfully built an
OCI-compliant container image manually by using simple shell commands
rather than using a Containerfile. Additionally, if you want to create
an image using an existing Containerfile or Dockerfile, you can use the
`buildah`{.fm-code-in-text} `build`{.fm-code-in-text}
[]{#A.htm#marker-1278213}[]{#A.htm#marker-1278214}[]{#A.htm#marker-1278215}command[]{#A.htm#marker-1278216}.

### []{#A.htm#pgfId-1278218}A.2.8 Building an image from Containerfiles {#A.htm#heading_id_12 .fm-head1}

[]{#A.htm#pgfId-1278223}You
[]{#A.htm#marker-1278219}[]{#A.htm#marker-1278220}[]{#A.htm#marker-1278221}can
use the `buildah`{.fm-code-in-text} `build`{.fm-code-in-text} command to
build an OCI-compliant image from a Containerfile or a Dockerfile.
Buildah includes a parser that understands the Containerfile format and
can perform all tasks using previously described commands automatically.
In the next example, use the Containerfile from section 2.3.2:

``` programlisting
$ cat myapp/Containerfile
FROM ubi8/httpd-24
COPY index.html /var/www/html/index.html
```

[]{#A.htm#pgfId-1278227}You can build your container image using this
Containerfile by executing the following command:

``` programlisting
$ buildah build ./myapp
STEP 1/2: FROM ubi8/httpd-24
Resolved "ubi8/httpd-24" as an alias (/home/dwalsh/.cache/containers/
➥ short-name-aliases.conf)
Trying to pull registry.access.redhat.com/ubi8/httpd-24:latest
...
Getting image source signatures
Checking if image destination supports signatures
Copying blob adffa6963146 skipped: already exists 
...
STEP 2/2: COPY html/index.html /var/www/html/index.html
COMMIT
Getting image source signatures
Copying blob 352ba846236b skipped: already exists 
...
bbfcf76c994c738f8496c1f274bd009ddbc960334b59a74953691fff00442417
```

[]{#A.htm#pgfId-1278244}You've probably noticed that this output matches
precisely the output of the `podman`{.fm-code-in-text}
`build`{.fm-code-in-text} command[]{#A.htm#marker-1278245}. This is
because the `podman`{.fm-code-in-text} `build`{.fm-code-in-text}
command[]{#A.htm#marker-1278246} uses
[]{#A.htm#marker-1278247}[]{#A.htm#marker-1278248}[]{#A.htm#marker-1278249}Buildah.

### []{#A.htm#pgfId-1278251}A.2.9 Buildah as a library {#A.htm#heading_id_13 .fm-head1}

[]{#A.htm#pgfId-1278254}Buildah
[]{#A.htm#marker-1278252}[]{#A.htm#marker-1278253}was designed to not
only be used as a command-line tool but also to be a Golang-based
library. Buildah is being used in a few different tools, such as Podman
and the OpenShift image builder. Buildah allows these tools to
internally build OCI images. Every time you do a
`podman`{.fm-code-in-text} `build`{.fm-code-in-text}, you are executing
the Buildah library code. Having learned how to build container images
using Buildah, copy images between container storages using Skopeo, and
manage and run containers on the host using Podman, let's talk about how
all these tools are used in the
[]{#A.htm#marker-1278255}[]{#A.htm#marker-1278256}Kubernetes
[]{#A.htm#marker-1278257}[]{#A.htm#marker-1278258}ecosystem.

## []{#A.htm#pgfId-1278260}A.3 CRI-O: Container Runtime Interface for OCI containers {#A.htm#heading_id_14 .fm-head}

[]{#A.htm#pgfId-1278263}When
[]{#A.htm#marker-1278261}[]{#A.htm#marker-1278262}Kubernetes was being
developed, it used the Docker API internally to run containers.
Kubernetes relied on features of Docker that changed from release to
release, sometimes breaking Kubernetes. At the same time, CoreOS wanted
their alternative container engine, called RKT
([https://github.com/rkt/rkt](https://github.com/rkt/rkt){.url}), to
work with Kubernetes. Kubernetes developers decided, then, to split out
the Docker functionality and use a new API called the Container Runtime
Interface (CRI; [http://mng.bz/yaDq](http://mng.bz/yaDq){.url}). This
interface allows Kubernetes to use other container engines in addition
to Docker.

[]{#A.htm#pgfId-1278264}When Kubernetes wants to pull a container image,
it calls out to a remote socket via the CRI and asks the listener to
pull an OCI image for it. When it wants to launch a Pod/Container, it
calls out to the socket and asks it to launch the container.

[]{#A.htm#pgfId-1278265}[Note]{.fm-callout-head} CoreOS was eventually
acquired by Red Hat, and the RKT project has ended. Kubernetes has
deprecated Docker as a container runtime.

[]{#A.htm#pgfId-1278266}Red Hat saw the CRI as an opportunity to develop
a new container engine, which they ended up calling the Container
Runtime Interface for OCI containers (CRI-O;
[https://cri-o.io/](https://cri-o.io/){.url}). CRI-O is based on the
same containers/storage and containers/image libraries as Skopeo,
Buildah, and Podman and can be used in conjunction with these tools.
CRI-O's primary objective is replacing the Docker service as the
container engine for Kubernetes.

::: figure
![](images/OEBPS/Images/A-UN03.png){.calibre18}
:::

[]{#A.htm#pgfId-1278271}CRI-O is tied to Kubernetes releases. When a new
version of Kubernetes is released, the version numbers are synchronized.
CRI-O is optimized for Kubernetes workloads; the engineers working on it
understand what Kubernetes is trying to do and are making sure CRI-O
does it in the most efficient way possible. Since CRI-O has no other
users, Kubernetes doesn't have to worry about breaking changes
[]{#A.htm#marker-1278272}[]{#A.htm#marker-1278273}in
[]{#A.htm#marker-1278274}CRI-O.

[]{#A.htm#pgfId-1278275}[Note]{.fm-callout-head} CRI-O is the core
technology used with Red Hat's OpenShift Kubernetes-based product.
OpenShift uses Podman to install and configure CRI-O before Kubernetes
starts running. The OpenShift image builder embeds Buildah functionality
to allow users to build images within their OpenShift clusters.

[]{#B.htm}
