# 8 Working with Kubernetes

[]{#08.htm#pgfId-1118670}This chapter []{#08.htm#marker-1119983}covers

-   []{#08.htm#pgfId-1118671 .calibre17}Creating Kubernetes YAML files
    from existing Podman pods and containers
-   []{#08.htm#pgfId-1118672 .calibre17}Creating Podman containers and
    pods from a Kubernetes YAML file
-   []{#08.htm#pgfId-1118673 .calibre17}Shutting down and removing pods
    and containers using the Kubernetes YAML file
-   []{#08.htm#pgfId-1118674 .calibre17}Building container images on the
    fly before launching pods and containers from a Kubernetes YAML file
-   []{#08.htm#pgfId-1118675 .calibre17}Running Podman inside of a
    Podman and Kubernetes container

[]{#08.htm#pgfId-1118676}Some readers come to this chapter expecting to
see how Podman can be used as the container engine for Kubernetes,
similar to how it has used Docker in the past. While there have been
some efforts to use Podman as the container engine for Podman (the kind
project supports this), I do not generally recommend that you use Podman
for this purpose. I recommend you use CRI-O described in appendix A,
since it was built specifically to work with Kubernetes and shares the
underlying libraries of Podman. Kubernetes is now discouraging users
from using the Docker backend and encouraging them to use a CRI-O or
containerd as a backend.

[]{#08.htm#pgfId-1118677}This chapter covers using the same structured
language with both Kubernetes and Podman as well as how to run Podman
containers inside a Kubernetes cluster. You have learned how to create
microservices as containers and pods from the command line using Podman.
Often software developers and packagers need to take their applications
and run them on multiple machines. You might want to take your web
application and add a database backend. If the web application becomes
popular, you will need to run multiple instances on different nodes.
Wiring different microservices together and orchestrating all of them
together is not something Podman does. This is where Kubernetes comes
in.

[]{#08.htm#pgfId-1118678}In this chapter, you will learn about running
these same containers and pods in Kubernetes. The kubernetes.io website
says, "Kubernetes, also known as K8s, is an open-source system for
automating deployment, scaling, and management of containerized
applications." I look at Kubernetes as the tool for running containers
on multiple machines at the same time---a way to orchestrate large
clusters of containerized microservices.

[]{#08.htm#pgfId-1118680}One problem you may encounter is that most
container development happens with tools like Podman and Docker, which
use fairly simple command-line interfaces to create containers and pods.
But Kubernetes uses a declarative language written in YAML files.

[]{#08.htm#pgfId-1118681}I will not be diving deep into how Kubernetes
works in this chapter because there are already many in-depth books on
the topic, including *Kubernetes in Action* by Marko
Lukša[]{#08.htm#marker-1118684}[]{#08.htm#marker-1118685} (Manning,
2020) and *Kubernetes for Developers*[]{#08.htm#marker-1118687} by
William Denniss (Manning, 2020), that describe all the features of
Kubernetes. But I will be describing the developer language of
Kubernetes: the Kubernetes YAML file.

[]{#08.htm#pgfId-1118690}[Note]{.fm-callout-head} The yaml.org website
first describes YAML as the "YAML Ain't Markup Language." It further
elaborates, "YAML is a human-friendly data serialization language for
all programming languages."

[]{#08.htm#pgfId-1118691}Translating command-line options to a
structured language like YAML presents a barrier for developers moving
from containers on a single node to containers running at scale. How do
you specify volumes, the image to be used, the security constraints, the
network ports, and so on? In section 8.2, you will learn to use Podman
to take your locally created pods and containers to generate Kubernetes
YAML files from them.

[]{#08.htm#pgfId-1118692}After writing and deploying your application in
a pod using Kubernetes YAML files, users are likely to find problems
with your application running within Kubernetes. Testing the application
at scale can be difficult, and often you just want to run the
application locally on your system, without having to set up and
configure a Kubernetes cluster. In section 8.3, you will learn about
`podman`{.fm-code-in-text} `play`{.fm-code-in-text}
`kube`{.fm-code-in-text}. This Podman command allows you to run the
Kubernetes YAML file locally, without Kubernetes, so you can test and
debug problems.

[]{#08.htm#pgfId-1118693}The final part of this chapter will cover
running Podman within containers, including running it within a
Kubernetes cluster. Administrators, developers, and quality engineers
need to test containers within their continuous integration
(CI[]{#08.htm#marker-1118694}) systems using Podman. Often these CI
systems are built on Kubernetes clusters. Section 8.4 teaches you
different ways to run the Podman command within containers launched by
Podman and Kubernetes.

## []{#08.htm#pgfId-1118696}8.1 Kubernetes YAML files {#08.htm#heading_id_3 .fm-head}

[]{#08.htm#pgfId-1118698}The []{#08.htm#marker-1118697}Kubernetes YAML
file is the object used to launch pods and containers within Kubernetes.
In chapter 5, you learned the configuration files used by Podman are
written using TOML, which is very similar to YAML. Both configuration
languages are attempting to be human readable. YAML relies on indenting
substanzas, which is different syntax than you learned with TOML. You
can go to the [yaml.org](http://yaml.org){.url} website to learn more
about the language.

[]{#08.htm#pgfId-1118700}If you are going to work a lot with Kubernetes
YAML files, it is nice to have a text editor or IDE, like Visual Studio
and VS Code, that can at least understand YAML; it is even better if it
knows the Kubernetes language. Kubernetes YAML is descriptive and
powerful. It allows you to model the desired state of your application
in a declarative language. As stated in the introduction to this
chapter, writing these YAML files is a barrier for developers to get
through when moving their containers from a local system to Kubernetes.
Most developers just web search an existing Kubernetes YAML file and
then begin cutting and pasting their container command, image, and
options into the YAML file. While this works, it can lead to unintended
consequences---and often unnecessary work.

[]{#08.htm#pgfId-1118701}Scott McCarty, product manager of Podman,
tossed out an idea: "What I would really like to do is help users get
from Podman to orchestrating their containers with Kubernetes." This led
the Podman developers to create a new Podman command:
[]{#08.htm#marker-1118702}`podman`{.fm-code-in-text}
`generate`{.fm-code-in-text}
`kube`{.fm-code-in-text}[]{#08.htm#marker-1118703}.

## []{#08.htm#pgfId-1118705}8.2 Generating Kubernetes YAML files with Podman {#08.htm#heading_id_4 .fm-head}

[]{#08.htm#pgfId-1118708}Imagine
[]{#08.htm#marker-1118706}[]{#08.htm#marker-1118707}you want to take the
containers you generated in the previous chapters and run them within
Kubernetes. You need to write the Kubernetes YAML file to make this
happen. Where should you start?

[]{#08.htm#pgfId-1118710}In this chapter, you will learn a new command:
`podman`{.fm-code-in-text} `generate`{.fm-code-in-text}
`kube`{.fm-code-in-text}[]{#08.htm#marker-1118709}. This Podman command
captures the description of local pods and containers and then
translates them into Kubernetes YAML. This helps you transition to a
more sophisticated orchestration environment like Kubernetes. The
generated Kubernetes YAML file can then be used by Kubernetes commands
to launch your pods and containers into a Kubernetes cluster.

[]{#08.htm#pgfId-1118711}You can re-create the containers or pods
locally using Podman on the command line with the same Podman
`run`{.fm-code-in-text}[]{#08.htm#marker-1118712},
`create`{.fm-code-in-text}[]{#08.htm#marker-1118713}, and
`stop`{.fm-code-in-text} commands[]{#08.htm#marker-1118714} you have
learned in the previous chapters. Using the following commands,
re-create the container you have been working with.

[]{#08.htm#marker-1121704}[]{#08.htm#pgfId-1118715}First, remove the
container if it exists using `podman`{.fm-code-in-text}
`rm`{.fm-code-in-text}. You will introduce a new flag,
`--ignore`{.fm-code-in-text}[]{#08.htm#marker-1118716}, which tells the
`podman`{.fm-code-in-text} `rm`{.fm-code-in-text}
command[]{#08.htm#marker-1118717} not to report errors when the
container does not exist. Then, re-create the container from the command
line:

``` programlisting
$ podman rm -f --ignore myapp
$ podman create -p 8080:8080 --name myapp quay.io/rhatdan/myimage
9305822e6089ca28a1fdbb005c12f57f4a26be273fe5d49a1908eadbcfdcb7d4
```

[]{#08.htm#pgfId-1118722}Now, use the command `podman`{.fm-code-in-text}
`generate`{.fm-code-in-text} `kube`{.fm-code-in-text}
`myapp`{.fm-code-in-text}[]{#08.htm#marker-1118721} to generate the
Kubernetes YAML file. Podman inspects the existing container or pod in
its database for all of the fields required to run the container in
Kubernetes and then populates them in the Kubernetes YAML file:

``` programlisting
$ podman generate kube myapp > myapp.yaml
```

[]{#08.htm#pgfId-1118725}Figure 8.1 shows the result of a
`podman`{.fm-code-in-text} `generate`{.fm-code-in-text}
`kube`{.fm-code-in-text} command[]{#08.htm#marker-1118724}.

::: figure
![](images/OEBPS/Images/08-01.png){.calibre18}

[]{#08.htm#pgfId-1123535}Figure 8.1 Shows the generated myapp.yaml file
from the `myapp`{.fm-code-in-text} container[]{#08.htm#marker-1123534}
:::

[]{#08.htm#pgfId-1118733}Examine parts of the YAML file. Understand that
Kubernetes works with pods, even though you created a container,
`podman`{.fm-code-in-text} `generate`{.fm-code-in-text}
`kube`{.fm-code-in-text}, that creates a pod specification. Podman names
the pod `myapp-pod`{.fm-code-in-text}[]{#08.htm#marker-1118734} and the
container `myapp`{.fm-code-in-text}[]{#08.htm#marker-1118735} within the
specification, based on the name of the original container:

``` programlisting
metadata:
  creationTimestamp: "2021-11-22T11:57:12Z"
  labels:
    app: myapppod
  name: myapp-pod
spec:
  containers:
  - args:
    - /usr/bin/run-httpd
    image: quay.io/rhatdan/myimage:latest
    name: myapp
```

[]{#08.htm#pgfId-1118747}Notice, in the containers section, that the
image name, quay.io/rhatdan/myimage: latest, is recorded, which tells
Kubernetes where to download the image for the container from. It also
tells Kubernetes the command arguments to start the app within the
container, /usr/bin/run-httpd:

``` programlisting
spec:  
  containers:
  - args:
    - /usr/bin/run-httpd
    image: quay.io/rhatdan/myimage:latest
```

[]{#08.htm#pgfId-1118753}In the same container section, you see that the
Podman ports are recorded, `-p`{.fm-code-in-text}
`8080: 8080`{.fm-code-in-text} `spec`{.fm-code-in-text}:

``` programlisting
  containers:
  - args:
    - /usr/bin/run-httpd
    image: quay.io/rhatdan/myimage:latest
    name: myapp
    ports:
    - containerPort: 8080
    hostPort: 8080
```

[]{#08.htm#pgfId-1118762}Finally, at the end of the containers section,
you see `securityContext`{.fm-code-in-text}, which records that Podman,
by default, drops three additional Linux capabilities:
`CAP_MKNOD`{.fm-code-in-text}[]{#08.htm#marker-1118763},
`CAP_NET_RAW`{.fm-code-in-text}[]{#08.htm#marker-1118764}, and
`CAP_AUDIT_WRITE`{.fm-code-in-text}[]{#08.htm#marker-1118765}:

``` programlisting
    securityContext:
    capabilities:
    drop:
    - CAP_MKNOD
    - CAP_NET_RAW
    - CAP_AUDIT_WRITE
```

[]{#08.htm#pgfId-1118772}Most containers run fine without these Linux
capabilities, but the OCI specification enables these three by default.
This tells Kubernetes that this pod can run more securely without these
capabilities, and Kubernetes will drop them. You can find out more about
Linux capabilities by running the command `man`{.fm-code-in-text}
`capabilities`{.fm-code-in-text}.

[]{#08.htm#pgfId-1118774}At this point, you can just run this Kubernetes
YAML file in any Kubernetes cluster, usually running a command like the
following:

``` programlisting
kubectl create -f myapp.yml
```

[]{#08.htm#pgfId-1118776}Often you will have to add sophistication and
orchestration to the YAML file and leverage advanced functions of
Kubernetes. For example, the generated Kubernetes YAML file will only
generate a single instance of your application. If you want to run
multiple versions of your applications on different nodes, you could add
a `replicas`{.fm-code-in-text} option[]{#08.htm#marker-1118777} to your
YAML file, as seen in figure 8.2.

::: figure
![](images/OEBPS/Images/08-02.png){.calibre18}

[]{#08.htm#pgfId-1123573}Figure 8.2 The modified Kubernetes YAML file
ready to run two replicas
:::

[]{#08.htm#pgfId-1118788}The `replicas`{.fm-code-in-text}
flag[]{#08.htm#marker-1118787} tells Kubernetes that the myapp.yaml file
wants to have two `myapp`{.fm-code-in-text}
pods[]{#08.htm#marker-1118789} running on two different nodes at all
times. Replicas and other advanced Kubernetes features are out of the
scope of Podman. The `podman`{.fm-code-in-text} `play`{.fm-code-in-text}
`kube`{.fm-code-in-text} command[]{#08.htm#marker-1118790} ignores these
fields.

[]{#08.htm#pgfId-1118791}Some notable `podman`{.fm-code-in-text}
`generate`{.fm-code-in-text} `kube`{.fm-code-in-text} options include
the following:

-   []{#08.htm#pgfId-1118792 .calibre17}`-f,`{.fm-code-in-text}
    `--filename`{.fm-code-in-text}---This writes output to the specified
    path.

-   []{#08.htm#pgfId-1118793 .calibre17} `-s,`{.fm-code-in-text}
    `--service`{.fm-code-in-text}---This generates YAML for a Kubernetes
    service object.

[]{#08.htm#pgfId-1118794}Now that you have generated a Kubernetes YAML
file, it'd be nice to be able to reverse the process. If you had a
Kubernetes YAML file, you may want to generate Podman pods and
[]{#08.htm#marker-1118795}[]{#08.htm#marker-1118796}containers.

## []{#08.htm#pgfId-1118798}8.3 Generating Podman pods and containers from Kubernetes YAML {#08.htm#heading_id_5 .fm-head}

[]{#08.htm#pgfId-1118803}Imagine
[]{#08.htm#marker-1118799}[]{#08.htm#marker-1118800}[]{#08.htm#marker-1118801}[]{#08.htm#marker-1118802}you
get a Kubernetes YAML file and want to examine it running locally. You
could set up a local Kubernetes cluster, but it would be nice if you
could just play the pods locally. Podman provides a command for doing
this. The `podman`{.fm-code-in-text} `play`{.fm-code-in-text}
`kube`{.fm-code-in-text} command[]{#08.htm#marker-1118804} creates pods,
containers, and volumes based on structured Kubernetes YAML files. The
created pods and containers are automatically started. To test this, you
can simply remove the container you created and then run the generated
myapp.yaml file with the following commands:

``` programlisting
$ podman rm -f --ignore myapp
$ podman play kube myapp.yaml
Pod:
b70aedd8105a6915428928a2b33fd7ecede632298088ea25d9db74ba9b16201e
Container:
a4d78fdfa5d8f751aafb06f3782e36a3aaf5b3804ca57694385de2ea1e400fe6
```

[]{#08.htm#pgfId-1118811}Kubernetes only runs pods with containers; it
does not run just containers by themselves. When the
`podman`{.fm-code-in-text} `play`{.fm-code-in-text}
`kube`{.fm-code-in-text} command[]{#08.htm#marker-1118812} reads the
YAML, file it launches the pod along with the container. Notice in
figure 8.3 that the `play`{.fm-code-in-text}
command[]{#08.htm#marker-1118813} created a Pod with your container
along with the infra containers.

::: figure
![](images/OEBPS/Images/08-03.png){.calibre18}

[]{#08.htm#pgfId-1123611}Figure 8.3 The `myapp-pod`{.fm-code-in-text}
running with the `myapp`{.fm-code-in-text} container and the infra
container
:::

[]{#08.htm#pgfId-1118822}The `podman`{.fm-code-in-text}
`generate`{.fm-code-in-text} `kube`{.fm-code-in-text}
command[]{#08.htm#marker-1118820} creates the pod named
`myapp-pod`{.fm-code-in-text}[]{#08.htm#marker-1118821}, based on the
name within the myapp.yaml file. The names of the containers are
generated by appending the name of the pod to the name of the container:
`myapp-pod-myapp`{.fm-code-in-text}[]{#08.htm#marker-1118823}. If the
YAML file defines additional containers, they need to be labeled
similarly:

``` programlisting
$ cat myapp.yaml
...
  name: myapp-pod
spec:
  containers:
  - args:
    name: myapp
```

[]{#08.htm#pgfId-1118832}You can display the pods running on your system
with the `podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`ps`{.fm-code-in-text} command[]{#08.htm#marker-1118831}. Add the
`--ctr-names`{.fm-code-in-text} option[]{#08.htm#marker-1118833} to also
list the containers running within the pod:

``` programlisting
$ podman pod ps --ctr-names
POD ID       NAME      STATUS  CREATED   INFRA ID     NAMES
b70aedd8105a myapp-pod Running 1 day ago b7a276c62c1d 
➥ myapp-pod-myapp ,b70aedd8105a-infra
```

[]{#08.htm#pgfId-1118839}Now examine the two containers running with the
`podman`{.fm-code-in-text} `ps`{.fm-code-in-text}
command,[]{#08.htm#marker-1118838} using the following command:

``` programlisting
$ podman ps
CONTAINER ID  IMAGE                         COMMAND             CREATED   
➥ STATUS          PORTS                 NAMES
b7a276c62c1d  k8s.gcr.io/pause:3.5                                
➥ 3 minutes ago  Up 3 minutes ago  0.0.0.0:8080->8080/tcp  b70aedd8105a-infra
a4d78fdfa5d8  quay.io/rhatdan/myimage:latest  /usr/bin/run-http...  
➥ 3 minutes ago  Up 3 minutes ago  0.0.0.0:8080->8080/tcp myapp-pod-myapp
```

[]{#08.htm#pgfId-1118847}Shut down the Pod and container with the
`podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`stop`{.fm-code-in-text} command:

``` programlisting
$ podman pod stop myapp-pod
b70aedd8105a6915428928a2b33fd7ecede632298088ea25d9db74ba9b16201e
```

[]{#08.htm#pgfId-1118850}`podman`{.fm-code-in-text}
`play`{.fm-code-in-text} `kube`{.fm-code-in-text} can execute much more
complex YAML files, including with multiple pods, volumes, and
containers defined. In the previous simple example, you can just shut
down the pod with the `podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`stop`{.fm-code-in-text} command[]{#08.htm#marker-1118851}, but when
`podman`{.fm-code-in-text} `play`{.fm-code-in-text}
`kube`{.fm-code-in-text} generates multiple unique pods, it gets a
little more complex to shut them down.

### []{#08.htm#pgfId-1118853}8.3.1 Shutting down pods and containers based on a Kubernetes YAML file {#08.htm#heading_id_6 .fm-head1}

[]{#08.htm#pgfId-1118859}Although
[]{#08.htm#marker-1118854}[]{#08.htm#marker-1118855}[]{#08.htm#marker-1118856}[]{#08.htm#marker-1118857}[]{#08.htm#marker-1118858}you
can stop each pod started by `podman`{.fm-code-in-text}
`play`{.fm-code-in-text} `kube`{.fm-code-in-text}, sometimes you don't
only want to stop the pods and containers but actually remove them from
the system. The `podman`{.fm-code-in-text} `play`{.fm-code-in-text}
`kube`{.fm-code-in-text} `--down`{.fm-code-in-text}
command[]{#08.htm#marker-1118860} tears down the pods that were created
by a previous run of `play`{.fm-code-in-text} `kube`{.fm-code-in-text}.
The pods are stopped and then removed. Any volumes created are left
intact. Shut down the myapp.yaml pod created in the previous example:

``` programlisting
$ podman play kube myapp.yaml --down
Pods stopped:
B70aedd8105a6915428928a2b33fd7ecede632298088ea25d9db74ba9b16201e
Pods removed:
b70aedd8105a6915428928a2b33fd7ecede632298088ea25d9db74ba9b16201e
```

[]{#08.htm#pgfId-1118866}Notice that Podman not only stopped the pod but
also removed it. You can verify the pod is gone with the
`podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`ps`{.fm-code-in-text} command[]{#08.htm#marker-1118867}:

``` programlisting
$ podman pod ps
POD ID     NAME       STATUS     CREATED    INFRA ID    # OF CONTAINERS
```

[]{#08.htm#pgfId-1118870}This leaves you back in a state where you can
run `podman`{.fm-code-in-text} `play`{.fm-code-in-text}
`kube`{.fm-code-in-text} again, which will create fresh pods and
containers:

``` programlisting
$ podman play kube myapp.yaml
Pod:
302b1d2c0048a49ea32c2e6ffa0e0549af199ab2bc32de285eef5da628efe28c
Container:
b9f080dc6e13b4a4c37fa66a9b727dbeb2af30f0c3824044aba8a46eebfe15c5
```

[]{#08.htm#pgfId-1118876}This mimics what happens with Kubernetes
running pods and containers. Kubernetes always creates pods and
containers fresh and tears them down when it completes. The ability to
generate all of the pods and containers from the YAML file and then
remove them with the `--down`{.fm-code-in-text}
flag[]{#08.htm#marker-1118877} is similar to the workflow of
`docker-compose`{.fm-code-in-text}. Podman has the big advantage of
using the same YAML file for running the pods and containers as in a
multinode, orchestrated environment with Kubernetes. One other feature
`docker-compose`{.fm-code-in-text} has is the ability to build the
images defined within the YAML file, which the Podman developers also
added to
[]{#08.htm#marker-1118878}[]{#08.htm#marker-1118879}[]{#08.htm#marker-1118880}[]{#08.htm#marker-1118881}[]{#08.htm#marker-1118882}`podman`{.fm-code-in-text}
`play`{.fm-code-in-text} `kube`{.fm-code-in-text}.

### []{#08.htm#pgfId-1118884}8.3.2 Building images using Podman and Kubernetes YAML files {#08.htm#heading_id_7 .fm-head1}

[]{#08.htm#pgfId-1118890}Users
[]{#08.htm#marker-1118885}[]{#08.htm#marker-1118886}[]{#08.htm#marker-1118887}[]{#08.htm#marker-1118888}[]{#08.htm#marker-1118889}who
were using `podman`{.fm-code-in-text} `play`{.fm-code-in-text}
`kube`{.fm-code-in-text} as a replacement for
`docker-compose`{.fm-code-in-text} requested Podman to add a feature to
build images, rather than always pull them from a container registry.
While Kubernetes does not support such a feature, Podman developers
decided to add the `--build`{.fm-code-in-text}
flag[]{#08.htm#marker-1118891} to `podman`{.fm-code-in-text}
`play`{.fm-code-in-text} `kube`{.fm-code-in-text}. Because
`podman`{.fm-code-in-text} `build`{.fm-code-in-text} can process
Containerfiles or Dockerfiles, enhancing `podman`{.fm-code-in-text}
`play`{.fm-code-in-text} `kube`{.fm-code-in-text} was simple.

[]{#08.htm#pgfId-1118892}The idea is to create a containerized
application via a container image that is produced on demand. Normal
Kubernetes workflow requires developers to build the image using
`podman`{.fm-code-in-text} `build`{.fm-code-in-text} and push it to a
container registry using `podman`{.fm-code-in-text}
`push`{.fm-code-in-text}, as you learned in chapter 2. Then you can
retrieve the image from the registry using `podman`{.fm-code-in-text}
`play`{.fm-code-in-text} `kube`{.fm-code-in-text}. The
`podman`{.fm-code-in-text} `play`{.fm-code-in-text}
`kube`{.fm-code-in-text} `--build`{.fm-code-in-text} option allows it to
execute `podman`{.fm-code-in-text} `build`{.fm-code-in-text} internally
and generate the image on demand, rather than forcing you to use a
container registry.

[]{#08.htm#pgfId-1118894}[Note]{.fm-callout-head} The
`--build`{.fm-code-in-text1} option[]{#08.htm#marker-1118893} is not
available with the remote Podman client, so you can't use it on Mac or
Windows.

[]{#08.htm#pgfId-1118895}In this example, you are going to re-create the
Containerfile used in section 6.1.3:

``` programlisting
$ cat > ./Containerfile << _EOF
FROM ubi8-init
RUN dnf -y install httpd; dnf -y clean all
RUN systemctl enable httpd.service
_EOF
```

[]{#08.htm#pgfId-1118901}Recall that this Containerfile builds a
container image with systemd running as the init system and the HTTPD
service running and listening on port `80`{.fm-code-in-text}. First,
remove all pods and containers:

``` programlisting
$ podman pod rm --all --force
$ podman rm --all --force
```

[]{#08.htm#pgfId-1118905}Now rebuild the `my-systemd`{.fm-code-in-text}
image[]{#08.htm#marker-1118904}:

``` programlisting
$ podman build -t mysystemd.
STEP 1/3: FROM ubi8-init
STEP 2/3: RUN dnf -y install httpd; dnf -y clean all
Updating Subscription Management repositories.
Unable to read consumer identity
...
Successfully tagged localhost/mysystemd:latest
bb1634ce1457f2eb70f84af33599d211eae64cb5f951e40e91481b6e58b747bf
```

[]{#08.htm#pgfId-1118914}Now re-create a container on the image with the
./html directory (using a code example from section 3.1) mounted into
the container:

``` programlisting
$ podman create --rm -p 8080:80 --name myapp -v ./:/var/www/
➥ html:Z mysystemd
fec6de5716ac246613723a4cc26407005e0bc315affdc62b56883bd94acd795e
```

[]{#08.htm#pgfId-1118918}Now generate the Kubernetes YAML file using
`podman`{.fm-code-in-text} `generate`{.fm-code-in-text}
`kube`{.fm-code-in-text}:

``` programlisting
$ podman generate kube myapp > myapp2.yaml
```

[]{#08.htm#pgfId-1118920}Notice that this time Podman generated the YAML
file with a volumes section for `html`{.fm-code-in-text}:

``` programlisting
$ cat myapp2.yaml
...
spec:
  containers:
  - image: localhost/mysystemd:latest
    ...
    volumeMounts:
    - mountPath: /var/www/html
    name: home-dwalsh-podman-html-host-0
  volumes:
  - hostPath:
    path: /home/dwalsh/podman/html
    type: Directory
    name: home-dwalsh-podman-html-host-0
```

[]{#08.htm#pgfId-1118935}Get back to a fresh environment by removing all
of the pods with the `podman`{.fm-code-in-text} `pod`{.fm-code-in-text}
`rm`{.fm-code-in-text} `--all`{.fm-code-in-text}
`--force`{.fm-code-in-text} command[]{#08.htm#marker-1118936}. Remove
all containers and images using the `podman`{.fm-code-in-text}
`rm`{.fm-code-in-text}[]{#08.htm#marker-1118937} and
`podman`{.fm-code-in-text} `rmi`{.fm-code-in-text}
commands[]{#08.htm#marker-1118938}, so you can start with a clean slate:

``` programlisting
$ podman pod rm --all --force
$ podman rm --all --force
fec6de5716ac246613723a4cc26407005e0bc315affdc62b56883bd94acd795e
$ podman rmi mysystemd
Untagged: localhost/mysystemd:latest
Deleted: bb1634ce1457f2eb70f84af33599d211eae64cb5f951e40e91481b6e58b747bf
Deleted: 70e0c1a7580089420267b5928210ad59fdd555603e647b462159ea94f97946f9
```

[]{#08.htm#pgfId-1118947}The `podman`{.fm-code-in-text}
`play`{.fm-code-in-text} `kube`{.fm-code-in-text}
`--build`{.fm-code-in-text} command[]{#08.htm#marker-1118946} requires
subdirectories matching the image names to exist for images to be built.
Podman examines the Kubernetes YAML file for all images and then looks
for the matching subdirectory. Each directory is treated as a context
directory and should contain a Containerfile or a Dockerfile. Podman
then executes `podman`{.fm-code-in-text} `build`{.fm-code-in-text} on
each subdirectory. Since the YAML file needs the mysystemd image, you
need to create a mysystemd directory and place the Containerfile in the
directory:

``` programlisting
$ mkdir mysystemd
$ mv Containerfile mysystemd/
```

[]{#08.htm#pgfId-1118950}You can now run `podman`{.fm-code-in-text}
`play`{.fm-code-in-text} `kube`{.fm-code-in-text}
`--build`{.fm-code-in-text}, and it will rebuild the container image and
launch the Pod and containers for your application:

``` programlisting
$ podman play kube myapp2.yaml --build
STEP 1/3: FROM ubi8-init
STEP 2/3: RUN dnf -y install httpd; dnf -y clean all
Updating Subscription Management repositories.
...
--> 305bb9b8da1
Successfully tagged localhost/mysystemd:latest
305bb9b8da12db682b0eae93ad492e632d2ba43e03f6a6b68467d7429a8a2664
a container exists with the same name ("myapp") as the pod in your YAML file;
➥ changing podname to myapp-pod
Pod:
30739dd554acfeab66a9767301127bab0fe994461686f45a3a89b137c3954840
Container:
ce633ac4e7a1e4d08e0428a8401fcfc4ac75fbcca4be07bc167add6093a44afa
```

[]{#08.htm#pgfId-1118965}Podman rebuilt the mysystemd image based on the
mysystemd/Containerfile and then generated the
`myapp-pod`{.fm-code-in-text} pod[]{#08.htm#marker-1118966} and the
`myapp`{.fm-code-in-text} container[]{#08.htm#marker-1118967} for your
application, without even reaching out to a container registry.

[]{#08.htm#pgfId-1118968}You can share this YAML file and the mysystemd
directory with other users, and they can build and launch your
application all with Podman. Remember, though, if they wanted to launch
it inside of Kubernetes, you need to push the built image to a container
registry, and then edit the YAML file to point the image to the registry
image. Now that you have been exposed to the integration of Podman with
Kubernetes, I want to explore one last idea: running Podman within
Podman and
[]{#08.htm#marker-1118969}[]{#08.htm#marker-1118970}[]{#08.htm#marker-1118971}[]{#08.htm#marker-1118972}[]{#08.htm#marker-1118973}Kubernetes
[]{#08.htm#marker-1118974}[]{#08.htm#marker-1118975}[]{#08.htm#marker-1118976}[]{#08.htm#marker-1118977}containers.

## []{#08.htm#pgfId-1118979}8.4 Running Podman within a container {#08.htm#heading_id_8 .fm-head}

[]{#08.htm#pgfId-1118982}Running
[]{#08.htm#marker-1118980}[]{#08.htm#marker-1118981}Podman within a
container, or within a Kubernetes cluster, is a common problem. Users
want to be able to test container images and tools within CI/CD systems
using containers. Often, they want to build container images with
`podman`{.fm-code-in-text} `build`{.fm-code-in-text}. Sometimes, they
just want to test a newer version of Podman than has been released
within their distribution.

[]{#08.htm#pgfId-1118983}One challenge with Podman is that it can be
configured in so many different ways that users were looking for best
practices for running Podman within a container. Because of this I,
along with some of my colleagues, decided to create a container image,
quay.io/podman`/`{.fm-code-in-text}stable, which makes it easier to run
Podman within a container. As you understand, Podman can run in two
different modes: rootful and rootless. By default, Podman containers
start as the container root within their user namespace. To help you
understand running Podman within a container, you will first experiment
with running Podman within Podman. Table 8.1 describes the different
ways you can run a container within a container and the capabilities
required to allow the internal Podman to execute a container.

[]{#08.htm#pgfId-1121343}Table 8.1 Requirements for running Podman
within a container

+-------------+-------------+-------------+---------------------------+
| []{#08.     | [           | []{#        | []{#08.htm#               |
| htm#pgfId-1 | ]{#08.htm#p | 08.htm#pgfI | pgfId-1121357}Explanation |
| 121351}Host | gfId-112135 | d-1121355}C |                           |
| mode        | 3}Container | apabilities |                           |
|             | mode        |             |                           |
+-------------+-------------+-------------+---------------------------+
| []{#08.htm  | []{#08.htm  | []{#        | []{                       |
| #pgfId-1121 | #pgfId-1121 | 08.htm#pgfI | #08.htm#pgfId-1121365}Has |
| 359}Rootful | 361}Rootful | d-1121363}` | full access to the host   |
|             |             | CAP_SYS_ADM | user's namespace          |
|             |             | IN`{.fm-cod |                           |
|             |             | e-in-text1} |                           |
+-------------+-------------+-------------+---------------------------+
| []{#08.htm  | []{#08.htm# | []{#08.htm# | []{#                      |
| #pgfId-1121 | pgfId-11213 | pgfId-11213 | 08.htm#pgfId-1121373}Runs |
| 367}Rootful | 69}Rootless | 71}`CAP_SET | in a separate user's      |
|             |             | UIDCAP_SETG | namespace based on        |
|             |             | ID`{.fm-cod | /etc/subuid and           |
|             |             | e-in-text1} | /etc/subgid inside the    |
|             |             |             | container                 |
+-------------+-------------+-------------+---------------------------+
| []{#08.htm# | []{#08.htm  | []          | []{                       |
| pgfId-11213 | #pgfId-1121 | {#08.htm#pg | #08.htm#pgfId-1121381}Has |
| 75}Rootless | 377}Rootful | fId-1121379 | full access to the user's |
|             |             | }Namespaced | user namespace            |
|             |             | `           |                           |
|             |             | CAP_SYS_ADM |                           |
|             |             | IN`{.fm-cod |                           |
|             |             | e-in-text1} |                           |
+-------------+-------------+-------------+---------------------------+
| []{#08.htm# | []{#08.htm# | []          | []{#                      |
| pgfId-11213 | pgfId-11213 | {#08.htm#pg | 08.htm#pgfId-1121389}Runs |
| 83}Rootless | 85}Rootless | fId-1121387 | in a separate user        |
|             |             | }Namespaced | namespace based on        |
|             |             | `CAP_SETUI  | /etc/subuid and           |
|             |             | D`{.fm-code | /etc/subgid inside the    |
|             |             | -in-text1}, | container. The user       |
|             |             | `CAP_SETG   | namespace must be a       |
|             |             | ID`{.fm-cod | subset of the user        |
|             |             | e-in-text1} | namespace in which you    |
|             |             |             | are running the Podman    |
|             |             |             | command.                  |
+-------------+-------------+-------------+---------------------------+

### []{#08.htm#pgfId-1119032}8.4.1 Running Podman within a Podman container {#08.htm#heading_id_9 .fm-head1}

[]{#08.htm#pgfId-1119035}In
[]{#08.htm#marker-1121493}[]{#08.htm#marker-1121494}the first example,
you will run a rootful Podman within a rootless container. You need to
use the `--privileged`{.fm-code-in-text}
command[]{#08.htm#marker-1121496} because, to run successfully, Podman
needs to be able to mount filesystems. When Podman is run as root,
mounting requires the `CAP_SYS_ADMIN`{.fm-code-in-text}
capability[]{#08.htm#marker-1121497}, which is given by the
`--privileged`{.fm-code-in-text} option[]{#08.htm#marker-1121498}. Try
it out by executing the following command:

``` programlisting
$ podman run --privileged quay.io/podman/stable podman version
Trying to pull quay.io/podman/stable:latest...
Getting image source signatures
Copying blob b1f89b7294d7 done 
...
Version:    4.1.0
API Version:     4.1.0
Go Version:      go1.18.2
Built:      Mon May 30 12:03:28 2022
OS/Arch:    linux/amd64
```

[]{#08.htm#pgfId-1119049}The quay.io/podman/stable image is also
configured to run a rootless Podman within a Podman container. You can
activate this behavior by adding running as the Podman user with the
`--user`{.fm-code-in-text} `podman`{.fm-code-in-text}
option[]{#08.htm#marker-1119050}. In this mode, Podman within the
container needs `CAP_SETUID`{.fm-code-in-text} and
`CAP_SETGID`{.fm-code-in-text} to set up the user namespace. Luckily,
Podman gives this access to containers by default:

``` programlisting
$ podman run --user podman quay.io/podman/stable podman version
```

[]{#08.htm#pgfId-1119052}If you really want to lock the container down,
you can drop all capabilities other than `CAP_SETUID`{.fm-code-in-text}
and `CAP_SETGID`{.fm-code-in-text}, using the
`--cap-drop=all`{.fm-code-in-text} `--cap-add`{.fm-code-in-text}
`CAP_SETUID,CAP_ SETGID`{.fm-code-in-text}
options[]{#08.htm#marker-1122583}:

``` programlisting
$ podman run --cap-drop=all --cap-add CAP_SETUID,CAP_SETGID 
➥ --user podman quay.io/podman/stable podman version
Version:    4.1.0
API Version:     4.1.0
Go Version:      go1.18.2
Built:      Mon May 30 12:03:28 2022
OS/Arch:    linux/amd64
```

[]{#08.htm#pgfId-1119061}These examples, which show how you can run
Podman within a Podman container, can also easily be done with Docker
running Podman within a container.

[]{#08.htm#pgfId-1119063}Note that Docker runs with a seccomp
filter[]{#08.htm#marker-1119062}, which blocks the unshare and mount
system calls. You need to either disable seccomp filtering in Docker---

``` programlisting
docker run –security-opt seccomp=unconfined ...
```

[]{#08.htm#pgfId-1119066}---or run Docker with Podman's seccomp
filters[]{#08.htm#marker-1119065}:

``` programlisting
docker run –security-opt seccomp=/usr/share/containers/seccomp.json ... .
```

[]{#08.htm#pgfId-1119068}In this section, you learned about Podman
integration with Kubernetes. In the next section, you will learn how to
configure Podman to run within a Kubernetes pod or
[]{#08.htm#marker-1119069}[]{#08.htm#marker-1119070}container.

### []{#08.htm#pgfId-1119072}8.4.2 Running Podman within a Kubernetes pod {#08.htm#heading_id_10 .fm-head1}

[]{#08.htm#pgfId-1119075}A
[]{#08.htm#marker-1119073}[]{#08.htm#marker-1119074}common use case for
CI/CD systems is using Podman to run containers within Kubernetes. As
you learned, running Podman within a container requires either
`CAP_SYS_ADMIN`{.fm-code-in-text} for rootful containers or
`CAP_SETUID`{.fm-code-in-text} and `CAP_SETGID`{.fm-code-in-text} to run
in rootless mode. Understand that Podman containers almost always
require more than one UID to run, especially when running
`podman`{.fm-code-in-text} `build`{.fm-code-in-text}. Lots of Podman
problems have been raised by users of Kubernetes attempting to run
Podman in a locked-down Kubernetes container, with only one UID and
without Linux capabilities. These containers are the default for
OpenShift and lots of the cloud-based Kubernetes environments. Running a
container engine like Podman in environments without some Linux
capabilities and access to more than one UID is impossible.

[]{#08.htm#pgfId-1119076}The equivalent version of running rootful
Podman using the quay.io/podman/stable image within a
`privileged`{.fm-code-in-text} Kubernetes
container[]{#08.htm#marker-1119077} can be launched with this Kubernetes
YAML file:

``` programlisting
apiVersion: v1
kind: Pod
metadata:
 name: podman-priv
spec:
 containers:
   - name: priv
     image: quay.io/podman/stable
     args:
       - podman
       - version
     securityContext:
       privileged: true
```

[]{#08.htm#pgfId-1119091}Similarly, you can launch a rootless Podman
within a Kubernetes container by using the following YAML file. Note
that you specify the `runAsUser`{.fm-code-in-text}:
`1000`{.fm-code-in-text} as the UID, not the `podman`{.fm-code-in-text}
user. Kubernetes does not support translating usernames within
containers to UIDs:

``` programlisting
apiVersion: v1
kind: Pod
metadata:
  name: podman-rootless
spec:
  containers:
  - name: rootless
    image: quay.io/podman/stable
    args:
      - podman 
      - version
    securityContext:
      capabilities:
        add:
          - "SETUID"
          - "SETGID"
      runAsUser: 1000
```

[]{#08.htm#pgfId-1119109}[Note]{.fm-callout-head} See the following
articles written by me along with my colleague, Urvashi Mohnani, that
offer many more examples on running Podman within containers:

-   []{#08.htm#pgfId-1119110 .calibre17}"How to Use Podman inside of a
    Container" ([http://mng.bz/vXDM](http://mng.bz/vXDM){.url1})

-   []{#08.htm#pgfId-1119111 .calibre17}"How to Use Podman inside of
    Kubernetes" ([http://mng.bz/49EV](http://mng.bz/49EV){.url1})

[]{#08.htm#pgfId-1119112}As you can see, it is fairly easy to run Podman
containers within Kubernetes, as long as you understand the Podman
requirements. There is ongoing work within the Kubernetes community to
take advantage of user namespaces, making it easier to run Podman
containers within Kubernetes containers and making them
[]{#08.htm#marker-1119113}[]{#08.htm#marker-1119114}more
[]{#08.htm#marker-1119115}[]{#08.htm#marker-1119116}secure.

## []{#08.htm#pgfId-1119118}Summary {#08.htm#heading_id_11 .fm-head}

-   []{#08.htm#pgfId-1119119 .calibre17}The `podman`{.fm-code-in-text}
    `generate`{.fm-code-in-text} `kube`{.fm-code-in-text} command easily
    allows you to move locally running pods and containers into a
    Kubernetes YAML file suitable for running within a Kubernetes
    cluster.

-   []{#08.htm#pgfId-1119120 .calibre17}These YAML files can also be
    used to generate local pods and containers via the
    `podman`{.fm-code-in-text} `play`{.fm-code-in-text}
    `kube`{.fm-code-in-text} command.

-   []{#08.htm#pgfId-1119121 .calibre17}The `--down`{.fm-code-in-text}
    option allows `podman`{.fm-code-in-text} `play`{.fm-code-in-text}
    `kube`{.fm-code-in-text} to shut down all pods and containers
    launched by a previous `podman`{.fm-code-in-text}
    `play`{.fm-code-in-text} `kube`{.fm-code-in-text} command.

-   []{#08.htm#pgfId-1119122 .calibre17}The `--build`{.fm-code-in-text}
    option allows `podman`{.fm-code-in-text} `play`{.fm-code-in-text}
    `kube`{.fm-code-in-text} to generate the container image defined
    within the Kubernetes YAML file based on a Containerfile/Dockerfile,
    eliminating the need to push the image to a container registry.

-   []{#08.htm#pgfId-1119123 .calibre17}`podman`{.fm-code-in-text}
    `play`{.fm-code-in-text} `kube`{.fm-code-in-text} is a suitable
    replacement for `docker-compose`{.fm-code-in-text} because it shares
    the same YAML format as Kubernetes.

-   []{#08.htm#pgfId-1119124 .calibre17}Running Podman within Podman and
    Kubernetes containers is possible as long as you understand the
    Podman requirements for running in a locked-down
    []{#08.htm#marker-1119125 .calibre17}environment.

[]{#09.htm}
