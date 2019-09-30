# Compose file version 3 reference
There are several versions of the Compose file format – 1, 2, 2.x, and 3.x.
# Service configuration reference
- The Compose file is a YAML file defining services, networks and volumes. 
- The default path for a Compose file is ./docker-compose.yml.
- You can use either a .yml or .yaml extension for this file. They both work.
- A service definition contains configuration that is applied to each container started for that service, much like passing command-line parameters to docker container create. 
- Likewise, network and volume definitions are analogous to docker network create and docker volume create.
- As with docker container create, options specified in the Dockerfile, such as CMD, EXPOSE, VOLUME, ENV, are respected by default - you don’t need to specify them again in docker-compose.yml.

# build
Configuration options that are applied at build time. `build` can be specified either as a string containing a path to the build context:
```
version: "3.7"
services:
  webapp:
    build: ./dir
```
Or, as an object with the path specified under context and optionally Dockerfile and args:
```
version: "3.7"
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```
If you specify image as well as build, then Compose names the built image with the webapp and optional tag specified in image:
```
build: ./dir
image: webapp:tag
```
This results in an image named webapp and tagged tag, built from ./dir.

## CONTEXT
Either a path to a directory containing a Dockerfile, or a url to a git repository.

When the value supplied is a relative path, it is interpreted as relative to the location of the Compose file. This directory is also the build context that is sent to the Docker daemon.

Compose builds and tags it with a generated name, and uses that image thereafter.
```
build:
  context: ./dir
```
## DOCKERFILE
Alternate Dockerfile.
Compose uses an alternate file to build with. A build path must also be specified.
```
build:
  context: .
  dockerfile: Dockerfile-alternate
```
## ARGS
Add build arguments, which are environment variables accessible only during the build process. 
```
build:
  context: .
  args:
    buildno: 1
    gitcommithash: cdc3b19
```
Or 
```
build:
  context: .
  args:
    - buildno=1
    - gitcommithash=cdc3b19
```
## CACHE_FROM
A list of images that the engine uses for cache resolution.

```
build:
  context: .
  cache_from:
    - alpine:latest
    - corp/web_app:3.14
```
## LABELS
Add metadata to the resulting image using Docker labels. You can use either an array or a dictionary.
```
build:
  context: .
  labels:
    com.example.description: "Accounting webapp"
    com.example.department: "Finance"
    com.example.label-with-empty-value: ""
```
Or 
```
build:
  context: .
  labels:
    - "com.example.description=Accounting webapp"
    - "com.example.department=Finance"
    - "com.example.label-with-empty-value"
```
## SHM_SIZE
Set the size of the /dev/shm partition for this build’s containers. Specify as an integer value representing the number of bytes or as a string expressing a byte value.
```
build:
  context: .
  shm_size: '2gb'
```
## TARGET
Build the specified stage as defined inside the Dockerfile. See the multi-stage build docs for details.
```
build:
  context: .
  target: prod
```
# cap_add, cap_drop
Add or drop container capabilities. See man 7 capabilities for a full list.

```
cap_add:
  - ALL

cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```
# cgroup_parent
Specify an optional parent cgroup for the container.
```
cgroup_parent: m-executor-abcd
```
# command
Override the default command.
```
command: bundle exec thin -p 3000
```
The command can also be a list, in a manner similar to dockerfile:

```
command: ["bundle", "exec", "thin", "-p", "3000"]
```
# configs
## SHORT SYNTAX
The short syntax variant only specifies the config name. This grants the container access to the config and mounts it at `/<config_name>` within the container. The source name and destination mountpoint are both set to the config name.
```
version: "3.7"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - my_config
      - my_other_config
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```
## LONG SYNTAX
The long syntax provides more granularity in how the config is created within the service’s task containers.
- source: The name of the config as it exists in Docker.
- target: The path and name of the file to be mounted in the service’s task containers. Defaults to `/<source>` if not specified.
- uid and gid: The numeric UID or GID that owns the mounted config file within in the service’s task containers. Both default to 0 on Linux if not specified. Not supported on Windows.
- mode: The permissions for the file that is mounted within the service’s task containers, in octal notation. For instance, 0444 represents world-readable. The default is 0444. Configs cannot be writable because they are mounted in a temporary filesystem, so if you set the writable bit, it is ignored. The executable bit can be set. If you aren’t familiar with UNIX file permission modes, you may find this permissions calculator useful.

