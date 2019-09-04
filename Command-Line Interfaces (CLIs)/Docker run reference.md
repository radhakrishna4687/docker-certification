# Docker run reference
- Docker runs processes in isolated containers. 
- A container is a process which runs on a host. 
- The host may be local or remote. 
- When an operator executes docker run, the container process that runs is isolated in that it has its own file system, its own networking, and its own isolated process tree separate from the host.
# General form
The basic `docker run` command takes this form:
```
$ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```
The `docker run` command must specify an IMAGE to derive the container from. An image developer can define image defaults related to:
- detached or foreground running
- container identification
- network settings
- runtime constraints on CPU and memory

With the `docker run [OPTIONS]` an operator can add to or override the image defaults set by a developer. And, additionally, operators can override nearly all the defaults set by the Docker runtime itself. The operator’s ability to override image and Docker runtime defaults is why run has more options than any other docker command.

# Operator exclusive options
Only the operator (the person executing docker run) can set the following options.
1. Detached vs foreground
- Detached (-d)
- Foreground
2. Container identification
- Name (--name)
- PID equivalent
3. IPC settings (--ipc)
4. Network settings
5. Restart policies (--restart)
6. Clean up (--rm)
7. Runtime constraints on resources
8. Runtime privilege and Linux capabilities

# Detached vs foreground
When starting a Docker container, you must first decide if you want to run the container in the background in a “detached” mode or in the default foreground mode:
```
-d=false: Detached mode: Run container in the background, print new container id
```
## Detached (-d)
- To start a container in detached mode, you use `-d=true` or just `-d` option. 
- By design, containers started in detached mode exit when the root process used to run the container exits, unless you also specify the --rm option. 
- If you use `-d` with `--rm`, the container is removed when it exits or when the daemon exits, whichever happens first.
- Do not pass a `service x start` command to a detached container. For example, this command attempts to start the nginx service.
```
$ docker run -d -p 80:80 my_image service nginx start
```
- This succeeds in starting the nginx service inside the container. 
- However, it fails the detached container paradigm in that, the root process (`service nginx start`) returns and the detached container stops as designed. As a result, the nginx service is started but could not be used. 
- Instead, to start a process such as the nginx web server do the following:
```
$ docker run -d -p 80:80 my_image nginx -g 'daemon off;'
```
> To reattach to a detached container, use docker attach command.
## Foreground
- In foreground mode (the default when -d is not specified), docker run can start the process in the container and attach the console to the process’s standard input, output, and standard error. 
- It can even pretend to be a TTY (this is what most command line executables expect) and pass along signals. 
- All of that is configurable:
```
-a=[]           : Attach to `STDIN`, `STDOUT` and/or `STDERR`
-t              : Allocate a pseudo-tty
--sig-proxy=true: Proxy all received signals to the process (non-TTY mode only)
-i              : Keep STDIN open even if not attached
```
- If you do not specify `-a` then Docker will attach to both stdout and stderr . 
- You can specify to which of the three standard streams (STDIN, STDOUT, STDERR) you’d like to connect instead, as in:
```
$ docker run -a stdin -a stdout -i -t ubuntu /bin/bash
```
- For interactive processes (like a shell), you must use `-i` `-t` together in order to allocate a tty for the container process. 
- `-i` `-t` is often written `-it` as you’ll see in later examples. 
- Specifying `-t` is forbidden when the client is receiving its standard input from a pipe, as in:
```
echo test | docker run -i busybox cat
```
>  A process running as PID 1 inside a container is treated specially by Linux: it ignores any signal with the default action. So, the process will not terminate on SIGINT or SIGTERM unless it is coded to do so.

