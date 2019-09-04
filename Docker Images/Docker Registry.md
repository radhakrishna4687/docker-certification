# Docker Registry
## What it is?
- The Registry is a stateless, highly scalable server side application that stores and lets you distribute Docker images.
## Why use it?
You should use the Registry if you want to:
- tightly control where your images are being stored
- fully own your images distribution pipeline
- integrate image storage and distribution tightly into your in-house development workflow
## Basic commands
### Start your registry
```
docker run -d -p 5000:5000 --name registry registry:2
```
### Pull (or build) some image from the hub
```
docker pull ubuntu
```
### Tag the image so that it points to your registry
docker image tag ubuntu localhost:5000/myfirstimage
### Push it
```
docker push localhost:5000/myfirstimage
```
### Pull it back
```
docker pull localhost:5000/myfirstimage
```
### Stop your registry and remove all data
```
docker container stop registry && docker container rm -v registry
```