```
version: "3.7"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - source: my_config
        target: /redis_config
        uid: '103'
        gid: '103'
        mode: 0440
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```
# container_name
Specify a custom container name, rather than a generated default name.
```
container_name: my-web-container

```
Because Docker container names must be unique, you cannot scale a service beyond 1 container if you have specified a custom name. Attempting to do so results in an error.
# credential_spec
Configure the credential spec for managed service account. This option is only used for services using Windows containers. The credential_spec must be in the format `file://<filename>` or `registry://<value-name>`.
```
credential_spec:
  file: my-credential-spec.json
```
## EXAMPLE GMSA CONFIGURATION
```
version: "3.8"
services:
  myservice:
    image: myimage:latest
    credential_spec:
      config: my_credential_spec

configs:
  my_credentials_spec:
    file: ./my-credential-spec.json|
```
# depends_on
Express dependency between services, Service dependencies cause the following behaviors:
- `docker-compose up` starts services in dependency order.
- `docker-compose up SERVICE` automatically includes SERVICE’s dependencies.
- `docker-compose stop` stops services in dependency order.
```
version: "3.7"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```
# deploy
Specify configuration related to the deployment and running of services. This only takes effect when deploying to a swarm with docker stack deploy, and is ignored by docker-compose up and docker-compose run.
```
version: "3.7"
services:
  redis:
    image: redis:alpine
    deploy:
      replicas: 6
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```
## ENDPOINT_MODE
Specify a service discovery method for external clients connecting to a swarm.

### endpoint_mode: vip
Docker assigns the service a virtual IP (VIP) that acts as the front end for clients to reach the service on a network. Docker routes requests between the client and available worker nodes for the service, without client knowledge of how many nodes are participating in the service or their IP addresses or ports. (This is the default.)
### endpoint_mode: dnsrr
DNS round-robin (DNSRR) service discovery does not use a single virtual IP. Docker sets up DNS entries for the service such that a DNS query for the service name returns a list of IP addresses, and the client connects directly to one of these. DNS round-robin is useful in cases where you want to use your own load balancer, or for Hybrid Windows and Linux applications.
```
version: "3.7"

services:
  wordpress:
    image: wordpress
    ports:
      - "8080:80"
    networks:
      - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: vip

  mysql:
    image: mysql
    volumes:
       - db-data:/var/lib/mysql/data
    networks:
       - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: dnsrr

volumes:
  db-data:

networks:
  overlay:
```
## LABELS
Specify labels for the service. These labels are only set on the service, and not on any containers for the service.
```
version: "3.7"
services:
  web:
    image: web
    deploy:
      labels:
        com.example.description: "This label will appear on the web service"
```
To set labels on containers instead, use the labels key outside of deploy:

```
version: "3.7"
services:
  web:
    image: web
    labels:
      com.example.description: "This label will appear on all containers for the web service"
```
## MODE
Either global (exactly one container per swarm node) or replicated (a specified number of containers). The default is replicated
```
version: "3.7"
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    deploy:
      mode: global
```
## PLACEMENT
Specify placement of constraints and preferences. See the docker service create documentation for a full description of the syntax and available types of constraints and preferences.
```
version: "3.7"
services:
  db:
    image: postgres
    deploy:
      placement:
        constraints:
          - node.role == manager
          - engine.labels.operatingsystem == ubuntu 14.04
        preferences:
          - spread: node.labels.zone
```
## REPLICAS
If the service is replicated (which is the default), specify the number of containers that should be running at any given time.

```
version: "3.7"
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 6
```
## RESOURCES
Configures resource constraints. 

In this general example, the redis service is constrained to use no more than 50M of memory and 0.50 (50% of a single core) of available processing time (CPU), and has 20M of memory and 0.25 CPU time reserved (as always available to it).
```
version: "3.7"
services:
  redis:
    image: redis:alpine
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 50M
        reservations:
          cpus: '0.25'
          memory: 20M
```
## Out Of Memory Exceptions (OOME)
If your services or containers attempt to use more memory than the system has available, you may experience an Out Of Memory Exception (OOME) and a container, or the Docker daemon, might be killed by the kernel OOM killer. 
To prevent this from happening, ensure that your application runs on hosts with adequate memory and see Understand the risks of running out of memory.
## RESTART_POLICY
Configures if and how to restart containers when they exit. Replaces restart.

