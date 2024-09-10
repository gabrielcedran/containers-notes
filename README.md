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
