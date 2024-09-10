# containers-notes

In a nutshell, containers are 3 main linux features bundled together: `Jailed Process` (change root / charoot) + `Namespaces` (unshared env) + `cgroups` (control groups).

_running dockers from a docker container: `docker run -it --name {name} --rm --privileged ubuntu:jammy`_

_attaching to a running docker `docker exec -it {name} bash`_

## Jailed Process / Change Root / Charoot

It's the action of limiting a given process (and its sub-processes) to a given directory and its sub-directories. That process (or user in this case) wouldn't be able to see anything outside that directory tree.

`chroot /{dir} bash` jails a process under `/{dir}`.

For more details about how to create a functional jailed process, follow this [documentation](./chroot/README.md).

## Namespaces

It allows the isolation of processes so that one cannot see the other nor meddle with each other. Processes are contained within itself and its sub-processes.

To isolate a process, the `unshare` program is used and it takes all the resources it should unshare:

`unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot /{dir} bash`

Once a process is unshared, it is necessary mount linux's resources so that is works:

```sh

unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot new-complete-root/ bash

mount -t proc none /proc
mount -t sysfs none /sys
mount -t tmpfs none /tmp

```

## cgroups - control groups

It allows to restrict the amount of resource (memory, cpu, disk, etc) a process can use.

The current api is v2 and to check which api your API distro run `grep -c cgroup /proc/mounts`

cgroups has an api based on directory and files. The root cgroup resides under `/sys/fs/cgroup` - notice all the files like:

1. cpu.max
2. memory.max
3. pids.max

To learn how to implement limitations to processes, read this [file](./cgroups/README.md).