# Container identification
## Name (--name)
The operator can identify a container in three ways:
1. UUID long identifier for eg “f78375b1c487e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778”
2. UUID short identifier (first 12 of UUID) for eg `f78375b1c487`
3. Name for eg `evil_ptolemy`
- The UUID identifiers come from the Docker daemon. 
- If you do not assign a container name with the --name option, then the daemon generates a random string name for you. 
- Defining a name can be a handy way to add meaning to a container. 
- If you specify a name, you can use it when referencing the container within a Docker network. 
- This works for both background and foreground Docker containers.
> Containers on the default bridge network must be linked to communicate by name.
## PID equivalent
- Finally, to help with automation, you can have Docker write the container ID out to a file of your choosing. 
- This is similar to how some programs might write out their process ID to a file (you’ve seen them as PID files):
```
--cidfile="": Write the container ID to the file
```
## Image[:tag]
- While not strictly a means of identifying a container, you can specify a version of an image you’d like to run the container with by adding image[:tag] to the command. For example, `docker run ubuntu:14.04`.
## Image[@digest]
- Images using the v2 or later image format have a content-addressable identifier called a digest. 
- As long as the input used to generate the image is unchanged, the digest value is predictable and referenceable.
- The following example runs a container from the alpine image with the `sha256:9cacb71397b640eca97488cf08582ae4e4068513101088e9f96c9814bfda95e0` digest:
```
$ docker run alpine@sha256:9cacb71397b640eca97488cf08582ae4e4068513101088e9f96c9814bfda95e0 date
```
## PID settings (--pid)
```
--pid=""  : Set the PID (Process) Namespace mode for the container,
             'container:<name|id>': joins another container's PID namespace
             'host': use the host's PID namespace inside the container
```
- By default, all containers have the PID namespace enabled.
- PID namespace provides separation of processes. The PID Namespace removes the view of the system processes, and allows process ids to be reused including pid 1.
- In certain cases you want your container to share the host’s process namespace, basically allowing processes within the container to see all of the processes on the system. 
- For example, you could build a container with debugging tools like strace or gdb, but want to use these tools when debugging processes within the container.
### Example: run htop inside a container
Create this Dockerfile:
```
FROM alpine:latest
RUN apk add --update htop && rm -rf /var/cache/apk/*
CMD ["htop"]
```
### Build the Dockerfile and tag the image as myhtop:
```
docker build -t myhtop .
```
### Use the following command to run htop inside a container:
```
docker run -it --rm --pid=host my-htop
```
Joining another container’s pid namespace can be used for debugging that container.
Example:
Start a container running a redis server:
```
docker run --name my-redis -d redis
```
Debug the redis container by running another container that has strace in it:
```
$ docker run -it --pid=container:my-redis my_strace_docker_image bash
$ strace -p 1
```
## UTS settings (--uts)
```
--uts=""  : Set the UTS namespace mode for the container,
       'host': use the host's UTS namespace inside the container
```
- The UTS namespace is for setting the hostname and the domain that is visible to running processes in that namespace. 
- By default, all containers, including those with --network=host, have their own UTS namespace. 
- The host setting will result in the container using the same UTS namespace as the host. Note that --hostname and --domainname are invalid in host UTS mode.
## IPC settings (--ipc)
```
--ipc="MODE"  : Set the IPC mode for the container
```
- If not specified, daemon default is used, which can either be "private" or "shareable", depending on the daemon version and configuration.
- IPC (POSIX/SysV IPC) namespace provides separation of named shared memory segments, semaphores and message queues.
# Network settings
```
--dns=[]           : Set custom dns servers for the container
--network="bridge" : Connect a container to a network
                      'bridge': create a network stack on the default Docker bridge
                      'none': no networking
                      'container:<name|id>': reuse another container's network stack
                      'host': use the Docker host network stack
                      '<network-name>|<network-id>': connect to a user-defined network
--network-alias=[] : Add network-scoped alias for the container
--add-host=""      : Add a line to /etc/hosts (host:IP)
--mac-address=""   : Sets the container's Ethernet device's MAC address
--ip=""            : Sets the container's Ethernet device's IPv4 address
--ip6=""           : Sets the container's Ethernet device's IPv6 address
--link-local-ip=[] : Sets one or more container's Ethernet device's link local IPv4/IPv6 addresses
```
- By default, all containers have networking enabled and they can make any outgoing connections. 
- The operator can completely disable networking with `docker run --network none` which disables all incoming and outgoing networking. 
- In cases like this, you would perform I/O through files or STDIN and STDOUT only.
- Publishing ports and linking to other containers only works with the default (bridge). 
- The linking feature is a legacy feature. 
- You should always prefer using Docker network drivers over linking.
- Your container will use the same DNS servers as the host by default, but you can override this with --dns.
- By default, the MAC address is generated using the IP address allocated to the container. 
- You can set the container’s MAC address explicitly by providing a MAC address via the `--mac-address parameter (format:12:34:56:78:9a:bc)`. 
- Be aware that Docker does not check if manually specified MAC addresses are unique.
Supported networks :
1. none
2. bridge
3. host
4. container 
5. custom network 
## NETWORK: NONE
- With the network is none a container will not have access to any external routes. 
- The container will still have a loopback interface enabled in the container but it does not have any routes to external traffic.
## NETWORK: BRIDGE
- With the network set to bridge a container will use docker’s default networking setup. 
- A bridge is setup on the host, commonly named docker0, and a pair of veth interfaces will be created for the container. 
- One side of the veth pair will remain on the host attached to the bridge while the other side of the pair will be placed inside the container’s namespaces in addition to the loopback interface. 
- An IP address will be allocated for containers on the bridge’s network and traffic will be routed though this bridge to the container.
- Containers can communicate via their IP addresses by default. To communicate by name, they must be linked.
## NETWORK: HOST
- With the network set to host a container will share the host’s network stack and all interfaces from the host will be available to the container. 
- The container’s hostname will match the hostname on the host system. 
- Note that --mac-address is invalid in host netmode. 
- Even in host network mode a container has its own UTS namespace by default. 
- As such --hostname and --domainname are allowed in host network mode and will only change the hostname and domain name inside the container. 
- Similar to --hostname, the --add-host, --dns, --dns-search, and --dns-option options can be used in host network mode. T
- hese options update /etc/hosts or /etc/resolv.conf inside the container. 
- No change are made to /etc/hosts and /etc/resolv.conf on the host.

