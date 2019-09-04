# docker export
Export a container’s filesystem as a tar archive
```
docker export [OPTIONS] CONTAINER
```
- The docker export command does not export the contents of volumes associated with the container. 
- If a volume is mounted on top of an existing directory in the container, docker export will export the contents of the underlying directory, not the contents of the volume.

```
$ docker export red_panda > latest.tar
```

## --output , -o
Write to a file, instead of STDOUT
```
$ docker export --output="latest.tar" red_panda
```