1. condition: One of `none`, `on-failure` or `any` (default: `any`).
2. delay: How long to wait between restart attempts, specified as a duration (default: 0).
3. max_attempts: How many times to attempt to restart a container before giving up (default: never give up). If the restart does not succeed within the configured window, this attempt doesn’t count toward the configured max_attempts value. For example, if max_attempts is set to ‘2’, and the restart fails on the first attempt, more than two restarts may be attempted.
4. window: How long to wait before deciding if a restart has succeeded, specified as a duration (default: decide immediately).
```
version: "3.7"
services:
  redis:
    image: redis:alpine
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

## ROLLBACK_CONFIG
Configures how the service should be rollbacked in case of a failing update.
- parallelism: The number of containers to rollback at a time. If set to 0, all containers rollback simultaneously.
- delay: The time to wait between each container group’s rollback (default 0s).
- failure_action: What to do if a rollback fails. One of continue or pause (default pause)
- monitor: Duration after each task update to monitor for failure (ns|us|ms|s|m|h) (default 0s).
- max_failure_ratio: Failure rate to tolerate during a rollback (default 0).
- order: Order of operations during rollbacks. One of stop-first (old task is stopped before starting new one), or start-first (new task is started first, and the running tasks briefly overlap) (default stop-first).
## UPDATE_CONFIG
Configures how the service should be updated. Useful for configuring rolling updates.
- parallelism: The number of containers to update at a time.
- delay: The time to wait between updating a group of containers.
- failure_action: What to do if an update fails. One of continue, rollback, or pause (default: pause).
- monitor: Duration after each task update to monitor for failure (ns|us|ms|s|m|h) (default 0s).
- max_failure_ratio: Failure rate to tolerate during an update.
- order: Order of operations during updates. One of stop-first (old task is stopped before starting new one), or start-first (new task is started first, and the running tasks briefly overlap) (default stop-first) Note: Only supported for v3.4 and higher.
```
version: "3.7"
services:
  vote:
    image: dockersamples/examplevotingapp_vote:before
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
        order: stop-first
```
# NOT SUPPORTED FOR DOCKER STACK DEPLOY
The following sub-options (supported for `docker-compose up` and `docker-compose run`) are not supported for `docker stack deploy` or the deploy key.
- build
- cgroup_parent
- container_name
- devices
- tmpfs
- external_links
- links
- network_mode
- restart
- security_opt
- sysctls
- userns_mode

# devices
List of device mappings. Uses the same format as the `--device` docker client create option.
```
devices:
  - "/dev/ttyUSB0:/dev/ttyUSB0"
```
# depends_on
Express dependency between services, Service dependencies cause the following behaviors:
- `docker-compose up` starts services in dependency order
- `docker-compose up` SERVICE automatically includes SERVICE’s dependencies
- `docker-compose stop` stops services in dependency order
```
version: "3.7"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```
- The depends_on option is ignored when deploying a stack in swarm mode with a version 3 Compose file.
# dns
Custom DNS servers. Can be a single value or a list.
```
dns: 8.8.8.8
```
```
dns:
  - 8.8.8.8
  - 9.9.9.9
```
# dns_search
Custom DNS search domains. Can be a single value or a list.
```
dns_search: example.com
```
```
dns_search:
  - dc1.example.com
  - dc2.example.com
```
# entrypoint
Override the default entrypoint.
```
entrypoint: /code/entrypoint.sh
```
The entrypoint can also be a list, in a manner similar to dockerfile:

```
entrypoint:
    - php
    - -d
    - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
    - -d
    - memory_limit=-1
    - vendor/bin/phpunit
```
# env_file
Add environment variables from a file. Can be a single value or a list.
```
env_file: .env
```
```
env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```
```
# common.env
VAR=hello
```
# environment
- Add environment variables. 
- You can use either an array or a dictionary. 
- Any boolean values; true, false, yes no, need to be enclosed in quotes to ensure they are not converted to True or False by the YML parser.
```
environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:
```
```
environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```
# expose
Expose ports without publishing them to the host machine - they’ll only be accessible to linked services. Only the internal port can be specified.
```
expose:
 - "3000"
 - "8000"
