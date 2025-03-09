# 9 Podman as a service

[]{#09.htm#pgfId-1109411}This chapter []{#09.htm#marker-1116839}covers

-   []{#09.htm#pgfId-1109412 .calibre17}Running Podman as a service
-   []{#09.htm#pgfId-1109413 .calibre17}Podman service support for two
    REST APIs
-   []{#09.htm#pgfId-1109415 .calibre17}Python libraries
    podman-py[]{#09.htm#marker-1116843 .calibre17} and docker-py for
    managing Podman containers
-   []{#09.htm#pgfId-1109416 .calibre17}Support for
    `docker-compose`{.fm-code-in-text}
-   []{#09.htm#pgfId-1109417 .calibre17}Remote command-line
    communication with the Podman service
-   []{#09.htm#pgfId-1109418 .calibre17}Managing SSH communications with
    remote Podman instances

[]{#09.htm#marker-1111843}[]{#09.htm#pgfId-1109419}In previous chapters,
you learned about the Podman command line. The problem with this is
sometimes you want to work with containers from a remote system.
Similarly, you might want to write code in a scripting language to
interact with containers. Docker, being written as a client-server
application, supports a popular remote API, which led to the creation of
libraries written in Python and JavaScript to access the daemon.
Docker-py is a popular Python library used to interact with the Docker
daemon.

[]{#09.htm#pgfId-1109421}Many CI/CD, GUI, and remote management systems
have been built to manage Docker containers. Code editors like Visual
Studio even have built-in plug-ins that talk directly to the Docker API.
Advanced tools like `docker-compose`{.fm-code-in-text} led to a new
programming language that is used to orchestrate multiple containers on
a host by interacting with the Docker daemon.

[]{#09.htm#pgfId-1109423}Podman provides similar features and can be run
as a service. Podman supports running the Podman service in rootless as
well as rootful mode. In this chapter, you will learn about the service
and how to interact with it. You will write a simple program in Python
that uses the docker-py and newer podman-py libraries to interact with
the Podman service. You will learn how to set up remote Docker-based
tools, including `docker-compose`{.fm-code-in-text}, to actually use the
Podman service, with no Docker daemon available.

[]{#09.htm#pgfId-1109429}[Note]{.fm-callout-head} The Podman service is
only supported on Linux. Because the Podman service launches Linux
containers, it only runs on Linux machines. Windows and Mac versions of
Podman communicate with the Podman service over the REST API to launch
containers. For more information on Podman on Mac, see appendix E, and
for Windows, see appendix F.

[]{#09.htm#pgfId-1109431}The Podman command has a
`--remote`{.fm-code-in-text} option[]{#09.htm#marker-1109430} that
allows you to interact with the Podman service, either on the local
machine or, most often, on a remote machine. You will learn to set up
the Podman connections to make interacting with remote services easy and
secure. But first you need to know how to enable the Podman service.

## []{#09.htm#pgfId-1109433}9.1 Introducing the Podman service {#09.htm#heading_id_3 .fm-head}

[]{#09.htm#pgfId-1109435}The []{#09.htm#marker-1115468}Podman project
supports a REST (or RESTful) API. The `podman`{.fm-code-in-text}
`system`{.fm-code-in-text} `service`{.fm-code-in-text}
command[]{#09.htm#marker-1115470} creates a listening service that
answers API calls for Podman. The service can be run in rootful or
rootless mode. This command offers an optional argument to specify a URI
on which the Podman service will listen. For example, the unix:///tmp/
podman.sock URI tells Podman to listen on the /tmp/podman.sock UNIX
domain socket. The tcp:localhost:10000 URI socket tells Podman to listen
on TCP socket, port `10000`{.fm-code-in-text}. By default, Podman
listens on a UNIX domain socket under the /run directory (table 9.1).

[]{#09.htm#pgfId-1109438}[Note]{.fm-callout-head} If you are not
familiar with REST API or remote APIs in general, I recommend that you
read "What is a REST API?" by Red Hat:
[https://www.redhat.com/en/topics/api/what-is-a-rest-api](https://www.redhat.com/en/topics/api/what-is-a-rest-api){.url}.

[]{#09.htm#pgfId-1109440}Podman running as a service in this case is
different from having a centralized daemon, like Docker does, in
multiple ways. The biggest difference is that the Podman command can run
without the service and interacts with containers and images created by
the service. Other container tools can interact with the storage and
containers without going through the service. The service also exits
when there are no connections to it. You could even run multiple
services at the same time on the same datastore (although I would not
recommend this). The Docker daemon forces all interaction with
containers and images to go through the daemon. Table 9.1 shows the
default locations where the Podman service listens for incoming
connections.[]{#09.htm#id_b2mt0jexvb3t}

[]{#09.htm#marker-1112574}[]{#09.htm#pgfId-1112127}Table 9.1 Default
locations for the podman.socket

+-----------------+-----------------------------------------------------+
| []{#09.htm#pgf  | []{#09.htm#pgfId-1112133}Default location           |
| Id-1112131}Mode |                                                     |
+-----------------+-----------------------------------------------------+
| []              | []{#                                                |
| {#09.htm#pgfId- | 09.htm#pgfId-1112137}unix:///run/podman/podman.sock |
| 1112135}Rootful |                                                     |
+-----------------+-----------------------------------------------------+
| []{             | []{#09.htm#pgfId-                                   |
| #09.htm#pgfId-1 | 1112141}unix://\$XDG_RUNTIME_DIR/podman/podman.sock |
| 112139}Rootless |                                                     |
|                 | []{#09.htm#pgfId-1112148}example                    |
|                 | unix:///run/user/1000/podman/podman.sock)           |
+-----------------+-----------------------------------------------------+

[]{#09.htm#pgfId-1109460}Although the Podman service can be set up to
run on a TCP socket as well, I caution you to be very careful because
there is no authorization or additional security built into the service
to prevent hackers from gaining access. The service relies on the SSH
service to gain remote access to the Podman service, and this approach
is recommended.

[]{#09.htm#pgfId-1109461}The Podman service was designed to run as an
on-demand service, exiting 5 seconds after the last connection. This
time limit avoids a long-running daemon that uses system resources even
when the service is not being used. While the Podman service could
launch a separate process for each connection, this could become a
bottleneck. Try this out by running the following command; after 5
seconds you will see the command exit. If you had active connections to
the service, it would continue to run:

``` programlisting
$ podman system service
```

[]{#09.htm#pgfId-1109464}You can specify the timeout for this exit in
seconds with the `--time`{.fm-code-in-text}
option[]{#09.htm#marker-1109463}. Specifying `--time`{.fm-code-in-text}
`0`{.fm-code-in-text} causes the `podman`{.fm-code-in-text}
`system`{.fm-code-in-text} `service`{.fm-code-in-text}
command[]{#09.htm#marker-1109465} to run until you stop it. Most users
never interact directly with the Podman system service to activate the
service but rely on systemd services to manage it.

### []{#09.htm#pgfId-1109467}9.1.1 Systemd services {#09.htm#heading_id_4 .fm-head1}

[]{#09.htm#pgfId-1109470}Podman
[]{#09.htm#marker-1117579}[]{#09.htm#marker-1117580}provides multiple
systemd unit files for running Podman as a service. Because Podman was
not designed as a daemon, and the developers did not want to always have
a long-running daemon, they decided to take advantage of systemd socket
activation. This allows the Podman service to be launched as an
on-demand service. Figure 9.1 shows how systemd listens on the Podman
socket and then launches the Podman service when it receives a
connection.

::: figure
![](images/OEBPS/Images/09-01.png){.calibre18}

[]{#09.htm#pgfId-1117624}[]{#09.htm#id_i88bi4plbk5b}Figure 9.1 Podman
service running under systemd
:::

[]{#09.htm#pgfId-1109477}The Podman package provides two podman.socket
unit files: one for rootful Podman and the other for rootless Podman.
Table 9.2 defines the location of the systemd socket files to be used in
rootful and rootless mode.[]{#09.htm#id_fu3h9sxk6mdw}

[]{#09.htm#pgfId-1112621}Table 9.2 Podman socket unit files

+-----------------+-----------------------------------------------------+
| []{#09.htm#pgf  | []{#09.htm#pgfId-1112627}Systemd socket file        |
| Id-1112625}Mode |                                                     |
+-----------------+-----------------------------------------------------+
| []              | []{#09.htm#                                         |
| {#09.htm#pgfId- | pgfId-1112631}/usr/lib/systemd/system/podman.socket |
| 1112629}Rootful |                                                     |
+-----------------+-----------------------------------------------------+
| []{             | []{#09.ht                                           |
| #09.htm#pgfId-1 | m#pgfId-1112635}/usr/lib/systemd/user/podman.socket |
| 112633}Rootless |                                                     |
+-----------------+-----------------------------------------------------+

[]{#09.htm#pgfId-1109495}These two socket activation services tell
systemd to listen on the default UNIX domain socket listed in table 9.1.
When a process connects to the socket, systemd launches the matching
service, which runs the `podman`{.fm-code-in-text}
`system`{.fm-code-in-text} `service`{.fm-code-in-text}
command[]{#09.htm#marker-1109496}. Systemd then hands the socket off to
the service. After the Podman service completes the API request, it
waits for another connection. If no connection happens for 5 seconds,
Podman exits, freeing up the resources it was using. If a new connection
comes in, systemd repeats the process and launches a new instance of the
Podman service.

[]{#09.htm#pgfId-1109497}In the rest of this chapter, you will be
interacting with the Podman service, so you need to start running it.
You can enable and start the Podman socket on your machine using the
`--user`{.fm-code-in-text} option[]{#09.htm#marker-1109498}, which tells
systemd to enable the user service (or rootless mode service):

``` programlisting
$ systemctl --user enable podman.socket
Created symlink 
➥ /home/dwalsh/.config/systemd/user/sockets.target.wants/podman.socket → 
➥ /usr/lib/systemd/user/podman.socket. 
$ systemctl --user start podman.socket
```

[]{#09.htm#pgfId-1109505}You can see that the podman.sock has been
created in your `XDG_RUNTIME_DIR`{.fm-code-in-text}:

``` programlisting
$ ls $XDG_RUNTIME_DIR/podman/podman.sock
/run/user/3267/podman/podman.sock
```

[]{#09.htm#pgfId-1109508}At this point, the systemd is listening on the
socket, and there is no Podman process running. When a packet comes into
the service, systemd launches the Podman service process to handle the
connection.

[]{#09.htm#pgfId-1109510}To try out the service, you can run the
following `curl`{.fm-code-in-text} command[]{#09.htm#marker-1109509} to
probe for the version on the Podman service:

``` programlisting
$ curl -s --unix-socket $XDG_RUNTIME_DIR/podman/podman.sock 
➥ http://d/v1.0.0/libpod/version | jq
{
  "Platform": {
  "Name": "linux/amd64/fedora-35"
  },
  "Components": [
  {
    "Name": "Podman Engine",
    "Version": "4.0.0-dev",
    "Details": {
        "APIVersion": "4.0.0-dev",
          "Arch": "amd64",
          "BuildTime": "2022-01-04T13:42:14-05:00",
          "Experimental": "false",
        "GitCommit": "66ffbc845d1f0fd5c29611ac3f09daa24749dc1e-dirty",
        "GoVersion": "go1.16.12",
        "KernelVersion": "5.15.10-200.fc35.x86_64",
        "MinAPIVersion": "3.1.0",
        "Os": "linux"
      }
  },
  {
      "Name": "Conmon",
      "Version": "conmon version 2.0.30, commit: ",
      "Details": {
        "Package": "conmon-2.0.30-2.fc35.x86_64"
      }
  },
  {
      "Name": "OCI Runtime (crun)",
      "Version": "crun version 1.4\ncommit:
3daded072ef008ef0840e8eccb0b52a7efbd165d\nspec: 1.0.0\n+SYSTEMD 
➥ +SELINUX +APPARMOR +CAP +SECCOMP +EBPF +CRIU +YAJL",
      "Details": {
        "Package": "crun-1.4-1.fc35.x86_64"
    }
  }
  ],
  "Version": "4.0.0-dev",
  "ApiVersion": "1.40",
  "MinAPIVersion": "1.24",
  "GitCommit": "66ffbc845d1f0fd5c29611ac3f09daa24749dc1e-dirty",
  "GoVersion": "go1.16.12",
  "Os": "linux",
  "Arch": "amd64",
  "KernelVersion": "5.15.10-200.fc35.x86_64",
  "BuildTime": "2022-01-04T13:42:14-05:00"
}
```

[]{#09.htm#pgfId-1109563}Now that you have the service running, it's
time to investigate
[]{#09.htm#marker-1109560}[]{#09.htm#marker-1109561}the
[]{#09.htm#marker-1109562}APIs.

## []{#09.htm#pgfId-1109565}9.2 Podman-supported APIs {#09.htm#heading_id_5 .fm-head}

[]{#09.htm#pgfId-1109568}The
[]{#09.htm#marker-1109566}[]{#09.htm#marker-1109567}Podman service
provides two APIs over the same socket (table 9.1). The compatibility
API targets the latest released version of the Docker API, implementing
all endpoints, except the Swarm APIs. The Podman team treats any problem
concerning a difference with the Docker API as a bug. If the API works
against the Docker daemon, it must work against the Podman service.

[]{#09.htm#pgfId-1109569}The Podman Libpod API provides support for
Podman's unique features, such as pods. While it would be great for all
projects to support the native Libpod API, it takes time to transition,
and it may be impossible for older, no-longer-maintained projects based
on the Docker API.

[]{#09.htm#pgfId-1109570}I recommend that all new users of Podman work
with the Libpod API, but if you are using legacy code or want to develop
code that will work with both Podman and Docker, then you should use the
compatibility API. Table 9.3 lists the two different REST APIs provided
by Podman.[]{#09.htm#id_58xsmdggrfxp}

[]{#09.htm#pgfId-1112682}Table 9.3 Podman-supported APIs

+-----------------+-----------------+-----------------------------------+
| []{#09.htm#pgf  | []{#09          | []{#0                             |
| Id-1112688}Mode | .htm#pgfId-1112 | 9.htm#pgfId-1112692}Documentation |
|                 | 690}Description |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#09.h        | []{#09.htm#     | []{#09                            |
| tm#pgfId-111269 | pgfId-1112696}A | .htm#pgfId-1112698}[https://docs. |
| 4}Compatibility | compatibility   | docker.com/engine/api/](https://d |
|                 | layer offering  | ocs.docker.com/engine/api/){.url} |
|                 | support for the |                                   |
|                 | Docker v1.40    |                                   |
|                 | API             |                                   |
+-----------------+-----------------+-----------------------------------+
| [               | []{#09.htm#     | []                                |
| ]{#09.htm#pgfId | pgfId-1112702}A | {#09.htm#pgfId-1112704}[https://d |
| -1112700}Libpod | Podman-native   | ocs.podman.io/en/latest/\_static/ |
|                 | Libpod layer    | api.html](https://docs.podman.io/ |
|                 |                 | en/latest/_static/api.html){.url} |
+-----------------+-----------------+-----------------------------------+

[]{#09.htm#pgfId-1109595}The easiest way to interact with the remote API
is via the `curl`{.fm-code-in-text} command[]{#09.htm#marker-1109594}.
Examine the list of images available with the `curl`{.fm-code-in-text}
command[]{#09.htm#marker-1109596} and the `jq`{.fm-code-in-text}
command[]{#09.htm#marker-1109597} to pretty-print the JSON code. Also
notice the `libpod`{.fm-code-in-text} field in the URL. This field tells
Podman to use its native API.

[]{#09.htm#pgfId-1109598}Listing 9.1 The default output when connecting
`curl`{.fm-code-in-text} to the Podman socket

``` programlisting
$ curl -s --unix-socket $XDG_RUNTIME_DIR/podman/podman.sock 
➥ http://d/v1.0.0/libpod/images/json | jq 
[
  {
  "Id":
"Sha256:2c7e43d880382561ebae3fa06c7a1442d0da2912786d09ea9baaef87f73c29ae",
  "ParentId": "",
  "RepoTags": [
    "quay.io/rhatdan/myimage:latest"    ❶
  ],
...
  }
]
```

[]{#09.htm#pgfId-1117132}[❶]{.fm-combinumeral} The image you have been
working on

[]{#09.htm#pgfId-1109613}You can also run the Docker API by eliminating
the `libpod`{.fm-code-in-text} field. For this command, you get the same
output because the APIs have the same output:

``` programlisting
$ curl -s --unix-socket $XDG_RUNTIME_DIR/podman/podman.sock 
➥ http://d/v1.0.0/images/json | jq
[
  {
  "Id":
"Sha256:2c7e43d880382561ebae3fa06c7a1442d0da2912786d09ea9baaef87f73c29ae",
  "ParentId": "",
  "RepoTags": [
    "quay.io/rhatdan/myimage:latest"
  ],
...
  }
]
```

[]{#09.htm#pgfId-1109627}An example in which the APIs differ is listing
pods, since Docker does not support the concept of a pod, the
`compat`{.fm-code-in-text} API does not have interfaces for it.

[]{#09.htm#pgfId-1109628}First, create a pod for the test by running the
following command:

``` programlisting
$ podman pod create --name mypod
116291543d5691c597132ec73a428f29f2c1f71a65fdfbaca17eb5440a5d47f6
```

[]{#09.htm#pgfId-1109631}Now, use the Libpod pods or JSON API to see
JSON related to the pod you just created:

``` programlisting
$ curl -s --unix-socket $XDG_RUNTIME_DIR/podman/podman.sock 
➥ http://d/v1.0.0/libpod/pods/json | jq
[
  {
    "Cgroup": "user.slice",
    “Containers": [
      {
      "Id": "8eeceeb4fd6aa3897e05b5361b5c27c6e98bc29707484f95994f49437536599e",
      "Names": "4b10a21c5b8c-infra",
      "Status": "running"
      }
    ],
    "Created": "2022-01-05T06:51:52.604528462-05:00",
    "Id": "4b10a21c5b8c2b4f8a598de1eace7b94918d813055891276c2472df856a7fbc1",
    "InfraId": 
➥ "8eeceeb4fd6aa3897e05b5361b5c27c6e98bc29707484f95994f49437536599e",
    "Name": "test_pod",
    "Namespace": "",
    “Networks": [],
    "Status": "Running",
    "Labels": {}
  },
  {
    "Cgroup": "user.slice",
    "Containers": [
      {
      "Id": "7a7405a31917da7bde01a6000809e0ee12f40b69fc76963d87a8ae254b34d8c7",
      "Names": "e10eb9303705-infra",
      "Status": "configured"
      }
    ],
    "Created": "2022-01-05T09:18:01.648324833-05:00",
    "Id": "e10eb930370592834fc168a7460fabe9b3e0e20a54b48a2bf3236cecd75f8138",
    "InfraId": 
➥ "7a7405a31917da7bde01a6000809e0ee12f40b69fc76963d87a8ae254b34d8c7",
    "Name": "mypod",
    "Namespace": "",
    "Networks": [],
    "Status": "Created",
    "Labels": {}
  }
]
```

[]{#09.htm#pgfId-1109674}If you try the same query against the Docker
API endpoint, it fails with a Not Found error.

``` programlisting
$ curl -s --unix-socket $XDG_RUNTIME_DIR/podman/podman.sock 
➥ http://d/v1.0.0/pods/json
Not Found
```

[]{#09.htm#pgfId-1109678}This is because the Docker API and Docker
itself do not understand pods. While you can do a lot of testing with
the API directly with tools like `curl`{.fm-code-in-text}, it is better
to have higher-level languages to interact with the API, such as
[]{#09.htm#marker-1109679}[]{#09.htm#marker-1109680}Python.

## []{#09.htm#pgfId-1109682}9.3 Python libraries for interacting with Podman {#09.htm#heading_id_6 .fm-head}

[]{#09.htm#pgfId-1109685}Python
[]{#09.htm#marker-1109683}[]{#09.htm#marker-1109684}is arguably the most
popular scripting language on Linux platforms. Almost every Linux system
has Python installed by default. Just like the API, there are two very
similar Python libraries available: the docker-py library, which works
with the compatibility library, and podman-py, which supports the newer
Libpod API. This section uses some Python commands and might require a
limited knowledge of Python but is easy enough for you to follow along
if you have limited experience.

### []{#09.htm#pgfId-1109687}9.3.1 Using docker-py with the Podman API {#09.htm#heading_id_7 .fm-head1}

[]{#09.htm#pgfId-1109691}The
[]{#09.htm#marker-1114805}[]{#09.htm#marker-1114806}[]{#09.htm#marker-1114807}most
popular Python package for interacting with containers is docker-py
([https://github.com/docker/docker-py](https://github.com/docker/docker-py){.url}).
Docker-py is a Python bindings library used originally to communicate
with the Docker daemon. It can also communicate with the Podman
compatibility service. The Docker-py library allows you to run the same
containers as the Podman command, except you can do it from Python.

[]{#09.htm#pgfId-1109692}Thousands of tools and examples built on
docker-py exist and are running in production. These tools have been
used for CI/CD systems as well as GUIs, management tools, and debugging
tools. For these commands, you can use the Podman
`compat`{.fm-code-in-text} API[]{#09.htm#marker-1109693}, which works
fine with docker-py.

[]{#09.htm#pgfId-1109694}Usually, you can install docker-py with
`apt-get`{.fm-code-in-text} or `dnf`{.fm-code-in-text}
`install`{.fm-code-in-text}. It is also available via PyPI. Consult the
install commands for your Linux platform. On RPM-based systems, the
package is called
`python-docker`{.fm-code-in-text}[]{#09.htm#marker-1109695}.

[]{#09.htm#pgfId-1109697}On my Red Hat-based system, I install it using
the following `dnf`{.fm-code-in-text} command[]{#09.htm#marker-1109696}:

``` programlisting
$ sudo dnf install -y python-docker
```

[]{#09.htm#pgfId-1109699}After docker-py is installed, you can start
using it to interact with the Podman service. Imagine you want to build
a Python script to interact with the Podman service to list the
currently available images. Notice I have to reset the
`DockerClient`{.fm-code-in-text} URL to point at the Podman socket. You
might have to modify the location of podman.sock on your system:

``` programlisting
$ cat > images.py << _EOF
import docker
client=docker.DockerClient(base_url='unix:/run/user/1000/podman/podman.sock')
print(client.images.list(all=True))
_EOF
```

[]{#09.htm#pgfId-1109705}Run the images.py script, and see the images
installed on your box:

``` programlisting
$ python images.py
[<Image: 'quay.io/rhatdan/myimage:latest'>, <Image: 'k8s.gcr.io/pause:3.5'>]
```

[]{#09.htm#pgfId-1109708}It is inconvenient to have to fully specify the
path to the Podman socket inside the Python script, but luckily, Docker
tools support a special environment variable called
`DOCKER_HOST`{.fm-code-in-text}[]{#09.htm#marker-1109709}. You can set
`DOCKER_HOST`{.fm-code-in-text} to point at the socket that implements
the Docker API.

[]{#09.htm#pgfId-1109711}First, set the `DOCKER_HOST`{.fm-code-in-text}
environment variable[]{#09.htm#marker-1109710} to point at podman.sock:

``` programlisting
$ export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/podman/podman.sock
```

[]{#09.htm#pgfId-1109714}Now, change the script to use the
`docker.from_env()`{.fm-code-in-text}
function[]{#09.htm#marker-1109713}:

``` programlisting
$ cat > images.py << _EOF
import docker
client=docker.from_env()
print(client.images.list(all=True))
_EOF
```

[]{#09.htm#pgfId-1109721}Run the new script, and you see that it uses
the `DOCKER_HOST`{.fm-code-in-text} environment
variable[]{#09.htm#marker-1109720} to discover the Podman service
socket:

``` programlisting
$ python images.py
[<Image: 'quay.io/rhatdan/myimage:latest'>, <Image: 'k8s.gcr.io/pause:3.5'>]
```

[]{#09.htm#pgfId-1109725}[Note]{.fm-callout-head} On many Linux
distributions, the `podman-docker`{.fm-code-in-text1}
package[]{#09.htm#marker-1109724} is available locally. When you install
this package, it installs a Docker script that redirects Docker commands
to run Podman commands. It also links all of the Docker man pages to
Podman man pages. Finally, it sets up a symbolic link between the
docker.sock and the podman.sock for rootful containers, allowing Docker
tools to use /var/run/podman/podman.sock, with no environment
modifications.

[]{#09.htm#pgfId-1109726}The great thing is that this
`DOCKER_HOST`{.fm-code-in-text} trick can be used with most docker-py
scripts that have been written over the years, and you can easily switch
your scripts from using the Docker daemon to using the Podman service.
If you want to use more advanced Podman features, you need to use the
[]{#09.htm#marker-1109727}[]{#09.htm#marker-1109728}[]{#09.htm#marker-1109729}podman-py
package.

### []{#09.htm#pgfId-1109731}9.3.2 Using podman-py with the Podman API {#09.htm#heading_id_8 .fm-head1}

[]{#09.htm#pgfId-1109735}Podman-py
[]{#09.htm#marker-1109732}[]{#09.htm#marker-1109733}[]{#09.htm#marker-1109734}([https://github.com/containers/podman-py](https://github.com/containers/podman-py){.url}),
like docker-py, is a Python bindings library used to communicate with
the Podman service. The podman-py library is newer than the docker-py
library and supports all of the advanced features of Podman using the
Libpod API.

[]{#09.htm#pgfId-1109736}The Podman Python library uses the default
locations of the podman.sock and connects to it automatically. When run
as non-root, the library connects to the rootless socket located in
/run/user/\$UID/podman/podman.sock. Running Python with the Podman
library as root connects automatically to /run/podman/podman.sock.

[]{#09.htm#pgfId-1109737}Similarly to docker-py, on my system I can
install the podman-py library via the `python-podman`{.fm-code-in-text}
package:

``` programlisting
$ sudo dnf install -y python-podman
Last metadata expiration check: 0:27:40 ago on Sun 19 Jun 2022 02:14:49 PM EDT.
Dependencies resolved.
...
Installed:
  python3-podman-3:4.0.0-1.fc36.noarch
Complete!
```

[]{#09.htm#pgfId-1109745}Now build a functionally similar script,
podman-images.py, using the podman-py library. This time you don't need
to worry about the location of the Podman socket. The podman-py library
connects to the default location:

``` programlisting
$ cat > podman-images.py << _EOF
import podman
client=podman.PodmanClient()
print(client.images.list())
_EOF
```

[]{#09.htm#pgfId-1109751}Run the script, and you will see the same
results as the docker-py example, but this library uses the Libpod API:

``` programlisting
$ python podman-images.py
[<Image: 'quay.io/rhatdan/myimage:latest'>, <Image: 'k8s.gcr.io/pause:3.5'>]
```

[]{#09.htm#pgfId-1109754}If you want to show advanced features, like
information on all the pods in the Podman database, call the
`pod.lists()`{.fm-code-in-text} function[]{#09.htm#marker-1109755}, and
iterate through each pod:

``` programlisting
$ cat >> podman-images.py << _EOF
for i in client.pods.list():
    print(i.attrs)
_EOF
Now the script shows the images as well as information on the pods.
$ python podman-images.py
[<Image: 'quay.io/rhatdan/myimage:latest'>, <Image: 'k8s.gcr.io/pause:3.5'>]
{'Cgroup': 'user.slice', 'Containers': [{'Id': 
➥ 'f8679839c25729eb422d38e505ae3a4b7ffe18942e2f77a997bd388e0f52313e', 
➥ 'Names': '116291543d56-infra', 'Status': 'configured'}], 'Created': 
➥ '2021-12-14T06:44:04.56055485-05:00', 'Id': '116291543d5691c597132ec73a428f29f2c1f71a65fdfbaca17eb5440a5d47f6', 
➥ 'InfraId': 'f8679839c25729eb422d38e505ae3a4b7ffe18942e2f77a997bd388e0f52313e', 
➥ 'Name': 'mypod', 'Namespace': '', 'Networks': None, 'Status': 
➥ 'Created', 'Labels': {}}
```

[]{#09.htm#pgfId-1109770}As you can see with the Python bindings, you
could begin to build a Python version of Podman, which can communicate
with the remote
[]{#09.htm#marker-1109771}[]{#09.htm#marker-1109772}[]{#09.htm#marker-1109773}socket.

### []{#09.htm#pgfId-1109775}9.3.3 Which Python library should you use? {#09.htm#heading_id_9 .fm-head1}

[]{#09.htm#pgfId-1109778}The
[]{#09.htm#marker-1109776}[]{#09.htm#marker-1109777}developers of the
podman-py library based their design on the
docker-py[]{#09.htm#marker-1116555} library to make it easier for
developers to transition. If you want to build an application that works
with Podman and Docker, the only choice is docker-py because podman-py
does not work with Docker. If you want to take advantage of advanced
features of Podman, you have to use podman-py. Podman-py is under heavy
development, but docker-py has a huge installed base. Podman-py works
out of the box with rootful and rootless Podman service, while if you
use docker-py you have to set the `DOCKER_ HOST`{.fm-code-in-text}
environment variable[]{#09.htm#marker-1109779} to point at the
podman.socket. Table 9.4 compares the features of the podman-py and
docker-py libraries to help you understand when to use a particular
library.[]{#09.htm#id_7luw6aqoktk5}

[]{#09.htm#pgfId-1112881}Table 9.4 Podman-py vs. docker-py

+-----------------------------------+-----------------+-----------------+
| []{#09.htm#pgfId-1112887}Support  | []{#            | []{#            |
|                                   | 09.htm#pgfId-11 | 09.htm#pgfId-11 |
|                                   | 12889}Podman-py | 12891}Docker-py |
+-----------------------------------+-----------------+-----------------+
| []{#09.htm#pgfId-1112893}Podman   | []{#09          | []{#09          |
| service                           | .htm#pgfId-1112 | .htm#pgfId-1112 |
|                                   | 895}[✔]{.segoe} | 897}[✔]{.segoe} |
+-----------------------------------+-----------------+-----------------+
| []{#09.htm#pgfId-1112899}Docker   | []{#09          | []{#09          |
| daemon                            | .htm#pgfId-1112 | .htm#pgfId-1112 |
|                                   | 901}[✘]{.segoe} | 903}[✔]{.segoe} |
+-----------------------------------+-----------------+-----------------+
| []{#09.htm#pgfId-1112905}Supports | []{#09          | []{#09          |
| pods                              | .htm#pgfId-1112 | .htm#pgfId-1112 |
|                                   | 907}[✔]{.segoe} | 909}[✘]{.segoe} |
+-----------------------------------+-----------------+-----------------+
| []{#09.htm#pgfId-1112911}Advanced | []{#09          | []{#09          |
| Podman features                   | .htm#pgfId-1112 | .htm#pgfId-1112 |
|                                   | 913}[✔]{.segoe} | 915}[✘]{.segoe} |
+-----------------------------------+-----------------+-----------------+

[]{#09.htm#pgfId-1109815}Using the low-level Python libraries docker-py
and podman-py for communicating with container engine daemons and
services, engineers developed higher-level tools to orchestrate and
manage containers. The most popular of these
[]{#09.htm#marker-1109816}[]{#09.htm#marker-1109817}is
[]{#09.htm#marker-1109818}[]{#09.htm#marker-1109819}`docker-compose`{.fm-code-in-text}.

## []{#09.htm#pgfId-1109821}9.4 Using docker-compose with the Podman service {#09.htm#heading_id_10 .fm-head}

[]{#09.htm#pgfId-1109824}In []{#09.htm#marker-1109822}the previous
chapters, you have seen how to manage containers with the Podman command
line as well as how to manage multiple containers using Kubernetes YAML
launched with `podman`{.fm-code-in-text} `play`{.fm-code-in-text}
`kube`{.fm-code-in-text}. You were introduced to launching containers
with Kubernetes. In this section, you will work with yet another
orchestration tool, `docker-compose`{.fm-code-in-text}
([https://docs.docker.com/compose](https://docs.docker.com/compose){.url}),
often referred to as just `compose`{.fm-code-in-text}.

[]{#09.htm#pgfId-1109828}`compose`{.fm-code-in-text}[]{#09.htm#marker-1109826}
is one of the most popular tools for launching containers. The
`compose`{.fm-code-in-text} tool[]{#09.htm#marker-1109827} predates
Kubernetes and concentrates on orchestrating multiple containers on a
single node, whereas Kubernetes orchestrates multiple containers on
multiple nodes. `compose`{.fm-code-in-text}, like Kubernetes, uses a
YAML file for its container definitions. One of the reasons
`compose`{.fm-code-in-text} was created was that building complex
command lines to run multiple containers can be complicated. Using a
structured language like YAML makes it easier to support running complex
applications with multiple containers on a single node.

[]{#09.htm#pgfId-1109832}`compose`{.fm-code-in-text} has a huge user
base, and it is likely you might want to run a
`compose`{.fm-code-in-text} YAML file in your infrastructure. If you
don't believe this will happen, you can skip this section.

[]{#09.htm#pgfId-1109834}The `compose`{.fm-code-in-text} tool was
written using docker-py and launches containers by using the Docker REST
API. Since Podman now supports the `compat`{.fm-code-in-text} REST
API[]{#09.htm#marker-1109835}, it also supports using
`docker-compose`{.fm-code-in-text} to launch Podman containers. Because
Podman works in rootless as well as rootful mode, you can even use
`docker-compose`{.fm-code-in-text} to launch rootless Podman containers.

[]{#09.htm#pgfId-1109836}In the rest of this section, you will create a
`compose`{.fm-code-in-text} YAML file just to get a feel for how the
`compose`{.fm-code-in-text} command works with the Podman service. You
first need to install `docker-compose`{.fm-code-in-text}. On my Fedora
system, I can do this with the following command:

``` programlisting
$ sudo dnf -y install docker-compose
```

[]{#09.htm#pgfId-1109838}Make sure the Podman systemd socket-activated
service is running by running the following command:

``` programlisting
$ systemctl –user start podman.socket
```

[]{#09.htm#pgfId-1109840}Verify the system service is running by hitting
the ping endpoint, and see if you get a response. This step needs to be
successful before you can proceed further.

``` programlisting
$ curl -H "Content-Type: application/json" --unix-socket 
➥ $XDG_RUNTIME_DIR/podman/podman.sock http://localhost/_ping
OK
```

[]{#09.htm#pgfId-1109845}Since `docker-compose`{.fm-code-in-text}
supports the `DOCKER_HOST`{.fm-code-in-text} environment
variable[]{#09.htm#marker-1109844}, make sure it is set using this
command:

``` programlisting
$ export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/podman/podman.sock
```

[]{#09.htm#pgfId-1109848}As was stated earlier in this section,
`compose`{.fm-code-in-text}[]{#09.htm#marker-1109847} supports its own
YAML file, which is different than the Kubernetes YAML described in
chapter 8.

[]{#09.htm#pgfId-1109849}First, create a directory called example, and
then navigate into it. Move the html directory you have been using into
the example directory:

``` programlisting
$ mkdir example
$ mv ./html example
$ cd example
```

[]{#09.htm#pgfId-1109853}You need to create the docker-compose.yaml file
in the example directory you have been working in. The YAML file will
create a container called
`myapp`{.fm-code-in-text}[]{#09.htm#marker-1109854} based on quay.io/
rhatdan/myimage:latest. Set up the container to use volumes from the
host ./html directory as well as a built-in volume,
`myapp_vol`{.fm-code-in-text}[]{#09.htm#marker-1109855}, used just for
the example:

``` programlisting
cat > docker-compose.yaml << _EOF
version: "3.7"
services:
  myapp:
    image: quay.io/rhatdan/myimage:latest
    volumes:
      - ./html:/var/www/html
      - myapp_vol:/vol
    ports:
      - 8080:80
volumes:
  myapp_vol: {}
_EOF
```

[]{#09.htm#pgfId-1109869}Now clean up the images and containers you have
on your system to make sure you are starting from a clean slate. Run the
following commands to do that:

``` programlisting
$ podman pod rm --all --force
$ podman rm --all --force
$ podman rmi --all --force
$ podman volume rm --all --force
```

[]{#09.htm#pgfId-1109875}To show how
`compose`{.fm-code-in-text}[]{#09.htm#marker-1109874} interacts with the
Podman service, launch the container with the
`compose`{.fm-code-in-text} command[]{#09.htm#marker-1109876}. Notice
that `compose`{.fm-code-in-text} tells Podman to pull down the image.
Then `compose`{.fm-code-in-text} tells Podman to create a container
named `example_myapp_1`{.fm-code-in-text} along with a volume named
`example_myapp_vol`{.fm-code-in-text}[]{#09.htm#marker-1109877}, which
will be volume mounted into the container along with the ./html
directory.

[]{#09.htm#pgfId-1109878}Listing 9.2 The output of executing
`docker-compose`{.fm-code-in-text} against the Podman socket

``` programlisting
$ docker-compose up
Pulling myapp (quay.io/rhatdan/myimage:latest)...   ❶
59bf1c3509f3: Download complete
c059bfaa849c: Download complete
Creating example_myapp_1 ... done                   ❷
Attaching to example_myapp_1
```

[]{#09.htm#pgfId-1116983}[❶]{.fm-combinumeral} Pulling the myimage image

[]{#09.htm#pgfId-1117011}[❷]{.fm-combinumeral} Creating the
example_myapp_1 container

[]{#09.htm#pgfId-1109887}In a different terminal, run the
`podman`{.fm-code-in-text} `ps`{.fm-code-in-text} command:

``` programlisting
$ podman ps --format "{{.ID}}  {{.Image}}  {{.Ports}}  {{.Names}}"
230fce823ff6  quay.io/rhatdan/myimage:latest  0.0.0.0:8080->80/tcp  
➥ example_myapp_1
```

[]{#09.htm#pgfId-1109891}Now check to see if Podman created a volume:

``` programlisting
$ podman volume ls
DRIVER    VOLUME NAME
local     example_myapp_vol
```

[]{#09.htm#pgfId-1109895}Go back to the original window, and enter
Ctrl-C to stop `docker-compose`{.fm-code-in-text}:

``` programlisting
^CGracefully stopping... (press Ctrl+C again to force)
Stopping example_myapp_1   ... done
```

[]{#09.htm#pgfId-1109898}This will shut down the container:

``` programlisting
$ podman ps --format "{{.ID}}  {{.Image}}  {{.Ports}}  {{.Names}}"
```

[]{#09.htm#pgfId-1109900}If you execute the `podman`{.fm-code-in-text}
`ps`{.fm-code-in-text} `-a`{.fm-code-in-text} command, you will see that
the container still exists but is not running:

``` programlisting
$ podman ps -a --format "{{.ID}}  {{.Image}}  {{.Ports}}  {{.Names}}"
230fce823ff6  docker.io/library/alpine:latest  0.0.0.0:8080->80/tcp  
➥ example_myapp_1
```

[]{#09.htm#pgfId-1109904}Now, if you run
`docker-compose`{.fm-code-in-text} `down`{.fm-code-in-text}, it will
tell Podman to remove the container from the system:

``` programlisting
$ docker-compose down
Removing example_myapp_1 ... done
Removing network example_default
```

[]{#09.htm#pgfId-1109909}Verify all containers are gone with the
`podman`{.fm-code-in-text} `ps`{.fm-code-in-text} `-a`{.fm-code-in-text}
command[]{#09.htm#marker-1109908} again:

``` programlisting
$ podman ps -a --format "{{.ID}}  {{.Image}}  {{.Ports}}  {{.Names}}"
```

[]{#09.htm#marker-1109930}[]{#09.htm#pgfId-1109911}As you can see,
Podman works nicely with `docker-compose`{.fm-code-in-text} to
orchestrate containers.

[]{#09.htm#pgfId-1109912}[Tip]{.fm-callout-head} While
`docker-compose`{.fm-code-in-text1} works nicely with the Podman
service, I think if you are starting a fresh project, it is better to
work with Kubernetes YAML and `podman`{.fm-code-in-text1}
`play`{.fm-code-in-text1} `kube`{.fm-code-in-text1} because this allows
you to more easily move your containers into Kubernetes.

[]{#09.htm#pgfId-1109913}As you have seen, the Podman service is useful
for allowing remote processes to manipulate your pods and containers.
Even the Podman command can be used as a client and communicate with the
Podman []{#09.htm#marker-1109914}[]{#09.htm#marker-1109915}service.

## []{#09.htm#pgfId-1109917}9.5 podman - -remote {#09.htm#heading_id_11 .fm-head}

[]{#09.htm#pgfId-1109920}As
[]{#09.htm#marker-1109918}[]{#09.htm#marker-1109919}you scale out
applications, you probably want to run your containerized applications
on multiple machines. You could `ssh`{.fm-code-in-text} into each box
and run Podman commands locally to manage the environment, or you could
write code to use the Python library described in section 9.4. The
Podman developers also built client support into the Podman command. You
can use the `podman`{.fm-code-in-text} command[]{#09.htm#marker-1109921}
to directly connect to these remote Podman services and manage the
container environment on the remote machines.

[]{#09.htm#pgfId-1109923}The Podman command has a special option,
`--remote`{.fm-code-in-text}[]{#09.htm#marker-1117659}, allowing it to
communicate with the socket-activated Podman service. Instead of
executing the commands and containers as a child of the Podman process,
it communicates with the service over the REST API.

::: figure
![](images/OEBPS/Images/09-02.png){.calibre18}

[]{#09.htm#pgfId-1117688}[]{#09.htm#id_y3fj7xwuq1cf}Figure 9.2
`podman`{.fm-code-in-text} `--remote`{.fm-code-in-text} connecting to
local podman.socket
:::

[]{#09.htm#pgfId-1109931}Because Podman is a tool for running Linux
containers, the `complete`{.fm-code-in-text} `podman`{.fm-code-in-text}
command[]{#09.htm#marker-1117669} can only be run on Linux. The Podman
developers wanted to support other operating systems, at least in client
mode. To support running Podman on non-Linux machines, Podman can be
built in two different ways. Up until now, you have been working with
the fully fledged Podman, which has the `--remote`{.fm-code-in-text}
option[]{#09.htm#marker-1117670}. The Podman executable can be compiled
with only support for communicating with the Podman service. Podman
built this way is often called `podman-remote`{.fm-code-in-text}. The
`podman-remote`{.fm-code-in-text} command[]{#09.htm#marker-1117671} is
the command that is shipped on some operating systems, like Mac and
Windows (covered more fully in appendixes E and F). If you have been
testing Podman on a Mac or Windows machine while reading this book, then
you have already been using `podman-remote`{.fm-code-in-text}, which
transparently communicates with the Podman service running in a VM or on
a different machine.

### []{#09.htm#pgfId-1109935}9.5.1 Local connections {#09.htm#heading_id_12 .fm-head1}

[]{#09.htm#pgfId-1109941}[]{#09.htm#id_vtahb6ww8z16}As[]{#09.htm#marker-1115139}[]{#09.htm#marker-1115140}[]{#09.htm#marker-1115141}
previously mentioned, the `podman`{.fm-code-in-text}
`--remote`{.fm-code-in-text} command[]{#09.htm#marker-1115154} connects,
by default, to the local podman.socket, referred to as a local
connection (figure 9.2). Try out `podman`{.fm-code-in-text}
`--remote`{.fm-code-in-text} with the Podman system service you enabled
in section 9.1.1. Notice how the `podman`{.fm-code-in-text}
`--remote`{.fm-code-in-text} version shows you the version of the Podman
client as well as the Podman server; in this case, they are the same
executable.

[]{#09.htm#pgfId-1109942}Listing 9.3 The output of
`podman`{.fm-code-in-text} `--remote`{.fm-code-in-text} executing the
version API

``` programlisting
$ podman --remote version
Client:
Version:      4.1.0        ❶
API Version:  4.1.0
Go Version:   go1.18.2
Built:        Sun Jun 19 07:35:42 2022
OS/Arch:      linux/amd64
Server:
Version:      4.1.0        ❷
API Version:  4.1.0
Go Version:   go1.18.2
Git Commit:   a2b78b627f0a9deef83a5b5e4ecffc9cdb5a72b1-dirty
Built:        Sun Jun 19 07:35:42 2022
OS/Arch:      linux/amd64
```

[]{#09.htm#pgfId-1116895}[❶]{.fm-combinumeral} Client version of Podman

[]{#09.htm#pgfId-1116896}[❷]{.fm-combinumeral} Server version of Podman

[]{#09.htm#pgfId-1109959}You can use the exact same commands to start
the container:

``` programlisting
$ podman --remote run ubi8 echo hi
Resolved "ubi8" as an alias (/etc/containers/registries.conf.d/
➥ 000-shortnames.conf)
Trying to pull registry.access.redhat.com/ubi8:latest...
..
hi
```

[]{#09.htm#pgfId-1109967}[]{#09.htm#id_aphjd9kd4zks}As you can imagine,
it is not that useful in this mode, since you can run Podman without the
`--remote`{.fm-code-in-text} optio[]{#09.htm#marker-1109968}n and manage
the same container environment. Local connections are mainly used for
testing of the API, especially in continuous integration
([]{#09.htm#marker-1109969}CI) systems. `podman`{.fm-code-in-text}
`--remote`{.fm-code-in-text} becomes much more interesting when you use
it to communicate with truly
remo[]{#09.htm#marker-1109970}[]{#09.htm#marker-1109971}[]{#09.htm#marker-1109972}te
machines.

### []{#09.htm#pgfId-1109974}9.5.2 Remote connections {#09.htm#heading_id_13 .fm-head1}

[]{#09.htm#pgfId-1109978}The
[]{#09.htm#marker-1112090}[]{#09.htm#marker-1112091}[]{#09.htm#marker-1112092}main
purpose of the `podman`{.fm-code-in-text} `--remote`{.fm-code-in-text}
command is allowing you to manipulate pods and containers on a separate
machine using the Podman service. Install Podman on a Linux machine or
VM, which also has the SSH daemon running. On the local operating
system, when you run a Podman command, Podman connects to the server via
SSH. It then connects to the Podman service by using systemd socket
activation and communicating with our REST API, as shown in figure
9.3.[]{#09.htm#id_x2sukq7nfn9t}[]{#09.htm#id_xehzr5i0ipj2}[]{#09.htm#id_b0mmeax3hyir}[]{#09.htm#id_u61su794t7r7}[]{#09.htm#id_h07up9l04rih}[]{#09.htm#id_w7f0w92kjs8q}

::: figure
![](images/OEBPS/Images/09-03.png){.calibre18}

[]{#09.htm#pgfId-1117726}Figure 9.3 `podman --remote`{.fm-code-in-text}
connecting over SSH to the server machine
:::

[]{#09.htm#pgfId-1109991}The command-line interface of Podman with the
`--remote`{.fm-code-in-text} option[]{#09.htm#marker-1116001} is exactly
the same as the regular Podman commands. When you run the Podman
commands, it feels like you are running the containers locally; however,
the container processes are running on the remote machine. There are a
few options that are not supported in remote mode, listed in table
9.5.[]{#09.htm#id_deb3mbem3z0l}

[]{#09.htm#pgfId-1113372}Table 9.5 Options not supported by the
`podman --remote`{.fm-code-in-text} command

+-----------------+-----------------------------------------------------+
| []              | []{#09.htm#pgfId-1113378}Explanation                |
| {#09.htm#pgfId- |                                                     |
| 1113376}Options |                                                     |
+-----------------+-----------------------------------------------------+
| []{#09.htm      | []{#09.htm#pgfId-1113382}The environment on two     |
| #pgfId-1113380} | different machines makes little sense to share; in  |
| `--env-host`    | some cases these can be two different operating     |
| {.fm-code-in-te | systems, like Windows and Macs talking to a Linux   |
| xt1}[]{#09.htm# | Podman service.                                     |
| marker-1116014} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#0           | []{#09.htm#pgfId-1113386}The                        |
| 9.htm#pgfId-111 | `--group-add`{.fm-code-in-text1} option works in    |
| 3384}`--group-a | `--remote`{.fm-code-in-text1} mode, but the         |
| dd=keep-groups` | `keep-groups`{.fm-code-in-text1} special flag does  |
| {.fm-code-in-te | not. The `keep-groups`{.fm-code-in-text1} flag      |
| xt1}[]{#09.htm# | tells Podman to leak the groups that the current    |
| marker-1116020} | process has access to into the container. Since     |
|                 | this is a client-server procedure, the leaking is   |
|                 | impossible.                                         |
+-----------------+-----------------------------------------------------+
| []{#09.ht       | []{#09.htm#pgfId-1113390}The                        |
| m#pgfId-1113388 | `--http-proxy`{.fm-code-in-text1} option tells      |
| }`--http-proxy` | Podman to use the HTTP proxy environment variables  |
| {.fm-code-in-te | off of the client machine and leak them into the    |
| xt1}[]{#09.htm# | server. Since the proxy is normally set up on the   |
| marker-1116024} | server, the `--http-proxy`{.fm-code-in-text1}       |
|                 | option is not allowed with the                      |
|                 | `--remote`{.fm-code-in-text1} option.               |
+-----------------+-----------------------------------------------------+
| []{#09.htm#     | []{#09.htm#pgfId-1113394}The                        |
| pgfId-1113392}` | `--preserve-fds`{.fm-code-in-text1} option leaks    |
| --preserve-fds` | file descriptors from the calling process into the  |
| {.fm-code-in-te | container; since this is a remote connection, there |
| xt1}[]{#09.htm# | is no way to leak the file descriptors.             |
| marker-1116029} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#0           | []{#09.htm#pgfId-1113398}This is supported, except  |
| 9.htm#pgfId-111 | that the source volume will come from the remote    |
| 3396}`--volume` | machine, not necessarily the one that is running    |
| {.fm-code-in-te | the `podman`{.fm-code-in-text1} command (unless     |
| xt1}[]{#09.htm# | they are on the same machine). If you are using a   |
| marker-1116034} | VM, you need to mount the directory on the host     |
|                 | machine into the VM first; then Podman inside of    |
|                 | the VM sees the mount and mounts it into the        |
|                 | container.                                          |
+-----------------+-----------------------------------------------------+
| []{#09.htm      | []{#09.htm#pgfId-1113402}Since there are            |
| #pgfId-1113400} | potentially multiple different users talking to the |
| `--latest`{.fm- | same server at the same time, the concept of        |
| code-in-text1}, | `--latest`{.fm-code-in-text1} was too racy, so it   |
| `-l`            | is not supported.                                   |
| {.fm-code-in-te |                                                     |
| xt1}[]{#09.htm# |                                                     |
| marker-1116039} |                                                     |
+-----------------+-----------------------------------------------------+

[]{#09.htm#pgfId-1110031}Podman commands are executed on the server.
From the client's point of view, it seems like Podman runs locally. Now
you need to complete the configuration of the Podman service on the
remote server.

[]{#09.htm#pgfId-1110033}Enabling SSHD connections

[]{#09.htm#pgfId-1110037}For
[]{#09.htm#marker-1110034}[]{#09.htm#marker-1110035}[]{#09.htm#marker-1110036}the
Podman client to communicate with the server, you need to enable and
start the SSH daemon on your Linux machine, if it is not already
enabled:

``` programlisting
$ sudo systemctl enable --now -s sshd
```

[]{#09.htm#pgfId-1110039}Now that the SSHD daemon is running, you need
to enable the Podman service on the remote
[]{#09.htm#marker-1116093}[]{#09.htm#marker-1116094}[]{#09.htm#marker-1116095}machine.

[]{#09.htm#pgfId-1110044}Enabling the Podman service on the server
machine

[]{#09.htm#pgfId-1110048}Before
[]{#09.htm#marker-1116097}[]{#09.htm#marker-1116098}[]{#09.htm#marker-1116099}performing
any Podman client commands, you must enable the podman.sock systemd
service on the Linux server or VM. In these examples, you are running
Podman as a normal, unprivileged user. For rootless Podman on a server
to run properly, enable this socket permanently using the following
command:

``` programlisting
$ systemctl --user enable --now podman.socket
```

[]{#09.htm#pgfId-1110050}Normally, when you log out of a system, systemd
stops all processes on the system. You need to tell systemd to allow the
remote users processes to `linger`{.fm-code-in-text} for rootless mode:

``` programlisting
$ sudo loginctl enable-linger $USER
```

[]{#09.htm#pgfId-1110052}This also tells systemd to start listening on
this socket at boot time. Once you have the service running on one
system, you can verify the socket is listening with a Podman command:

``` programlisting
$ podman --remote info
Host:
  arch: amd64
  buildahVersion: 1.16.0-dev
...
```

[]{#09.htm#pgfId-1110058}[Note]{.fm-callout-head} You can enable the
rootful `podman`{.fm-code-in-text1} service with the following command:

``` programlisting
$ sudo systemctl enable --now podman.socket
```

[]{#09.htm#pgfId-1110060}The previous `enable-linger`{.fm-code-in-text}
command is only for rootless mode. Now that you have the remote service
enabled and running along with the SSHD daemon, you can go back to the
[]{#09.htm#marker-1110061}[]{#09.htm#marker-1110062}[]{#09.htm#marker-1110063}client
[]{#09.htm#marker-1110064}[]{#09.htm#marker-1110065}[]{#09.htm#marker-1110066}machine.

### []{#09.htm#pgfId-1110068}9.5.3 Setting up SSH on the client machine {#09.htm#heading_id_14 .fm-head1}

[]{#09.htm#pgfId-1110073}Remote
[]{#09.htm#marker-1110069}[]{#09.htm#marker-1110070}[]{#09.htm#marker-1110071}[]{#09.htm#marker-1110072}Podman
uses SSH to communicate between the client and server when they are on
separate machines. By default, SSH will ask you to provide the usernames
and passwords on each command, unless you set up SSH keys. To set up
your SSH connection, you need to generate an SSH key pair from your
client machine. If you have existing SSH keys, you can just use them;
it's even better if you already have shared keys with the server. On my
Linux system, I can generate SSH keys with a command like the following:

``` programlisting
$ ssh-keygen -t ed25519
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/myuser/.ssh/id_ed25519):
```

[]{#09.htm#pgfId-1110077}Once you have finished generating your keys,
you can set up trust between the client and server machine with the
`ssh-copy-id`{.fm-code-in-text} command[]{#09.htm#marker-1110078} or
some similar command. The public key, by default, will be in your home
directory under \$HOME/.ssh/id_ed25519 .pub. You need to copy the
contents of id_ed25519.pub and append it into \~/.ssh/ authorized_keys
on the Linux server. See
[https://red.ht/3HuxPT6](https://red.ht/3HuxPT6){.url} for more
information on configuring your SSH environment:

``` programlisting
$ ssh-copy-id myuser@192.168.122.1
passwd:
```

[]{#09.htm#pgfId-1110081}If you do not wish to use SSH keys, you will be
prompted with each Podman command for your login password. Now that you
have shared your SSH keys with the server, the next step is configuring
the connection with
[]{#09.htm#marker-1110082}[]{#09.htm#marker-1110083}[]{#09.htm#marker-1110084}[]{#09.htm#marker-1110085}Podman.

### []{#09.htm#pgfId-1110087}9.5.4 Configuring a connection {#09.htm#heading_id_15 .fm-head1}

[]{#09.htm#pgfId-1110091}The
[]{#09.htm#marker-1110088}[]{#09.htm#marker-1110089}`podman`{.fm-code-in-text}
`system`{.fm-code-in-text} `connection`{.fm-code-in-text} command
allows[]{#09.htm#marker-1110090} you to manage SSH connections to be
used by the `podman`{.fm-code-in-text} `--remote`{.fm-code-in-text}
command. You can add a connection by using the
`podman`{.fm-code-in-text} `system`{.fm-code-in-text}
`connection`{.fm-code-in-text} `add`{.fm-code-in-text}
command[]{#09.htm#marker-1110092}; name the connection
`server1`{.fm-code-in-text}. The default identity file will be chosen,
or you can use the `-–identity`{.fm-code-in-text}
option[]{#09.htm#marker-1110093} to specify the SSH key to use. Finally,
you need to specify the full SSH URL for the Podman socket. This
includes the user account,
`myuser`{.fm-code-in-text}[]{#09.htm#marker-1110094}, and IP address, as
well as the path to the Podman socket for the user account:

``` programlisting
$ podman system connection add server1 --identity ~/.ssh/id_ed25519 
➥ ssh://myuser@192.168.122.1/run/user/1000/podman/podman.sock
```

[]{#09.htm#pgfId-1110097}This Podman command adds a remote connection to
Podman. Since this was the first connection added, Podman marks the
connection as the default.

[]{#09.htm#pgfId-1110099}List the available connections with the
`podman`{.fm-code-in-text} `system`{.fm-code-in-text}
`connection`{.fm-code-in-text} `list`{.fm-code-in-text}
command[]{#09.htm#marker-1110098}. Notice that the `*`{.fm-code-in-text}
after the connection name indicates it is the default connection:

``` programlisting
$ podman system connection list
Name    Identity          URI
system1*    id_ed25519     
➥ ssh://myuser@192.168.122.1/run/user/1000/podman/podman.sock
```

[]{#09.htm#pgfId-1110104}Now you can test the connection with
`podman`{.fm-code-in-text} `info`{.fm-code-in-text}:

``` programlisting
$ podman --remote info
host:
  arch: amd64
  buildahVersion: 1.23.1
  cgroupControllers:
...
```

[]{#09.htm#pgfId-1110111}[Note]{.fm-callout-head} You can use the
`--connection`{.fm-code-in-text1} `(-c)`{.fm-code-in-text1} if you have
more than one connection and want to choose the non-default
`man`{.fm-code-in-text1} `podman-system-connection`{.fm-code-in-text1}
for all possible options.

[]{#09.htm#pgfId-1110114}You can use the `podman`{.fm-code-in-text}
option[]{#09.htm#marker-1116219} or the
`podman-remote`{.fm-code-in-text} clients[]{#09.htm#marker-1116220} to
manage containers running on Linux servers or VMs. The communication
between client and server relies heavily on SSH connections, and the use
of SSH keys is encouraged. Once you have Podman installed on your remote
server, you need to set up a connection using `podman`{.fm-code-in-text}
`system`{.fm-code-in-text} `connection`{.fm-code-in-text}
`add`{.fm-code-in-text}, which can then be used by subsequent
[]{#09.htm#marker-1116222}[]{#09.htm#marker-1116223}Podman
[]{#09.htm#marker-1116224}[]{#09.htm#marker-1116225}commands.[]{#09.htm#id_n2kx94l968}
Table 9.6 lists the available Podman system commands.

[]{#09.htm#pgfId-1113524}Table 9.6 Podman system commands

+---------+------------------------+-----------------------------------+
| []{#    | []{#09                 | []{                               |
| 09.htm# | .htm#pgfId-1113532}Man | #09.htm#pgfId-1113534}Description |
| pgfId-1 | page                   |                                   |
| 113530} |                        |                                   |
| Command |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#0   | []{#09.                | []{#09.htm#pgfId-1113540}Manages  |
| 9.htm#p | htm#pgfId-1113538}`pod | remote SSH destinations           |
| gfId-11 | man-system-connection( |                                   |
| 13536}` | 1)`{.fm-code-in-text1} |                                   |
| connect |                        |                                   |
| ion`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 9.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 116332} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#    | []{#09.htm#pgfId-1113  | []{#09.htm#pgfId-1113546}Shows    |
| 09.htm# | 544}`podman-system-df( | Podman's disk usage               |
| pgfId-1 | 1)`{.fm-code-in-text1} |                                   |
| 113542} |                        |                                   |
| `df`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 9.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 116339} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#09  | [                      | []{#09.htm#pgfId-1113552}Displays |
| .htm#pg | ]{#09.htm#pgfId-111355 | Podman system information         |
| fId-111 | 0}`podman-system-info( |                                   |
| 3548}`i | 1)`{.fm-code-in-text1} |                                   |
| nfo`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 9.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 116346} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []      | []{#                   | []{#09.htm#pgfId-1113558}Migrates |
| {#09.ht | 09.htm#pgfId-1113556}` | containers to a new user          |
| m#pgfId | podman-system-migrate( | namespace                         |
| -111355 | 1)`{.fm-code-in-text1} |                                   |
| 4}`migr |                        |                                   |
| ate`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 9.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 116353} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#09. | []                     | []{#09.htm#pgfId-1113564}Removes  |
| htm#pgf | {#09.htm#pgfId-1113562 | unused pod, container, volume,    |
| Id-1113 | }`podman-system-prune( | and image data                    |
| 560}`pr | 1)`{.fm-code-in-text1} |                                   |
| une`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 9.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 116360} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{     | []{#0                  | []{#09.htm#pgfId-1113570}Migrates |
| #09.htm | 9.htm#pgfId-1113568}`p | lock numbers                      |
| #pgfId- | odman-system-renumber( |                                   |
| 1113566 | 1)`{.fm-code-in-text1} |                                   |
| }`renum |                        |                                   |
| ber`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 9.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 116367} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#09. | []                     | []{#09.htm#pgfId-1113576}Resets   |
| htm#pgf | {#09.htm#pgfId-1113574 | Podman storage                    |
| Id-1113 | }`podman-system-reset( |                                   |
| 572}`re | 1)`{.fm-code-in-text1} |                                   |
| set`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 9.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 116374} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []      | []{#                   | []{#09.htm#pgfId-1113582}Runs the |
| {#09.ht | 09.htm#pgfId-1113580}` | API service                       |
| m#pgfId | podman-system-service( |                                   |
| -111357 | 1)`{.fm-code-in-text1} |                                   |
| 8}`serv |                        |                                   |
| ice`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 9.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 116381} |                        |                                   |
+---------+------------------------+-----------------------------------+

## []{#09.htm#pgfId-1110187}Summary {#09.htm#heading_id_16 .fm-head}

-   []{#09.htm#pgfId-1110188 .calibre17}Podman can be run as a REST API
    service.

-   []{#09.htm#pgfId-1110189 .calibre17}Podman supports two REST API
    endpoints.

-   []{#09.htm#pgfId-1110190 .calibre17}The Podman socket supports two
    APIs.

-   []{#09.htm#pgfId-1110191 .calibre17}Compatibility mode or Docker
    mode allows Docker client tools to work with Podman.

-   []{#09.htm#pgfId-1110192 .calibre17}Podman mode allows remote
    clients to take advantage of advanced Podman features.

-   []{#09.htm#pgfId-1110193 .calibre17}Podman-py is a Python bindings
    library used to communicate with the Podman service.

-   []{#09.htm#pgfId-1110194 .calibre17}Docker-py is a Python bindings
    library used to communicate with the Podman compatibility service.

-   []{#09.htm#pgfId-1110195 .calibre17}Podman supports running
    `docker-compose`{.fm-code-in-text} with the compatibility service to
    orchestrate `compose`{.fm-code-in-text} containers on a single node.

-   []{#09.htm#pgfId-1110196 .calibre17}The `podman`{.fm-code-in-text}
    `--remote`{.fm-code-in-text} command communicates with the Podman
    service over SSH to manage containers.

-   []{#09.htm#pgfId-1110197 .calibre17}The `podman`{.fm-code-in-text}
    `system`{.fm-code-in-text} `connect`{.fm-code-in-text} command
    manages SSH connections to remote Podman services, making it easier
    to manage containers in your []{#09.htm#marker-1110198
    .calibre17}environment.

[]{#p4.htm}

## Part 4. Container security

[]{#p4.htm#pgfId-1017170}[I]{.fm-part-initial-cap}n the final part of
the book, part 4, I divulge all I know about container security. This
part is very technical, but you learn some key concepts that will help
you understand when a container gets permission denied. It also explains
the benefits of running applications within a container from a security
point of view. Containerizing applications adds tremendous protection
from potential hacks to your host system.

[]{#p4.htm#pgfId-1017171}In chapter 10, I explain all of the features of
the kernel that Podman uses to isolate containers from each other as
well as the host system. I explain SELinux, seccomp, Linux capabilities,
read-only mount points, and many other features.

[]{#p4.htm#pgfId-1017172}Chapter 11 digs into security considerations.
You learn the security best practices for running your containers in
production, how you should design your application, and how you should
run your containerized application in production.

[]{#10.htm}