> --network="host" gives the container full access to local system services such as D-bus and is therefore considered insecure.

## NETWORK: CONTAINER
- With the network set to container a container will share the network stack of another container. 
- The other container’s name must be provided in the format of `--network container:<name|id>`. 
- Note that --add-host --hostname --dns --dns-search --dns-option and --mac-address are invalid in container netmode, and --publish --publish-all --expose are also invalid in container netmode.
```
$ docker run -d --name redis example/redis --bind 127.0.0.1
$ # use the redis container's network stack to access localhost
$ docker run --rm -it --network container:redis example/redis-cli -h 127.0.0.1
```
## USER-DEFINED NETWORK
- You can create a network using a Docker network driver or an external network driver plugin. 
- You can connect multiple containers to the same network. 
- Once connected to a user-defined network, the containers can communicate easily using only another container’s IP address or name.

# Managing /etc/hosts
- Your container will have lines in /etc/hosts which define the hostname of the container itself as well as localhost and a few other common things. 
- The `--add-host` flag can be used to add additional lines to /etc/hosts.
```
$ docker run -it --add-host db-static:86.75.30.9 ubuntu cat /etc/hosts
```
# Restart policies (--restart)
- Using the `--restart` flag on Docker run you can specify a restart policy for how a container should or should not be restarted on exit.
- When a restart policy is active on a container, it will be shown as either `Up` or `Restarting` in `docker ps`. 
- It can also be useful to use docker events to see the restart policy in effect.

Docker supports the following restart policies:
1. no : Do not automatically restart the container when it exits. This is the default.
2. on-failure[:max-retries]	: Restart only if the container exits with a non-zero exit status. Optionally, limit the number of restart retries the Docker daemon attempts.
3. always: Always restart the container regardless of the exit status. When you specify always, the Docker daemon will try to restart the container indefinitely. The container will also always start on daemon startup, regardless of the current state of the container.
4. unless-stopped: Always restart the container regardless of the exit status, including on daemon startup, except if the container was put into a stopped state before the Docker daemon was stopped.

- Combining `--restart` (restart policy) with the `--rm` (clean up) flag results in an error. 
- On container restart, attached clients are disconnected. 

Examples
```$ docker run --restart=always redis```
This will run the redis container with a restart policy of always so that if the container exits, Docker will restart it.
```$ docker run --restart=on-failure:10 redis```
This will run the redis container with a restart policy of on-failure and a maximum restart count of 10. If the redis container exits with a non-zero exit status more than 10 times in a row Docker will abort trying to restart the container. Providing a maximum restart limit is only valid for the on-failure policy.

# Exit Status
- The exit code from `docker run` gives information about why the container failed to run or why it exited. 
- When docker run exits with a non-zero code, the exit codes follow the `chroot` standard, see below:
1. 125 if the error is with Docker daemon itself
```
$ docker run --foo busybox; echo $?
# flag provided but not defined: --foo
  See 'docker run --help'.
  125
```
2. 126 if the contained command cannot be invoked
```
$ docker run busybox /etc; echo $?
# docker: Error response from daemon: Container command '/etc' could not be invoked.
  126
```
3. 127 if the contained command cannot be found
```
$ docker run busybox foo; echo $?
# docker: Error response from daemon: Container command 'foo' not found or does not exist.
  127
```
# Clean up (--rm)
- By default a container’s file system persists even after the container exits. 
- This makes debugging a lot easier (since you can inspect the final state) and you retain all your data by default. 
- But if you are running short-term foreground processes, these container file systems can really pile up. 
- If instead you’d like Docker to automatically clean up the container and remove the file system when the container exits, you can add the `--rm` flag:
```--rm=false: Automatically remove the container when it exits```
> Note: When you set the --rm flag, Docker also removes the anonymous volumes associated with the container when the container is removed. This is similar to running docker rm -v my-container. Only volumes that are specified without a name are removed. For example, with docker run --rm -v /foo -v awesome:/bar busybox top, the volume for /foo will be removed, but the volume for /bar will not. Volumes inherited via --volumes-from will be removed with the same logic -- if the original volume was specified with a name it will not be removed.

