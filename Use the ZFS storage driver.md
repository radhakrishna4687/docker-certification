# Use the ZFS storage driver
ZFS is a next generation filesystem that supports many advanced storage technologies 
- volume management
- snapshots
- checksumming
- compression
- deduplication
- replication

It was created by Sun Microsystems (now Oracle Corporation) and is open sourced under the CDDL license. 
- Due to licensing incompatibilities between the CDDL and GPL, ZFS cannot be shipped as part of the mainline Linux kernel.
- However, the ZFS On Linux (ZoL) project provides an out-of-tree kernel module and userspace tools which can be installed separately.
- The ZFS on Linux (ZoL) port is healthy and maturing. However, at this point in time it is not recommended to use the zfs Docker storage driver for production use unless you have substantial experience with ZFS on Linux.
# How the zfs storage driver works
ZFS uses the following objects
## filesystems
thinly provisioned, with space allocated from the zpool on demand.
## snapshots
read-only space-efficient point-in-time copies of filesystems
## clones
Read-write copies of snapshots. Used for storing the differences from the previous layer.

The process of creating a clone
![](https://docs.docker.com/storage/storagedriver/images/zfs_clones.jpg)
- A read-only snapshot is created from the filesystem
- A writable clone is created from the snapshot. This contains any differences from the parent layer.
# Image and container layers on-disk
Each running container’s unified filesystem is mounted on a mount point in /var/lib/docker/zfs/graph/.
# Image layering and sharing
The base layer of an image is a ZFS filesystem. Each child layer is a ZFS clone based on a ZFS snapshot of the layer below it. A container is a ZFS clone based on a ZFS Snapshot of the top layer of the image it’s created from.

The diagram below shows how this is put together with a running container based on a two-layer image.
![](https://docs.docker.com/storage/storagedriver/images/zfs_zpool.jpg)

# Image layering and sharing
The base layer of an image is a ZFS filesystem. Each child layer is a ZFS clone based on a ZFS snapshot of the layer below it. A container is a ZFS clone based on a ZFS Snapshot of the top layer of the image it’s created from.

The diagram below shows how this is put together with a running container based on a two-layer image.
![](https://docs.docker.com/storage/storagedriver/images/zfs_zpool.jpg)
