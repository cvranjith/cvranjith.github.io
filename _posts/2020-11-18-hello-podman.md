    Created by Ranjith Vijayan, last modified on Dec 26, 2020

### TL;DR

** Containers != Docker **
Our Journey towards Containerization (a.k.a 'Dockerization')

Ever since we embarked our journey in the container world with "Docker" technology, there has never been a go back. It enabled us provisioning and running our application environments in the quickest time possible. It made our path easier in exploring DevOps & Continuous Integration / Continuous Deployment (CI/CD) capabilities for our CEMLI projects and POCs, and it facilitated our developers deploying and testing their code quickly and efficiently.

We knew that containers were popular in micro-service world. But more we learnt about and experimented with Docker, it made us feel at home even while dealing with our legacy/monolithic applications. While Docker is a tool that exploded its popularity by enabling micro-services workflows, we realized that it also a good tool for all types of architectures and applications. We soon "Dockerized" FLEXCUBE Application which mainly used a J2EE Application Server and an Oracle Database as the major tech-stack components, even before we even went about ‘Dockerizing’ the newer micro-service based Oracle Banking products. We could spin the application environments as lightweight, portable and self-sufficient containers, in a matter of minutes, with zero pre-requisites, but the "Docker engine" on the host machine. Docker technology gave us the feeling of “Instant Application Portability”. We made some quick tools (also known as Reference Install - RI tools) to leverage the Docker power adaptable for our applications. We started embracing the docker technology more when delved deeper into it, and it never disappointed us.


... But what if I told you that we might have to start exploring container technologies other than docker?


Take a look at the updates on some of the major Linux Distros' sites.


Both Red Hat Enterprise Linux (RHEL) 8 and CentOS 8 have dropped official support for the Docker container runtime engine.

quote

> The docker package is not shipped or supported by Red Hat for Red Hat Enterprise Linux (RHEL) 8. The docker container engine is replaced by a suite of tools in the Container Tools module.

The podman container engine replaced docker as the preferred, maintained, and supported container runtime of choice for Red Hat Enterprise Linux 8 systems. Podman provides a docker compatible command line experience enabling users to find, run, build, and share containers. Podman uses Buildah and Skopeo as libraries for the build and push"


The Oracle Enterprise Linux (OEL) 8 documentation says

quote

> Podman is also intended as a drop-in replacement for Oracle Container Runtime for Docker”


Below note of December 2020 update of Kubernetes caused me to raise an eyebrow over my morning coffee
Docker as an underlying runtime is being deprecated. Docker-produced images will continue to work in your cluster with all runtimes, as they always have. The Kubernetes community has written a blog post about this in detail with a dedicated FAQ page for it.


So what are these new tools that we are now hearing about– Podman, Skopeo, Buildah etc?


Before getting into it, let's quickly take a look at a high level on, what containers are and what Docker is.


### Containers


Container technology is not new- it has been available in Linux over at least 40 years now. Back in 1979 Unix V7 introduced the “chroot” system for changing the root directory of a process and its children to a new location in the file system. This was the beginning of Process Isolation and container technoligy. Chroot literally created a "jail" for an application that has all of the root access, but within its boundaries. Similar operating system level virtualization has also been offered by FreeBSD jails, AIX Workload Partitions and Solaris Containers. Containers have emerged as a technology for building, packaging and distributing modern applications. It combines the flexibility of image based delivery, lighter-weight and minimal footprint and isolation of run-time environments. Benefits of using containers include Instant portability, Lower hardware footprint, Easier maintenance, Reusability, Environment isolation and Lowered development costs


### Docker


Docker has become synonymous with container technology from 2013, because it has been the most successful at popularizing it. Docker made the container technology popular amongst millions of developers and administrators around the world. Today Docker is the de facto standard to build and share containerized apps - from desktop, to the cloud.