```
# external_links
- Link to containers started outside this docker-compose.yml or even outside of Compose, especially for containers that provide shared or common services. 
- external_links follow semantics similar to the legacy option links when specifying both the container name and the link alias (CONTAINER:ALIAS).
```
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```
# extra_hosts
Add hostname mappings. Use the same values as the docker client --add-host parameter.
```
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"
```
# healthcheck
- Configure a check that’s run to determine whether or not containers for this service are “healthy”. 
```
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
```
To disable any default healthcheck set by the image, you can use disable: true. This is equivalent to specifying test: ["NONE"].
```
healthcheck:
  disable: true
```
# image
Specify the image to start the container from. Can either be a repository/tag or a partial image ID.
```
image: redis
image: ubuntu:14.04
image: tutum/influxdb
image: example-registry.com:4000/postgresql
image: a4bc65fd
```
# init
Run an init inside the container that forwards signals and reaps processes. Set this option to true to enable this feature for the service.
```
version: "3.7"
services:
  web:
    image: alpine:latest
    init: true
```
# isolation
- Specify a container’s isolation technology. 
- On Linux, the only supported value is default
- On Windows, acceptable values are default, process and hyperv
# labels
Add metadata to containers using Docker labels. You can use either an array or a dictionary.
```
labels:
  com.example.description: "Accounting webapp"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""
```
```
labels:
  - "com.example.description=Accounting webapp"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```
# links
Link to containers in another service. Either specify both the service name and a link alias (SERVICE:ALIAS), or just the service name.
```
web:
  links:
   - db
   - db:database
   - redis
```
- Links are not required to enable services to communicate 
- by default, any service can reach any other service at that service’s name.
# logging
Logging configuration for the service.
```
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```
The driver name specifies a logging driver for the service’s containers, as with the `--log-driver` option for docker run
- The default value is json-file.
```
driver: "json-file"
driver: "syslog"
driver: "none"
```
Specify logging options for the logging driver with the options key, as with the --log-opt option for docker run.
```
driver: "syslog"
options:
  syslog-address: "tcp://192.168.0.42:123"
```
# network_mode
Network mode. Use the same values as the `docker client --network` parameter, plus the special form service:[service name].
```
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```
- This option is ignored when deploying a stack in swarm mode with a (version 3) Compose file.
- network_mode: "host" cannot be mixed with links.
# networks
Networks to join, referencing entries under the top-level networks key.
services:
  some-service:
    networks:
     - some-network
     - other-network
# ALIASES
- Aliases (alternative hostnames) for this service on the network.
- Other containers on the same network can use either the service name or this alias to connect to one of the service’s containers.
- Since aliases is network-scoped, the same service can have different aliases on different networks.
- A network-wide alias can be shared by multiple containers, and even by multiple services. If it is, then exactly which container the name resolves to is not guaranteed.
```
services:
  some-service:
    networks:
      some-network:
        aliases:
         - alias1
         - alias3
      other-network:
        aliases:
         - alias2
```
# IPV4_ADDRESS, IPV6_ADDRESS
Specify a static IP address for containers for this service when joining the network.
The corresponding network configuration in the top-level networks section must have an ipam block with subnet configurations covering each static address.
- If IPv6 addressing is desired, the enable_ipv6 option must be set, and you must use a version 2.x Compose file. 
- IPv6 options do not currently work in swarm mode.
```
version: "3.7"

services:
  app:
    image: nginx:alpine
    networks:
      app_net:
        ipv4_address: 172.16.238.10
        ipv6_address: 2001:3984:3989::10

networks:
  app_net:
    ipam:
      driver: default
      config:
        - subnet: "172.16.238.0/24"
        - subnet: "2001:3984:3989::/64"
```
# pid
```
pid: "host"
```
- Sets the PID mode to the host PID mode.
- This turns on sharing between container and the host operating system the PID address space.
# ports
- Expose ports.
- Port mapping is incompatible with network_mode: host
## SHORT SYNTAX
Either specify both ports (HOST:CONTAINER), or just the container port (an ephemeral host port is chosen).
```
ports:
 - "3000"
 - "3000-3005"
 - "8000:8000"
 - "9090-9091:8080-8081"
 - "49100:22"
 - "127.0.0.1:8001:8001"
 - "127.0.0.1:5000-5010:5000-5010"
 - "6060:6060/udp"
