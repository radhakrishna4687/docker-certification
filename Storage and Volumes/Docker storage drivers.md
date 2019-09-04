# Docker storage drivers
- Ideally, very little data is written to a container’s writable layer, and you use Docker volumes to write data.
- However, some workloads require you to be able to write to the container’s writable layer. This is where storage drivers come in.
- Docker supports several different storage drivers, using a pluggable architecture. 
- The storage driver controls how images and containers are stored and managed on your Docker host.

Docker supports the following storage drivers:
1. overlay2 - is the preferred storage driver, for all currently supported Linux distributions, and requires no extra configuration.
2. aufs -  is the preferred storage driver for Docker 18.06 and older, when running on Ubuntu 14.04 on kernel 3.13 which has no support for overlay2
3. devicemapper - is supported, but requires direct-lvm for production environments, because loopback-lvm, while zero-configuration, has very poor performance. devicemapper was the recommended storage driver for CentOS and RHEL, as their kernel version did not support overlay2. However, current versions of CentOS and RHEL now have support for overlay2, which is now the recommended driver.
4. btrfs/zfs - storage drivers are used if they are the backing filesystem (the filesystem of the host on which Docker is installed). These filesystems allow for advanced options, such as creating “snapshots”, but require more maintenance and setup. Each of these relies on the backing filesystem being configured correctly.
5. vfs -  storage driver is intended for testing purposes, and for situations where no copy-on-write filesystem can be used. Performance of this storage driver is poor, and is not generally recommended for production use.
# Supported storage drivers per Linux distribution
At a high level, the storage drivers you can use is partially determined by the Docker edition you use.
# Docker Engine - Enterprise and Docker Enterprise
For Docker Engine - Enterprise and Docker Enterprise, the definitive resource for which storage drivers are supported is the Product compatibility matrix. 
# Docker Engine - Community
For Docker Engine - Community, only some configurations are tested, and your operating system’s kernel may not support every storage driver.
- The overlay storage driver is deprecated in Docker Engine - Enterprise 18.09, and will be removed in a future release. It is recommended that users of the overlay storage driver migrate to overlay2.
- The devicemapper storage driver is deprecated in Docker Engine 18.09, and will be removed in a future release. It is recommended that users of the devicemapper storage driver migrate to overlay2.
- When possible, overlay2 is the recommended storage driver. 
- When installing Docker for the first time, overlay2 is used by default.
- Previously, aufs was used by default when available, but this is no longer the case.
# Suitability for your workload
- overlay2, aufs, and overlay all operate at the file level rather than the block level. This uses memory more efficiently, but the container’s writable layer may grow quite large in write-heavy workloads.
- Block-level storage drivers such as devicemapper, btrfs, and zfs perform better for write-heavy workloads (though not as well as Docker volumes).
- btrfs and zfs require a lot of memory.
- zfs is a good choice for high-density workloads such as PaaS.

# Shared storage systems and the storage driver
- Each Docker storage driver is based on a Linux filesystem or volume manager.
- Be sure to follow existing best practices for operating your storage driver (filesystem or volume manager) on top of your shared storage system.
# Stability
In general, overlay2, aufs, overlay, and devicemapper are the choices with the highest stability.
# Check your current storage driver
```
docker info
```
```
Containers: 0
Images: 0
Storage Driver: overlay2
 Backing Filesystem: xfs
<output truncated>
```
