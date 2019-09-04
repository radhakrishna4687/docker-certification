# Choose the -v or --mount flag
- Originally, the `-v` or `--volume` flag was used for standalone containers and the `--mount` flag was used for swarm services. 
- However, starting with `Docker 17.06`, you can also use `--mount` with standalone containers. 
-  In general, `--mount` is more explicit and verbose.
- The biggest difference is that the `-v` syntax combines all the options together in one field, while the `--mount` syntax separates them.
- New users should try --mount syntax which is simpler than --volume syntax.
- If you need to specify volume driver options, you must use `--mount`.
## --volume
Consists of three fields, separated by colon characters (:). The fields must be in the correct order, and the meaning of each field is not immediately obvious.
- In the case of named volumes, the first field is the name of the volume, and is unique on a given host machine. For anonymous volumes, the first field is omitted.
- The second field is the path where the file or directory are mounted in the container.
- The third field is optional, and is a comma-separated list of options, such as ro. These options are discussed below.
## --mount
Consists of multiple key-value pairs, separated by commas and each consisting of a `<key>=<value>` tuple. The `--mount` syntax is more verbose than -v or --volume, but the order of the keys is not significant, and the value of the flag is easier to understand.
- The `type` of the mount, which can be `bind`, `volume`, or `tmpfs`. This topic discusses volumes, so the type is always volume.
- The `source` of the mount. For named volumes, this is the name of the volume. For anonymous volumes, this field is omitted. May be specified as source or src.
- The `destination` takes as its value the path where the file or directory is mounted in the container. May be specified as destination, dst, or target.
- The `readonly` option, if present, causes the bind mount to be mounted into the container as read-only.
- The `volume-opt` option, which can be specified more than once, takes a key-value pair consisting of the option name and its value.
```
docker service create \
     --mount 'type=volume,src=<VOLUME-NAME>,dst=<CONTAINER-PATH>,volume-driver=local,volume-opt=type=nfs,volume-opt=device=<nfs-server>:<nfs-path>,"volume-opt=o=addr=<nfs-address>,vers=4,soft,timeo=180,bg,tcp,rw"'
    --name myservice \
    <IMAGE>
```
- When using volumes with services, only `--mount` is supported.
# Create and manage volumes
## Create a volume:
```
docker volume create my-vol
```
## List volumes:
```
docker volume ls
```
## Inspect a volume:
```
docker  volume inspect vol-name
```
```
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
```
## Remove a volume:
```
docker volume rm my-vol
```
# Start a container with a volume
- If you start a container with a volume that does not yet exist, Docker creates the volume for you.
```
docker run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest
```
# Start a service with volumes
- When you start a service and define a volume, each service container uses its own local volume.
- None of the containers can share this data if you use the `local` volume driver, but some volume drivers do support shared storage.
- Docker for AWS and Docker for Azure both support persistent storage using the Cloudstor plugin.
- The following example starts a nginx service with four replicas, each of which uses a local volume called myvol2.
```
docker service create -d \
  --replicas=4 \
  --name devtest-service \
  --mount source=myvol2,target=/app \
  nginx:latest
```
```
docker service ps devtest-service
```
- Removing the service does not remove any volumes created by the service. Volume removal is a separate step.

# Populate a volume using a container
```
docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html \
  nginx:latest
  ```
# Use a read-only volume
```
docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
  nginx:latest
  ```
# Share data among machines
When building fault-tolerant applications, you might need to configure multiple replicas of the same service to have access to the same files.
![](https://docs.docker.com/storage/images/volumes-shared-storage.svg)

There are several ways to achieve this when developing your applications.
1. Add logic to your application to store files on a cloud object storage system like Amazon S3. 
2. Create volumes with a driver that supports writing files to an external storage system like NFS or Amazon S3.

- Volume drivers allow you to abstract the underlying storage system from the application logic.

# Use a volume driver
## Initial set-up
```
docker plugin install --grant-all-permissions vieux/sshfs
```
## Create a volume using a volume driver
```
docker volume create --driver vieux/sshfs \
  -o sshcmd=test@node2:/home/test \
  -o password=testpassword \
  sshvolume
```
## Start a container which creates a volume using a volume driver
```
docker run -d \
  --name sshfs-container \
  --volume-driver vieux/sshfs \
  --mount src=sshvolume,target=/app,volume-opt=sshcmd=test@node2:/home/test,volume-opt=password=testpassword \
  nginx:latest
  ```
## Create a service which creates an NFS volume
### NFSV3
```
docker service create -d \
  --name nfs-service \
  --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/var/docker-nfs,volume-opt=o=addr=10.0.0.10' \
  nginx:latest
```
### NFSV4
```
docker service create -d \
    --name nfs-service \
    --mount 'type=volume,source=nfsvolume,target=/app,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/,"volume-opt=o=10.0.0.10,rw,nfsvers=4,async"' \
    nginx:latest
```
# Backup, restore, or migrate data volumes
# Remove all volumes
To remove all unused volumes and free up space:
```
docker volume prune
```
