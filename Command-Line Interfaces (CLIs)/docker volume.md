# docker volume create
```
docker volume create [OPTIONS] [VOLUME]
```
Creates a new volume that containers can consume and store data in. If a name is not specified, Docker generates a random name.

## Options
- --driver , -d
- --label	
- --name	
- --opt , -o	
# Examples
Create a volume and then configure the container to use it:
```
$ docker volume create hello
$ docker run -d -v hello:/world busybox ls /world
```
## Driver-specific options
```
$ docker volume create --driver local \
    --opt type=tmpfs \
    --opt device=tmpfs \
    --opt o=size=100m,uid=1000 \
    foo
```


