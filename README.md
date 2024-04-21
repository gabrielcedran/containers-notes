# containers-notes

These are the my main takeaways and notes from the Complete Intro to Containers [course](https://btholt.github.io/complete-intro-to-containers/) by Brian Holt that I would like to have easy access in the future should I need them.


## Intro

Bear-metal servers, VMs and containers.

*Abstraction level*

Containers operate on OS level and share the host operating system's kernel but have their own isolated filesystem, process space and network interfaces. VMs on the other hand abstract the entire hardware layer - each VM runs its own guest operating system on top of a hypervisor (which sits on top of the physical hardware).


*Resource usage*

Containers do not require a separate operating system for each application, thus are known as being lightweight - they share resources with the host system. VMs are heavier because they include a full operating system and require more resources (CPU, memory, disk space, etc). 


*Isolation*

Containers provide process isolation level (between containers and from the host system) by putting together 3 basic kernel features: chroot ("change root"), namespaces and control groups ("c-groups"). VMs provide hardware-level isolation and each VM has its own dedicated resources and operates independently of other VMs on the same host.


*Concepts*

Typically docker running on Linux does not use virtual machines to run containers - it leverages the host Linux kernel's features and enables the containers to run directly on the host system without the overhead of running a separate VM. However lightweight VMs are used under the hood when running docker on non-linux based hosts (like windows and macos - which partially bsd based / unix based).

Ps: it's possible to configure docker to run on Linux using a virtual machine based approach for environments where full isolation is required.



_For the following academic examples I'll be using a docker container as my computer is a macos, thus not a linux based OS. If I were on a linux based computer, I could run the examples directly on the host computer. `docker run -it --name docker-host --rm --privileged ubuntu:bionic`_

### chroot (charoot or change root or linux jails)

This is the operation / utility that allows you to change the root directory for a process and its children - it's known as jailed process as it restricts the process so that it can only see what is inside the specifiec directory, nothing outisde. 

When you run a process within a `chroot` environment, the process sees a modified file system hierarchy where the specified directory becomes the root (`/`) directory.

_chroot is about sharing file system - 2 chrooted processes still can see each other's via ps and manage them (kill, sigint, etc)_

#### Showcasing chroot

The basics to run a process with the changed root is executing the command `chroot . bash` from the desired directory (`bash` argument specifies that you want to execute the process with bash shell rather than the default `sh`).

However the new process will have no access to the basic necessary libraries and modules (like bash and its dependencies). It's necessary to mimic the host structure in order to have it working:

```bash

mkdir -p my-new-root # directory where the process with changed root will be launched

mkdir -p my-new-root/bin
cp /bin/bash my-new-root/bin

ldd /bin/bash # to discover the modules and libraries required by bash

# given the previous command output, copy everything that has a directory into my-new-root directory following the same structure

# the following is what was necessary at the time of writing this:

mkdir -p my-new-root/lib{,64}

cp /lib/x86_64-linux-gnu/libtinfo.so.5 /lib/x86_64-linux-gnu/libdl.so.2 /lib/x86_64-linux-gnu/libc.so.6 my-new-root/lib
cp /lib64/ld-linux-x86-64.so.2 my-new-root/lib64

cd my-new-root
chroot . bash # or chroot my-new-root/ bash

# note that some basic programs that are not built into linux kernel (ls, cp, mv, etc) are not available since they were not provided
# however it is just a matter of following the steps above using ldd program
```

_In order to discover which linux distro and version you are running, run the command `cat /etc/issue`_


Instead of going and checking and handcopying all the necessary tools, modules and libraries, it's possible to create a valid and changerootable environment with a tool called `debootstrap` (DEbian BOOT STRAP), which bootstraps a directory already with what you need:

```bash

apt update

apt install debootstrap -y

debootstrap --variant=minbase bionic /my-directory-root 
# --variant=minbase creates the environment with the most minimal tools that you will possibly need to run a debian based ubuntu
# tells the type of ubuntu (or debian) you want
# last parameter is the name of the wanted directory

chroot /my-directory-root bash

# ls, cd, cp, all work out of the box

```


### namespaces

This is the operation / utility to limit processes to not see or interfere others - which is essential for containerisation. They get their own process ids, get their own networking layer, etc.


_Namespaces is about sharing capabilities or controlling the capabilities that flow into processes_

#### Showcasing namespaces

Problem illustration:

On a running container, create a file and tail it with -f to create a long running process: `tail -f {file}`.

On a new shell (emulator), connect (or attach) to the previously created and running docker `docker exec -it {container id / name} bash` - this opens another shell in the same container. On this shell, type `ps aux`. Amongst the processes, you can find the tail process and kill it (`kill -9 {pid}`).


Solution:

Unshare all the resources that you don't want the new process(es) to have access (or shared access) using the tool... `unshare`. Like unshare the network, unshare the filesystem, unshare the UTS (process managing system), etc.

`unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot /desired-new-root bash` after the flags you just tell what you want to run with that unshare. In this case, chroot pointing to the desired new root dictory and with which shell.

_Each flag above is a namespace._

After running the previous command, `ps aux` won't return other's processes. It's necessary to mount some things tho to let the environment know it can access the process system and file system:

```
mount -t proc none /proc
mount -t sysfs none /sys
mount -t tmpfs none /tmp
```

_Now the process spawned above would be truly containerised and if access to the network was needed, it would actually be necessary to create a separate network, which would connect to a broader network._











