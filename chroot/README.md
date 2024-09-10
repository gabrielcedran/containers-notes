## Manual process: having programs working for jailed processes

In order to have basic programs working, like bash, cat, ls, etc, they (and their dependencies) have to be copied into the directory that the process will be jailed following their structure. To discover a program's dependencies run `ldd {program}` - only care for the dependencies that have a path.

Example (for this example I will use the directory `/new-root`):

```sh
mkdir /new-root/bin
cp /bin/bash /new-root/bin


# dependencies
ldd /bin/bash
# at the time of writing, ldd produced the following output:
#	linux-vdso.so.1 (0x00007f95bd692000)
#	libtinfo.so.6 => /lib/x86_64-linux-gnu/libtinfo.so.6 (0x00007f95bd4f7000)
#	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f95bd200000)
#	/lib64/ld-linux-x86-64.so.2 (0x00007f95bd694000)

mkdir /new-root/lib
cp /lib/x86_64-linux-gnu/libtinfo.so.6 /new-root/lib
cp /lib/x86_64-linux-gnu/libc.so.6 /new-root/lib

mkdir /new-root/lib64
cp /lib64/ld-linux-x86-64.so.2 /new-root/lib64

chroot /new-root bash

# from this point, the process is jailed under /new-root.
# however it cannot do much as there is no other program
# to have other programs working, follow the steps carried out for `bash`. Example ldd cat, etc
```

## Minimal debian copy: Having programs working for jailed processes

Debootstrap is a tool that install a debian based system into a subdirectory, avoiding the manual and tedious process above.

In order to have a minimal installation of debian, with the main programs, carry out the following steps (for the following examples, I'll use the directory `/new-complete-root`):

```sh

# debootstrap installation
apt-get update
apt-get install debootstrap -y

# install debian into /new-complete-root - ps jammy is the ubuntu version and can be changed
debootstrap --variant=minbase jammy /new-complete-root

chroot /new-complete-root bash

# all basic shell commands are available out of the box, like cat, ls, etc
```