```
## LONG SYNTAX
The long form syntax allows the configuration of additional fields that can’t be expressed in the short form.
- target: the port inside the container
- published: the publicly exposed port
- protocol: the port protocol (tcp or udp)
- mode: host for publishing a host port on each node, or ingress for a swarm mode port to be load balanced.
```
ports:
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```
# restart
- `no` is the default restart policy, and it does not restart a container under any circumstance.
- When always is specified, the container always restarts. 
- The on-failure policy restarts a container if the exit code indicates an on-failure error.
```
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```
- This option is ignored when deploying a stack in swarm mode
# secrets
Grant access to secrets on a per-service basis using the per-service secrets configuration. Two different syntax variants are supported.
## SHORT SYNTAX
- The short syntax variant only specifies the secret name.
- This grants the container access to the secret and mounts it at `/run/secrets/<secret_name>` within the container.
- The source name and destination mountpoint are both set to the secret name.
-  If the external secret does not exist, the stack deployment fails with a secret not found error.
```
version: "3.7"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - my_secret
      - my_other_secret
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```
- The value of my_secret is set to the contents of the file ./my_secret.txt
- The value of my_other_secret is defined as an external resource, which means that it has already been defined in Docker, either by running the docker secret create command or by another stack deployment
## LONG SYNTAX
The long syntax provides more granularity in how the secret is created within the service’s task containers.
- source: The name of the secret as it exists in Docker.
- target: The name of the file to be mounted in /run/secrets/ in the service’s task containers. Defaults to source if not specified.
- uid and gid: The numeric UID or GID that owns the file within /run/secrets/ in the service’s task containers. Both default to 0 if not specified.
- mode: The permissions for the file to be mounted in /run/secrets/ in the service’s task containers, in octal notation. For instance, 0444 represents world-readable. The default in Docker 1.13.1 is 0000, but is be 0444 in newer versions. Secrets cannot be writable because they are mounted in a temporary filesystem, so if you set the writable bit, it is ignored. The executable bit can be set. If you aren’t familiar with UNIX file permission modes, you may find this permissions calculator useful.
```
version: "3.7"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - source: my_secret
        target: redis_secret
        uid: '103'
        gid: '103'
        mode: 0440
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```
# security_opt
Override the default labeling scheme for each container.
```
security_opt:
  - label:user:USER
  - label:role:ROLE
```
# stop_grace_period
- Specify how long to wait when attempting to stop a container if it doesn’t handle SIGTERM (or whatever stop signal has been specified with stop_signal), before sending SIGKILL. 
- Specified as a duration.
```
stop_grace_period: 1s
stop_grace_period: 1m30s
```
# stop_signal
- Sets an alternative signal to stop the container. 
- By default stop uses `SIGTERM`. 
- Setting an alternative signal using stop_signal causes stop to send that signal instead.
```
stop_signal: SIGUSR1
```
# sysctls
Kernel parameters to set in the container. You can use either an array or a dictionary.
```
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0
```
```
sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```
# tmpfs
Mount a temporary file system inside the container. Can be a single value or a list.
```
tmpfs: /run
```
```
tmpfs:
  - /run
  - /tmp
```
- This option is ignored when deploying a stack in swarm mode 
Mount a temporary file system inside the container. Size parameter specifies the size of the tmpfs mount in bytes. Unlimited by default.
```
 - type: tmpfs
     target: /app
     tmpfs:
       size: 1000
```  
# ulimits
Override the default ulimits for a container. You can either specify a single limit as an integer or soft/hard limits as a mapping.
```
ulimits:
  nproc: 65535
  nofile:
    soft: 20000
    hard: 40000
```
# userns_mode
```
userns_mode: "host"
```
# volumes
- Mount host paths or named volumes, specified as sub-options to a service.
- You can mount a host path as part of a definition for a single service, and there is no need to define it in the top level volumes key.
- But, if you want to reuse a volume across multiple services, then define a named volume in the top-level volumes key. Use named volumes with services, swarms, and stack files.

```
version: "3.7"
services:
  web:
    image: nginx:alpine
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

  db:
    image: postgres:latest
    volumes:
      - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
      - "dbdata:/var/lib/postgresql/data"

