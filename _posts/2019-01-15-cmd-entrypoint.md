---
layout:      post
title:       CMD VS Entrypoint 
date:        2019-01-15
author:      Sandip
summary:     CMD VS Entrypoint 
categories:  containerization
thumbnail:   docker
comment:     true
tags:
      -  CMD
      -  Entrypoint 
---

some developers, especially newbies, still get confused when looking at the instructions that are available for use in a Dockerfile, because there are a few that may initially appear to be redundant (or, at least, have significant overlap) . RUN, CMD and ENTRYPOINT are a good example of this, and in this post I will explain the difference between CMD, RUN, and ENTRYPOINT on examples.

{% highlight ruby %}
Short version
{% endhighlight %}

   RUN executes the command(s) that you give in a new layer and creates a new image. This is mainly used for installing a new package.

   CMD is the default command to be run by the entrypoint. It sets default command and/or parameters, however, we can overwrite those commands or pass in and bypass the default parameters from the command line when docker runs

   ENTRYPOINT is the program to run the given command. It is used when yo want to run a container as an executable.

{% highlight ruby %}
 1. Shell vs. Exec
{% endhighlight %}

All three instructions RUN, CMD and ENTRYPOINT support two different forms: the shell form and the exec form.

When using the shell form, the specified binary is executed with an invocation of the shell using /bin/sh -c.
< instruction > < command >
For example let's consider the following Dockerfile:

{% highlight ruby %}
 FROM ubuntu:trusty
 
 CMD ping localhost
{% endhighlight %}


You can see this clearly if you run a container and then look at the docker ps output:

```
$ docker run -d med/test
98aa7c371139d81d376abdc9ce01ea53cfac1f87506d9e758fee14696a0fa621
$ docker ps -l
CONTAINER ID        IMAGE               COMMAND             CREATED
98aa7c371139        med/test   "/bin/sh -c 'ping loc"   5 seconds ago

```
Here we've run the test image and you can see that the command which was executed was /bin/sh -c 'ping localhost'.

You may run into problems with the shell form if you're building a minimal image which doesn't even include a shell binary. When Docker is constructing the command to be run it doesn't check to see if the shell is available inside the container, if you don't have /bin/sh in your image, the container will simply fail to start.

A better option is to use the exec form of the ENTRYPOINT/CMD instructions which looks like this:

CMD ["executable","param1","param2"]

Note that the content appearing after the CMD instruction in this case is formatted as a JSON array.

When the exec form of the CMD instruction is used the command will be executed without a shell.

Let's change our Dockerfile from the example above to see this in action:

{% highlight ruby %}
FROM ubuntu:trusty

CMD ["/bin/ping","localhost"]
{% endhighlight %}

Rebuild the image and look at the command that is generated for the running container:
{% highlight ruby %}
$ docker run -d med/test
fc9e3c759ea8f9793c1be8695d43e04050c9f14a4b0c723c95f2b76ee29c7628
$ docker ps -l
CONTAINER ID        IMAGE               COMMAND                 CREATED             
fc9e3c759ea8        med/test     "/bin/ping localhost"   2 seconds ago 
{% endhighlight %}

Now /bin/ping is being run directly without the intervening shell process.

### RUN

As mentioned above, the RUN command is mainly used to install a new package on top of the main OS distribution. When you use the RUN command, it will execute the instruction and will create a new layer.

RUN command can be used in two forms:

Shell form  
RUN <command>

Exec form  
RUN ["executable", "param1", "param2"]




### CMD

CMD instruction allows you to set a default command and default parameters which will be executed when docker is run. 
But these commands and parameters can be overwritten by passing the values over the command line.

CMD can be specified in three forms:

exec form, preferred way  
CMD ["executable","param1","param2"]

(sets additional default parameters for ENTRYPOINT in exec form)  
CMD ["param1","param2"] 

Shell form  
CMD command param1 param2

Again, the first and third forms should look familar to you as they were already covered above. 
The second one is used together with ENTRYPOINT instruction in exec form. 
It sets default parameters that will be added after ENTRYPOINT parameters if container runs without command line arguments.

Let's have a look how CMD instruction works. The following snippet in Dockerfile

CMD echo "Hello world"
when container runs as docker run -it <image> will produce output

Hello world

but when container runs with a command, e.g., docker run -it <image> /bin/bash, CMD is ignored and bash interpreter runs instead:
{% highlight ruby %}
root@98e4bed87725:/#
{% endhighlight %}

### ENTRYPOINT

ENTRYPOINTinstruction should be used when you need your container to be run as an executable. 
I might look similar to CMD, but in fact, it is different and should be used in a different context
