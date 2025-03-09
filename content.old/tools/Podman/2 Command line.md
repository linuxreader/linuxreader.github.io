# 2 Command line 

This chapter covers:
-   The Podman command line
-   Running an OCI application
-   Comparing containers and images
-   Building an OCI-based image

Podman is an excellent tool for running and building containerized applications. In this chapter, you'll get started
by building a simple web application to demonstrate commonly used features of the Podman command line.

If you don't have Podman installed on your machine, you can jump to appendix C, and then return here. This chapter assumes that Podman 4.1 or later is already installed. Older versions of Podman probably work fine, but all examples were tested with Podman 4.1.

The example base image I use is the registry.access.redhat.com/ubi8/httpd-24 image.

Universal Base Images (UBI can be used anywhere, but container software maintained and vetted by Red Hat as well as run on a Red Hat operating system is fully supported. There are hundreds of Apache images that work similarly to this image that you can also try out.

Chapter 2 shows how Podman is a great tool for working with containers. In this chapter, I walk you through running the
scenario you might use to build a containerized application. You launch a container, modify its contents, create an image, and ship it to a registry. Then I explain how you can do this in an automated way to maintain the security of your container image. Through it all, you will be exposed to many of the Podman command-line interfaces and get a good understanding of how to work with Podman.

If you are an experienced Docker user, you probably just want to skim through this chapter. You will know a lot of
it, but there are many features unique to Podman, such as the ability to mount container images (section 2.2.10) and different transports

Let's start by running our first container.

Podman is an open source project under heavy development. Podman is packaged and provided
on many different Linux distributions as well as Mac and Windows. These distributions might be shipping older versions of Podman without some of the current features covered in this book. Some examples in this book assume you are using Podman 4.1 or later. If an example does not work, please update your version of Podman to the latest version. Refer to appendix C for further information on installing Podman.

## 2.1 Working with containers

There are thousands of different container images sitting at container registries. Developers, administrators, quality engineers, and general users primarily use the `podman run` command  to pull down and run, test, or explore these container images. To start building out containerized applications, the first thing you need to do is start working with a base image. In our examples, you pull and run the registry.access.redhat.com/ubi8/httpd-24 image to container storage in your home directory and start exploring inside the container.

### 2.1.1 Exploring containers

In this section, you will examine a typical Podman command, step by step. You will execute
the `podman run` command, which reaches out to the registry.access.redhat.com container registry and begins pulling down the image and storing it locally in your home directory:

``` programlisting
$ podman run -ti --rm registry.access.redhat.com/ubi8/httpd-24 bash
```

Now I will break down the command you just executed. By default the `podman run` command executes the
containerized command in the foreground until the container exits. In this case, you end up at a Bash prompt running within the container and showing the `bash-4.4$` prompt. When you exit this Bash prompt, Podman stops the container.

In this example, you used two options:
`-t` and `-i`, as `-ti`, which tells Podman to hook up to the terminal.

This connects to the input, output, and error stream of the `bash` process within the container to your screen, which allows you to interact within the container:

```bash
$ podman run -ti --rm registry.access.redhat.com/ubi8/httpd-24 bash
```

The `--rm` option tells Podman to delete the container as soon as the container exits, freeing up all of the container's storage:

```bash
$ podman run -ti --rm registry.access.redhat.com/ubi8/httpd-24 bash
```

Next, specify the container image, registry.access.redhat.com/ubi8/httpd-24, you are working with. The `podman` command reaches out to the container registry at registry.access.redhat.com and begins copying down the ubi8/httpd-24:latest image. Podman copies multiple layers (aka blobs), as shown in the following listing, and stores them
in the local container storage. You see the progress as the image layers
are pulled down. Some images are rather large and can take a long time
while being pulled down. If you later run a different container on the
same image, Podman skips the image-pulling step, since you already have
the correct image in local container storage.

Listing 2.1 Pulling and running a container
image from a registry

``` programlisting
$ podman run -ti --rm registry.access.redhat.com/ubi8/httpd-24 bash
Trying to pull registry.access.redhat.com/
➥ ubi8/httpd-24:latest...                                   ❶
Getting image source signatures
Checking if image destination supports signatures        
Copying blob 296e14ee2414 skipped: already exists            ❷
Copying blob 356f18f3a935 skipped: already exists            ❷
Copying blob 359fed170a21                                    ❷
➥ [========================>---------] 11.8MiB / 16.2MiB    ❷
Copying blob 226cafc3a0c6                                    ❷
➥ [=====>----------------------------]                      ❷
➥ 10.1MiB / 61.1MiB                                         ❷
```

[❶]{.fm-combinumeral} Contact with the registry

[❷]{.fm-combinumeral} Layer pulling is skipped.

Finally, specify the executable to be run
within the container, in this case, `bash`:

``` programlisting
$ podman run -ti --rm registry.access.redhat.com/ubi8/httpd-24 bash 
...
bash-4.4$
```

[Note]{.fm-callout-head} Images almost always
have default commands they execute. You only have to specify a command
if you want to override the default application the image runs with. In
the case of the registry.access.redhat.com/ubi8/httpd-24 image, it runs
the Apache web server.

While inside the bash shell container, cat
/etc/os-release, and notice it is likely a different OS or a different
version than the /etc/os-release outside the container. Explore around
in the container, and notice how different it is from your host
environment:

``` programlisting
bash-4.4$ grep PRETTY_NAME /etc/os-release 
PRETTY_NAME="Red Hat Enterprise Linux 8.4 (Ootpa)"
```

On my host on a different terminal, the same
command outputs

``` programlisting
$ grep PRETTY_NAME /etc/os-release 
PRETTY_NAME="Fedora Linux 35 (Workstation Edition Prerelease)"
```

Back inside the container, you will notice
there are a lot fewer commands available:

``` programlisting
bash-4.4$ ls /usr/bin | wc -l
525
```

However, on the host you see

``` programlisting
$ ls -l /usr/bin | wc -l
3303
```

Execute the `ps`
command to see what processes are running
inside of the container:

``` programlisting
$ ps
PID TTY        TIME CMD
1 pts/0    00:00:00 bash
2 pts/0    00:00:00 ps
```

You only see two processes: the
`bash` script and the
`ps` command. Needless to
say, on my host machine, there are hundreds of processes running
(including these two processes). You can further explore the inside of
the container to gain an understanding of what is going on within a
container.

When you are done, you exit the
`bash` script, and the container shuts down. Since you
ran with the `--rm` option,
Podman removes all the container storage and deletes the container. The
container image remains in container/storage. Now that you have explored
the inner workings of a container, it is time to start working with the
default application within the
container.

### 2.1.2 Running the containerized application {#02.htm#heading_id_5 .fm-head1}

In
the
previous example, you pulled and ran `bash` within a
containerized application, but you did not run the application the
developer intended you to run. In this next example, you will run the
actual application by removing the command and running with a couple of
new options.

First, remove the `-ti` and
the `--rm` options, since you want the container to
remain running when the `podman`
command exits. You are not a shell running
within the container interactively, since it is just running the
containerized web service:

``` programlisting
$ podman run -d -p 8080:8080 --name myapp registry.access.redhat.com/ubi8/httpd-24
37a1d2e31dbf4fa311a5ca6453f53106eaae2d8b9b9da264015cc3f8864fac22
```

The first option to notice is the
`-d` `(--detach)`
option, which tells Podman to launch the
container and then detach from it. Basically, run the container in the
background. The Podman command actually exits and leaves the container
running. Chapter 6 goes much deeper into what is going on behind the
scenes:

``` programlisting
$ podman run -d -p 8080:8080 --name myapp registry.access.redhat.com/ubi8/httpd-24
```

The `-p`
`(--publish)` option tells Podman to publish or bind
the container port `8080` to the host port
`8080` when the container is running. With the
`-p` option, the field
before the colon refers to the host port, while the field after the
colon refers to the container port. In this case, you see that the ports
are the same. If you specify only one port,
Podman considers this port a container port and randomly picks a host
port on which the container port is bound. You can use the
`podman` `port`
command to discover which ports are bound to a
container.

Listing 2.2 Example of the
`podman` `port` command

``` programlisting
$ podman port myapp
8080/tcp -> 0.0.0.0:8080    ❶
```

[❶]{.fm-combinumeral} Shows that port 8080/tcp
inside the container is bound to all of the host networks (0.0.0.0) at
port 8080

By default, containers are created within their
own network namespace, meaning they are not bound to the host network
but to their virtualized network. Suppose I execute the container
without the `-p` option. In
that case, the Apache server within the container binds to the network
interface within the container's network namespace, but Apache is not
bound to the host network.

Only processes within the container are able to
connect to port `8080` to communicate with the web
server. By executing the command with the `-p`
option, Podman connects the port from inside
the container to the host network at the specified port. The connection
allows external processes like a web browser to read from the web
service.

[Note]{.fm-callout-head} If you are running
containers in rootless mode, covered in chapter 3, Podman users are by
default not permitted to bind to ports \< 1024 by the kernel. Some
containers want to bind to lower ports like port 80, which is allowed
inside the container, but `-p`{.fm-code-in-text1}
`80:80`{.fm-code-in-text1} fails, since 80 is less than 1024. Using
`-p`{.fm-code-in-text1} `8080:80`{.fm-code-in-text1} causes Podman to
bind the host's port `8080`{.fm-code-in-text1} to port
`80`{.fm-code-in-text1} within the container. The upstream Podman repo
contains troubleshooting information on problems like binding to ports
less than 1024 and many others (see
[http://mng.bz/69ry](http://mng.bz/69ry){.url}).

The `-p`
option can map port numbers inside the
container to different port numbers outside the container:

``` programlisting
$ podman run -d -p 8080:8080 --name myapp registry.access.redhat.com/ubi8/httpd-24
```

In the example name, the container
`myapp` is using the `--name`
`myapp` option. Specifying a
name makes it easier to find the container, and it allows you to specify
a name that can then be used for other commands (e.g.,
`podman` `stop`
`myapp`). If you don't
specify a name, Podman automatically generates a unique container name
along with a container ID. All of the Podman commands that interact with
containers can use either the name or the ID:

``` programlisting
$ podman run -d --name myapp -p 8080:8080 registry.access.redhat.com/ubi8/httpd-24
```

When the `podman`
`run` command completes, the
container is running. Since this container is running in detached mode,
Podman prints out the container ID and exits, but the container remains
running:

``` programlisting
$ podman run -d -p 8080:8080 --name myapp registry.access.redhat.com/ubi8/httpd-24
37a1d2e31dbf4fa311a5ca6453f53106eaae2d8b9b9da264015cc3f8864fac22
```

Now that the container is running, you can
launch a web browser to communicate with the web server inside of the
container at localhost port `8080` (see figure 2.1):

``` programlisting
$ web-browser localhost:8080
```

::: figure
![](images//02-01.png)

Figure 2.1 Web browser window connecting to the
ubi8/httpd-24 container running in Podman
:::

Congratulations! You have launched your first
containerized application.

Now imagine you want to start another
container. You can execute a similar command with just a couple of
changes:

``` programlisting
$ podman run -d -p 8081:8080 --name myapp1 \ 
➥ registry.access.redhat.com/ubi8/httpd-24
fa41173e4568a8fa588690d3177150a454c63b53bdfa52865b5f8f7e4d7de1e1
```

Notice you need to change the name of the
container to `myapp1`; otherwise, the
`podman` `run` command fails with
the `myapp` name because the container previously
existed. Also you need to change the `-p` option to
use `8081` for the host port because the previous
container, `myapp`, is
currently running and is bound to port `8080`. The
second container isn't allowed to bind to port `8080`
until the first container exits:

``` programlisting
$ podman run -d -p 8081:8080 --name myapp1  
     registry.access.redhat.com/ubi8/httpd-24
```

The `podman`
`create` command is almost
identical to the `podman` `run`
command. The `create`
command pulls the image if it is not in container storage and configures
the container information to make it ready to run but never executes the
container. It is often used together with the `podman`
`start` command described in
section 2.1.4. You might want to create a container and then later use a
systemd unit file to start and stop the container.

[]{#02.htm#id_qoinjfin0ayc1111}Some notable
`podman` `run` options include the
following:

-   \--`user`
    `USERNAME`
    .calibre17}---This tells Podman to run the container as a specific
    user defined in the image. By default, Podman will run the container
    as root, unless the container image specifies a default user.

-   
    .calibre17}`--rm`
    .calibre17}---This automatically removes the container when it
    exits.

-   `--tty`
    `-(t`
    .calibre17}`)`---This allocates a pseudo
    `-tty` and attaches it to the standard input of
    the container.

-   
    .calibre17}`--interactive`
    `(-i`
    .calibre17}`)`---This connects
    `stdin` to the primary process of the container.
    These options give you an interactive shell within the container.

[Note]{.fm-callout-head} There are dozens of
`podman`{.fm-code-in-text1} `run`{.fm-code-in-text1} options available,
allowing you to change security features, namespaces, volumes, and so
on. Some of these I use and explain throughout the book. Refer to the
`podman-run`{.fm-code-in-text1} man page for a
description of all of the options. Most of the
`podman`{.fm-code-in-text1} `create`{.fm-code-in-text1} options defined
in table 2.1 are also available for `podman`{.fm-code-in-text1}
`run`{.fm-code-in-text1}.

