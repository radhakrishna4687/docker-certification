# Use the AUFS storage driver
- AUFS is a union filesystem.
- The aufs storage driver was previously the default storage driver used for managing images and layers on Docker for Ubuntu, and for Debian versions prior to Stretch.
-  If your Linux kernel is version 4.0 or higher, and you use Docker Engine - Community, consider using the newer overlay2, which has potential performance advantages over the aufs storage driver.

# Configure Docker with the aufs storage driver
- If the AUFS driver is loaded into the kernel when you start Docker, and no other storage driver is configured, Docker uses it by default.
## Use the following command to verify that your kernel supports AUFS.
```
grep aufs /proc/filesystems
```
## Check which storage driver Docker is using.
```
docker info
```
## AUFS is not included in the kernel 
If you are using a different storage driver, either AUFS is not included in the kernel (in which case a different default driver is used) or that Docker has been explicitly configured to use a different driver.
1. Check /etc/docker/daemon.json
2. `ps auxw | grep dockerd` to see if Docker has been started with the --storage-driver flag.
```
root      2120  0.5 10.2 534680 103996 ?       Sl   00:03   2:47 dockerd --data-root /var/lib/docker -H unix:// --label provider=virtualbox -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=/var/lib/boot2docker/ca.pem --tlskey=/var/lib/boot2docker/server-key.pem --tlscert=/var/lib/boot2docker/server.pem --storage-driver overlay2 --pidfile /var/run/docker.pid
docker   16499  0.0  0.0   4164  1008 pts/0    S+   08:09   0:00 grep dockerd
```
# How the aufs storage driver works?
- AUFS is a union filesystem, which means that it layers multiple directories on a single Linux host and presents them as a single directory.
- These directories are called branches in AUFS terminology, and layers in Docker terminology.
- The unification process is referred to as a union mount.
The diagram below shows a Docker container based on the ubuntu:latest image.
![](https://docs.docker.com/storage/storagedriver/images/aufs_layers.jpg)

- Each image layer, and the container layer, are represented on the Docker host as subdirectories within /var/lib/docker/. 
- The union mount provides the unified view of all layers. 
- The directory names do not directly correspond to the IDs of the layers themselves.
- AUFS uses the Copy-on-Write (CoW) strategy to maximize storage efficiency and minimize overhead.
- All of the information about the image and container layers is stored in subdirectories of /var/lib/docker/aufs/
# How container reads and writes work with aufs?
## Reading files
### The file does not exist in the container layer
- If a container opens a file for read access and the file does not already exist in the container layer, the storage driver searches for the file in the image layers, starting with the layer just below the container layer. 
- It is read from the layer where it is found.
### The file only exists in the container layer
- If a container opens a file for read access and the file exists in the container layer, it is read from there.
### The file exists in both the container layer and the image layer
- If a container opens a file for read access and the file exists in the container layer and one or more image layers, the file is read from the container layer.
-  Files in the container layer obscure files with the same name in the image layers.
## Modifying files or directories
### Writing to a file for the first time:
- The first time a container writes to an existing file, that file does not exist in the container (upperdir).
- The aufs driver performs a copy_up operation to copy the file from the image layer where it exists to the writable container layer.
- The container then writes the changes to the new copy of the file in the container layer.
- However, AUFS works at the file level rather than the block level. This means that all copy_up operations copy the entire file, even if the file is very large and only a small part of it is being modified.
- This can have a noticeable impact on container write performance. 
- AUFS can suffer noticeable latencies when searching for files in images with many layers.
- However, it is worth noting that the copy_up operation only occurs the first time a given file is written to. 
- Subsequent writes to the same file operate against the copy of the file already copied up to the container.
### Deleting files and directories
- When a file is deleted within a container, a whiteout file is created in the container layer. The version of the file in the image layer is not deleted (because the image layers are read-only). However, the whiteout file prevents it from being available to the container.
- When a directory is deleted within a container, an opaque file is created in the container layer. This works in the same way as a whiteout file and effectively prevents the directory from being accessed, even though it still exists in the image layer.
### Renaming directories
- Calling rename(2) for a directory is not fully supported on AUFS. 
- 