# Use the BTRFS storage driver
- Btrfs is a next generation copy-on-write filesystem that supports many advanced storage technologies that make it a good fit for Docker.
-  Btrfs is included in the mainline Linux kernel.
- Docker’s btrfs storage driver leverages many Btrfs features for image and container management.
- Among these features are 
1. block-level operations, 
2. thin provisioning, 
3. copy-on-write snapshots, and 
4. ease of administration
- You can easily combine multiple physical block devices into a single Btrfs filesystem.
- This article refers to Docker’s Btrfs storage driver as btrfs and the overall Btrfs Filesystem as Btrfs.
- btrfs support must exist in your kernel. To check this, run the following command:
```
sudo cat /proc/filesystems | grep btrfs
```
# How the btrfs storage driver works?
The btrfs storage driver works differently from devicemapper or other storage drivers in that your entire /var/lib/docker/ directory is stored on a Btrfs volume.
## Image and container layers on-disk
- Information about image layers and writable container layers is stored in `/var/lib/docker/btrfs/subvolumes/`.
- This subdirectory contains one directory per image or container layer, with the unified filesystem built from a layer plus all its parent layers. 
- Subvolumes are natively copy-on-write and have space allocated to them on-demand from an underlying storage pool.
- They can also be nested and snapshotted.
- The diagram below shows 4 subvolumes. ‘Subvolume 2’ and ‘Subvolume 3’ are nested, whereas ‘Subvolume 4’ shows its own internal directory tree.
![](https://docs.docker.com/storage/storagedriver/images/btfs_subvolume.jpg)
- Only the base layer of an image is stored as a true subvolume.
- All the other layers are stored as snapshots, which only contain the differences introduced in that layer. 
- You can create snapshots of snapshots as shown in the diagram below.
![](https://docs.docker.com/storage/storagedriver/images/btfs_snapshots.jpg)
- On disk, snapshots look and feel just like subvolumes, but in reality they are much smaller and more space-efficient. 
- Copy-on-write is used to maximize storage efficiency and minimize layer size, and writes in the container’s writable layer are managed at the block level. 
- The following image shows a subvolume and its snapshot sharing data.
![](https://docs.docker.com/storage/storagedriver/images/btfs_pool.jpg)
- For maximum efficiency, when a container needs more space, it is allocated in chunks of roughly 1 GB in size.
- Docker’s btrfs storage driver stores every image layer and container in its own Btrfs subvolume or snapshot.
- The base layer of an image is stored as a subvolume whereas child image layers and containers are stored as snapshots.
- This is shown in the diagram below.
![](https://docs.docker.com/storage/storagedriver/images/btfs_container_layer.jpg)

The high level process for creating images and containers on Docker hosts running the btrfs driver is as follows:
1. The image’s base layer is stored in a Btrfs subvolume under `/var/lib/docker/btrfs/subvolumes`.
2. Subsequent image layers are stored as a Btrfs snapshot of the parent layer’s subvolume or snapshot, but with the changes introduced by this layer. These differences are stored at the block level.
3. The container’s writable layer is a Btrfs snapshot of the final image layer, with the differences introduced by the running container. These differences are stored at the block level.
# How container reads and writes work with btrfs?
## Reading files
- A container is a space-efficient snapshot of an image.
- Metadata in the snapshot points to the actual data blocks in the storage pool.
- This is the same as with a subvolume. 
- Therefore, reads performed against a snapshot are essentially the same as reads performed against a subvolume.
## Writing files
### Writing new files
- Writing a new file to a container invokes an allocate-on-demand operation to allocate new data block to the container’s snapshot.
- The file is then written to this new space. 
- The allocate-on-demand operation is native to all writes with Btrfs and is the same as writing new data to a subvolume. 
- As a result, writing new files to a container’s snapshot operates at native Btrfs speeds.
### Modifying existing files
- Updating an existing file in a container is a copy-on-write operation (redirect-on-write is the Btrfs terminology). 
- The original data is read from the layer where the file currently exists, and only the modified blocks are written into the container’s writable layer. 
- Next, the Btrfs driver updates the filesystem metadata in the snapshot to point to this new data. 
- This behavior incurs very little overhead.
### Deleting files or directories
-  If a container deletes a file or directory that exists in a lower layer, Btrfs masks the existence of the file or directory in the lower layer.
- If a container creates a file and then deletes it, this operation is performed in the Btrfs filesystem itself and the space is reclaimed.

Note: With Btrfs, writing and updating lots of small files can result in slow performance.
# Btrfs and Docker performance
There are several factors that influence Docker’s performance under the btrfs storage driver.
## Page caching
- Btrfs does not support page cache sharing. 
- This means that each process accessing the same file copies the file into the Docker hosts’s memory.
- As a result, the btrfs driver may not be the best choice high-density use cases such as PaaS.
## Small writes
- Containers performing lots of small writes (this usage pattern matches what happens when you start and stop many containers in a short period of time, as well) can lead to poor use of Btrfs chunks.
- This can prematurely fill the Btrfs filesystem and lead to out-of-space conditions on your Docker host.
- Use btrfs filesys show to closely monitor the amount of free space on your Btrfs device.
## Sequential writes
- Btrfs uses a journaling technique when writing to disk.
- This can impact the performance of sequential writes, reducing performance by up to 50%.
## Fragmentation
- Fragmentation is a natural byproduct of copy-on-write filesystems like Btrfs.
- Many small random writes can compound this issue.
- Fragmentation can manifest as CPU spikes when using SSDs or head thrashing when using spinning disks. Either of these issues can harm performance.
## SSD performance
- Btrfs includes native optimizations for SSD media
- To enable these features, mount the Btrfs filesystem with the -o ssd mount option.
## Balance Btrfs filesystems often
- Use operating system utilities such as a cron job to balance the Btrfs filesystem regularly, during non-peak hours.
- This reclaims unallocated blocks and helps to prevent the filesystem from filling up unnecessarily. 
## Use fast storage
- Solid-state drives (SSDs) provide faster reads and writes than spinning disks.
## Use volumes for write-heavy workloads
- `Volumes` provide the best and most predictable performance for write-heavy workloads.
- This is because they bypass the storage driver and do not incur any of the potential overheads introduced by thin provisioning and copy-on-write. 
- 