# Security configuration
```
--security-opt="label=user:USER"     : Set the label user for the container
--security-opt="label=role:ROLE"     : Set the label role for the container
--security-opt="label=type:TYPE"     : Set the label type for the container
--security-opt="label=level:LEVEL"   : Set the label level for the container
--security-opt="label=disable"       : Turn off label confinement for the container
--security-opt="apparmor=PROFILE"    : Set the apparmor profile to be applied to the container
--security-opt="no-new-privileges:true|false"   : Disable/enable container processes from gaining new privileges
--security-opt="seccomp=unconfined"  : Turn off seccomp confinement for the container
--security-opt="seccomp=profile.json": White listed syscalls seccomp Json file to be used as a seccomp filter
```
- You can override the default labeling scheme for each container by specifying the `--security-opt` flag. 
- Specifying the level in the following command allows you to share the same content between containers.
```
$ docker run --security-opt label=level:s0:c100,c200 -it fedora bash
```
# Specify an init process
- You can use the `--init` flag to indicate that an init process should be used as the PID 1 in the container. 
- Specifying an init process ensures the usual responsibilities of an init system, such as reaping zombie processes, are performed inside the created container.
- The default init process used is the first docker-init executable found in the system path of the Docker daemon process. 
- This docker-init binary, included in the default installation, is backed by tini.
# Specify custom cgroups
- Using the `--cgroup-parent` flag, you can pass a specific `cgroup` to run a container in. 
- This allows you to create and manage `cgroups` on their own. 
- You can define custom resources for those `cgroups` and put containers under a common parent group.
# Runtime constraints on resources
The operator can also adjust the performance parameters of the container:
## -m, --memory=""
Memory limit (format: `<number>[<unit>]`). Number is a positive integer. Unit can be one of b, k, m, or g. Minimum is 4M.
## --memory-swap=""
Total memory limit (memory + swap, format: `<number>[<unit>]`). Number is a positive integer. Unit can be one of b, k, m, or g.
## --memory-reservation=""
Memory soft limit (format: `<number>[<unit>]`). Number is a positive integer. Unit can be one of b, k, m, or g.
## --kernel-memory=""
Kernel memory limit 
## -c, --cpu-shares=0
CPU shares (relative weight)
## --cpus=0.000
Number of CPUs. Number is a fractional number. 0.000 means no limit.
# User memory constraints
## memory=inf, memory-swap=inf (default)
There is no memory limit for the container. The container can use as much memory as needed.
## memory=L<inf, memory-swap=inf
(specify memory and set memory-swap as -1) The container is not allowed to use more than L bytes of memory, but can use as much swap as is needed (if the host supports swap memory).

## memory=L<inf, memory-swap=2*L
(specify memory without memory-swap) The container is not allowed to use more than L bytes of memory, swap plus memory usage is double of that.

