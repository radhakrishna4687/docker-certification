# Use the VFS storage driver
- The VFS storage driver is not a union filesystem
- instead, each layer is a directory on disk
- there is no copy-on-write support
- To create a new layer, a “deep copy” is done of the previous layer. 
- This leads to lower performance and more space used on disk than other storage drivers
- However, it is robust, stable, and works in every environment.
# Configure Docker with the vfs storage driver
```
/etc/docker/daemon.json
{
  "storage-driver": "vfs"
}
```
# How the vfs storage driver works?
- each image layer and the writable container layer are represented on the Docker host as subdirectories within /var/lib/docker/
