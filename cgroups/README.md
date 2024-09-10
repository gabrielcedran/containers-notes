# cgroups

## Limiting resources

To impose limits on processes, it's necessary to mirror the cgroup (`/sys/fs/cgroup`) directory into a sub-directory (e.g `/sys/fs/cgroup/limited-resources`).

Creating a sub-directory in the `/sys/fs/cgroup`, automatically creates the basic files - which isn't very useful in the way it is (notice the content of `cgroup.controllers` being empty and files diff between parent and child directories).

In order to have all the files created automatically, it's necessary to add some properties to `cgroup.subtree_control` -> `echo "+cpuset +cpu +io +memory +hugetlb +pids +rdma" > /sys/fs/cgroup/cgroup.subtree_control`. But before doing so, it's necessary to move all process away from the root cgroup to a sub-cgroup [read](./Process-handling).

Once all the files are created in the sub-cgroup, you just have to start limiting resources:

```sh

# limit processes assigned to limited-resources cgroup to 80mb
echo 83886080 > /sys/fs/cgroup/limited-resources/memory.max

# limit processes assigned to limited-resources to 5% of the cpu
# max is 100000, therefore 5000 out of 100000
echo "5000 100000" > /sys/fs/cgroup/limited-resources/cpu.max


# limit the number of subprocesses (to prevent fork bomb)
echo 3 > /sys/fs/cgroup/limited-resources/pids.max
```

### Process handling

All processes running on the host machine are mapped out in the `cgroup.procs` (either at the root cgroup or in a sub-cgroup).

Moving processes between cgroups is very easy, suffice adding the PID into `cgroup.procs` of the wanted cgroup: `echo {pid} > /sys/fs/cgroup/cgroup.procs` or `echo {pid} > /sys/fs/cgroup/{sub-cgroup-dir}/cgroup.procs`.

example:

```sh

ps aux # or cat /sys/fs/cgroup/cgroup.procs

# move {pid} to sub-cgroup `limited-resources`
echo {pid} > /sys/fs/cgroup/limited-resources/cgroup.procs

# move {pid} back to root cgroup
echo {pid} > /sys/fs/cgroup/limited-resources/cgroup.procs

```

#### Testing it in practice

##### Causing the issue

1. Run any docker image
2. Connect another terminal to the running container
3. Install `htop` program to follow memory utilisation `apt-get install htop`.
4. Execute `htop`
5. On the other connected terminal, run the command `yes | tr \\n x | head -c 1048576000 | grep n` to consume memory and cpu uncontrollably (on htop, notice memory and cpu consumption - usually one core of the cpu is hog)
6. Kill the process and notice the memory and cpu consumption going back to normal

##### Applying the solution

1. Create a new cgroup (e.g limited-resources)
2. Change memory and cpu limit as explained above
3. run the command `yes | tr \\n x | head -c 1048576000 | grep n` again
4. On `htop` notice that memory and cpu is never way above than the defined limits

### Points of attention

1: Processes can create subprocesses. Subprocesses are createD in the same cgroup of the parent process, but moving a parent process to another cgroup does not move the sub-process automatically.

If you want to limit the resources a given user can use, it's important to ensure that not only the main process is assigned to the correct cgroup but also all the subprocess that might have already been created.

_To reinforce: At the point when the main process is already assigned to the correct cgroup, all sub process will also be created in the cgroup._

2: Memory already allocated and used by a process won't be freed until that process itself release it.
