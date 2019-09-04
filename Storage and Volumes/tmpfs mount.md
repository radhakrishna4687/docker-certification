# Use tmpfs mounts
- If you’re running Docker on Linux, you have a third option: tmpfs mounts. 
- When you create a container with a tmpfs mount, the container can create files outside the container’s writable layer.
![](https://docs.docker.com/storage/images/types-of-mounts-tmpfs.png)
# Use a tmpfs mount in a container
```
docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app \
  nginx:latest
```
# Specify tmpfs options
- tmpfs-size
- tmpfs-mode
```
docker run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app,tmpfs-mode=1770 \
  nginx:latest
```