Use the `man`
`podman-run` command for
information about all options. Now that the container is up and running,
it is time to stop the container and go to the next
step.

### 2.1.3 Stopping containers {#02.htm#heading_id_6 .fm-head1}

You
have
two containers running and have tested them by running a web browser
against them. To continue the development by actually adding some
content to the web page, you can stop the containers using the
`podman` `stop`
command:

``` programlisting
$ podman stop myapp
```

The `stop` command stops the
container started with the previous `podman`
`run` command.

When stopping a container, Podman examines the
running container and sends a stop signal, usually
`SIGTERM`, to the primary process (PID1) of the
container, and then by default it waits 10 seconds for the container to
stop. The stop signal tells the primary process within the container to
exit gracefully. If the container doesn't stop within 10 seconds, Podman
sends the `SIGKILL` signal
to the process, forcing the container to stop. The 10-second wait gives
the processes in the container time to clean up and commit changes.

The default stop signal can be changed for a
container using the `podman` `run`
`--stop-signal` option.
Sometimes the primary or init process of a container ignores
`SIGTERM` (e.g., containers that use systemd as the
primary process inside a container). systemd ignores
`SIGTERM` and specifies that it shuts down using the
`SIGRTMIN+3` `(signal`
`#37)` signal. The stop signal can be embedded in
container images, as I describe in section 2.3.

Some containers ignore the
`SIGTERM` stop signal, which
means you have to wait 10 seconds for the container to exit. If you know
the container ignores the default stop signal, and you don't care about
the container cleaning up, you can just add the `-t`
`0` option to
`podman` `stop` to send the
`SIGKILL` signal right away:

``` programlisting
$ podman stop -t 0 myapp1
myapp1
```

Podman has a similar command,
`podman`
`kill`, which sends the
specified kill signal. The `podman`
`kill` command can be useful
when you want to send signals into the container without actually
stopping the container.

[]{#02.htm#id_26vkzzhe5enr}Some notable Podman
stop options include the following:

-   `--timeout`
    `(-t`
    .calibre17}`)`---This sets the timeout;
    `-t` `0` sends the
    `SIGKILL` without waiting for the container to
    stop.

-   `--latest`
    `(-l`
    .calibre17}`)`---This is a useful option to allow
    you to stop the last created container rather than having to use the
    container name or container ID. Most Podman commands that require
    you to specify a container name or ID also accept the
    `--latest` option
    .calibre17}. This is only available on Linux machines.

-   
    .calibre17}`--all`
    .calibre17}---This tells Podman to stop all running containers.
    Similarly to `--latest`, Podman commands that
    require a container name or container ID parameter also take the
    `--all` option
    .calibre17}.

[]{#02.htm#id_lgnto058jplf}Use the
`man` `podman-stop`
command for information about all options.

Eventually, your system will have lots of
stopped containers, and sometimes you will need to restart them (e.g.,
if the system was rebooted). Another common use case is to first create
a container and later start it. The next section explains how to start a
container.

### 2.1.4 Starting containers {#02.htm#heading_id_7 .fm-head1}

The
container
you created has now been stopped. Next, you may want to start it back up
again using the command in the following listing.

Listing 2.3 Example of starting a container

``` programlisting
$ podman start myapp
myapp                  ❶
```

[❶]{.fm-combinumeral} The start command prints
the names of the containers that were started.

The `podman`
`start` command starts one
or more containers. This command will output the container ID,
indicating that your container is up and running. You can now reconnect
to it with a web browser. One common use case for
`podman` `start` is starting a
container after a reboot to start all of the containers that were
stopped during shutdown.

Some favorite Podman start options include
these:

-   
    .calibre17}`--all`
    .calibre17}---This  .calibre17}starts all
    of the stopped containers in container storage.

-   
    .calibre17}`--attach`
    .calibre17}---This attaches your terminal to the output of the
    container.

-   
    .calibre17}`--interactive`
    `(-i`
    .calibre17}`)`---This attaches the terminal input
    to the container.

Use the `man`
`podman-start` command for
information about all options.

After you've been using Podman for a while and
have pulled down and run many different container images, you might want
to figure out which containers are running or which containers you have
in local storage. You will need to be able to list
these
containers.

### 2.1.5 Listing containers {#02.htm#heading_id_8 .fm-head1}

You
can
list the running containers and all of the containers that were
previously created. Use the `podman`
`ps` command to list
containers:

``` programlisting
$ podman ps
CONTAINER ID IMAGE                      COMMAND        CREATED \ 
➥   STATUS         PORTS          NAMES
b1255e94d084 registry.access.redhat.com/ubi8/httpd-24:latest /usr/bin/run-\ 
➥ http... 6 minutes ago Up 4 minutes ago 0.0.0.0:8080->8080/tcp myapp
```

Notice the `podman`
`ps` command by default
lists the running containers. Use the `--all`
option to see all of the containers:

``` programlisting
$ podman ps --all
CONTAINER ID IMAGE                       COMMAND        CREATED \ 
➥   STATUS          PORTS          NAMES
b1255e94d084 registry.access.redhat.com/ubi8/httpd-24:latest /usr/bin/run-\
➥ http... 9 minutes ago Up 8 minutes ago     0.0.0.0:8080->8080/tcp myapp
3efee4d39965 registry.access.redhat.com/ubi8/httpd-24:latest /usr/bin/run-\
➥ http... 7 minutes ago Exited (0) 3 minutes ago 0.0.0.0:8081->8080/tcp myapp1
```

[]{#02.htm#id_8i6xl26vaon5}Some notable
`podman` `ps` options include the
following:

-   
    .calibre17}`--all`
    .calibre17}---This tells Podman to list all containers rather than
    just running containers.

-   
    .calibre17}`--quiet`
    .calibre17}---This tells Podman to only print the container IDs.

-   
    .calibre17}`--size`
    .calibre17}---This tells Podman to return the amount of disk space
    currently used for each container other than the images they are
    based on.

Use the `man`
`podman-ps` command for
information about all options. Now that you know all of the containers
you have on the system, you might want to inspect their
internals.

### 2.1.6 Inspecting containers {#02.htm#heading_id_9 .fm-head1}

To
fully
understand a container, sometimes you want to know which image a
container was based on, which environment variables a container gets by
default, or what the security settings used for a container are. The
`podman` `ps`
command gives us some data about the
containers, but if you want to really examine information about the
container, you can use the `podman`
`inspect` command.

The `podman`
`inspect` command can also
be used to inspect images, networks, volumes, and pods. The
`podman` `container`
`inspect` command is also
available and specific to containers. But most users just type the
shorter `podman` `inspect`
command:

``` programlisting
$ podman inspect myapp
[
  {
      "Id": "240271ae90480d3836b1477e5c0b49fbd3883846ca474e3f6effdfb271f4ff54",
      "Created": "2021-09-27T05:27:47.163828842-04:00",
      "Path": "container-entrypoint",
      "Args": [
          "/usr/bin/run-httpd"
      ],
...
]
```

As you can see, the `podman`
`inspect` command outputs a
large JSON file---307 lines on my machine. All of this information is
eventually handed down the OCI runtime to launch the container. When
using the `inspect` command, it is often better to
pipe its output to `less` or `grep`
to find particular fields you are interested in. Alternatively, you can
use the format option. If you want to examine the command executed when
you start the container, execute the following.

Listing 2.4 Inspecting a specified command to
execute the container

``` programlisting
$ podman inspect --format '{{ .Config.Cmd }}' myapp     ❶
[/usr/bin/run-httpd]
```

[❶]{.fm-combinumeral} inspect is displaying
data from the OCI image specification.

Or if you want to see the stop signal, execute
the following.

Listing 2.5 Inspecting the stop signal to be
used when stopping the container

``` programlisting
$ podman inspect --format '{{ .Config.StopSignal }}' myapp
15       ❶
```

[❶]{.fm-combinumeral} The default stop signal
for all containers is 15 (SIGTERM).

[]{#02.htm#id_c6rw1dirvvel}Some notable
`podman` `inspect`
options include the following:

-   `--latest`
    `(-l`
    .calibre17}`)`---This is handy in that it allows
    you to quickly inspect the last created container rather than
    specifying the container name or container ID.

-   
    .calibre17}`--format`
    .calibre17}---This is useful, as shown previously, to extract
    particular fields out of the JSON.

-   
    .calibre17}`--size`
    .calibre17}---This adds the amount of disk space the container is
    using. Gathering this information takes a long time, so it is not
    done by default.

Use the `man`
`podman-inspect` command for
information about all options. After you inspect a container, you might
realize you no longer need that container taking up storage, so you need
to remove
it.

### 2.1.7 Removing containers {#02.htm#heading_id_10 .fm-head1}

If
you
are done using a container, you may want to remove the container to free
up disk space or reuse the container name. Remember when you started a
second container called `myapp1`? You no longer need
it, so you can remove it. Make sure to stop the container (section
2.1.3) before removing it. Then use the `podman`
`rm` command to remove
container:

``` programlisting
$ podman rm myapp1
3efee4d3996532769356ffea23e1f50710019d4efc704d39026c5bffd6aa18be
```

[]{#02.htm#id_t4e49sbt7wo71111}Some notable
`podman` `rm` options include the
following:

-   
    .calibre17}`--all`
    .calibre17}---This option is useful if you want to remove all your
    containers.

-   
    .calibre17}`--force`
    .calibre17}---This option tells Podman to stop all the running
    containers when removing.

Use the `man`
`podman-rm` command for
information about all options. Now that you understand a few commands,
it is time to start modifying the running
container.

### 2.1.8 exec-ing into a container {#02.htm#heading_id_11 .fm-head1}

Often,
when
a container is running, you might want to start another process within
the container to debug or examine what is going on. In some cases, you
may want to modify some of the content the container is using.

Imagine you want to go into your container and
modify the web page it is showing. You can `exec` into
the container using the `podman`
`exec` command. Use the
`--interactive` (`-i`)
option to allow you to execute commands within
the container. You need to specify the name of the container
`myapp` and execute the Bash
script while in the container. If you stopped the
`myapp` container, you need
to restart it, since `podman` `exec`
only works on running containers.

In the following example, you will
`exec` a `bash` process into the
container to create the /var/www/html/index.html file. You will write
HTML content that causes the containerized website to display
`Hello` `World`:

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

`exec`-ing back into the
container a second time, you can see that the file was successfully
modified. This shows that modifications to a container via
`exec` are permanent to the container and will remain
even if you stopped and restarted the container. A key difference
between `podman` `run` and
`podman` `exec` is that
`run` creates a new container off of an image with
processes running inside, while `exec` starts
processes inside of existing containers:

``` programlisting
$ podman exec myapp cat /var/www/html/index.html
<html>
 <head>
 </head>
 <body>
  <h1>Hello World</h1>
 </body>
</html>
```

Now let's connect a web browser to the
container to see if the content has changed (see figure 2.2):

``` programlisting
$ web-browser localhost:8080
```

::: figure
![](images//02-02.png)

Figure 2.2 Web browser window connecting to the
ubi8/httpd-24 container running in Podman with updated Hello World HTML
:::

[]{#02.htm#id_gb8ykx98avmq}Some notable
`podman` `exec`
options include the following:

-   
    .calibre17}`--tty`
    .calibre17}---This connects a `-tty` to the
    `exec` session.

-   
    .calibre17}`--interactive`
    .calibre17}---The `-i`
    option .calibre17} tells Podman to run in
    interactive mode, meaning you can interact with an
    `exec`-ed program, like a shell.

Use the `man`
`podman-exec` command for
information about all options.

Now that you have created an application, you
might want to share it with others. First, you need to commit the
container to an
image.

### 2.1.9 Creating an image from a container {#02.htm#heading_id_12 .fm-head1}

Developers
often
run containers from a base image to create a new container environment.
Once they are done, they pack this environment into a container image to
be able to share it with other users. Those users can then use Podman to
launch the containerized application. You can do this with Podman by
committing the container to an OCI image.

First, stop or pause the container to make sure
nothing gets modified while you are committing it:

``` programlisting
$ podman stop myapp
```

Now you can execute the
`podman` `commit`
command to take your application container,
`myapp`, and commit it,
creating a new image named `myimage`:

``` programlisting
$ podman commit myapp myimage
Getting image source signatures
Copying blob e39c3abf0df9 skipped: already exists
Copying blob 8f26704f753c skipped: already exists
Copying blob 83310c7c677c skipped: already exists
Copying blob 654b3bf1361e skipped: already exists
Copying blob 9e816183404c done              Copying config e38084bb8a done
Writing manifest to image destination
Storing signatures
e38084bb8a76104a7cac22b919f67646119aff235bb1cfcba5478cc1fbf1c9eb
```

Now you can continue running the existing
`myapp` container by calling
`podman` `start`, or you can create
a new container based on `myimage`:

``` programlisting
$ podman run -d --name myapp1 -p 8080:8080 myimage
0052cb32c8e63b845ac5dfd5ba176b8204535c2c6cafa3277453424de601263f
```