The name **"Docker"** came as a shortening of "Dock" + "Worker", and it may mean different things to different people. It may refer to as - The technology itself; an open source community project; The company Docker Inc., that primarily supports that project; and the tools that company formally supports.

The open source Docker community explore the design patterns and best practices about Docker and related projects in the Docker Ecosystem and works towards improving these technologies to benefit all users.

The company, Docker Inc., builds on the work of the Docker community, makes it more secure, and shares those advancements back to the greater community 

We even coined a word “dockerize” for the activities we perform in and around containers, that we use in our daily conversations and documentations so naturally.


So what are Podman, Buildah and Skopeo? ... and why do we need to look at alternatives if docker is soo good?


### Working with Podman

Podman refers to “Pod Manager”, which is a new open source, container engine that works seamlessly with OCI containers (OCI stands for Open Container Initiative) as well as pods. The concept of “pod” is borrowed from Kubernetes. A Pod can house zero to many containers.

Buildah is a utility for creating OCI compatible container images. Buildah provides a wider range of customization options than the more generic podman build command. The utility can push container images to container registries, so it is well-suited for use with deployment scripts and automated build pipelines.

Skopeo in Greek language means “look at”, “inspect” or “examine”, and it is exactly what skopeo does in the container world. It helps in copying, inspecting, deleting, and signing container images on local or remote container registries. In fact the skopeo project team originally wanted to build this functionality in Docker CLI, but the enhancement proposal was rejected because docker CLI was already complicated to do this kind of change easily, and was asked to launch it as a separate independent project… which now is available as a popular tool called Skopeo.

You might be wondering why do we have to look at alternatives to the well functioning Docker tool, and break it up to so many smaller parts. Here are some of the technical reasons why alternatives to docker might make sense.

   Docker basically employs a client-server architecture relying on a Docker daemon. The Docker CLI (that we use to run commands such as docker run, docker inspect, docker image etc) sends instruction to the “Docker Engine” running on the host machine. When we issue “docker run -d“ we tend to think that the container it spawns is the child of client process. But in fact it is the child of the daemon process that is tied to the Docker Engine. Unlike Docker,  Podman doesn’t depend on a daemon, but instead launches containers and pods as child processes.
    Think about remotely inspecting an image using Docker. That would require the Docker CLI to talk to a Daemon (dockerd) to talk to another Daemon (containerd) which talks to another Daemon (registry). All these are basically part of one big Daemon, which already supports a lot of use cases. Following the Unix philosophy “Do One Thing and Do It Well”, by breaking it up into smaller tools, allows Podman, Buildah, and Skopeo to experiment with its individual capabilities effectively.
    Over the years Docker engine has become a little monolith. The bigger it gets, the harder it is to change the fundamental architecture, making it difficult for the project to experiment with big changes.
    All of Docker operations (i.e. operations done by Daemon) had to be conducted by a user with the same full root authority. This can pause a potential security risk. If the container engine, runtime, or orchestrator is compromised, the attacker may gain root privileges on the host. Podman introduces Rootless container concept where containers can be created, run, and managed by users without admin rights. They allow for isolation inside of nested containers. Podman also can be run as root. But most of the use-cases can be achieved by running containers with non-privileged user accounts. Also for building and managing images one shouldn’t need a root account.
    Podman can ease the transition to Kubernetes. Configuring Kubernetes is an exercise in defining objects in YAML files. Kubernetes YAML is descriptive and powerful. Podman has a niche feature to generate Kubernetes YAML from the existing Podman pods using podman generate kube command. That is sounds sweet!


There may be also business and community reasons behind the genesis of Podman. We are not getting into the details of the same in this article.


Good news! If you have invested time in learning and playing with Docker CLI, not a single piece of that knowledge will go wasted when you go to Podman. Podman CLI tool supports exact same commands as Docker CLI. Anyone that has used the Docker CLI will feel immediately at home with Podman, so migration to from Docker to Podman should be seamless.

In fact you could set alias docker=podman and continue to use docker commands to work with Podman!


