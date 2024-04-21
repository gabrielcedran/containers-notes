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



_For the following examples I'll be using a docker container as my computer is a macos, thus not a linux based OS. If I were on a linux based computer, I could run the examples directly on the host computer._

### chroot (charoot or change root or linux jails)

This is the operation / utility that allows you to change the root directory for a process and its children - it's known as jailed process as it restricts the process so that it can only see what is inside the specifiec directory, nothing outisde. 

When you run a process within a `chroot` environment, the process sees a modified file system hierarchy where the specified directory becomes the root (`/`) directory.

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