[Note]{.fm-callout-head} Using the
`podman`{.fm-code-in-text1} `commit`{.fm-code-in-text1}
command to create an image is not a common
method. The entire process of building container images can be scripted
and automated using `podman`{.fm-code-in-text1}
`build`{.fm-code-in-text1}. See section 2.3 for more information on this
process.

[]{#02.htm#id_rll08y12fcy}Some notable
`podman` `commit`
options include the following:

-   
    .calibre17}`--pause`
    .calibre17}---This pauses a running container during the commit.
    Notice I stopped the container before doing the commit, while I
    could have simply paused it. The `podman`
    `pause` and `podman`
    `unpause` commands
    .calibre17} allow you to pause and unpause containers directly.

-   
    .calibre17}`--change`
    .calibre17}---This option allows you to commit instructions on using
    the image. The instructions are
    `CMD` .calibre17},
    `ENTRYPOINT` .calibre17},
    `ENV` .calibre17},
    `EXPOSE` .calibre17},
    `LABEL` .calibre17},
    `ONBUILD` .calibre17},
    `STOPSIGNAL` .calibre17},
    `USER` .calibre17},
    `VOLUME` .calibre17}, and
    `WORKDIR`. These instructions match up with the
    directives in the Containerfile or Dockerfile.

Use the `man`
`podman-commit` command for
information about all options. Table 2.1 lists all the Podman container
commands.

Now that you have committed your container to
an image, it is time to show how Podman can work
with
images.

[Note]{.fm-callout-head} You have examined a
few of the Podman container commands, but there are many more. Use the
`podman-container(1)`{.fm-code-in-text1} man
pages to explore all of them as well as a full
description of commands specified in this section.

Table 2.1 Podman container commands