### Testing Podman in Oracle Enterprise Linux 8


We installed and tested Podman on Oracle Enterprise Linux (OEL) 8 with our FLEXCUBE images that we have built on Docker, and it worked just fine. A couple of changes in our RI scripting was all we had to do to spin a FLEXCUBE environment using Podman.


In this section we will go over some basic Podman commands and run an Oracle Database container Image.

To Install Podman on Oracle Enterprise Linux 8, you can use dnf install podman command. To install all the container utilities including podman, buidah and skopeo , you can use

```
dnf module install container-tools:ol8
```

After installation you can check print podman info, much like docker info.

```
podman info
host:
  arch: amd64
  buildahVersion: 1.14.9
  cgroupVersion: v1
  conmon:
 
    package: conmon-2.0.17-1.0.1.module+el8.2.1+7658+86e51d52.x86_64
    path: /usr/bin/conmon
    version: 'conmon version 2.0.17, commit: 7d67f45a8a8085557bcec893982af6f4e7377514'
 
<<snipped>
```

Since Podman doesn't need to be run as a root user, it is recommended to run podman as a non-root user. In the below steps we will create a linux user by name "podman"

```
    Create Linux User
    useradd podman --home-dir /scratch/podman


    Connect as user podman and run podman command
    [root@whf00ovz ~]# su - podman
    [podman@whf00ovz ~]$ podman info
    host:
      arch: amd64
      buildahVersion: 1.14.9
      cgroupVersion: v1
      conmon:
        package: conmon-2.0.17-1.0.1.module+el8.2.1+7658+86e51d52.x86_64
        path: /usr/bin/conmon
        version: 'conmon version 2.0.17, commit: 7d67f45a8a8085557bcec893982af6f4e7377514'
    <<snipped>>

    Looks great, and it seems like we are good to pull an image and test.
    In this step we will pull Oracle Database 19c image from container-registry.oracle.com and run a container using it.

    First lets login to the registry using podman login command
    [podman@whf00ovz ~]$ podman login container-registry.oracle.com
    Username: ranjith.vijayan@oracle.com
    Password:
     
    Login Succeeded!
    [podman@whf00ovz ~]$
        As expected, the login succeeded by using the SSO credentials
    Now we can pull and run the image. In this example I am directly issuing the "podman run" command, which will implicitly pull the image from the registry as it was not pulled before on this host
    [podman@whf00ovz ~]$ podman run -d --name ora-db-19c-in-podman database/enterprise:19.3.0.0
    Trying to pull container-registry.oracle.com/database/enterprise:19.3.0.0...
    Getting image source signatures
    Copying blob bd7b86188ada done 
    Copying blob 35defbf6c365 done 
    Copying blob 043a075be1ea done 
    Copying blob 30dde8221db6 done 
    Copying blob 29ee28675c53 done 
    Copying blob b7267f7fbeee done 
    Copying blob c4bdd478d3d0 done 
    Copying blob a6654847368b done 
    Copying blob f5b4867f6b93 done 
    Copying blob bc5458a731fb done 
    Copying blob 554f6f42e624 done 
    Copying blob 0cf0c2777355 done 
    Copying blob c809a85a6250 done 
    Copying blob d25e60f1987a done 
    Copying blob 07f4457e2768 done 
    Copying blob 1a02ac6d1f8b done 
    Copying blob a047904c9d59 done 
    Copying blob b7fe2df9722e done 
    Copying blob 4e48b81b580d done 
    Copying config 2e375ab669 done 
    Writing manifest to image destination
    Storing signatures
    86c693cca10458250ddae9a208e5ff42fa6c7aa3951c4edb8a7619ad8d1b9226
    [podman@whf00ovz ~]$
 
 ```
 
In the above example I have just issued a very basic command. Of course you can customize the configurations, mount volumes, export network ports etc, as documented by Oracle.

