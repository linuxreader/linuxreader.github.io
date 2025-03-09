# 4 Pods 

[]{#04.htm#pgfId-1103748}This chapter []{#04.htm#marker-1104179}covers

-   []{#04.htm#pgfId-1103749 .calibre17}An introduction to pods
-   []{#04.htm#pgfId-1103750 .calibre17}Managing multiple containers
    within a pod
-   []{#04.htm#pgfId-1103751 .calibre17}Using volumes with pods

[]{#04.htm#pgfId-1103753}*Podman* is short for *Pod
Manager*[]{#04.htm#marker-1103752}. A *pod* is a concept popularized by
the Kubernetes project; it is a group of one or more containers working
together for a common purpose and sharing the same namespaces and
cgroups (resource constraints). Additionally, Podman ensures that on
SELinux machines, all container processes within a pod share the same
SELinux labels. This means they can all work together from an SELinux
point of view.

## []{#04.htm#pgfId-1103755}4.1 Running pods {#04.htm#heading_id_3 .fm-head}

[]{#04.htm#pgfId-1103757}Podman []{#04.htm#marker-1103756}pods (see
figure 4.1), just like Kubernetes Pods, always include a container
called the *infra* container[]{#04.htm#marker-1103758}---sometimes
called the *pause* container (not to be confused with the rootless pause
container mentioned in section 5.2). The infra container only holds open
the namespaces and cgroups from the kernel, allowing containers to come
and go within the pod. When Podman adds a container to a pod, it adds
the container process to the cgroups and namespaces. Notice that the
infra container has a container monitor process,
`conmon`{.fm-code-in-text}, monitoring it. Every container within a pod
has its own `conmon`{.fm-code-in-text}.

[]{#04.htm#pgfId-1103759}Conmon is a lightweight C program that monitors
the container until it exits, allowing the Podman executable to exit and
reconnect to the container. Conmon does the following when monitoring
the container:

1.  []{#04.htm#pgfId-1103760 .calibre17}Conmon executes the OCI runtime,
    handing it the path to the OCI spec file as well as pointing to the
    container layer mount point in containers/storage. The mount point
    is called the rootfs[]{#04.htm#marker-1103761 .calibre17}.

2.  []{#04.htm#pgfId-1103762 .calibre17}Conmon monitors the container
    until it exits and reports its exit code back.

3.  []{#04.htm#pgfId-1103763 .calibre17}Conmon handles when the user
    attaches to the container, providing a socket to stream the
    container's `STDOUT`{.fm-code-in-text} and
    `STDERR`{.fm-code-in-text}.

4.  []{#04.htm#pgfId-1103764 .calibre17}The `STDOUT`{.fm-code-in-text}
    and `STDERR`{.fm-code-in-text} are also logged to a file for Podman
    logs.

[]{#04.htm#pgfId-1103771}[Note]{.fm-callout-head} The infra container
(pause container; see figure 4.1) is similar to the rootless pause
container; its only purpose is to hold open the namespaces and cgroups,
while containers come and go. However, each pod will have a different
infra container.

::: figure
![](images/OEBPS/Images/04-01.png){.calibre18}

[]{#04.htm#pgfId-1108956}[]{#04.htm#id_84p7xz75sdgk}Figure 4.1 The
Podman pod launches conmon with the infra container, which will hold
cgroups and Linux namespaces.
:::

[]{#04.htm#pgfId-1103772}Podman pods also support init containers, as
seen in figure 4.2. These containers run before the primary containers
in the pods are executed. An example of an init container is a database
initialization on a volume. This would allow the primary container to
use the database. Podman supports the following two classes of init
containers:

-   []{#04.htm#pgfId-1103773 .calibre17}*Once*---Only runs the first
    time the pod is created

-   []{#04.htm#pgfId-1103774 .calibre17}*Always*---Runs every time the
    pod is started

[]{#04.htm#pgfId-1103775}The primary container runs the application.

::: figure
![](images/OEBPS/Images/04-02.png){.calibre18}

[]{#04.htm#pgfId-1109098}Figure 4.2 []{#04.htm#id_n238co82eabp}Podman
next launches any init containers with conmon. The init containers
examine the infra container and join its cgroups and namespaces.
:::

[]{#04.htm#pgfId-1103788}Pods also support additional containers, which
are often called *sidecar* containers (see figure 4.4). Sidecar
containers often monitor the primary container, as seen in figure 4.3,
or the environment where the primary container runs. The Kubernetes
documentation
([https://kubernetes.io/docs/concepts/workloads/pods](https://kubernetes.io/docs/concepts/workloads/pods){.url})
describes pods with sidecar containers as follows:

::: figure
![](images/OEBPS/Images/04-03.png){.calibre18}

[]{#04.htm#pgfId-1109134}[]{#04.htm#id_j3t1mdk8ldy}Figure 4.3 Podman
waits until the init containers complete before launching the primary
containers with their `conmon`{.fm-code-in-text} into the pod.
:::

[]{#04.htm#pgfId-1103790 .calibre17}A Pod can encapsulate an application
composed of multiple co-located containers that are tightly coupled and
need to share resources. These co-located containers form a single
cohesive unit of service---for example, one container serving data
stored in a shared volume to the public, while a separate sidecar
container refreshes or updates those files. The Pod wraps these
containers, storage resources, and an ephemeral network identity
together as a single unit.

[]{#04.htm#pgfId-1103791}If you want to dive deeper into sidecar
containers, there are several good articles on the following website:
[https://www.magalix.com/blog/the-sidecar-pattern](https://www.magalix.com/blog/the-sidecar-pattern){.url}.

::: figure
![](images/OEBPS/Images/04-04.png){.calibre18}

[]{#04.htm#pgfId-1109173}[]{#04.htm#id_8bq5c4isrfz1}Figure 4.4 Podman
can launch additional containers called sidecar containers.
:::

[]{#04.htm#pgfId-1103799}[Note]{.fm-callout-head} While pods can support
more than one sidecar container, I recommend you only use one. There is
a real temptation for people to abuse this capability, especially in
Kubernetes, but it can use up more resources and become unwieldy.

[]{#04.htm#pgfId-1103800}A big advantage of using pods is that you can
manage them as discrete units. Starting a pod starts all of the
containers within it, and stopping the pod stops all of the containers.

## []{#04.htm#pgfId-1103802}4.2 Creating a pod {#04.htm#heading_id_4 .fm-head}

[]{#04.htm#pgfId-1103804}In []{#04.htm#marker-1103803}this section, you
will create a pod in which you have the myimage application as the
primary container within the pod. You will also add a second container
to the pod, a sidecar container, which will update the web content used
by your application to show two containers working together within a
pod.

[]{#04.htm#pgfId-1103807}You can create a pod named
`mypod`{.fm-code-in-text}[]{#04.htm#marker-1103805} using the
`podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`create`{.fm-code-in-text} command[]{#04.htm#marker-1103806}, as
demonstrated in the following command:

``` programlisting
$ podman pod create -p 8080:8080 --name mypod --volume ./html:/var/www/html:z
790fefe97b280e5f67c526e3a421e9c9f958cf5a98f3709373ef1afd91965955
```

[]{#04.htm#pgfId-1103811}The `podman`{.fm-code-in-text}
`pod`{.fm-code-in-text} `create`{.fm-code-in-text}
command[]{#04.htm#marker-1103810} has many of the same options as the
`podman`{.fm-code-in-text} `container`{.fm-code-in-text}
`create`{.fm-code-in-text} command[]{#04.htm#marker-1103812}. When you
create a container within a pod, the container inherits these options as
its default (see figure 4.5).

::: figure
![](images/OEBPS/Images/04-05.png){.calibre18}

[]{#04.htm#pgfId-1109215}[]{#04.htm#id_jxxk1q3so2r7}Figure 4.5 Podman
creates a network namespace and binds port `8080`{.fm-code-in-text}
within the container to port `8080`{.fm-code-in-text} on the host.
Podman creates the infra container with the /var/www/html directory from
the host in the container and joins the cgroups and network namespace.
:::

[]{#04.htm#pgfId-1103813}Notice that, like the previous examples, you
are binding the pod to port `-p`{.fm-code-in-text}
`8080:8080`{.fm-code-in-text}:

``` programlisting
$ podman pod create -p 8080:8080 --name mypod --volume ./html:/var/www/html:z
```

[]{#04.htm#pgfId-1103815}Because the containers within the pod share the
same network namespace, this port binding is shared by all of the
containers. The kernel allows only one process to listen on port
`8080`{.fm-code-in-text}. Lastly, notice that the ./html directory was
volume mounted, `--volume`{.fm-code-in-text}
`./html:/var/www/html:z`{.fm-code-in-text}, into the pod:

``` programlisting
$ podman pod create -p 8080:8080 --name mypod --volume ./html:/var/www/html:z
```

[]{#04.htm#pgfId-1103818}The `:z`{.fm-code-in-text}
parameter[]{#04.htm#marker-1103817} causes Podman to relabel the content
of the directory. Podman will automatically volume mount this directory
into every container that joins the pod. Containers in pods share the
same SELinux label, which means they can share the same
[]{#04.htm#marker-1103819}volumes.

## []{#04.htm#pgfId-1103828}4.3 Adding a container to a pod {#04.htm#heading_id_5 .fm-head}

[]{#04.htm#pgfId-1103832}You
[]{#04.htm#marker-1103829}[]{#04.htm#marker-1103830}create a container
within a pod using the `podman`{.fm-code-in-text}
`create`{.fm-code-in-text} command[]{#04.htm#marker-1103831} (see figure
4.6). Add the quay.io/rhatdan/myimage container to the pod with the
`--pod`{.fm-code-in-text} `mypod`{.fm-code-in-text}
option[]{#04.htm#marker-1103833}:

``` programlisting
$ podman create --pod mypod --name myapp quay.io/rhatdan/myimage
Cec045acb1c2be4a6e4e88e21275076fb1de5519a25fb5a55f192da70708a640
```

::: figure
![](images/OEBPS/Images/04-06.png){.calibre18}

[]{#04.htm#pgfId-1109257}Figure 4.6 []{#04.htm#id_g2ujyja8w08f}Because
the pod does not have any init containers, the first
`myapp`{.fm-code-in-text} container is launched into the pod.
:::

[]{#04.htm#pgfId-1103842}When you add the first container to the pod,
Podman reads the information associated with the infra container and
adds the volume mount to the `myapp`{.fm-code-in-text}
container[]{#04.htm#marker-1103843} and then joins it to the namespaces
held by the infra container. The next step is adding the sidecar
container to the pod. The sidecar container will be updating the
index.html file in the /var/www/html volume, adding a new time stamp
every second.

[]{#04.htm#pgfId-1103845}Create a simple Bash script to update the
index.html used by the `myapp`{.fm-code-in-text}
container,[]{#04.htm#marker-1103844} called html/time.sh. You can create
it in the ./html directory, so it will be available to processes within
the pod:

``` programlisting
$ cat > html/time.sh << _EOL
#!/bin/sh
data() {
  echo "<html><head></head><body><h1>"; date;echo "Hello World</h1></body></html>"
  sleep 1
}
while true; do
   data > index.html
done
_EOL
```

[]{#04.htm#pgfId-1103857}Make sure it is executable. You can do this on
Linux with the `chmod`{.fm-code-in-text}
command[]{#04.htm#marker-1103856}:

``` programlisting
$ chmod +x html/time.sh
```

[]{#04.htm#pgfId-1103859}Now create the second container
(`--name`{.fm-code-in-text} `time`{.fm-code-in-text}), this time using a
different image: ubi8. Containers within pods can use totally different
images, even images from different distributions. Recall that container
images only share the host kernel by default:

``` programlisting
$ podman create --pod mypod --name time --workdir /var/www/html ubi8 ./time.sh
Resolved "ubi8" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull registry.access.redhat.com/ubi8:latest...
...
1be0b2fae53029d518e75def71c0d6961b662d0e8b4a1082edea5589d1353af3
```

[]{#04.htm#pgfId-1103865}Remember the concept of short names covered in
chapter 2. You can type the long name, registry.access.redhat.com/ubi8,
but that is a lot of typing. Luckily for us, the short name, ubi8,
already had an alias map to its long name, meaning you do not need to
select it from the list of registries. Podman shows you where it found
the alias for the long name in the output:

``` programlisting
$ podman create --pod mypod --name time --workdir /var/www/html ubi8 ./time.sh
Resolved "ubi8" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
```

[]{#04.htm#pgfId-1103869}You also used the `--workdir`{.fm-code-in-text}
command option[]{#04.htm#marker-1104510} to set the default directory
for the container to /var/www/html. When the container starts, the
./time.sh will run in the `workdir`{.fm-code-in-text} and is actually
/var/www/html/time.sh (see figure 4.7):

``` programlisting
$ podman create --pod mypod --name time --workdir /var/www/html ubi8 ./time.sh
```

[]{#04.htm#marker-1104888}[]{#04.htm#pgfId-1103872}Because this
container will be run within the `mypod`{.fm-code-in-text}
pod[]{#04.htm#marker-1103871}, it will inherit the
`-v`{.fm-code-in-text} ./html:/var/www/html option from the pod, meaning
the ./html/time.sh command in the host directory is available to every
container within the pod.

::: figure
![](images/OEBPS/Images/04-07.png){.calibre18}

[]{#04.htm#pgfId-1109296}[]{#04.htm#id_bsufq5aiah2b}Figure 4.7 Finally,
Podman launches the sidecar container named
`time`{.fm-code-in-text}[]{#04.htm#marker-1109297}.
:::

[]{#04.htm#pgfId-1103880}Podman examines the infra container, mounts the
/var/www/html volume, and joins the namespaces when it launches the
sidecar container. Now it is time to start the pod and see what
[]{#04.htm#marker-1103881}[]{#04.htm#marker-1103882}happens.

## []{#04.htm#pgfId-1103884}4.4 Starting a pod {#04.htm#heading_id_6 .fm-head}

[]{#04.htm#pgfId-1103888}You
[]{#04.htm#marker-1103885}[]{#04.htm#marker-1103886}can start the pod
with the `podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`start`{.fm-code-in-text} command[]{#04.htm#marker-1103887}:

``` programlisting
$ podman pod start mypod
790fefe97b280e5f67c526e3a421e9c9f958cf5a98f3709373ef1afd91965955
```

[]{#04.htm#pgfId-1103892}Use the `podman`{.fm-code-in-text}
`ps`{.fm-code-in-text} command[]{#04.htm#marker-1103891} to see which
containers the pod started:

``` programlisting
$ podman ps
CONTAINER ID  IMAGE                  COMMAND             CREATED       STATUS          PORTS                 NAMES
b9536ea4a8ab localhost/podman-pause:4.0.3-1648837314                      14 minutes ago  Up 5 seconds ago  0.0.0.0:8080->8080/tcp  8920b1ccd8b0-infra
a978e0005273  quay.io/rhatdan/myimage:latest        /usr/bin/run-http...  14 minutes ago  Up 5 seconds ago  0.0.0.0:8080->8080/tcp  myapp
be86937986e9  registry.access.redhat.com/ubi8:latest  ./time.sh           13 minutes ago  Up 5 seconds ago  0.0.0.0:8080->8080/tcp  time
```

[]{#04.htm#pgfId-1103898}Notice now that three containers have started.
The infra container is based on the k8s.gcr.io/pause image, your
application is based on quay.io/rhatdan/myimage:latest, and the update
container is based on the registry.access.redhat.com/ubi8:latest image.

[]{#04.htm#pgfId-1103899}When the ubi8 sidecar container starts, it
begins modifying the index.html via the time.sh script. Since the
`myapp`{.fm-code-in-text} container[]{#04.htm#marker-1103900} shares the
volume mount, /var/www/html, it can see the changes in the
/var/www/html/index.html file. Launch your favorite web browser, and
navigate to http://localhost:8080 to verify the application is working,
as seen in figure 4.8.

::: figure
![](images/OEBPS/Images/04-08.png){.calibre18}

[]{#04.htm#pgfId-1109339}[]{#04.htm#id_v0n4sa5bdmic}Figure 4.8 The web
browser communicates with `myapp`{.fm-code-in-text} running in a pod.
:::

[]{#04.htm#pgfId-1103913}A couple of seconds later, press the Refresh
button. Notice the date changes, indicating the sidecar container is
running and updating the data used by the `myapp`{.fm-code-in-text} web
server running within the primary container, as seen in figure 4.9.

::: figure
![](images/OEBPS/Images/04-09.png){.calibre18}

[]{#04.htm#pgfId-1109375}[]{#04.htm#id_h58wqqsm5erb}Figure 4.9 The web
browser shows that the content in `myapp`{.fm-code-in-text} has been
changed by the second container running in the pod.
:::

[]{#04.htm#pgfId-1103915}[]{#04.htm#id_vjrr3xilz1pl1}Some notable
`podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`start`{.fm-code-in-text} options include the following:

-   []{#04.htm#pgfId-1103917
    .calibre17}`--all`{.fm-code-in-text}[]{#04.htm#marker-1103916
    .calibre17}---This tells Podman to start all pods.

-   []{#04.htm#pgfId-1103919
    .calibre17}`--latest`{.fm-code-in-text}[]{#04.htm#marker-1103918
    .calibre17}---The `-l`{.fm-code-in-text} tells Podman to start the
    last pod created. (This is not available on Mac and Windows.)

[]{#04.htm#pgfId-1103920}Now that you have run the application within a
pod, you might want to stop the
[]{#04.htm#marker-1103921}[]{#04.htm#marker-1103922}application.

## []{#04.htm#pgfId-1103924}4.5 Stopping a pod {#04.htm#heading_id_7 .fm-head}

[]{#04.htm#pgfId-1103927}Now
[]{#04.htm#marker-1103925}[]{#04.htm#marker-1103926}that you see the
application ran successfully, you can stop the pod with the
`podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`stop`{.fm-code-in-text} command[]{#04.htm#marker-1103928}, as follows:

``` programlisting
$ podman pod stop mypod
790fefe97b280e5f67c526e3a421e9c9f958cf5a98f3709373ef1afd91965955
```

[]{#04.htm#pgfId-1103933}Use the `podman`{.fm-code-in-text}
`ps`{.fm-code-in-text} command[]{#04.htm#marker-1103932} to make sure
Podman stopped all the containers within the pod:

``` programlisting
$ podman ps
CONTAINER ID  IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

[]{#04.htm#pgfId-1103937}[]{#04.htm#id_o6adp572edcj1}Some notable
`podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`stop`{.fm-code-in-text} options include the following:

-   []{#04.htm#pgfId-1103939
    .calibre17}`--all`{.fm-code-in-text}[]{#04.htm#marker-1103938
    .calibre17}---This tells Podman to stop all pods.

-   []{#04.htm#pgfId-1103941
    .calibre17}`--latest`{.fm-code-in-text}[]{#04.htm#marker-1103940
    .calibre17}---The `-l`{.fm-code-in-text} tells Podman to stop the
    most recently started pod.

-   []{#04.htm#pgfId-1103943
    .calibre17}`--timeout`{.fm-code-in-text}[]{#04.htm#marker-1103942
    .calibre17}---The `-t`{.fm-code-in-text} tells Podman to set the
    timeout when attempting to stop the containers within a pod.

[]{#04.htm#pgfId-1103944}Now that you have created, run, and stopped the
pod, you can start examining it. First, list all the pods on your
[]{#04.htm#marker-1103945}[]{#04.htm#marker-1103946}system.

## []{#04.htm#pgfId-1103948}4.6 Listing pods {#04.htm#heading_id_8 .fm-head}

[]{#04.htm#pgfId-1103952}You
[]{#04.htm#marker-1103949}[]{#04.htm#marker-1103950}can list pods with
the `podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`list`{.fm-code-in-text} command[]{#04.htm#marker-1103951}:

``` programlisting
$ podman pod list
POD ID        NAME      STATUS    CREATED         INFRA ID      # OF CONTAINERS
790fefe97b28  mypod     Exited    22 minutes ago  b9536ea4a8ab  3
```

[]{#04.htm#pgfId-1103957}[]{#04.htm#id_qoinjfin0ayc1}Some notable
`podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`list`{.fm-code-in-text} options include the following:

-   []{#04.htm#pgfId-1103959
    .calibre17}`--ctr*`{.fm-code-in-text}[]{#04.htm#marker-1103958
    .calibre17}---This tells Podman to list container information within
    pods.

-   []{#04.htm#pgfId-1103961
    .calibre17}`--format`{.fm-code-in-text}[]{#04.htm#marker-1103960
    .calibre17}---This tells Podman to change the output of pods.

[]{#04.htm#pgfId-1103964}Now that you are done with the demonstration,
time to clean up the pods and
[]{#04.htm#marker-1103962}[]{#04.htm#marker-1103963}containers.

## []{#04.htm#pgfId-1103966}4.7 Removing pods {#04.htm#heading_id_9 .fm-head}

[]{#04.htm#pgfId-1103969}In
[]{#04.htm#marker-1103967}[]{#04.htm#marker-1103968}a chapter 8, I
discuss how you can generate Kubernetes YAML files to allow you to
launch your pod on other systems using Podman or within Kubernetes. But
for now, you can remove a pod with the `podman`{.fm-code-in-text}
`pod`{.fm-code-in-text} `rm`{.fm-code-in-text}
command[]{#04.htm#marker-1103970}.

[]{#04.htm#pgfId-1103972}Before you do this, list
`--all`{.fm-code-in-text} the containers on the system. Using the
`--format`{.fm-code-in-text} option[]{#04.htm#marker-1103971} to show
only the ID, image, and pod ID, you will see three containers that make
up your pod:

``` programlisting
$ podman ps --all --format "{{.ID}}  {{.Image}} {{.Pod}}"
b9536ea4a8ab  k8s.gcr.io/pause:3.5 790fefe97b28
a978e0005273  quay.io/rhatdan/myimage:latest 790fefe97b28
be86937986e9  registry.access.redhat.com/ubi8:latest 790fefe97b28
```

[]{#04.htm#pgfId-1103977}Now you can remove the pod with the following
command:

``` programlisting
$ podman pod rm mypod
790fefe97b280e5f67c526e3a421e9c9f958cf5a98f3709373ef1afd91965955
```

[]{#04.htm#pgfId-1103980}Make sure it is gone:

``` programlisting
$ podman pod ls
POD ID    NAME      STATUS    CREATED   INFRA ID  # OF CONTAINERS
```

[]{#04.htm#pgfId-1103983}Good! It looks like your pod is gone. Verify
that Podman removed all of the containers by running the following
command:

``` programlisting
$ podman ps -a --format "{{.ID}} {{.Image}}"
```

[]{#04.htm#pgfId-1103985}The system is fully cleaned up.

[]{#04.htm#pgfId-1103990}[]{#04.htm#id_x5vr0icnbkss}Some notable
`podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`rm`{.fm-code-in-text} options
include[]{#04.htm#marker-1103987}[]{#04.htm#marker-1103988}
t[]{#04.htm#marker-1103989}he following (also see table 4.1):

-   []{#04.htm#pgfId-1103992
    .calibre17}`--all`{.fm-code-in-text}[]{#04.htm#marker-1103991
    .calibre17}---This tells Podman to remove all the pods.

-   []{#04.htm#pgfId-1103994
    .calibre17}`--force`{.fm-code-in-text}[]{#04.htm#marker-1103993
    .calibre17}---This tells Podman to first stop all running containers
    before attempting to remove them. Otherwise, Podman will only remove
    non-running pods.[]{#04.htm#id_n2kx94l968 .calibre17}

[]{#04.htm#pgfId-1107258}Table 4.1 `podman`{.fm-code-in-text}
`pod`{.fm-code-in-text} commands

+---------+------------------------+-----------------------------------+
| []{#    | []{#04                 | []{                               |
| 04.htm# | .htm#pgfId-1107266}Man | #04.htm#pgfId-1107268}Description |
| pgfId-1 | page                   |                                   |
| 107264} |                        |                                   |
| Command |                        |                                   |
+---------+------------------------+-----------------------------------+
| [       | []{#04.htm#pgfId-11072 | []{#04.htm#pgfId-1107274}Create a |
| ]{#04.h | 72}`podman-pod-create( | new pod.                          |
| tm#pgfI | 1)`{.fm-code-in-text1} |                                   |
| d-11072 |                        |                                   |
| 70}`cre |                        |                                   |
| ate`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107359} |                        |                                   |
+---------+------------------------+-----------------------------------+
| [       | []{#04.htm#pgfId-11072 | []{#04.htm#pgfId-1107280}Check if |
| ]{#04.h | 78}`podman-pod-exists( | a pod exists.                     |
| tm#pgfI | 1)`{.fm-code-in-text1} |                                   |
| d-11072 |                        |                                   |
| 76}`exi |                        |                                   |
| sts`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107360} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []      | [                      | []{#04.htm#pgfId-1107286}Display  |
| {#04.ht | ]{#04.htm#pgfId-110728 | detailed information on a pod.    |
| m#pgfId | 4}`podman-pod-inspect( |                                   |
| -110728 | 1)`{.fm-code-in-text1} |                                   |
| 2}`insp |                        |                                   |
| ect`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107361} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#04  | []{#04.htm#pgfId-110   | []{#04.htm#pgfId-1107292}Send a   |
| .htm#pg | 7290}`podman-pod-kill( | signal to the primary processes   |
| fId-110 | 1)`{.fm-code-in-text1} | of the containers in the pod.     |
| 7288}`k |                        |                                   |
| ill`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107362} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#04  | []{#04.htm#pgfId-110   | []{#04.htm#pgfId-1107298}List all |
| .htm#pg | 7296}`podman-pod-list( | of the pods.                      |
| fId-110 | 1)`{.fm-code-in-text1} |                                   |
| 7294}`l |                        |                                   |
| ist`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107363} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#04  | []{#04.htm#pgfId-110   | []{#04.htm#pgfId-1107304}Fetch    |
| .htm#pg | 7302}`podman-pod-logs( | logs for the pod with one or more |
| fId-110 | 1)`{.fm-code-in-text1} | containers.                       |
| 7300}`l |                        |                                   |
| ogs`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107364} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#04. | []{#04.htm#pgfId-1107  | []{#04.htm#pgfId-1107310}Pause    |
| htm#pgf | 308}`podman-pod-pause( | all the containers in a pod.      |
| Id-1107 | 1)`{.fm-code-in-text1} |                                   |
| 306}`pa |                        |                                   |
| use`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107365} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#04. | []{#04.htm#pgfId-1107  | []{#04.htm#pgfId-1107316}Remove   |
| htm#pgf | 314}`podman-pod-prune( | all stopped pods and their        |
| Id-1107 | 1)`{.fm-code-in-text1} | containers.                       |
| 312}`pr |                        |                                   |
| une`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107366} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []      | [                      | []{#04.htm#pgfId-1107322}Restart  |
| {#04.ht | ]{#04.htm#pgfId-110732 | a pod.                            |
| m#pgfId | 0}`podman-pod-restart( |                                   |
| -110731 | 1)`{.fm-code-in-text1} |                                   |
| 8}`rest |                        |                                   |
| art`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107367} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#    | []{#04.htm#pgfId-1     | []{#04.htm#pgfId-1107328}Remove   |
| 04.htm# | 107326}`podman-pod-rm( | one or more pods.                 |
| pgfId-1 | 1)`{.fm-code-in-text1} |                                   |
| 107324} |                        |                                   |
| `rm`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107368} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#04. | []{#04.htm#pgfId-1107  | []{#04.htm#pgfId-1107334}Display  |
| htm#pgf | 332}`podman-pod-stats( | statistics for the containers in  |
| Id-1107 | 1)`{.fm-code-in-text1} | a pod.                            |
| 330}`st |                        |                                   |
| ats`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107369} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#04. | []{#04.htm#pgfId-1107  | []{#04.htm#pgfId-1107340}Start a  |
| htm#pgf | 338}`podman-pod-start( | pod.                              |
| Id-1107 | 1)`{.fm-code-in-text1} |                                   |
| 336}`st |                        |                                   |
| art`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107370} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#04  | []{#04.htm#pgfId-110   | []{#04.htm#pgfId-1107346}Stop a   |
| .htm#pg | 7344}`podman-pod-stop( | pod.                              |
| fId-110 | 1)`{.fm-code-in-text1} |                                   |
| 7342}`s |                        |                                   |
| top`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107371} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []{#0   | []{#04.htm#pgfId-11    | []{#04.htm#pgfId-1107352}Display  |
| 4.htm#p | 07350}`podman-pod-top( | running process in the pod.       |
| gfId-11 | 1)`{.fm-code-in-text1} |                                   |
| 07348}` |                        |                                   |
| top`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107372} |                        |                                   |
+---------+------------------------+-----------------------------------+
| []      | [                      | []{#04.htm#pgfId-1107358}Unpause  |
| {#04.ht | ]{#04.htm#pgfId-110735 | all the containers in a pod.      |
| m#pgfId | 6}`podman-pod-unpause( |                                   |
| -110735 | 1)`{.fm-code-in-text1} |                                   |
| 4}`unpa |                        |                                   |
| use`{.f |                        |                                   |
| m-code- |                        |                                   |
| in-text |                        |                                   |
| 1}[]{#0 |                        |                                   |
| 4.htm#m |                        |                                   |
| arker-1 |                        |                                   |
| 107373} |                        |                                   |
+---------+------------------------+-----------------------------------+

## []{#04.htm#pgfId-1104112}Summary {#04.htm#heading_id_10 .fm-head}

-   []{#04.htm#pgfId-1104113 .calibre17}Pods are a way of grouping
    containers together into more complex applications, sharing
    namespaces, and sharing resource constraints.

-   []{#04.htm#pgfId-1104114 .calibre17}Pods share most of the options
    containers use, and when you add a container to a pod, it shares
    these options with all containers in the []{#04.htm#marker-1104115
    .calibre17}pod.

[]{#p2.htm}

## Part 2. Design

[]{#p2.htm#pgfId-1015384}[P]{.fm-part-initial-cap}art 2 of the book
covers the underlying design of Podman. Chapter 5 explains all of the
different configuration files used with Podman. Podman is developed
using multiple different container libraries, each with a distinct
method of configuration. You learn how to configure your container
storage and where to store your containers as well as images. You also
learn how to configure the container registries you use for pulling and
pushing container images. Finally, you learn about containers.conf,
which allows you to fully customize the way Podman works. Basically, you
can change the default values used by the Podman CLI for every container
you create.

[]{#p2.htm#pgfId-1015385}Chapter 6 then takes a deep dive into how
rootless containers work. Rootless containers are a key feature of
Podman that allows you to fully work with containers and pods as a
normal user, without any additional privileges. This chapter also
introduces you to how the user namespace works and allows you to use
more than a single UID within a container, without being root. Finally,
you will learn some of the problems with rootless containers and how to
work around them. []{#p2.htm#id_u6n877dddx1z}[]{#p2.htm#id_7dubk56duarh}

[]{#05.htm}