## memory=L<inf, memory-swap=S<inf, L<=S
(specify both memory and memory-swap) The container is not allowed to use more than L bytes of memory, swap plus memory usage is limited by S.
```
$ docker run -it -m 300M --memory-swap -1 ubuntu:14.04 /bin/bash
## Memory reservation 
```
Memory reservation is a kind of memory soft limit that allows for greater sharing of memory. Under normal circumstances, containers can use as much of the memory as needed and are constrained only by the hard limits set with the -m/--memory option. 
```
$ docker run -it -m 500M --memory-reservation 200M ubuntu:14.04 /bin/bash
```
# Kernel memory constraints
- Kernel memory is fundamentally different than user memory as kernel memory can’t be swapped out. 
- The inability to swap makes it possible for the container to block system services by consuming too much kernel memory. Kernel memory includes：
- The inability to swap makes it possible for the container to block system services by consuming too much kernel memory. 
Kernel memory includes：
1. stack pages
2. slab pages
3. sockets memory pressure
4. tcp memory pressure
```$ docker run -it -m 500M --kernel-memory 50M ubuntu:14.04 /bin/bash```
# Swappiness constraint
- By default, a container’s kernel can swap out a percentage of anonymous pages. 
- To set this percentage for a container, specify a `--memory-swappiness` value between 0 and 100. 
- A value of 0 turns off anonymous page swapping. 
- A value of 100 sets all anonymous pages as swappable. 
- By default, if you are not using `--memory-swappiness`, memory swappiness value will be inherited from the parent.
```
$ docker run -it --memory-swappiness=0 ubuntu:14.04 /bin/bash
```
# CPU share constraint
By default, all containers get the same proportion of CPU cycles. This proportion can be modified by changing the container’s CPU share weighting relative to the weighting of all other running containers.
# CPU period constraint
```$ docker run -it --cpu-period=50000 --cpu-quota=25000 ubuntu:14.04 /bin/bash```
# Cpuset constraint
```$ docker run -it --cpuset-cpus="1,3" ubuntu:14.04 /bin/bash```
# CPU quota constraint
# Block IO bandwidth (Blkio) constraint
- By default, all containers get the same proportion of block IO bandwidth (blkio). 
- This proportion is 500. 
- To modify this proportion, change the container’s blkio weight relative to the weighting of all other running containers using the `--blkio-weight` flag.
```
$ docker run -it --name c1 --blkio-weight 300 ubuntu:14.04 /bin/bash
```
# Additional groups
```
--group-add: Add additional groups to run as
```
# Runtime privilege and Linux capabilities
```
--cap-add: Add Linux capabilities
--cap-drop: Drop Linux capabilities
--privileged=false: Give extended privileges to this container
--device=[]: Allows you to run devices inside the container without the --privileged flag.
```
- By default, Docker containers are “unprivileged” and cannot, for example, run a Docker daemon inside a Docker container. 
- This is because by default a container is not allowed to access any devices, but a “privileged” container is given access to all devices (see the documentation on cgroups devices).
- When the operator executes `docker run --privileged`, Docker will enable access to all devices on the host as well as set some configuration in AppArmor or SELinux to allow the container nearly all the same access to the host as processes running outside containers on the host. 
```
$ docker run --device=/dev/snd:/dev/snd ...
```
By default, the container will be able to read, write, and mknod these devices. This can be overridden using a third :rwm set of options to each --device flag:
```
$ docker run --device=/dev/sda:/dev/xvdc --rm -it ubuntu fdisk  /dev/xvdc
```
```
$ docker run --cap-add=ALL --cap-drop=MKNOD ...
```
# Logging drivers (--log-driver)
- The container can have a different logging driver than the Docker daemon. 
- Use the `--log-driver=VALUE` with the docker run command to configure the container’s logging driver. 
The following options are supported:
1. none
2. json-file	
3. syslog
4. journald
5. gelf
6. fluentd
7. awslogs
8. splunk
# Overriding Dockerfile image defaults
- When a developer builds an image from a Dockerfile or when she commits it, the developer can set a number of default parameters that take effect when the image starts up as a container.
- Four of the Dockerfile commands cannot be overridden at runtime: `FROM`, `MAINTAINER`, `RUN`, and `ADD`. 
- Everything else has a corresponding override in docker run. 
- We’ll go through what the developer might have set in each Dockerfile instruction and how the operator can override that setting.
1. CMD (Default Command or Options)
2. ENTRYPOINT (Default Command to Execute at Runtime)
3. EXPOSE (Incoming Ports)
4. ENV (Environment Variables)
5. HEALTHCHECK
6. VOLUME (Shared Filesystems)
7. USER
8. WORKDIR

## CMD (default command or options)
```
$ docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```
- This command is optional because the person who created the IMAGE may have already provided a default COMMAND using the Dockerfile CMD instruction. 
- As the operator (the person running a container from the image), you can override that CMD instruction just by specifying a new COMMAND.
- If the image also specifies an ENTRYPOINT then the CMD or COMMAND get appended as arguments to the ENTRYPOINT.
## ENTRYPOINT (default command to execute at runtime)
```
--entrypoint="": Overwrite the default entrypoint set by the image
```
- The ENTRYPOINT of an image is similar to a COMMAND because it specifies what executable to run when the container starts, but it is (purposely) more difficult to override. 
- The ENTRYPOINT gives a container its default nature or behavior, so that when you set an ENTRYPOINT you can run the container as if it were that binary, complete with default options, and you can pass in more options via the COMMAND. 
- But, sometimes an operator may want to run something else inside the container, so you can override the default ENTRYPOINT at runtime by using a string to specify the new ENTRYPOINT.  

