# Use bind mounts
- When you use a bind mount, a file or directory on the host machine is mounted into a container.
- By contrast, when you use a volume, a new directory is created within Docker’s storage directory on the host machine, and Docker manages that directory’s contents.
- The file or directory does not need to exist on the Docker host already. It is created on demand if it does not yet exist. 
![](https://docs.docker.com/storage/images/types-of-mounts-bind.png)
# Start a container with a bind mount
```
docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app \
  nginx:latest
```
# Mount into a non-empty directory on the container
```
docker run -d \
  -it \
  --name broken-container \
  --mount type=bind,source=/tmp,target=/usr \
  nginx:latest
  ```
# Use a read-only bind mount
```
docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app,readonly \
  nginx:latest
```
# Configure bind propagation
- Bind propagation defaults to `rprivate` for both bind mounts and volumes.
- It is only configurable for bind mounts, and only on Linux host machines.
- Bind propagation refers to whether or not mounts created within a given bind-mount or named volume can be propagated to replicas of that mount. 
# Configure the selinux label
# Configure mount consistency for macOS
- Docker Desktop for Mac uses `osxfs` to propagate directories and files shared from macOS to the Linux VM.
- This propagation makes these directories and files available to Docker containers running on Docker Desktop for Mac.
```
docker run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,destination=/app,consistency=cached \
  nginx:latest
  ```
  