One thing you might have noticed in the above command is that I didn't specify the image name with registry qualifier. i.e. I used database/enterprise:19.3.0.0 instead of container-registry.oracle.com/database/enterprise:19.3.0.0 that I would have used in docker pull. This is because docker is opinionated to "docker.io" registry, so If we don't give the fully qualified image name, it will search in docker.io registry by default. We could however provide the fully qualified name for the image. But in Podman we have the flexibility to control this using the configuration file /etc/containers/registries.conf. Using this feature we can control the access to registries, search multiple registries in an order that we define,  and even limit it to work with only specific local registry(ies) if you wish to. Using this you can also simplify your CI/CD scripts. In my set-up he default configuration points to container-registry-oracle.com before docker.io.

```
        [podman@whf00ovz ~]$ cat /etc/containers/registries.conf
         
        [registries.search]
        registries = ['container-registry.oracle.com', 'docker.io', 'registry.fedoraproject.org', 'quay.io', 'registry.centos.org']
 ```
 
 As the podman run command succeeded it has now created the container in detached mode (since we used -d). We can check the container status using "podman ps" command
 
 ```
    [podman@whf00ovz ~]$ podman ps
    CONTAINER ID  IMAGE                                                        COMMAND               CREATED             STATUS                 PORTS     NAMES
    86c693cca104  container-registry.oracle.com/database/enterprise:19.3.0.0   /bin/sh -c exec $...  About a minute ago  Up About a minute ago            ora-db-19c-in-podman
```

Lets connect to the database now. First we can podman exec and then use sql*plus.

```
    [podman@whf00ovz ~]$ podman exec -it ora-db-19c-in-podman bash
    [oracle@86c693cca104 ~]$ sqlplus / as sysdba
     
    SQL*Plus: Release 19.0.0.0.0 - Production on Fri Nov 20 01:01:07 2020
    Version 19.3.0.0.0
     
    Copyright (c) 1982, 2019, Oracle.  All rights reserved.
     
     
    Connected to:
    Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
    Version 19.3.0.0.0
     
    SQL> select banner from v$version;
     
    BANNER
    --------------------------------------------------------------------------------
    Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
     
    SQL>
```


As you can see we could pull an Oracle Database image, spin a container, and use it without any hassle. Remember the image we pulled was prepared in Docker. Thanks to the OCI standards that image built by a tool can be inter-operably used in any tool conforming to the standards, without issues


Some thoughts before we end this article...


As technologists we have seen this over and over again. Tools will come and go. Technology will keep evolving. While innovations happen, it is important that involved parties and the community agree to common standards for the benefit of every one. Containers existed for a long time now in Linux. When Docker emerged in 2013, containers exploded in popularity and millions of apps architected on micro-services easily moved to the Cloud. Some other players like Rockit also entered the scene around the same time. Luckily since 2015 we have a unified set of standards by Open Containers Initiative (OCI) and specifications for how to run containers and manage container images. In addition, we also have Container Runtime Interface (CRI) and Container Network Interface(CNI). These standards governed the definition of containers and container images, and it allowed broad inter-operability no matter what run-time(s) are in use.


When I started playing with the new set of tools for managing containers, it made me to realize that Docker was just another tool. There are only “containers” and “images”, and not “Docker containers” and “Docker images”.  Owing to the popularity, not forgetting the innovations they brought in, brand-names have turned into ordinary words - Google for Search Engine, Xerox for Photocopy, Jacuzzi for Hot Tub,... and Docker for Container. Even common words like Kerosene, Escalator & Ping Pong were once trademarked.  Probably we need to go and check if we have used the word “dockerization” anywhere in our documentations and presentations, to stand corrected and embrace technology evolution.


As of writing container-registry.oracle.com landing page displays a welcome message that informs "Easy access to Oracle products for use in Docker containers". From what I have seen in the above experiment, "container-registry.oracle.com" is the "Container Registry" of oracle.com, and its images can be used for running "Containers" and not only "Docker containers" (smile)