volumes:
  mydata:
  dbdata:
```
## SHORT SYNTAX
```
volumes:
  # Just specify a path and let the Engine create a volume
  - /var/lib/mysql

  # Specify an absolute path mapping
  - /opt/data:/var/lib/mysql

  # Path on the host, relative to the Compose file
  - ./cache:/tmp/cache

  # User-relative path
  - ~/configs:/etc/configs/:ro

  # Named volume
  - datavolume:/var/lib/mysql
```
## LONG SYNTAX
The long form syntax allows the configuration of additional fields that can’t be expressed in the short form.
- type: the mount type volume, bind, tmpfs or npipe
- source: the source of the mount, a path on the host for a bind mount, or the name of a volume defined in the top-level volumes key. Not applicable for a tmpfs mount.
- target: the path in the container where the volume is mounted
- read_only: flag to set the volume as read-only
- bind: configure additional bind options
- propagation: the propagation mode used for the bind
- volume: configure additional volume options
- nocopy: flag to disable copying of data from a container when a volume is created
- tmpfs: configure additional tmpfs options
- size: the size for the tmpfs mount in bytes
- consistency: the consistency requirements of the mount, one of consistent (host and container have identical view), cached (read cache, host view is authoritative) or delegated (read-write cache, container’s view is authoritative)
```
version: "3.7"
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

networks:
  webnet:

volumes:
  mydata:
```
## VOLUMES FOR SERVICES, SWARMS, AND STACK FILES
- When working with services, swarms, and docker-stack.yml files, keep in mind that the tasks (containers) backing a service can be deployed on any node in a swarm, and this may be a different node each time the service is updated.
- In the absence of having named volumes with specified sources, Docker creates an anonymous volume for each task backing a service. Anonymous volumes do not persist after the associated containers are removed.
- If you want your data to persist, use a named volume and a volume driver that is multi-host aware, so that the data is accessible from any node. Or, set constraints on the service so that its tasks are deployed on a node that has the volume present.
```
version: "3.7"
services:
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
```
## CACHING OPTIONS FOR VOLUME MOUNTS (DOCKER DESKTOP FOR MAC)
- consistent: Full consistency. The container runtime and the host maintain an identical view of the mount at all times. This is the default.
- cached: The host’s view of the mount is authoritative. There may be delays before updates made on the host are visible within a container.
- delegated: The container runtime’s view of the mount is authoritative. There may be delays before updates made in a container are visible on the host.
```
version: "3.7"
services:
  php:
    image: php:7.1-fpm
    ports:
      - "9000"
    volumes:
      - .:/var/www/project:cached
```
# domainname, hostname, ipc, mac_address, privileged, read_only, shm_size, stdin_open, tty, user, working_dir
Each of these is a single value, analogous to its docker run counterpart. Note that mac_address is a legacy option.
```
user: postgresql
working_dir: /code

domainname: foo.com
hostname: foo
ipc: host
mac_address: 02:42:ac:11:65:43

privileged: true


read_only: true
shm_size: 64M
stdin_open: true
tty: true
```
# Specifying durations
Some configuration options, such as the interval and timeout sub-options for check, accept a duration as a string in a format that looks like this:
```
2.5s
10s
1m30s
2h32m
5h34m56s
```
The supported units are 
- us 
- ms
- s
- m
- h
# Specifying byte values
Some configuration options, such as the shm_size sub-option for build, accept a byte value as a string in a format that looks like this:
```
2b
1024kb
2048k
300m
1gb
```
The supported units are 
- b
- k
- m 
- g
their alternative notation kb, mb and gb. Decimal values are not supported at this time.
# Volume configuration reference
```
version: "3.7"

services:
  db:
    image: db
    volumes:
      - data-volume:/var/lib/db
  backup:
    image: backup-service
    volumes:
      - data-volume:/var/lib/backup/data

volumes:
  data-volume:
```
## driver
```
driver: foobar
```
## driver_opts
```
volumes:
  example:
    driver_opts:
      type: "nfs"
      o: "addr=10.40.0.199,nolock,soft,rw"
      device: ":/docker/example"
