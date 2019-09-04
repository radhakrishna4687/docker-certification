# About storage drivers
- Storage drivers allow you to create data in the writable layer of your container. 
- The files won’t be persisted after the container is deleted, and both read and write speeds are lower than native file system performance.
# Images and layers
- A Docker image is built up from a series of layers.
- Each layer represents an instruction in the image’s Dockerfile. 
- Each layer except the very last one is read-only.

Let's discuss following sample `Dockerfile`
```
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```
- This Dockerfile contains four commands, each of which creates a layer.
- The FROM statement starts out by creating a layer from the ubuntu:18.04 image.
- The COPY command adds some files from your Docker client’s current directory. 
- The RUN command builds your application using the make command.
- last layer specifies what command to run within the container.

Following are more details.
- Each layer is only a set of differences from the layer before it. 
- The layers are stacked on top of each other. 
- When you create a new container, you add a new writable layer on top of the underlying layers.
- This layer is often called the “container layer”.
- All changes made to the running container, such as writing new files, modifying existing files, and deleting files, are written to this thin writable container layer. 
- The diagram below shows a container based on the Ubuntu 18.04 image.
![](https://docs.docker.com/storage/storagedriver/images/container-layers.jpg)

- A storage driver handles the details about the way these layers interact with each other. 
- Different storage drivers are available, which have advantages and disadvantages in different situations.

# Container and layers
- The major difference between a container and an image is the top writable layer. 
- All writes to the container that add new or modify existing data are stored in this writable layer. 
- When the container is deleted, the writable layer is also deleted. The underlying image remains unchanged.
- Because each container has its own writable container layer, and all changes are stored in this container layer, multiple containers can share access to the same underlying image and yet have their own data state. 
![](https://docs.docker.com/storage/storagedriver/images/sharing-layers.jpg)
- Docker uses storage drivers to manage the contents of the image layers and the writable container layer. 
- Each storage driver handles the implementation differently, but all drivers use stackable image layers and the copy-on-write (CoW) strategy.
# Container size on disk
```
docker ps -s
```
- size: the amount of data (on disk) that is used for the writable layer of each container.
- virtual size: the amount of data used for the read-only image data used by the container plus the container’s writable layer size. Multiple containers may share some or all read-only image data. Two containers started from the same image share 100% of the read-only data, while two containers with different images which have layers in common share those common layers. Therefore, you can’t just total the virtual sizes. This over-estimates the total disk usage by a potentially non-trivial amount.

This also does not count the following additional ways a container can take up disk space:
- Disk space used for log files if you use the json-file logging driver. This can be non-trivial if your container generates a large amount of logging data and log rotation is not configured.
- Volumes and bind mounts used by the container.
- Disk space used for the container’s configuration files, which are typically small.
- Memory written to disk (if swapping is enabled).
- Checkpoints, if you’re using the experimental checkpoint/restore feature.
# The copy-on-write (CoW) strategy
- Copy-on-write is a strategy of sharing and copying files for maximum efficiency.
- If a file or directory exists in a lower layer within the image, and another layer (including the writable layer) needs read access to it, it just uses the existing file.
- The first time another layer needs to modify the file (when building the image or running the container), the file is copied into that layer and modified.
- This minimizes I/O and the size of each of the subsequent layers. 

Following are advantages. 
## Sharing promotes smaller images
When you use `docker pull` to pull down an image from a repository, or when you create a container from an image that does not yet exist locally, each layer is pulled down separately, and stored in Docker’s local storage area, which is usually `/var/lib/docker/` on Linux hosts. 
You can see these layers being pulled in this example:
```
docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
f476d66f5408: Pull complete
8882c27f669e: Pull complete
d9af21273955: Pull complete
f5029279ec12: Pull complete
Digest: sha256:ab6cb8de3ad7bb33e2534677f865008535427390b117d7939193f8d1a6613e34
Status: Downloaded newer image for ubuntu:18.04
```
Each of these layers is stored in its own directory inside the Docker host’s local storage area. To examine the layers on the filesystem, list the contents of `/var/lib/docker/<storage-driver>`
This example uses the overlay2 storage driver:
```
ls /var/lib/docker/overlay2
16802227a96c24dcbeab5b37821e2b67a9f921749cd9a2e386d5a6d5bc6fc6d3
377d73dbb466e0bc7c9ee23166771b35ebdbe02ef17753d79fd3571d4ce659d7
3f02d96212b03e3383160d31d7c6aeca750d2d8a1879965b89fe8146594c453d
ec1ec45792908e90484f7e629330666e7eee599f08729c93890a7205a6ba35f5
l
```
- The directory names do not correspond to the layer IDs (this has been true since Docker 1.10).

Now imagine that you have two different Dockerfiles. You use the first one to create an image called `acme/my-base-image:1.0`.
```
FROM ubuntu:18.04
COPY . /app
```
The second one is based on acme/my-base-image:1.0, but has some additional layers:
```
FROM acme/my-base-image:1.0
CMD /app/hello.sh
```
- The second image contains all the layers from the first image, plus a new layer with the CMD instruction, and a read-write container layer.
- Docker already has all the layers from the first image, so it does not need to pull them again.
- The two images share any layers they have in common.
- If you build images from the two Dockerfiles, you can use `docker image ls` and `docker history` commands to verify that the cryptographic IDs of the shared layers are the same.

# Copying makes containers efficient
- When you start a container, a thin writable container layer is added on top of the other layers.
- Any changes the container makes to the filesystem are stored here.
- Any files the container does not change do not get copied to this writable layer.
- This means that the writable layer is as small as possible.
- When an existing file in a container is modified, the storage driver performs a copy-on-write operation. 
- The specifics steps involved depend on the specific storage driver.
- Following are few drivers 
1. aufs
2. overlay
3. overlay2

The copy-on-write operation follows following rough sequence:
- Search through the image layers for the file to update. The process starts at the newest layer and works down to the base layer one layer at a time. When results are found, they are added to a cache to speed future operations.
- Perform a copy_up operation on the first copy of the file that is found, to copy the file to the container’s writable layer.
- Any modifications are made to this copy of the file, and the container cannot see the read-only copy of the file that exists in the lower layer.

- Containers that write a lot of data consume more space than containers that do not. This is because most write operations consume new space in the container’s thin writable top layer.
- A copy_up operation can incur a noticeable performance overhead. This overhead is different depending on which storage driver is in use. 
- Large files, lots of layers, and deep directory trees can make the impact more noticeable.
- This is mitigated by the fact that each copy_up operation only occurs the first time a given file is modified.
- Not only does copy-on-write save space, but it also reduces start-up time. 
- When you start a container (or multiple containers from the same image), Docker only needs to create the thin writable container layer.
- 