Here is an example of how to run a shell in a container that has been set up to automatically run something else (like /usr/bin/redis-server):
```
$ docker run -it --entrypoint /bin/bash example/redis
```
or two examples of how to pass more parameters to that ENTRYPOINT:
```
$ docker run -it --entrypoint /bin/bash example/redis -c ls -l
$ docker run -it --entrypoint /usr/bin/redis-cli example/redis --help
```
You can reset a containers entrypoint by passing an empty string, for example:
```
$ docker run -it --entrypoint="" mysql bash
```
> Passing --entrypoint will clear out any default command set on the image (i.e. any CMD instruction in the Dockerfile used to build it).
## EXPOSE (incoming ports)
```
--expose=[]: Expose a port or a range of ports inside the container.
             These are additional to those exposed by the `EXPOSE` instruction
-P         : Publish all exposed ports to the host interfaces
-p=[]      : Publish a container's port or a range of ports to the host
               format: ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort | containerPort
               Both hostPort and containerPort can be specified as a
               range of ports. When specifying ranges for both, the
               number of container ports in the range must match the
               number of host ports in the range, for example:
                   -p 1234-1236:1234-1236/tcp

               When specifying a range for hostPort only, the
               containerPort must not be a range.  In this case the
               container port is published somewhere within the
               specified hostPort range. (e.g., `-p 1234-1236:1234/tcp`)

               (use 'docker port' to see the actual mapping)

--link=""  : Add link to another container (<name or id>:alias or <name or id>)
```
## ENV (environment variables)
Docker automatically sets some environment variables when creating a Linux container. Docker does not set any environment variables when creating a Windows container.

The following environment variables are set for Linux containers:
1. HOME
2. HOSTNAME
3. PATH
4. TERM

- Additionally, the operator can set any environment variable in the container by using one or more `-e` flags, even overriding those mentioned above, or already defined by the developer with a Dockerfile ENV. 
- If the operator names an environment variable without specifying a value, then the current value of the named variable is propagated into the container’s environment:
```
$ export today=Wednesday
$ docker run -e "deep=purple" -e today --rm alpine env
```
# HEALTHCHECK
```
  --health-cmd            Command to run to check health
  --health-interval       Time between running the check
  --health-retries        Consecutive failures needed to report unhealthy
  --health-timeout        Maximum time to allow one check to run
  --health-start-period   Start period for the container to initialize before starting health-retries countdown
  --no-healthcheck        Disable any container-specified HEALTHCHECK
```
Example:
```
$ docker run --name=test -d \
    --health-cmd='stat /etc/passwd || exit 1' \
    --health-interval=2s \
    busybox sleep 1d
```
# TMPFS (mount tmpfs filesystems)
```
--tmpfs=[]: Create a tmpfs mount with: container-dir[:<options>],
            where the options are identical to the Linux
            'mount -t tmpfs -o' command.
```
Example:
```
$ docker run -d --tmpfs /run:rw,noexec,nosuid,size=65536k my_image
```
# VOLUME (shared filesystems)
```
-v, --volume=[host-src:]container-dest[:<options>]: Bind mount a volume.
The comma-delimited `options` are [rw|ro], [z|Z],
[[r]shared|[r]slave|[r]private], and [nocopy].
The 'host-src' is an absolute path or a name value.

If neither 'rw' or 'ro' is specified then the volume is mounted in
read-write mode.

The `nocopy` mode is used to disable automatically copying the requested volume
path in the container to the volume storage location.
For named volumes, `copy` is the default mode. Copy modes are not supported
for bind-mounted volumes.

--volumes-from="": Mount all volumes from the given container(s)
```
# USER
- root (id = 0) is the default user within a container. 
- The image developer can create additional users. 
- Those users are accessible by name. 
- When passing a numeric ID, the user does not have to exist in the container.

The developer can set a default user to run the first process with the Dockerfile USER instruction. When starting a container, the operator can override the USER instruction by passing the -u option.

```
-u="", --user="": Sets the username or UID used and optionally the groupname or GID for the specified command.

The followings examples are all valid:
--user=[ user | user:group | uid | uid:gid | user:gid | uid:group ]
```
# WORKDIR
The default working directory for running binaries within a container is the root directory (/), but the developer can set a different default with the Dockerfile WORKDIR command. The operator can override this with:

```
-w="": Working directory inside the container
```



















































































