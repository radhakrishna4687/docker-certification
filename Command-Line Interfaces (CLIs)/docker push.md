# docker push
Push an image or a repository to a registry
```
docker push [OPTIONS] NAME[:TAG]
```
- `--disable-content-trust`, default is true i.e. Skip image signing
- Use docker push to share your images to the Docker Hub registry or to a self-hosted one.
# Push a new image to a registry
First save the new image by finding the container ID (using docker ps) and then committing it to a new image name. Note that only a-z0-9-_. are allowed when naming images:
```
$ docker commit c16378f943fe rhel-httpd
```
Now, push the image to the registry using the image ID. In this example the registry is on host named registry-host and listening on port 5000. To do this, tag the image with the host name or IP address, and the port of the registry:
```
$ docker tag rhel-httpd registry-host:5000/myadmin/rhel-httpd
$ docker push registry-host:5000/myadmin/rhel-httpd
```
