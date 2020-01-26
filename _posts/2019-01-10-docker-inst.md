---
layout:     post
title:      Installation of Docker
date:       2019-01-10
author:     Sandip
summary:    Installation of Docker on CentOS 7 
categories:  containerization
thumbnail:  linux 
tags:
  -Docker 
  -CentOS 7 
  
---

# Installation of docker on CentOS 7 

#### To install Docker on cent os 7  machine one should need to follow below steps.
{% highlight ruby %}
[root@sandip ~]# yum install docker -y 

  Installed:
    docker.x86_64 2:1.13.1-84.git07f3374.el7.centos

  Dependency Updated:
    docker-client.x86_64 2:1.13.1-84.git07f3374.el7.centos
    docker-common.x86_64 2:1.13.1-84.git07f3374.el7.centos

 Complete!
{% endhighlight %}
#### By default docker service is inactive as shown 
{% highlight ruby %}
[root@sandip ~]# systemctl status docker
 â docker.service - Docker Application Container Engine
    Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
    Active: inactive (dead)
      Docs: http://docs.docker.com

  Dec 01 19:55:25 sandip dockerd-current[19206]: time="2018-12-01T19:55:25.273799339+05:30" level=error msg... -f"
  Dec 01 19:55:35 sandip dockerd-current[19206]: time="2018-12-01T19:55:35.534426144+05:30" level=error msg...n\""
  Dec 06 20:22:30 sandip systemd[1]: Stopped Docker Application Container Engine.
Hint: Some lines were ellipsized, use -l to show in full.
{% endhighlight %}

#### To start docker service use 
{% highlight ruby %}
[root@sandip ~]# systemctl start docker 
{% endhighlight %}
#### now check the status 
{% highlight ruby %}
[root@sandip ~]# systemctl status docker
  â docker.service - Docker Application Container Engine
    Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
    Active: active (running) since Thu 2018-12-06 20:32:53 IST; 3s ago
      Docs: http://docs.docker.com
  Main PID: 2769 (dockerd-current)
     Tasks: 20
    CGroup: /system.slice/docker.service
             ââ2769 /usr/bin/dockerd-current --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current ...
             ââ2775 /usr/bin/docker-containerd-current -l unix:///var/run/docker/libcontainerd/docker-container...
{% endhighlight %}

#### to check the version 
{% highlight ruby %}
 [root@sandip ~]# docker -v
 Docker version 1.13.1, build 07f3374/1.13.1
{% endhighlight %}

#### As shown docker installation is done on machine and docker service is in running state, now lets run first image "Hello-World".

#### to know more about what are images and contaner and what is difference between them, please do check previous post.

#### to run container Hello-World firstly pull image from docker hub
{% highlight ruby %}
 [root@sandip ~]# docker pull hello-world
 Using default tag: latest
 Trying to pull repository docker.io/library/hello-world ...
 latest: Pulling from docker.io/library/hello-world
 d1725b59e92d: Pull complete
 Digest: sha256:0add3ace90ecb4adbf7777e9aacf18357296e799f81cabc9fde470971e499788
 Status: Downloaded newer image for docker.io/hello-world:latest
{% endhighlight %}

#### Now to check the downoaded image 
{% highlight ruby %}
[root@sandip ~]# docker images
 REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
 docker.io/hello-world   latest              4ab4c602aa5e        2 months ago        1.84 kB
{% endhighlight %}

#### here as one can find all images present.
#### now to run image "Hello-World"
{% highlight ruby %}
[root@sandip ~]# docker run hello-world
Hello from Docker!
his message shows that your installation appears to be working correctly.
{% endhighlight %}

#### This is the first container that we just spawned
#### to check the docker process:
{% highlight ruby %}
[root@sandip ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                          PORTS               NAMES
    8d260cc3a3a9        hello-world         "/hello"            About a minute ago   Exited (0) About a minute ago                       sharp_roentgen
{% endhighlight %}
#### this is exited process that shown because of "-a " option. 