```
## external
If set to true, specifies that this volume has been created outside of Compose. 
## name
Set a custom name for this volume. The name field can be used to reference volumes that contain special characters. The name is used as is and will not be scoped with the stack name.
```
version: "3.7"
volumes:
  data:
    name: my-app-data
```
It can also be used in conjunction with the external property:
```
version: "3.7"
volumes:
  data:
    external: true
    name: my-app-data
```
# Network configuration reference
## driver
Specify which driver should be used for this network.
```
driver: overlay
```
## BRIDGE
- Docker defaults to using a bridge network on a single host.
## OVERLAY
The overlay driver creates a named network across multiple nodes in a swarm.
## HOST OR NONE
```
version: "3.7"
services:
  web:
    networks:
      hostnet: {}

networks:
  hostnet:
    external: true
    name: host
```
## driver_opts
```
driver_opts:
  foo: "bar"
  baz: 1
```
## attachable
- Only used when the driver is set to overlay. 
-  If set to true, then standalone containers can attach to this network, in addition to services. 
```
networks:
  mynet1:
    driver: overlay
    attachable: true
```
## internal
## labels
## external
## name
# configs configuration reference
The top-level configs declaration defines or references configs that can be granted to the services in this stack. The source of the config is either file or external.

- file: The config is created with the contents of the file at the specified path.
- external: If set to true, specifies that this config has already been created. Docker does not attempt to create it, and if it does not exist, a config not found error occurs.
- name: The name of the config object in Docker. This field can be used to reference configs that contain special characters. The name is used as is and will not be scoped with the stack name. Introduced in version 3.5 file format.
```
configs:
  my_first_config:
    file: ./config_data
  my_second_config:
    external: true
```
Another variant for external configs is when the name of the config in Docker is different from the name that exists within the service. The following example modifies the previous one to use the external config called redis_config.

```
configs:
  my_first_config:
    file: ./config_data
  my_second_config:
    external:
      name: redis_config
```
# secrets configuration reference
The top-level secrets declaration defines or references secrets that can be granted to the services in this stack. The source of the secret is either file or external.

- file: The secret is created with the contents of the file at the specified path.
- external: If set to true, specifies that this secret has already been created. Docker does not attempt to create it, and if it does not exist, a secret not found error occurs.
- name: The name of the secret object in Docker. This field can be used to reference secrets that contain special characters. The name is used as is and will not be scoped with the stack name. Introduced in version 3.5 file format.
```
secrets:
  my_first_secret:
    file: ./secret_data
  my_second_secret:
    external: true
```
# Variable substitution
- Your configuration options can contain environment variables. 
- Compose uses the variable values from the shell environment in which docker-compose is run. 
- For example, suppose the shell contains POSTGRES_VERSION=9.3 and you supply this configuration:
```
db:
  image: "postgres:${POSTGRES_VERSION}"
```
When you run docker-compose up with this configuration, Compose looks for the POSTGRES_VERSION environment variable in the shell and substitutes its value in. 
You can set default values for environment variables using a .env file, which Compose automatically looks for. Values set in the shell environment override those set in the .env file.
# Extension fields
It is possible to re-use configuration fragments using extension fields. Those special fields can be of any format as long as they are located at the root of your Compose file and their name start with the x- character sequence.
```
version: '3.4'
x-custom:
  items:
    - a
    - b
  options:
    max-size: '12m'
  name: "custom"
  ```
  The contents of those fields are ignored by Compose, but they can be inserted in your resource definitions using YAML anchors.
  ```
  logging:
  options:
    max-size: '12m'
    max-file: '5'
  driver: json-file
  ```
  You may write your Compose file as follows:
  ```
version: '3.4'
x-logging:
  &default-logging
  options:
    max-size: '12m'
    max-file: '5'
  driver: json-file

services:
  web:
    image: myapp/web:latest
    logging: *default-logging
  db:
    image: mysql:latest
    logging: *default-logging
  ```
It is also possible to partially override values in extension fields using the YAML merge type. For example:
```
version: '3.4'
x-volumes:
  &default-volume
  driver: foobar-storage

services:
  web:
    image: myapp/web:latest
    volumes: ["vol1", "vol2", "vol3"]
volumes:
  vol1: *default-volume
  vol2:
    << : *default-volume
    name: volume02
  vol3:
    << : *default-volume
    driver: default
    name: volume-local
```
