# Use the Device Mapper storage driver
- Device Mapper is a kernel-based framework that underpins many advanced volume management technologies on Linux. 
- Docker’s devicemapper storage driver leverages the thin provisioning and snapshotting capabilities of this framework for image and container management. 
- This article refers to the Device Mapper storage driver as devicemapper, and the kernel framework as Device Mapper.
- For the systems where it is supported, devicemapper support is included in the Linux kernel.
- The devicemapper driver uses block devices dedicated to Docker and operates at the block level, rather than the file level. 
- These devices can be extended by adding physical storage to your Docker host, and they perform better than using a filesystem at the operating system (OS) level.
# Configure Docker with the devicemapper storage driver
## Configure loop-lvm mode for testing
- The loop-lvm mode makes use of a `loopback` mechanism that allows files on the local disk to be read from and written to as if they were an actual physical disk or block device.
- However, the addition of the loopback mechanism, and interaction with the OS filesystem layer, means that IO operations can be slow and resource-intensive. 
- Use of loopback devices can also introduce race conditions. 
- However, setting up loop-lvm mode can help identify basic issues (such as missing user space packages, kernel drivers, etc.) ahead of attempting the more complex set up required to enable direct-lvm mode. 
- loop-lvm mode should therefore only be used to perform rudimentary testing prior to configuring direct-lvm.
### Stop Docker
```
sudo systemctl stop docker
```
### Edit /etc/docker/daemon.json
```
{
  "storage-driver": "devicemapper"
}
```
### Start Docke
```
sudo systemctl start docker
```
### Verify that the daemon is using the devicemapper storage driver
```
docker info
```
## Configure direct-lvm mode for production
- Production hosts using the devicemapper storage driver must use direct-lvm mode. 
- This mode uses block devices to create the thin pool. 
- This is faster than using loopback devices, uses system resources more efficiently, and block devices can grow as needed. 
- However, more setup is required than in loop-lvm mode.
### ALLOW DOCKER TO CONFIGURE DIRECT-LVM MODE
Edit the daemon.json file and set the appropriate options, then restart Docker for the changes to take effect. 
```
{
  "storage-driver": "devicemapper",
  "storage-opts": [
    "dm.directlvm_device=/dev/xdf",
    "dm.thinp_percent=95",
    "dm.thinp_metapercent=1",
    "dm.thinp_autoextend_threshold=80",
    "dm.thinp_autoextend_percent=20",
    "dm.directlvm_device_force=false"
  ]
}
```
## CONFIGURE DIRECT-LVM MODE MANUALLY
### Identify the block device you want to use
The device is located under /dev/ (such as /dev/xvdf) and needs enough free space to store the images and container layers for the workloads that host runs. A solid state drive is ideal.
### Stop Docker
```
sudo systemctl stop docker
```
### Install the following packages
#### RHEL / CentOS
device-mapper-persistent-data, lvm2, and all dependencies
#### Ubuntu / Debian
thin-provisioning-tools, lvm2, and all dependencies
### Create a physical volume on your block device
```
sudo pvcreate /dev/xvdf
```
### Create a docker volume group
```
sudo vgcreate docker /dev/xvdf
```
### Create two logical volumes
```
sudo lvcreate --wipesignatures y -n thinpool docker -l 95%VG
```
```
sudo lvcreate --wipesignatures y -n thinpoolmeta docker -l 1%VG
```
### Convert the volumes to a thin pool 
```
 sudo lvconvert -y \
--zero n \
-c 512K \
--thinpool docker/thinpool \
--poolmetadata docker/thinpoolmeta
```
### Configure autoextension of thin pools via an lvm profile
```
sudo vi /etc/lvm/profile/docker-thinpool.profile
```
# Manage devicemapper
## Monitor the thin pool
- Do not rely on LVM auto-extension alone. 
- To view the LVM logs, you can use journalctl
```
sudo journalctl -fu dm-event.service
```
## Increase capacity on a running device
## RESIZE A DIRECT-LVM THIN POOL
## Activate the devicemapper after reboot
# How the devicemapper storage driver works
Use the lsblk command to see the devices and their pools, from the operating system’s point of view:
```
sudo lsblk
----------------
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0  19.5G  0 disk 
|-sda1   8:1    0  18.6G  0 part /mnt/sda1
`-sda2   8:2    0  1000M  0 part [SWAP]
sr0     11:0    1    56M  0 rom  
zram0  252:0    0 165.4M  0 disk [SWAP]
```
Use the mount command to see the mount-point Docker is using
```
mount |grep devicemapper
```
## Image and container layers on-disk
- The `/var/lib/docker/devicemapper/metadata/` directory contains metadata about the Devicemapper configuration itself and about each image and container layer that exist. 
- The devicemapper storage driver uses `snapshots`, and this metadata include information about those snapshots. These files are in `JSON` format.
- The `/var/lib/devicemapper/mnt/` directory contains a mount point for each image and container layer that exists.
## Image layering and sharing
The devicemapper storage driver uses dedicated `block` devices rather than formatted `filesystems`, and operates on files at the block level for maximum performance during copy-on-write (CoW) operations.
### SNAPSHOTS
- Another feature of devicemapper is its use of snapshots (also sometimes called thin devices or virtual devices)
- snapshot store the differences introduced in each layer as very small, lightweight thin pools. 

Snapshots provide many benefits:
- Layers which are shared in common between containers are only stored on disk once, unless they are writable. 
- Snapshots are an implementation of a copy-on-write (CoW) strategy. This means that a given file or directory is only copied to the container’s writable layer when it is modified or deleted by that container.
- Because devicemapper operates at the block level, multiple blocks in a writable layer can be modified simultaneously.
- Snapshots can be backed up using standard OS-level backup utilities. Just make a copy of /var/lib/docker/devicemapper/.
### DEVICEMAPPER WORKFLOW
When you start Docker with the devicemapper storage driver, all objects related to image and container layers are stored in /var/lib/docker/devicemapper/, which is backed by one or more block-level devices, either loopback devices (testing only) or physical disks.
- The base device is the lowest-level object.  This is the thin pool itself.  It contains a filesystem. This base device is the starting point for every image and container layer.The base device is a Device Mapper implementation detail, rather than a Docker layer.
- Metadata about the base device and each image or container layer is stored in /var/lib/docker/devicemapper/metadata/ in JSON format. These layers are copy-on-write snapshots, which means that they are empty until they diverge from their parent layers.
- Each container’s writable layer is mounted on a mountpoint in /var/lib/docker/devicemapper/mnt/.

Each image layer is a snapshot of the layer below it. The lowest layer of each image is a snapshot of the base device that exists in the pool. When you run a container, it is a snapshot of the image the container is based on. The following example shows a Docker host with two running containers. The first is a ubuntu container and the second is a busybox container.
![](https://docs.docker.com/storage/storagedriver/images/two_dm_container.jpg)
# How container reads and writes work with devicemapper
## Reading files
- With devicemapper, reads happen at the block level. 
- The diagram below shows the high level process for reading a single block (0x44f) in an example container.
![](https://docs.docker.com/storage/storagedriver/images/dm_container.jpg)
- An application makes a read request for block 0x44f in the container. 
- Because the container is a thin snapshot of an image, it doesn’t have the block, but it has a pointer to the block on the nearest parent image where it does exist, and it reads the block from there. 
- The block now exists in the container’s memory.
## Writing files
### Writing a new file
- With the devicemapper driver, writing new data to a container is accomplished by an allocate-on-demand operation. 
- Each block of the new file is allocated in the container’s writable layer and the block is written there.
### Updating an existing file
- The relevant block of the file is read from the nearest layer where it exists.
- When the container writes the file, only the modified blocks are written to the container’s writable layer.
### Deleting a file or directory
When you delete a file or directory in a container’s writable layer, or when an image layer deletes a file that exists in its parent layer, the devicemapper storage driver intercepts further read attempts on that file or directory and responds that the file or directory does not exist.
### Writing and then deleting a file
 If a container writes to a file and later deletes the file, all of those operations happen in the container’s writable layer.  In that case, if you are using direct-lvm, the blocks are freed. If you use loop-lvm, the blocks may not be freed. This is another reason not to use loop-lvm in production.
 # Device Mapper and Docker performance
 ## allocate-on demand performance impact
 - The devicemapper storage driver uses an allocate-on-demand operation to allocate new blocks from the thin pool into a container’s writable layer. 
 - Each block is 64KB, so this is the minimum amount of space that is used for a write.
 ## Copy-on-write performance impact
 - The first time a container modifies a specific block, that block is written to the container’s writable layer.
 - Because these writes happen at the level of the block rather than the file, performance impact is minimized. 
 - However, writing a large number of blocks can still negatively impact performance, and the devicemapper storage driver may actually perform worse than other storage drivers in this scenario.
 -  For write-heavy workloads, you should use data volumes, which bypass the storage driver completely.

 # Performance best practices
 ## Use direct-lvm
 The loop-lvm mode is not performant and should never be used in production.
## Use fast storage
Solid-state drives (SSDs) provide faster reads and writes than spinning disks.
## Memory usage
-  the devicemapper uses more memory than some other storage drivers. 
- Each launched container loads one or more copies of its files into memory, depending on how many blocks of the same file are being modified at the same time. 
-  Due to the memory pressure, the devicemapper storage driver may not be the right choice for certain workloads in high-density use cases.
### Use volumes for write-heavy workloads
### Note
when using devicemapper and the json-file log driver, the log files generated by a container are still stored in Docker’s dataroot directory, by default /var/lib/docker. If your containers generate lots of log messages, this may lead to increased disk usage or the inability to manage your system due to a full disk. You can configure a log driver to store your container logs externally.