+-----------------+-----------------+-----------------------------------+
| []              | []{#02.htm#pg   | []{                               |
| {#02.htm#pgfId- | fId-1044671}Man | #02.htm#pgfId-1044673}Description |
| 1044669}Command | page            |                                   |
+-----------------+-----------------+-----------------------------------+
| []{             | []{#02.htm#p    | Attach   |
| #02.htm#pgfId-1 | gfId-1044677}`p | to a running container.           |
| 044675}`attach` | odman-container |                                   |
| {.fm-code-in-te | -attach(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044866} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#02.         | [               | []                                |
| htm#pgfId-10446 | ]{#02.htm#pgfId | {#02.htm#pgfId-1044685}Checkpoint |
| 81}`checkpoint` | -1044683}`podma | a container.                      |
| {.fm-code-in-te | n-container-che |                                   |
| xt1}[]{#02.htm# | ckpoint(1)`{.fm |                                   |
| marker-1044867} | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#            | []{#02.htm#pg   | Clean up |
| 02.htm#pgfId-10 | fId-1044689}`po | network and mount points of a     |
| 44687}`cleanup` | dman-container- | container.                        |
| {.fm-code-in-te | cleanup(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044868} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{             | []{#02.htm#p    | Commit a |
| #02.htm#pgfId-1 | gfId-1044695}`p | container into an image.          |
| 044693}`commit` | odman-container |                                   |
| {.fm-code-in-te | -commit(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044869} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#02.htm#pgf  | []{#02.h        | Copy     |
| Id-1044699}`cp` | tm#pgfId-104470 | files or folders into and out of  |
| {.fm-code-in-te | 1}`podman-conta | containers.                       |
| xt1}[]{#02.htm# | iner-cp(1)`{.fm |                                   |
| marker-1044870} | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{             | []{#02.htm#p    | Create a |
| #02.htm#pgfId-1 | gfId-1044707}`p | new container.                    |
| 044705}`create` | odman-container |                                   |
| {.fm-code-in-te | -create(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044871} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| [               | []{#02.htm      | Inspect  |
| ]{#02.htm#pgfId | #pgfId-1044713} | changes in a container's          |
| -1044711}`diff` | `podman-contain | filesystem.                       |
| {.fm-code-in-te | er-diff(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044872} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| [               | []{#02.htm      | Run a    |
| ]{#02.htm#pgfId | #pgfId-1044719} | process in a container.           |
| -1044717}`exec` | `podman-contain |                                   |
| {.fm-code-in-te | er-exec(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044873} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#02.         | []{#02.htm#p    | Check if |
| htm#pgfId-10447 | gfId-1044725}`p | a container                       |
| 23}`exists`{.fm | odman-container | exists. |
| -code-in-text1} | -exists(1)`{.fm |                                   |
|                 | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{             | []{#02.htm#p    | Export a |
| #02.htm#pgfId-1 | gfId-1044731}`p | container\'s filesystem as a TAR  |
| 044729}`export` | odman-container | archive.                          |
| {.fm-code-in-te | -export(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044875} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| [               | []{#02.htm      | Init a   |
| ]{#02.htm#pgfId | #pgfId-1044737} | container.                        |
| -1044735}`init` | `podman-contain |                                   |
| {.fm-code-in-te | er-init(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044876} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#            | []{#02.htm#pg   | Display  |
| 02.htm#pgfId-10 | fId-1044743}`po | detailed information on a         |
| 44741}`inspect` | dman-container- | container.                        |
| {.fm-code-in-te | inspect(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044877} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| [               | []{#02.htm      | Send a   |
| ]{#02.htm#pgfId | #pgfId-1044749} | signal to the primary process in  |
| -1044747}`kill` | `podman-contain | the container.                    |
| {.fm-code-in-te | er-kill(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044878} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#0           | []{#02.htm      | List all |
| 2.htm#pgfId-104 | #pgfId-1044755} | of the containers.                |
| 4753}`List`{.fm | `podman-contain |                                   |
| -code-in-text1} | er-list(1)`{.fm |                                   |
| `(ps`{.fm-co    | -code-in-text1} |                                   |
| de-in-text1}[]{ |                 |                                   |
| #02.htm#marker- |                 |                                   |
| 1044879}`)`{.fm |                 |                                   |
| -code-in-text1} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#0           | []{#02.htm      | Fetch    |
| 2.htm#pgfId-104 | #pgfId-1044761} | logs    |
| 4759}`logs`{.fm | `podman-contain | for a container.                  |
| -code-in-text1} | er-logs(1)`{.fm |                                   |
|                 | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []              | []{#02.htm#     | Mount a  |
| {#02.htm#pgfId- | pgfId-1044767}` | container\'s root filesystem.     |
| 1044765}`mount` | podman-containe |                                   |
| {.fm-code-in-te | r-mount(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044881} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []              | []{#02.htm#     | Pause    |
| {#02.htm#pgfId- | pgfId-1044773}` | container.                        |
| 1044771}`pause` | podman-containe |                                   |
| {.fm-code-in-te | r-pause(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044882} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#0           | []{#02.htm      | List     |
| 2.htm#pgfId-104 | #pgfId-1044779} | port mappings for a container.    |
| 4777}`port`{.fm | `podman-contain |                                   |
| -code-in-text1} | er-port(1)`{.fm |                                   |
|                 | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []              | []{#02.htm#     | Remove   |
| {#02.htm#pgfId- | pgfId-1044785}` | all non-running containers.       |
| 1044783}`prune` | podman-containe |                                   |
| {.fm-code-in-te | r-prune(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044883} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{             | []{#02.htm#p    | Rename   |
| #02.htm#pgfId-1 | gfId-1044791}`p | an existing container.            |
| 044789}`rename` | odman-container |                                   |
| {.fm-code-in-te | -rename(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044884} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#            | []{#02.htm#pg   | Restart  |
| 02.htm#pgfId-10 | fId-1044797}`po | a container.                      |
| 44795}`restart` | dman-container- |                                   |
| {.fm-code-in-te | restart(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044885} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#            | []{#02.htm#pg   | Restore  |
| 02.htm#pgfId-10 | fId-1044803}`po | a checkpointed container.         |
| 44801}`restore` | dman-container- |                                   |
| {.fm-code-in-te | restore(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044886} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#02.htm#pgf  | []{#02.h        | Remove a |
| Id-1044807}`rm` | tm#pgfId-104480 | container.                        |
| {.fm-code-in-te | 9}`podman-conta |                                   |
| xt1}[]{#02.htm# | iner-rm(1)`{.fm |                                   |
| marker-1044887} | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#            | []{#02.ht       | Run a    |
| 02.htm#pgfId-10 | m#pgfId-1044815 | command in a new container.       |
| 44813}`run`{.fm | }`podman-contai |                                   |
| -code-in-text1} | ner-run(1)`{.fm |                                   |
|                 | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#0           | []{#02.htm#pgf  | Execute  |
| 2.htm#pgfId-104 | Id-1044821}`pod | the command described by an image |
| 4819}`runlabel` | man-container-r | label.                            |
| {.fm-code-in-te | unlabel(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044888} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []              | []{#02.htm#     | Start a  |
| {#02.htm#pgfId- | pgfId-1044827}` | container.                        |
| 1044825}`start` | podman-containe |                                   |
| {.fm-code-in-te | r-start(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044889} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []              | []{#02.htm#     | Display  |
| {#02.htm#pgfId- | pgfId-1044833}` | statistics for a container.       |
| 1044831}`stats` | podman-containe |                                   |
| {.fm-code-in-te | r-stats(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044890} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| [               | []{#02.htm      | Stop a   |
| ]{#02.htm#pgfId | #pgfId-1044839} | container.                        |
| -1044837}`stop` | `podman-contain |                                   |
| {.fm-code-in-te | er-stop(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044891} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#02.htm#pgfI | []{#02.ht       | Display  |
| d-1044843}`top` | m#pgfId-1044845 | running process in a container.   |
| {.fm-code-in-te | }`podman-contai |                                   |
| xt1}[]{#02.htm# | ner-top(1)`{.fm |                                   |
| marker-1044892} | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#            | []{#02.htm#pg   | Unmount  |
| 02.htm#pgfId-10 | fId-1044851}`po | a container\'s root filesystem.   |
| 44849}`unmount` | dman-container- |                                   |
| {.fm-code-in-te | unmount(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044893} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#            | []{#02.htm#pg   | Unpause  |
| 02.htm#pgfId-10 | fId-1044857}`po | all the containers in a pod.      |
| 44855}`unpause` | dman-container- |                                   |
| {.fm-code-in-te | unpause(1)`{.fm |                                   |
| xt1}[]{#02.htm# | -code-in-text1} |                                   |
| marker-1044894} |                 |                                   |
+-----------------+-----------------+-----------------------------------+
| []{#0           | []{#02.htm      | Wait for |
| 2.htm#pgfId-104 | #pgfId-1044863} | a container to exit.              |
| 4861}`wait`{.fm | `podman-contain |                                   |
| -code-in-text1} | er-wait(1)`{.fm |                                   |
|                 | -code-in-text1} |                                   |
+-----------------+-----------------+-----------------------------------+

## 2.2 Working with container images {#02.htm#heading_id_13 .fm-head}

In
the previous
section, you tried basic operations with containers, including
inspecting and committing to a container image. In this section, you
will try working with container images, learn how they differ from
containers, and learn how to share them through container registries.

### 2.2.1 Differences between a container and an image {#02.htm#heading_id_14 .fm-head1}

One
of
the problems with computer programming is that the same names are
constantly used for different purposes. In the container world, there is
no more overused term than *container*. Often *container* refers to the
running processes launched by Podman. But *container* can also refer to
container data as the non-running objects sitting in container storage.
As you saw in the previous section,
`podman` `ps`
`--all` shows running and non-running containers.

Another example is the term
*namespace*, which is used in many different
ways. I often get confused when people talk about namespaces within
Kubernetes. Some people hear the term and think of *virtual clusters*,
but when I hear it I think of Linux namespaces used with Pods and
Containers. Similarly, *image* can refer to a VM image, a container
image, an OCI image, or a Docker image stored at a container registry.

I think of containers as executing processes
within an environment or something that is being prepared to run. In
contrast, images are *committed* *containers*, which are prepared to be
shared with others. Other users or systems can use these images to
create new containers.

Container images are just committed containers.
The OCI defines the format of an image. Podman uses the container/image
library
([https://github.com/containers/image](https://github.com/containers/image){.url})
for all of its interaction with images. Container images can be stored
in different types of storage or transports, as *container/image* refers
to them. These transports can be container registries, Docker archives,
OCI archives, `docker-daemon`, as well as
containers/storage. See section 2.2.4 for more information on
transports.

In the context of Podman, I usually refer to
images as the content stored locally in a container storage or in
container registries like docker.io and quay.io. Podman uses the GitHub
container/storage library
([https://github.com/containers/storage](https://github.com/containers/storage){.url})
for handling locally stored images. Let's take a closer look at it.

The container/storage library provides the
concept of a storage container. Basically, storage containers are
intermediate storage content that hasn't been committed yet. Think of
them as files on disk and some JSON describing the content. Podman has
its own datastore of data related to a Podman container, and Podman
needs to deal with multiple users of its containers at the same time. It
relies on filesystem locking provided by containers/storage to make sure
hundreds of Podman executables can reliably share the same datastore.

When you commit a container to storage, Podman
copies the container storage to the image storage. Images are stored in
a series of layers, with every commit creating a new layer.

I like to think of an image like a wedding cake
(figure 2.3). In our previous example, you used the ubi8/httpd-24 image,
which is two layers: the base layer is ubi8, and then the image provided
added the `httpd` package and a few others to create
the ubi8/httpd-24. Now when you commit your container in the previous
section, Podman adds another layer on top of the ubi8/httpd-24 image
called `myimage`.

::: figure
![](images//02-03.png)

Figure 2.3 A wedding cake display showing the
images making up our Hello World application.
:::

One handy Podman command for showing the layers
of an image is the `podman` `image`
`tree` command:

``` programlisting
$ podman image tree myimage
Image ID: 2c7e43d88038
Tags:   [localhost/myimage:latest]
Size:   461.7MB
Image Layers
├── ID: e39c3abf0df9 Size: 233.6MB
├── ID: 42c81bd2b468 Size: 20.48kB Top Layer of: [registry.access.redhat.com/ubi8:latest]
├── ID: 51a7beaa0b88 Size: 57.43MB
├── ID: 519e681b5702 Size: 170.6MB Top Layer of: [registry.access.redhat.com/ubi8/httpd-24:latest]
└── ID: bc3dcdefdac3 Size: 69.63kB Top Layer of: [localhost/myimage:latest localhost/myapp:latest]
```

You can see that the image
`myimage` consists of five
layers.

Another useful Podman
command, `podman`
`image`
`diff`, allows you to see
the actual files and directories that have been changed (C), added (A),
or deleted (D) compared to another image or the lower layer:

``` programlisting
$ podman image diff myimage ubi8/httpd-24 
C /etc/group
C /etc/httpd/conf
C /etc/httpd/conf/httpd.conf
C /etc/httpd/conf.d
C /etc/httpd/conf.d/ssl.conf
C /etc/httpd/tls
C /etc
C /etc/httpd
A /etc/httpd/tls/localhost.crt
A /etc/httpd/tls/localhost.key
...
```

Images are just TAR diffs of software applied
on lower-level images, and container content is an uncommitted layer of
software. Once a container is committed, you can create other containers
on top of your image. You can also share the image with others, so they
can create other containers on your image. Now let's look at all the
images in your container
storage.

### 2.2.2 Listing images {#02.htm#heading_id_15 .fm-head1}

In
the
container section, you were working with images and used command
`podman`
`images` to list the images
in local storage:

``` programlisting
$ podman images
REPOSITORY                   TAG        IMAGE ID       CREATED     SIZE
localhost/myimage         latest    2c7e43d88038  46 hours ago   462 MB
registry.access.redhat
➥.com/ubi8/httpd-24      latest    8594be0a0b57   5 weeks ago   462 MB
registry.access.redhat
➥.com/ubi8               latest    ad42391b9b46   5 weeks ago   234 MB
```

Let's look at the different fields in the
default output. Table 2.2 describes the different fields and data
available with the `podman` `images`
command. You will use the
`podman` `images`
command throughout this section.

Table 2.2 Default fields listed by the
`podman` `images` command

+-----------------+-----------------------------------------------------+
| []              | Description                |
| {#02.htm#pgfId- |                                                     |
| 1044972}Heading |                                                     |
+-----------------+-----------------------------------------------------+
| []{#02.         | Complete name of the       |
| htm#pgfId-10449 | image.                                              |
| 76}`Repository` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#02.htm# |                                                     |
| marker-1044995} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#02.htm#pgfI | Version (tag) of the       |
| d-1044980}`TAG` | image. Image tagging is covered in section 2.2.6.   |
| {.fm-code-in-te |                                                     |
| xt1}[]{#02.htm# |                                                     |
| marker-1044996} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#0           | Unique identifier of the   |
| 2.htm#pgfId-104 | image. It is generated by Podman as a SHA256 hash   |
| 4984}`IMAGE ID` | of the image\'s JSON configuration object.          |
| {.fm-code-in-te |                                                     |
| xt1}[]{#02.htm# |                                                     |
| marker-1044997} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#            | Elapsed time since the     |
| 02.htm#pgfId-10 | image was created. Images are sorted by this field  |
| 44988}`CREATED` | by default.                                         |
| {.fm-code-in-te |                                                     |
| xt1}[]{#02.htm# |                                                     |
| marker-1044998} |                                                     |
+-----------------+-----------------------------------------------------+
| [               | The amount of storage      |
| ]{#02.htm#pgfId | being used by the image.                            |
| -1044992}`SIZE` |                                                     |
| {.fm-code-in-te |                                                     |
| xt1}[]{#02.htm# |                                                     |
| marker-1044999} |                                                     |
+-----------------+-----------------------------------------------------+

[Note]{.fm-callout-head} Over time, the amount
of storage used by all the images you pull grows. It is relatively
common for users to run out of disk space, so you should monitor the
size of images and containers, removing them when you are no longer
using them. Use the `man`{.fm-code-in-text1}
`podman-system-prune`{.fm-code-in-text1} command for more information on
cleaning up.

[]{#02.htm#id_u8l9oyck7hrk}One notable
`podman` `image` option
is the following:

-   
    .calibre17}`--all`
    .calibre17}---This option is useful for listing all images. By
    default, `podman-images` lists only the images
    currently in use. When an image is replaced by a newer image with
    the same tag, the previous image is tagged as
    `<none><none>`; These images are called dangling
    images. I cover dangling images in section 2.3.1.

Use the `man`
`podman-images` command for
information about all options. Similarly to containers, you will likely
want to examine the configuration information associated with an image
by inspecting
it.

### 2.2.3 Inspecting images {#02.htm#heading_id_16 .fm-head1}

In
the
previous sections, I mentioned a couple of commands to examine images. I
used the `podman` `image`
`diff` to examine files and directories created or
deleted between images. I also showed you a way to see the image
hierarchy or wedding cake layers of images using the
`podman` `image`
`tree` command.

Sometimes you may want to examine the
configuration of an image; use the `podman`
`image` `inspect`
command for this.
The `podman` `inspect`
command can also be used to inspect images,
but the names can conflict with containers, so I prefer to use the
specific image command:

``` programlisting
$ podman image inspect myimage
[
  {
      "Id": "3b8fcf9081b4c4e6c16d763b8d02684df0737f3557a1e03ebfe4cc7cd6562135",
      "Digest":
"sha256:ff49aa6253ae47569d5aadbd73d70e7d0431bcf3a2f57b1b56feecdb531029a3",
      "RepoTags": [
          "localhost/myimage:latest"
      ],
      "RepoDigests": [       "localhost/myimage@sha256:ff49aa6253ae47569d5aadbd73d70e7d0431bcf3a2f57b1b\
➥ 56feecdb531029a3"
      ],
...
]
```

As you can see, this command outputs a large
JSON array---153 lines in the previous example---that includes the data
used for the OCI Image Format specification. When you create a container
from an image, this information is used as one of the inputs to create
the container.

When using the `inspect`
command, it is often better to pipe its output to
`less` or `grep` to find particular
fields you are interested in. Alternatively, you can use the
`--format` option.

If you want to to examine the default command
to be executed from this image, execute the following:

``` programlisting
$ podman image inspect --format '{{ .Config.Cmd }}' myimage
[/usr/bin/run-httpd]
```

Or if you want to see the stop signal, execute

``` programlisting
$ podman image inspect --format '{{ .Config.StopSignal }}' myimage
```

As you can see, nothing is output, meaning the
developer of the application did not specify a
`STOPSIGNAL`. When you build a container off of this
image, the `STOPSIGNAL` is the default,
`15`, unless you override it via the command line.

[]{#02.htm#id_b2ilzpve6pxb}One notable
`podman` `image`
`inspect` option is the
following:

-   
    .calibre17}`--format`
    .calibre17}---This is useful as you see above to extract particular
    fields out of the json.

Use the `man`
`podman-image-inspect`
command for information about the command.

Once you are happy with a container and commit
it to an image, the next step is sharing it with others or perhaps
running it on another system. You need to push the image out to other
types of container storage, usually a container
registry.

### 2.2.4 Pushing images {#02.htm#heading_id_17 .fm-head1}

In
Podman,
you use the `podman` `push`
command to copy an image and all of its layers
out of container storage and push it to other forms of container image
storage, like a container registry. Podman supports a few different
types of container storage, which it calls transports.

Container transports

Podman
uses the
containers/image library
([https://github.com/containers/image](https://github.com/containers/image){.url})
for pulling and pushing images. I describe the containers/image project
as a library for copying images between different types of container
storage. One storage, as you have seen, is containers/storage.

When pushing an image, the
`[destination]` is specified using
`transport:ImageName`
format. If no transport is specified, the
`docker` (container registry) transport is used by
default.

One of the novel things that Docker did, as I
explained earlier, was invent the container registry
concept---basically, a web server that contains container images. The
docker.io, quay.io, and Artifactory web servers are all examples of
container registries. The Docker engineering team defined a protocol for
pulling and pushing these images from the container registries, which I
refer to as the container registry or `docker`
transport.

When I want to run a container of an image, I
can fully specify the image name, including the transport like the
following command:

``` programlisting
$ podman run docker://registry.access.redhat.com/ubi8/httpd-24:latest echo hello
hello
```

For Podman, `docker://`
transport is the default; it can be skipped for convenience:

``` programlisting
$ podman run registry.access.redhat.com/ubi8/httpd-24:latest echo hello
hello
```

The `myimage`
image you created in the previous section was
created locally, which means it doesn't have a registry associated with
it. By default, locally created images have the localhost registry
associated with them. You can see the images in the containers/storage
using the `podman` `images` command:

``` programlisting
$ podman images
REPOSITORY                        TAG        IMAGE ID       CREATED    SIZE
localhost/myimage              latest    2c7e43d88038  46 hours ago  462 MB
registry.access.redhat
➥.com/ubi8/httpd-24           latest    8594be0a0b57   5 weeks ago  462 MB
registry.access.redhat
➥.com/ubi8                    latest    ad42391b9b46   5 weeks ago  234 MB
```

If the image has a remote
registry associated with it (e.g.,
registry.access.redhat.com/ ubi8), it can be pushed without specifying
the `[destination]` field. On the contrary, since
localhost/myimage does not have a registry associated with it, remote
registry needs to be specified (e.g., quay.io/rhatdan):

``` programlisting
$ podman push myimage quay.io/rhatdan/myimage
Getting image source signatures
Copying blob 164d51196137 done 
Copying blob 8f26704f753c done 
Copying blob 83310c7c677c done 
Copying blob 654b3bf1361e [==================>-------------------] 82.0MiB / 162.7MiB
Copying blob e39c3abf0df9 [================>---------------------] 100.0MiB / 222.8MiB
```

[Note]{.fm-callout-head} Before executing the
`podman`{.fm-code-in-text1} `push`{.fm-code-in-text1}
command, I logged into the quay.io/ rhatdan
account using p`odman`{.fm-code-in-text1} `login`{.fm-code-in-text1},
which is covered in the next section.

After the `push` command is
finished, the image becomes available for pull for other users, given
they have access to this container registry. Table 2.3 describes the
supported transports for different types of container's storage.

Table 2.3 Podman-supported transports

+-----------------+-----------------------------------------------------+
| []{#            | Description                |
| 02.htm#pgfId-10 |                                                     |
| 45280}Transport |                                                     |
+-----------------+-----------------------------------------------------+
| []{#            | Default transport. This    |
| 02.htm#pgfId-10 | references a container image stored in a remote     |
| 45284}Container | container image registry. Container registry is a   |
| registry        | place for storing and sharing container images      |
| (Doc            | (e.g., docker.io or quay.io).                       |
| ker[]{#02.htm#m |                                                     |
| arker-1045311}) |                                                     |
+-----------------+-----------------------------------------------------+
| []{#02.htm#pgfI | References a container     |
| d-1045288}`oci` | image, compliant with the Open Container Image      |
| {.fm-code-in-te | Layout Specification. The manifest and layer        |
| xt1}[]{#02.htm# | tarballs as individual files are located in the     |
| marker-1045312} | local directory.                                    |
+-----------------+-----------------------------------------------------+
| []{#02.htm#pgfI | References a container     |
| d-1045292}`dir` | image, compliant with the Docker image layout. It   |
| {.fm-code-in-te | is very similar to the `oci`{.fm-code-in-text1}     |
| xt1}[]{#02.htm# | transport but stores the files using the legacy     |
| marker-1045313} | Docker format. It is a nonstandardized format,      |
|                 | primarily useful for debugging or noninvasive       |
|                 | container inspection.                               |
+-----------------+-----------------------------------------------------+
| []{#02.htm#     | References a container     |
| pgfId-1045296}` | image in Docker image layout, which is packed into  |
| docker-archive` | a TAR archive.                                      |
| {.fm-code-in-te |                                                     |
| xt1}[]{#02.htm# |                                                     |
| marker-1045314} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#02.htm#p    | References an image        |
| gfId-1045300}`o | compliant with the Open Container Image Layout      |
| ci-archive`{.fm | Specification, which is packed into a TAR archive.  |
| -code-in-text1} | It is very similar to the                           |
|                 | `docker-archive`{.fm-code-in-text1} transport, but  |
|                 | it stores an image in OCI format.                   |
+-----------------+-----------------------------------------------------+
| []{#02.htm      | References an image stored |
| #pgfId-1045304} | in the Docker daemon's internal storage. Since the  |
| `docker-daemon` | Docker daemon requires root privileges, Podman has  |
| {.fm-code-in-te | to be run by the root user.                         |
| xt1}[]{#02.htm# |                                                     |
| marker-1045315} |                                                     |
+-----------------+-----------------------------------------------------+
| []{#02.htm      | References an image        |
| #pgfId-1045308} | located in a local container storage. It is not a   |
| `container-stor | transport but more of a mechanism for storing       |
| age`{.fm-code-i | images. It can be used to convert other transports  |
| n-text1}[]{#02. | into `container-storage`{.fm-code-in-text1}. Podman |
| htm#marker-1045 | defaults to using                                   |
| 316}[]{#02.htm# | `container-storage`{.fm-code-in-text1} for local    |
| marker-1045317} | images.                                             |
+-----------------+-----------------------------------------------------+

You want to push your image to a container
registry, but if you try to push it, the container registry rejects your
push, since you have not provided login authorization information. You
need to execute `podman` `login` to
create the
authorization.

### 2.2.5 podman login: Logging into a container registry {#02.htm#heading_id_18 .fm-head1}

In
the
previous section, I pushed the image to my container registry by
executing the following:

``` programlisting
$ podman push myimage quay.io/rhatdan/myimage
```

However, I left out a key step: logging into a
container registry using correct credentials. This is a necessary step
for pushing a container image. It is also required for pulling a
container image from a private registry.

To follow along in this section, you need to
set up an account at a container registry; there are several container
registries available to choose from. The
[https://quay.io](https://quay.io){.url} and
[https://docker.io](https://docker.io){.url} registries both provide
free accounts and storage. Your company might have a private registry,
where you can also get an account.

For the examples, I will continue to use my
rhatdan account at quay.io. Log in to get your credentials:

``` programlisting
$ podman login quay.io
Username: rhatdan
Password: 
Login Succeeded!
```

Notice the Podman command prompts you for your
username and password at the registry. The `podman`
`login` command has options
to pass the username/password information on the command line to avoid
the prompt, allowing you to automate the login process.

To store authentication information for the
user, the `podman` `login`
command creates an auth.json file. By default,
this is stored in the /run/user/\$UID/containers/auth.json file:

``` programlisting
cat /run/user/3267/containers/auth.json 
{
  "auths": {
    "quay.io": {
      "auth": "OBSCURED-BASE64-PASSWORD"
    }
  }
}
```

The auth.json file contains your registry
password in a Base64-encoded string; there is no cryptography involved.
Therefore, the auth.json file needs to be protected. Podman defaults to
storing the file in /run because it is a temporary filesystem and is
destroyed when you log out or the system is rebooted. The
/run/user/\$UID/containers directory is not accessible by other users on
the system.

It is possible to override the location by
specifying the `--auth-file`
option. Alternatively, you can use the
`REGISTRY_AUTH_FILE` environment variable to modify
its location. If both are specified, the `--auth-file`
option is used. All container tools use this
file to access the container registry.

It is possible to run the
`podman` `login`
command multiple times to log in to multiple
registries, storing the login information in the same authorization file
with a different stanza.

[Note]{.fm-callout-head} Podman supports other
mechanisms for storing the password information. These are called
*credential helpers*.

After you are done using the registry, you can
log out by executing `podman`
`logout`. This command deletes the cached credentials
stored in the auth.json file:

``` programlisting
$ podman logout quay.io
Removed login credentials for quay.io
```

[]{#02.htm#id_kl9hru67otu4}Some notable
`podman`
`login` and
`logout` options include the following:

-   `--username`,
    `(-u` .calibre17}---This
    provides the Podman username to use when logging into the registry.

-   
    .calibre17}`--authfile`
    .calibre17}---This tells Podman to store the authorization file in a
    different location. You can also use the
    `REGISTRY_AUTH_FILE` environment
    variable .calibre17} to change the
    location.

-   
    .calibre17}`--all`
    .calibre17}---This allows you to log out of all of the registries.

Use the `man`
`podman-login` and `man`
`podman-logout` commands for
information about all options.

Notice when you pushed the image to a container
registry, you renamed `myimage` to
quay.io/rhatdan/myimage:

``` programlisting
$ podman push myimage quay.io/rhatdan/myimage
```

It'd be nice to just have the local image named
quay.io/rhatdan/myimage, in which case you could have just executed

``` programlisting
$ podman push quay.io/rhatdan/myimage
```

In the next section, you will learn how to add
names
to
images.

### 2.2.6 Tagging images {#02.htm#heading_id_19 .fm-head1}

Earlier
in
this chapter, I pointed out that locally created images are created with
a localhost registry. Images get created with the localhost registry
when you commit a container to an image or if you use
`podman` `build` to build an image.
Podman has a mechanism to add additional names to images; it calls these
names tags, and the command is `podman`
`tag`.

Using the `podman`
`images` command, list the
image(s) in container/storage:

``` programlisting
$ podman images
REPOSITORY                      TAG        IMAGE ID         CREATED     SIZE
localhost/myimage            latest    2c7e43d88038   46 \hours ago   462 MB
registry.access.redhat
➥.com/ubi8/httpd-24         latest    8594be0a0b57     5 weeks ago   462 MB
registry.access.redhat
➥.com/ubi8                  latest    ad42391b9b46     5 weeks ago   234 MB
```

You will want the final image you plan on
shipping to be referred to as quay.io/rhatdan/ myimage. To achieve this,
add that name with the following `podman`
`tag` command:

``` programlisting
$ podman tag myimage quay.io/rhatdan/myimage 
```

Now run
`podman` `images` again to examine
the images. You will see that the name is now quay.io/rhatdan/myimage.
Notice that the localhost/myimage and quay.io/ rhatdan/myimage have the
same image ID of `2c7e43d88038`:

``` programlisting
$ podman images
REPOSITORY                      TAG        IMAGE ID        CREATED      SIZE
localhost/myimage            latest    2c7e43d88038   46 hours ago    462 MB
quay.io/rhatdan/myimage      latest    2c7e43d88038   46 hours ago    462 MB
registry.access.redhat
➥.com/ubi8/httpd-24         latest    8594be0a0b57    5 weeks ago    462 MB
registry.access.redhat
➥.com/ubi8                  latest    ad42391b9b46    5 weeks ago    234 MB
```

Since the images have the same image ID, they
are the same image with multiple names. Now you can interact directly
with quay.io/rhatdan/myimage. First, you need to log back in to quay.io:

``` programlisting
$ podman login --username rhatdan quay.io
Password: 
Login Succeeded!
```

Now push without requiring the destination
name:

``` programlisting
$ podman push quay.io/rhatdan/myimage
Getting image source signatures
...
Storing signatures
```

That was much simpler.

Let's tag the previously used image with a
version, 1.0:

``` programlisting
$ podman tag quay.io/rhatdan/myimage quay.io/rhatdan/myimage:
```

Once again, examine the images; notice that
`myimage` now has three different names/tags. All
three have the same image ID of `2c7e43d88038`:

``` programlisting
$ podman images
REPOSITORY                      TAG        IMAGE ID        CREATED     SIZE
localhost/myimage            latest    2c7e43d88038   46 hours ago   462 MB
quay.io/rhatdan/myimage         1.0    2c7e43d88038   46 hours ago   462 MB
quay.io/rhatdan/myimage      latest    2c7e43d88038   46 hours ago   462 MB
registry.access.redhat
➥.com/ubi8/httpd-24         latest    8594be0a0b57    5 weeks ago   462 MB
registry.access.redhat
➥.com/ubi8                  latest    ad42391b9b46    5 weeks ago   234 MB
```

Now you can push the 1.0 version of the
`myimage` (application) to the registry:

``` programlisting
$ podman push quay.io/rhatdan/myimage:1.0
Getting image source signatures
Copying blob 8f26704f753c skipped: already exists 
Copying blob e39c3abf0df9 skipped: already exists 
Copying blob 654b3bf1361e skipped: already exists 
Copying blob 83310c7c677c skipped: already exists 
Copying blob 164d51196137 [--------------------------------------] 0.0b / 0.0b
Copying config 2c7e43d880 [--------------------------------------] 0.0b / 4.0KiB
Writing manifest to image destination
Storing signatures
```

Users can pull either the latest image or the
1.0 version. Later, when you build version 2.0 of your application, you
can store both images at the registry. You can run both version 1.0 and
2.0 of your application on the host at the same time.

Use a web browser (e.g., Firefox, Chrome,
Safari, Internet Explorer, or Microsoft Edge) to look at the images at
quay.io. You can see 1.0 and the latest image in figure 2.4:

``` programlisting
$ web-browser quay.io/repository/rhatdan/myimage?tab=tags
```

::: figure
![](images//02-04.png)

Figure 2.4 List of `myimage`
tags on quay.io
([https://quay.io/repository/rhatdan/myimage/?tab=tags](https://quay.io/repository/rhatdan/myimage/?tab=tags){.url})
:::

Now that you have pushed your image to a
container registry, you may want to free up storage from your home
directory by removing the
images.

[Note]{.fm-callout-head} Contrary to common
sense, the tag `latest`{.fm-code-in-text1} does not refer to the most
up-to-date image in the repository. It is just another tag with no magic
involved. Even worse, because it is being used as a default tag for
images pushed without tags, it could refer to any random version of an
image. There could be newer images in the container registry than your
local container's storage with this tag. Thus, it is always better to
refer to the specific version of the image you want to use, rather than
relying on the `latest`{.fm-code-in-text1}.

### 2.2.7 Removing images {#02.htm#heading_id_20 .fm-head1}

Over
time,
images can take up a lot of disk space. Thus, it will be a good idea to
remove images you no longer use. Let's list local images first:

``` programlisting
$ podman images
REPOSITORY                          TAG        IMAGE ID       CREATED    SIZE
localhost/myimage                   1.0    2c7e43d88038  46 hours ago  462 MB
quay.io/rhatdan/myimage             1.0    2c7e43d88038  46 hours ago  462 MB
quay.io/rhatdan/myimage          latest    2c7e43d88038  46 hours ago  462 MB
registry.access.redhat
➥.com/ubi8/httpd-24             latest    8594be0a0b57   5 weeks ago  462 MB
registry.access.redhat
➥.com/ubi8                      latest    ad42391b9b46   5 weeks ago  234 MB
```

Use the `podman`
`rmi` command to remove
local images:

``` programlisting
$ podman rmi localhost/myimage
Untagged: localhost/myimage:latest
```

Listing the local images again, you will see
that the command didn't actually remove the image but only the
`localhost` tag from the
image. Podman still has two references to the same image ID: the actual
content of the image has not been removed. None of the disk space was
freed up:

``` programlisting
$ podman images
REPOSITORY                         TAG        IMAGE ID      CREATED     SIZE
quay.io/rhatdan/myimage            1.0    2c7e43d88038  46 hours ago  462 MB
quay.io/rhatdan/myimage         latest    2c7e43d88038  46 hours ago  462 MB
registry.access.redhat
➥.com/ubi8/httpd-24            latest    8594be0a0b57   5 weeks ago  462 MB
registry.access.redhat
➥.com/ubi8                     latest    ad42391b9b46   5 weeks ago   234 MB
```

You can remove the other tags using a short
name (see section 2.2.8). Podman uses the short name and finds the first
name in local storage that matches the short name without a registry and
removes it, which is why I need to remove it twice to get rid of both
images. Tags other than `latest` need to be specified
explicitly:

``` programlisting
$ podman rmi myimage
Untagged: quay.io/rhatdan/myimage:latest
$ podman rmi myimage:1.0
Untagged: quay.io/rhatdan/myimage:1.0
Deleted: 2c7e43d88038669e8cdbdff324a9f9605d99697215a0d21c360fe8dfa8471bab
 
```

It is only when the last tag is removed that
the actual disk space is reclaimed:

``` programlisting
$ podman images
REPOSITORY                         TAG        IMAGE ID       CREATED     SIZE
registry.access.redhat
➥.com/ubi8/httpd-24            latest    8594be0a0b57   5 weeks ago   462 MB
registry.access.redhat
➥.com/ubi8                     latest    ad42391b9b46   5 weeks ago   234 MB
```

Alternatively, you can try removing the images
by specifying the image ID:

``` programlisting
$ podman rmi 14119a10abf4
Error: unable to delete image\ 
➥ "2c7e43d88038669e8cdbdff324a9f9605d99697215a0d21c360fe8dfa8471bab" by\ 
➥ ID with more than one tag ([quay.io/rhatdan/myimage:1.0\ 
➥ quay.io/rhatdan/myimage:latest]): please force removal
```

But that fails because there are multiple tags
for the same image. Adding the `--force`
option removes the image and all of its tags:

``` programlisting
$ podman rmi 14119a10abf4 --force
Untagged: quay.io/rhatdan/myimage:1.0
Untagged: quay.io/rhatdan/myimage:latest
Deleted: 2c7e43d88038669e8cdbdff324a9f9605d99697215a0d21c360fe8dfa8471bab
```

As your image sizes and numbers grow and more
containers are created, it becomes harder to figure out which images are
no longer needed. Podman has another useful
command---`podman` `image`
`prune`---for removing all
dangling images. *Dangling
images* are images that no longer have a tag
associated with them or a container using them. The
`prune` command also has the
`--all` option, which
removes all images that are currently not in use by any containers,
including dangling images:

``` programlisting
$ podman image prune -a
WARNING! This command removes all images without at least one container \
➥ associated with them.
Are you sure you want to continue? [y/N] y
6d633c2626113fb4e5aa75babb2af39268948497893f7bb5b4c2043d7a986ba0
B9097177b416944cabdcfcab0e74a319223ad1acaed38ac57a262b2421732355
```

[Note]{.fm-callout-head} Having no containers
running the `podman`{.fm-code-in-text1} `image`{.fm-code-in-text1}
`prune`{.fm-code-in-text1} command removes all
of the local images. This frees up all of the disk space in the home
directory. You can use the `podman`{.fm-code-in-text1}
`system`{.fm-code-in-text1} `df`{.fm-code-in-text1} command to show all
of the storage in your home directory used by Podman.

``` programlisting
$ podman images
REPOSITORY                              TAG       IMAGE ID    CREATED    SIZE
```

[]{#02.htm#id_k1qw18a94248}Some notable
`podman` `image`
`prune` options include the
following:

-   
    .calibre17}`--all`
    .calibre17}---This tells Podman to remove all images, freeing up all
    storage. Images that have containers running on them are not
    removed.

-   
    .calibre17}`--force`
    .calibre17}---This tells Podman to stop and remove any containers
    that are running on them and remove any images dependent on the
    image you are attempting to remove.

Use the `man`
`podman-image-prune` command
for information about all options.

Images pushed to the registry could also be
pulled for various reasons, including but not limited to sharing your
applications with others, testing other versions, getting back removed
local versions, and working on a new version of an
image.

### 2.2.8 Pulling images {#02.htm#heading_id_21 .fm-head1}

Although
you
previously removed all local images, you can pull the previously pushed
image at quay.io/rhatdan/myimage. Podman has the
`podman` `pull`
command to pull
images from container registries (transports) into local container
storage:

``` programlisting
$ podman pull quay.io/rhatdan/myimage
Trying to pull quay.io/rhatdan/myimage:latest...
Getting image source signatures
Copying blob dfd8c625d022 done 
Copying blob e21480a19686 done 
Copying blob 68e8857e6dcb done 
Copying blob 3f412c5136dd done 
Copying blob fbfcc23454c6 done 
Copying config 2c7e43d880 done 
Writing manifest to image destination
Storing signatures
2c7e43d880382561ebae3fa06c7a1442d0da2912786d09ea9baaef87f73c29ae
```

Does the output look familiar? You probably
remember similar output from the `podman`
`run` command from section
2.1.2:

``` programlisting
$ podman run -d -p 8080:8080 --name myapp\ 
➥ registry.access.redhat.com/ubi8/httpd-24
Trying to pull registry.access.redhat.com/ubi8/httpd-24:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 296e14ee2414 skipped: already exists  
Copying blob 356f18f3a935 skipped: already exists  
Copying blob 359fed170a21 done
Copying blob 226cafc3a0c6 done
Writing manifest to image destination
Storing signatures
37a1d2e31dbf4fa311a5ca6453f53106eaae2d8b9b9da264015cc3f8864fac22
```

Many Podman commands implicitly execute the
`podman` `pull`
command if the required image is not present
locally.

So executing `podman`
`images` shows the image back in container storage,
ready to be used for containers:

``` programlisting
$ podman images
REPOSITORY                   TAG        IMAGE ID     CREATED    SIZE
quay.io/rhatdan/myimage   latest    2c7e43d88038  2 days ago  462 MB
```

Up until now, you have been typing the image
with the full names as registry.access .redhat.com/ubi8/httpd-24 or
quay.io/rhatdan/myimage, but if you are like me and not a great typist,
this can be a pain. You really need a way to refer to the images via
short names.

Short names and container registries

When
Docker
first hit the scene, they defined an image reference as a combination of
the container registry where the image was stored, repository, image
name, and a tag or version of the image. In our examples, we have been
using quay.io/rhatdan/myimage. In table 2.4, you can see this image name
breakdown; note that the `latest`
tag was used implicitly, as the image version
wasn't specified.

Table 2.4 Container image name table

+-----------------+-----------------+-----------------+-----------------+
| []{             | []{#0           | []{#02.htm#pgf  | []{#02.htm#pg   |
| #02.htm#pgfId-1 | 2.htm#pgfId-104 | Id-1045380}Name | fId-1045382}Tag |
| 045376}Registry | 5378}Repository |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| []{#02.htm#pg   | []              | []              | []{#02.         |
| fId-1045384}qua | {#02.htm#pgfId- | {#02.htm#pgfId- | htm#pgfId-10453 |
| y.io[]{#02.htm# | 1045386}rhatdan | 1045388}myimage | 90}`latest`{.fm |
| marker-1045447} |                 |                 | -code-in-text1} |
+-----------------+-----------------+-----------------+-----------------+

The Docker command line has internally set the
docker.io registry as the only registry, thus making every short image
name refer to images at docker.io. There is also a special repository
library, which is used for certified images.

So rather than typing

``` programlisting

`# docker pull docker.io/library/alpine:latest`

You can just execute

``` programlisting
```bash
# docker pull alpine
```

Conversely, if you want to pull an image from a
different registry, you need to specify the full name of the image:

``` programlisting
# docker pull registry.access.redhat.com/ubi8/httpd-24:latest
```

Table 2.5 shows the difference between the
image name used in a short name versus the fully specified image name.
Notice that when using the short name, the registry, repository, and tag
were not specified.

Table 2.5 Short name to container image name
table

+-----------------+-----------------+-----------------+-----------------+
| []{             | []{#0           | []{#02.htm#pgf  | []{#02.htm#pg   |
| #02.htm#pgfId-1 | 2.htm#pgfId-104 | Id-1045504}Name | fId-1045506}Tag |
| 045500}Registry | 5502}Repository |                 |                 |
+-----------------+-----------------+-----------------+-----------------+
| []{#02.htm#p    | []{#02.htm#p    | [               | []{#02.htm#p    |
| gfId-1045508}   | gfId-1045510}   | ]{#02.htm#pgfId | gfId-1045514}   |
|                 |                 | -1045512}alpine |                 |
+-----------------+-----------------+-----------------+-----------------+
| []{#02.htm#pgfI | []              | [               | []{#02.         |
| d-1045516}docke | {#02.htm#pgfId- | ]{#02.htm#pgfId | htm#pgfId-10455 |
| r.io[]{#02.htm# | 1045518}library | -1045520}alpine | 22}`latest`{.fm |
| marker-1045523} |                 |                 | -code-in-text1} |
+-----------------+-----------------+-----------------+-----------------+

Since I am lazy and hate to type extra
characters, I almost always use short names. With Podman, the developers
did not want to hardcode one registry, docker.io, into the tool. Podman
allows distributions, companies, and you to control which registries to
use and to be able to configure multiple registries. At the same time,
Podman provides support for the easier-to-use short names.

Podman usually comes with multiple registries
defined, controlled by the distribution that packaged Podman. You can
use the `podman` `info`
command to see what registries are defined for
your Podman installation:

``` programlisting
$ podman info
...
registries:
  search:
  - registry.fedoraproject.org
  - registry.access.redhat.com
  - docker.io
  - quay.io
```

The list of registries can be modified in the
registries.conf file, which is described in section 5.2.1.

Let's discuss the security side of things using
these commands:

``` programlisting
$ podman pull rhatdan/myimage
$ podman pull quay.io/rhatdan/myimage
```

From a security perspective, it is always
better to specify the full image name when pulling it from a registry.
That way, Podman guarantees that it pulls from the specified registry.
Imagine you are attempting to pull rhatdan/myimage. Using the previous
search order, there is a chance someone could set up an account on
docker.io/rhatdan and trick you into mistakenly pulling
docker.io/rhatdan/myimage.

To help protect against this, on the first pull
of an image, Podman prompts you to select an exact image from the list
of found images in configured registries:

``` programlisting
$ podman create -p 8080:8080 ubi8/httpd-24 
? Please select an image: 
   registry.fedoraproject.org/ubi8/httpd-24:latest
 ▸ registry.access.redhat.com/ubi8/httpd-24:latest
   docker.io/ubi8/httpd-24:latest
   quay.io/ubi8/httpd-24:latest
```

Once you have selected and pulled an image
successfully, Podman records the short name mapping. In the future, when
you run a container with this short name, Podman uses the short name
mapping to pick the correct registry and does not prompt.

Linux distributions also ship mappings of the
most commonly used short names, as they want you to pull from their
supported registries. You can find these short name configuration files
in the /etc/containers/registries.conf.d directory on the Linux host.
Companies can also drop short name alias files in this directory:

``` programlisting
$ cat /etc/containers/registries.conf.d/000-shortnames.conf
[aliases]
```bash
# centos
  "centos" = "quay.io/centos/centos"
  # containers
  "skopeo" = "quay.io/skopeo/stable"
  "buildah" = "quay.io/buildah/stable"
  "podman" = "quay.io/podman/stable"
...
```

[]{#02.htm#id_alh8ymvwn26r}[]{#02.htm#id_xpx7qets8jwc}Some
notable `podman` `pull` options
include the following:

-   
    .calibre17}`--arch`
    .calibre17}---This tells Podman to pull an image for a different
    architecture. For example, on my x86_64 machine, I can pull an arm64
    image. By default, `podman`
    `pull` pulls images for the native architecture.

-   `--quiet`
    `(-q`
    .calibre17}`)`---This tells Podman not to print
    out all the progress information. It just prints the image ID when
    it completes.

Use the `man`
`podman-pull` command for
information about all options.

I have mentioned a few images in this book, but
there are thousands and thousands of images available. You need a
mechanism to be able to search through these images for the
perfect
match.

### 2.2.9 Searching for images {#02.htm#heading_id_22 .fm-head1}

You
might
not know the name of a particular image you want to run or use as a base
for your own image. Podman provides the command
`podman`
`search`, which allows you
to search container registries for matching names:

``` programlisting
$ podman search registry.access.redhat.com/httpd
INDEX     NAME                                                     
➥   DESCRIPTION                      redhat.com 
➥ registry.access.redhat.com/rhscl/httpd-24-rhel7
➥ Apache HTTP 2.4\ Server                                          
redhat.com  registry.access.redhat.com/ubi8/httpd-24\                
➥ Platform for running Apache httpd 2.4 or bui...                   
redhat.com  registry.access.redhat.com/rhscl/varnish-6-rhel7        Varnish\
➥ available as container is a base pla...
...
```

In this example, we are searching for images
that include the string *httpd* in their name on the repository
registry.access.redhat.com.

[]{#02.htm#id_3adi657j8luj}Some notable
`podman` `search` options include
the following:

-   
    .calibre17}`--no-trunc`---This tells Podman to
    show the full description of the image.

-   
    .calibre17}`--format`
    .calibre17} .calibre17}---This allows you
    to customize which fields are displayed by Podman.

Use the `man`
`podman-search` command for
information about all options.

Up until now, you have seen several ways of
managing and manipulating container images, including inspecting,
pushing, pulling, and searching for them. But you have only been able to
look at the contents of an image by running it as a container. One way
to simplify the process is mounting a container
image.

## 2.2.10 Mounting images

Often
you
might want to examine the contents of a container image, and one way to
do this is launching a shell inside a running container from the image.
The problem with this is that the tools you use to examine the container
image might not be available within the container. There is also a
security risk that the application in the container is malicious, making
use of this container undesirable.

To help with these problems, Podman provides
the `podman` `image`
`mount` command to mount an
image's root filesystem in a read-only mode without creating a container
from it. The mounted image becomes immediately available on the host
system, allowing you to examine its contents.

Now try mounting the image you pulled
previously:

``` programlisting
$ podman mount quay.io/rhatdan/myimage
Error: cannot run command "podman mount" in rootless mode, must execute `podman unshare` first
```

The reason for this error is that rootless mode
does not allow mounting images. You need to enter a user namespace and
separate mount namespace. Chapter 5 explains how most rootless Podman
commands enter the user namespace and mount namespace when they execute.
For now, it is enough to know that the `podman`
`unshare` command enters the
user and mount namespaces and will shut down when you execute the
`exit` command of your
shell.

[Note]{.fm-callout-head} The name
`unshare`{.fm-code-in-text1} comes from the Linux syscall
`unshare`{.fm-code-in-text1} (`man`{.fm-code-in-text1}
`2`{.fm-code-in-text1} `unshare`{.fm-code-in-text1}). Linux also
includes an unshare tool (`man`{.fm-code-in-text1}
`1`{.fm-code-in-text1} `unshare`{.fm-code-in-text1}), which allows you
to create namespaces by hand. Another low-level tool called
`nsenter`{.fm-code-in-text1}, or namespace
enter (`man`{.fm-code-in-text1} `1`{.fm-code-in-text1}
`nsenter`{.fm-code-in-text1}), allows you to join processes to different
namespaces. Podman `unshare`{.fm-code-in-text1} uses the same kernel
features. It simplifies the process of creating and configuring
namespaces and inserting processes into the namespaces.

The `podman`
`unshare` command leaves you
at a `#` prompt, where you can actually mount an
image:

``` programlisting
$ podman unshare
#
```

Mount the image, and save the location of the
mounted filesystem in an environment variable:

``` programlisting
# mnt=$(podman image mount quay.io/rhatdan/myimage)
```

Now you can actually examine the content of the
image. Let's print the contents of a file on the terminal:

``` programlisting
# cat $mnt/var/www/html/index.html 
<html>
 <head>
 </head>
 <body>
  <h1>Hello World</h1>
 </body>
</html>
```

When you are done, unmount the image, and exit
the unshare session:

``` programlisting
# podman image unmount quay.io/rhatdan/myimage
# exit
```

[Note]{.fm-callout-head} You have examined
about a half of the `podman`{.fm-code-in-text1}
`image`{.fm-code-in-text1} subcommands,
arguably the most used ones. Refer to the Podman man pages for a full
explanation of these and other subcommands of the
`podman`{.fm-code-in-text1} `image`{.fm-code-in-text1} command:
`$`{.fm-code-in-text1} `man`{.fm-code-in-text1}
`podman-image`{.fm-code-in-text1}.

Now that you have a better understanding of
containers and images, the next important step is updating your image.
The main reasons for this are the need to update your application and
the availability of new versions for the base image you use. You can
build scripts to manually run the commands to build the image, but
luckily, Podman optimized
the
experience.

## 2.3 Building images

So
far you have been
working with images, which were already created and uploaded to a
container registry. The process of creating a container image is called
*building*.

When building container images, you manage not
only your application but also the image content used by this
application. In the days prior to containers, you shipped applications
as an RPM or DEB package, and then it was up to the distribution to make
sure the other parts of the OS were kept up to date and secure. But in
the container world, the container image includes the application along
with a subset of the OS. It is the developers' responsibility to keep
all of the image contents up to date and secure.

A coworker of mine, Scott McCarty
(smccarty@redhat.com, \@fatherlinux), has a saying, "Container images
don't age like wine but more like cheese. As the image gets older it
gets stinky."

This means that if the developer doesn't keep
up with the security updates, the number of vulnerabilities in the image
will grow at an alarming rate. Luckily for developers, Podman has a
special mechanism for helping you with image building for your
applications. The `podman` `build`
command uses the Buildah tool
([https://github.com/containers/buildah](https://github.com/containers/buildah){.url})
as a library to build container images; Buildah is covered in appendix
A.

The `podman`
`build` uses a special text document called
Containerfile or Dockerfile to automate the building of container
images. This document lists commands used to build a container image.

[Note]{.fm-callout-head} The concept of a
Dockerfile and its syntax was originally created for the Docker tool,
developed by Docker, Inc. Podman defaults to using Containerfile for the
name, which uses the exact same syntax. Dockerfile is supported as well
for legacy purposes. The Docker build command does not support
Containerfile by default but can use the Containerfile. You can specify
the `-f`{.fm-code-in-text1} option: `#`{.fm-code-in-text1}
`docker`{.fm-code-in-text1} `build`{.fm-code-in-text1}
`-f`{.fm-code-in-text1} `Containerfile.`{.fm-code-in-text1}

### 2.3.1 Format of a Containerfile or Dockerfile {#02.htm#heading_id_25 .fm-head1}

Containerfiles
take
many directives. I break these down into two categories, adding content
to the container image and describing and documenting how to use the
image[]{#02.htm#id_ozq815u6tf54}.

Adding content to an image

Recall
back
in section 1.1.2 that I described a container image as a tdirectory on
disk that looks like root on a Linux system. This directory is called a
rootfs. Several of the directives in a container job are adding content
to this rootfs. This rootfs eventually contains all of the content used
to create your container image.

Every Containerfile must include a
`FROM` line. The `FROM` line
specifies the image that the new image is based off, often called a base
image. The `podman` `build`
command supports a special image named
`scratch`, which means to
start your image with no content. When Podman sees the
`FROM` `scratch`
directive, it just allocates space in
containers/storage for an empty rootfs, then `COPY`
can be used to populate the rootfs. More often, the
`FROM` directive uses an
existing image. For example, `FROM`
`registry.access.redhat.com/ubi8` causes Podman to
pull the ubi8 image from the registry.access.redhat.com container
registry and copy it to container storage.
`podman` `build` pulls the same
image as the `podman` `pull`
command you learned about in section 2.2.8.
When the image is pulled, Podman uses container storage to mount the
image on the rootfs directory, using a copy on the write filesystem,
like OverlayFS, where the other directives can start to add content.
This image becomes the base layer of the rootfs.

The `COPY` directive is often
used to copy files, directories, or tarballs off of the local host into
the newly created rootfs. The `RUN`
directive is one of
the most commonly used Containerfile directives. `RUN`
tells Podman to actually run a container on the image. Package
management tools, like DNF/YUM and `apt-get`, are run
to install packages from distributions onto your new image. The
`RUN` directive runs any
command within the container image as a container. The
`podman` `build`
command runs the commands with the same
security constraints as the `podman`
`run` command.

As an example, imagine you want to add the
`ps` command to a container
image; you can create a directive like the following. The
`RUN` command executes a container, which updates all
of the packages from the base image, and then installs the
`procps-ns` package, which includes the
`ps` command. Finally the
containerized command executes
`yum` to clean up after itself, so cruft is removed
from the container image:

``` programlisting
RUN yum -y update; yum -y install procps-ng; yum -y clean all
```

Adding content to the container image is only
half of what you need to do when creating a container image. You also
need to describe and document how the image will be used when other
users download and run your
image.

Documenting how to use the image

Recall that
back
in section 1.1.2, I also described the JSON file that included the image
specification. This specification describes how the container image is
to be run, the command, which user to run it with, and other
requirements of the image. The Containerfile also supports many
directives, which tells Podman how to run containers. These include the
following:

-   *The*
    `ENTRYPOINT` .calibre17}
    *and* `CMD` *directives*
    .calibre17}---These instrument the image with the default command to
    be executed when users execute the image with Podman
    `run`. `CMD` is the actual
    command to run. `ENTRYPOINT` can cause the entire
    image to execute as a single command.

-   *The* `ENV`
    *directive*---This sets up the default environment variables to run
    when Podman runs a container on the image.

-   *The* `EXPOSE`
    *directive*
    .calibre17} .calibre17}---This records the
    network ports for Podman to expose in containers based on the image.
    If you execute `podman` `run`
    `--publish-all` `...`, Podman
    looks inside of the image for the `EXPOSE` network
    ports and connects them to the host.

Table 2.6 explains the directives used in a
Containerfile to add content to a container image.

Table 2.6 Containerfile directives that update
the image

+----------------------+----------------------------------------------+
| []{#02.htm#pgf       | Explanation         |
| Id-1045585}Directive |                                              |
| examples             |                                              |
+----------------------+----------------------------------------------+
| []{#02.htm           | Sets the base image |
| #pgfId-1045589}`FROM | for subsequent instructions. Containerfiles  |
| `{.fm-code-in-text1} | must have `FROM`{.fm-code-in-text1} as their |
| `qua                 | first instruction. The                       |
| y.io/rhatdan/myimage | `FROM`{.fm-code-in-text1} may appear         |
| `{.fm-code-in-text1} | multiple times within a single Containerfile |
|                      | to create multiple build stages.             |
+----------------------+----------------------------------------------+
| []{#02.ht            | Copies new files,   |
| m#pgfId-1045593}`ADD | directories, or remote file URLs to the      |
| `{.fm-code-in-text1} | filesystem of the container at a specified   |
| `start.sh            | path.                                        |
| `{.fm-code-in-text1} |                                              |
| `/usr/bin/start.sh   |                                              |
| `{.fm-code-in-text1} |                                              |
+----------------------+----------------------------------------------+
| []{#02.htm           | Copies files to the |
| #pgfId-1045597}`COPY | filesystem of the container at a specified   |
| `{.fm-code-in-text1} | path.                                        |
| `start.sh            |                                              |
| `{.fm-code-in-text1} |                                              |
| `/usr/bin/start.sh   |                                              |
| `{.fm-code-in-text1} |                                              |
+----------------------+----------------------------------------------+
| []{#02.ht            | Executes commands   |
| m#pgfId-1045601}`RUN | in a new layer on top of the current image   |
| `{.fm-code-in-text1} | and commits the results. The committed image |
| `dnf                 | is used for the next step in the             |
| `{.fm-code-in-text1} | Containerfile.                               |
| `-y                  |                                              |
| `{.fm-code-in-text1} |                                              |
| `update              |                                              |
| `{.fm-code-in-text1} |                                              |
+----------------------+----------------------------------------------+
| []{#02.htm#p         | Creates a mount     |
| gfId-1045605}`VOLUME | point with the specified name and marks it   |
| `{.fm-code-in-text1} | as holding externally mounted volumes from   |
| `/var/lib/mydata     | the native host or from other containers.    |
| `{.fm-code-in-text1} | For more on volumes, see chapter 3.          |
+----------------------+----------------------------------------------+

Table 2.7 explains the directives used in a
Containerfile to populate the OCI Runtime Specifications with
information that tells container engines like Podman information about
the image and how to run the image. You can find much more information
on Containerfiles in the `containerfile(5)` man
page.

Table 2.7 Containerfile directives that define
the OCI Runtime Specification

+----------------------+----------------------------------------------+
| []{#02.htm#pgf       | Explanation         |
| Id-1045703}Directive |                                              |
| examples             |                                              |
+----------------------+----------------------------------------------+
| []{#02.ht            | Specifies the       |
| m#pgfId-1045707}`CMD | default command to run when launching a      |
| `{.fm-code-in-text1} | container off this image. If                 |
| `/usr/bin/start.sh   | `CMD`{.fm-code-in-text1} is not specified,   |
| `{.fm-code-in-text1} | the parent image's `CMD`{.fm-code-in-text1}  |
|                      | is inherited. Note that                      |
|                      | `RUN`{.fm-code-in-text1} and                 |
|                      | `CMD`{.fm-code-in-text1} are very different. |
|                      | `RUN`{.fm-code-in-text1} runs the commands   |
|                      | during the build process, while              |
|                      | `CMD`{.fm-code-in-text1} is only used when a |
|                      | user launches the image without specifying a |
|                      | command.                                     |
+----------------------+----------------------------------------------+
| []{#02.htm#pgfId     | Allows you to       |
| -1045711}`ENTRYPOINT | configure a container to run as an           |
| `{.fm-code-in-text1} | executable. The                              |
| `“/bin/sh -c”`{.fm-  | `ENTRYPOINT`{.fm-code-in-text1} instruction  |
| code-in-text1}[]{#02 | is not overwritten when arguments are passed |
| .htm#marker-1045746} | to `podman run`{.fm-code-in-text1}. This     |
|                      | allows arguments to be passed to the         |
|                      | entrypoint---for instance,                   |
|                      | `podman run <image> -d`{.fm-code-in-text1}   |
|                      | passes the `-d`{.fm-code-in-text1} argument  |
|                      | to the `ENTRYPOINT`{.fm-code-in-text1}.      |
+----------------------+----------------------------------------------+
| []{#02.ht            | Adds an environment |
| m#pgfId-1045715}`ENV | variable to be used during both the image    |
| `{.fm-code-in-text1} | build and container execution.               |
| `foo=”bar”           |                                              |
| `{.fm-code-in-text1} |                                              |
+----------------------+----------------------------------------------+
| []{#02.htm#p         | Announces the port  |
| gfId-1045719}`EXPOSE | that containerized applications will be      |
| `{.fm-code-in-text1} | exposing. This does not actually map or open |
| `8080`{.fm-          | any ports.                                   |
| code-in-text1}[]{#02 |                                              |
| .htm#marker-1045747} |                                              |
+----------------------+----------------------------------------------+
| []{#                 | Adds metadata to an |
| 02.htm#pgfId-1045723 | image.                                       |
| }`LABEL Description= |                                              |
| ”Web browser which d |                                              |
| isplays Hello World” |                                              |
| `{.fm-code-in-text1} |                                              |
+----------------------+----------------------------------------------+
| []{#02.ht            | Sets the            |
| m#pgfId-1045727}`MAI | `Author`{.fm-code-in-text1} field for the    |
| NTAINER Daniel Walsh | generated images.                            |
| `{.fm-code-in-text1} |                                              |
+----------------------+----------------------------------------------+
| []{#                 | Sets the default    |
| 02.htm#pgfId-1045731 | stop signal sent to the container to exit.   |
| }`STOPSIGNAL SIGTERM | The signal can be a valid unsigned number or |
| `{.fm-code-in-text1} | a signal name in the format                  |
|                      | `SIGNAME`{.fm-code-in-text1}.                |
+----------------------+----------------------------------------------+
| []{#02.htm#pgfId-    | Sets the user name  |
| 1045735}`USER apache | (or UID) and group name (or GID) to use for  |
| `{.fm-code-in-text1} | any `RUN`{.fm-code-in-text1},                |
|                      | `CMD`{.fm-code-in-text1}, and                |
|                      | `ENTRYPOINT`{.fm-code-in-text1} specified    |
|                      | after it.                                    |
+----------------------+----------------------------------------------+
|   | Adds a trigger      |
| 45739}`ONBUILD`{.fm- | instruction to the image to be executed at a |
| code-in-text1}[]{#02 | later time, when the image is used as the    |
| .htm#marker-1045748} | base for another build.                      |
+----------------------+----------------------------------------------+
| []{#02.htm#pg        | Sets the working    |
| fId-1045743}`WORKDIR | directory for `RUN`{.fm-code-in-text1},      |
| `{.fm-code-in-text1} | `CMD`{.fm-code-in-text1},                    |
| /var/www/html        | `ENTRYPOINT`{.fm-code-in-text1}, and         |
|                      | `COPY`{.fm-code-in-text1} directives. A      |
|                      | directory will be created if it doesn't      |
|                      | exist.                                       |
+----------------------+----------------------------------------------+

Committing the image

When
`podman`
`build` finishes processing the Containerfile, it
commits the image, using the same code as `podman`
`commit` you learned about in section 2.1.9.
Basically, Podman TARs up all of the differences between the new content
in the rootfs and the base image, pulled down by the
`FROM` directive. Podman
also commits the JSON file and saves this as an image in container
storage. Now you can take the steps used to build out containerized
applications and automate them using a Containerfile and Podman build.

[Tip]{.fm-callout-head} Use the
`--tag`{.fm-code-in-text1} option to name the
new image you are creating with `podman`{.fm-code-in-text1}
`build`{.fm-code-in-text1}. This tells Podman to add the specified tag
or name to the image in container storage in the same way as the
`podman`{.fm-code-in-text1}
`tag`{.fm-code-in-text1}
command.

### 2.3.2 Automating the building of our application {#02.htm#heading_id_26 .fm-head1}

First,
create
a directory to put your Containerfile and any other content for the
container image in. The directory is called a context directory:

``` programlisting
mkdir myapp
```

Next, create the index.html file you plan to
use in the containerized application in the `myapp`
directory:

``` programlisting
$ cat > myapp/index.html << _EOF
<html>
 <head>
 </head>
 <body>
 <h1>Hello World</h1>
 </body>
</html>
_EOF
```

Next, create a simple Containerfile to build
your application in the `myapp` directory. The first
line of the Containerfile is the `FROM`
directive to pull the ubi8/httpd-24 image you
are treating as your base image. Then add a `COPY`
command to copy the index.html file into the
image. The `COPY` directive
tells Podman to copy the index.html file out of the context directory
(./myapp) and copy it to the /var/www/html/index.html file within the
image:

``` programlisting
$ cat > myapp/Containerfile << _EOF
FROM ubi8/httpd-24
COPY index.html /var/www/html/index.html
_EOF
```

Finally, use `podman`
`build` to build your containerized application.
Specify the `--tag` (`-t`) to name
the image quay.io/rhatdan/myimage. You also need to specify the context
directory ./myapp:

``` programlisting
$ podman build -t quay.io/rhatdan/myimage ./myapp
STEP 1/2: FROM ubi8/httpd-24
STEP 2/2: COPY index.html /var/www/html/index.html
COMMIT quay.io/rhatdan/myimage
--> f81b8ace4f1
Successfully tagged quay.io/rhatdan/myimage:latest
F81b8ace4f134d08cedb20a9156ae727444ae4d4ec1ceb3b12d3aff23d18128b
```

When the `podman`
`build` command completes,
it commits the image and tags (`-t`) it with the
quay.io/rhatdan/myimage name. It is now ready to be pushed to the
container registry using the `podman`
`push` command.

Now you can set up a CI/CD system or even a
simple cron job to regularly build and replace
`myapplication`:

``` programlisting
$ cat > myapp/automate.sh << _EOF
#!/bin/bash
podman build -t quay.io/rhatdan/myimage ./myapp
podman push quay.io/rhatdan/myimage
_EOF
$ chmod +x myapp/automate.sh
```

Add some test scripts as well to make sure your
application works the way it was designed before replacing the previous
version. Let's take a look at the images that were built:

``` programlisting
$ podman images
REPOSITORY                         TAG        IMAGE ID        CREATED    SIZE
quay.io/rhatdan/myimage         latest    f81b8ace4f13  2 minutes ago  462 MB
<none>                          <none>    2c7e43d88038     2 days ago  462 MB
registry.access.redhat
➥.com/ubi8/httpd-24            latest    8594be0a0b57    5 weeks ago  462 MB
```

Notice the old version of
quay.io/rhatdan/myimage, image ID `2c7e43d88038`,
still exists in container storage but now has a
`REPOSITORY` and `TAG` of
`<none>` `<none>`. Images like these
are called dangling images. Since I have created a new version of
quay.io/rhatdan/myimage with the `podman`
`build` command, the
previous image loses that name. You can still use the Podman commands
with the image ID, or if the new image doesn't work, simply use
`podman` `tag` to rename the old
image back to quay.io/ rhatdan/myimage. If the new image works
correctly, you can remove the old image with `podman`
`rmi`. These `<none><none>` images
tend to build up over time, wasting space, but you can periodically use
the `podman` `image`
`prune` command to remove
them.

The `podman`
`build` could really use a chapter or even a book to
itself. People build images in thousands of different ways using the
commands briefly described here.

`--tag`[]{#02.htm#id_t7sehvoh2hfl1111}
is a notable `podman` `build` option
that specifies the image tag or name for the image. Remember that you
can always add additional names after you create the image with the
`podman` `tag`
command you used in section 2.2.6. Use the
`man` `podman-build`
command for information about all
options
(see table 2.8).

Table 2.8 Podman image commands

+---------+------------------------+-----------------------------------+
| []{#    | []{#02                 | []{                               |
| 02.htm# | .htm#pgfId-1045952}Man | #02.htm#pgfId-1045954}Description |
| pgfId-1 | page                   |                                   |
| 045950} |                        |                                   |
| Command |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#02. | [                      | Builds   |
| htm#pgf | ]{#02.htm#pgfId-104595 | an image using instructions from  |
| Id-1045 | 8}`podman-image-build( | Containerfiles                    |
| 956}`bu | 1)`{.fm-code-in-text1} |                                   |
| ild`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 2.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 046087} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#02  |  | Inspects |
| .htm#pg | 64}`podman-image-diff( | changes in image's filesystem     |
| fId-104 | 1)`{.fm-code-in-text1} |                                   |
| 5962}`d |                        |                                   |
| iff`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 2.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 046088} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{     | []                     | Checks   |
| #02.htm | {#02.htm#pgfId-1045970 | whether an image                  |
| #pgfId- | }`podman-image-exists( | exists  |
| 1045968 | 1)`{.fm-code-in-text1} |                                   |
| }`exist |                        |                                   |
| s`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#    | []{                    | Shows a  |
| 02.htm# | #02.htm#pgfId-1045976} | history |
| pgfId-1 | `podman-image-history( | of a specified image              |
| 045974} | 1)`{.fm-code-in-text1} |                                   |
| `histor |                        |                                   |
| y`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| [       | []                     | Imports  |
| ]{#02.h | {#02.htm#pgfId-1045982 | a tarball to create a filesystem  |
| tm#pgfI | }`podman-image-import( | image                             |
| d-10459 | 1)`{.fm-code-in-text1} |                                   |
| 80}`imp |                        |                                   |
| ort`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 2.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 046091} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#    | []{                    | Displays |
| 02.htm# | #02.htm#pgfId-1045988} | the configuration of an image     |
| pgfId-1 | `podman-image-inspect( |                                   |
| 045986} | 1)`{.fm-code-in-text1} |                                   |
| `inspec |                        |                                   |
| t`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#02  |  | Lists    |
| .htm#pg | 94}`podman-image-list( | all of the images                 |
| fId-104 | 1)`{.fm-code-in-text1} |                                   |
| 5992}`l |                        |                                   |
| ist`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 2.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 046092} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#02  |  | Loads    |
| .htm#pg | 00}`podman-image-load( | image(s) from a tarball           |
| fId-104 | 1)`{.fm-code-in-text1} |                                   |
| 5998}`l |                        |                                   |
| oad`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 2.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 046093} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#02. | [                      | Mounts   |
| htm#pgf | ]{#02.htm#pgfId-104600 | an image's root filesystem        |
| Id-1046 | 6}`podman-image-mount( |                                   |
| 004}`mo | 1)`{.fm-code-in-text1} |                                   |
| unt`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 2.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 046094} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#02. | [                      | Removes  |
| htm#pgf | ]{#02.htm#pgfId-104601 | unused images                     |
| Id-1046 | 2}`podman-image-prune( |                                   |
| 010}`pr | 1)`{.fm-code-in-text1} |                                   |
| une`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 2.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 046095} |                        |                                   |
+---------+------------------------+-----------------------------------+
| [       |  | Pulls an |
| ]{#02.h | 18}`podman-image-pull( | image from a registry             |
| tm#pgfI | 1)`{.fm-code-in-text1} |                                   |
| d-10460 |                        |                                   |
| 16}`pul |                        |                                   |
| l`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| [       |  | Pushes   |
| ]{#02.h | 24}`podman-image-push( | an image to a registry            |
| tm#pgfI | 1)`{.fm-code-in-text1} |                                   |
| d-10460 |                        |                                   |
| 22}`pus |                        |                                   |
| h`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#02  |    | Removes  |
| .htm#pg | 6030}`podman-image-rm( | an image                          |
| fId-104 | 1)`{.fm-code-in-text1} |                                   |
| 6028}`r |                        |                                   |
| m`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| [       |  | Saves    |
| ]{#02.h | 36}`podman-image-save( | image(s) to an archive            |
| tm#pgfI | 1)`{.fm-code-in-text1} |                                   |
| d-10460 |                        |                                   |
| 34}`sav |                        |                                   |
| e`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#02. |   | Securely |
| htm#pgf | 042}`podman-image-scp( | copies images to other            |
| Id-1046 | 1)`{.fm-code-in-text1} | containers/storage                |
| 040}`sc |                        |                                   |
| p`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{     | []                     | Searches |
| #02.htm | {#02.htm#pgfId-1046048 | the registry for an image         |
| #pgfId- | }`podman-image-search( |                                   |
| 1046046 | 1)`{.fm-code-in-text1} |                                   |
| }`searc |                        |                                   |
| h`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| [       |  | Signs an |
| ]{#02.h | 54}`podman-image-sign( | image                             |
| tm#pgfI | 1)`{.fm-code-in-text1} |                                   |
| d-10460 |                        |                                   |
| 52}`sig |                        |                                   |
| n`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#02. |   | Adds an  |
| htm#pgf | 060}`podman-image-tag( | additional name to a local image  |
| Id-1046 | 1)`{.fm-code-in-text1} |                                   |
| 058}`ta |                        |                                   |
| g`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| [       |  | Prints   |
| ]{#02.h | 66}`podman-image-tree( | the layer hierarchy of an image   |
| tm#pgfI | 1)`{.fm-code-in-text1} | in a tree format                  |
| d-10460 |                        |                                   |
| 64}`tre |                        |                                   |
| e`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []      | [                      | Manages  |
| {#02.ht | ]{#02.htm#pgfId-104607 | container image trust policy      |
| m#pgfId | 2}`podman-image-trust( |                                   |
| -104607 | 1)`{.fm-code-in-text1} |                                   |
| 0}`trus |                        |                                   |
| t`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#    | []{                    | Unmounts |
| 02.htm# | #02.htm#pgfId-1046078} | an image's root filesystem        |
| pgfId-1 | `podman-image-unmount( |                                   |
| 046076} | 1)`{.fm-code-in-text1} |                                   |
| `unmoun |                        |                                   |
| t`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []      | [                      | Removes  |
| {#02.ht | ]{#02.htm#pgfId-104608 | a name from a local image         |
| m#pgfId | 4}`podman-image-untag( |                                   |
| -104608 | 1)`{.fm-code-in-text1} |                                   |
| 2}`unta |                        |                                   |
| g`{.fm- |                        |                                   |
| code-in |                        |                                   |
| -text1} |                        |                                   |
+---------+------------------------+-----------------------------------+

## Summary 

-   Podman's simple command-line
    interface makes working with containers easy.

-   Podman `run`,
    `stop`, `start`,
    `ps`,
    `inspect` .calibre17},
    `rm` .calibre17}, and
    `commit` are all commands for working with
    containers.

-   Podman
    `pull` .calibre17},
    `push`, `login`, and
    `rmi` are tools for working with images and
    sharing them via container registries.

-   Podman `build`
    is a great command for automating the build of container images.

-   Podman's command line is based
    on the Docker CLI and supports it exactly, allowing us to tell
    people to just alias Docker = Podman.

-   Podman has additional commands
    and options to support more advanced concepts
    like .calibre17}
    .calibre17}
    .calibre17}
    .calibre17}
    .calibre17}
    .calibre17}
    .calibre17}
    .calibre17} .calibre17}
     .calibre17}`podman`
    `image` `mount`.

[]{#03.htm}
