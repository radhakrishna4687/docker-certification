# Docker Machine
## What is Docker Machine?
- Docker Machine is a tool that lets you install Docker Engine on virtual hosts and manage hosts using `docker-machine` command.
- Docker Machine was the only way to run Docker on Mac or Windows previous to Docker v1.12. 
## Why should I use it?
- Docker Machine enables you to provision multiple remote Docker hosts on various flavors of Linux.
- Docker Engine runs natively on Linux systems.
- Whether your primary system is Mac, Windows, or Linux, you can install Docker Machine on it and use docker-machine commands to provision and manage large numbers of Docker hosts. 
-  It automatically creates hosts, installs Docker Engine on them, then configures the docker clients.
- Each managed host (“machine”) is the combination of a Docker host and a configured client.
## What is Docker Engine?
It is a client-server application made up of the following
- Docker daemon
- REST API - specifies interfaces for interacting with the daemon
- Command line interface (CLI) client - talks to the daemon (through the REST API wrapper)
![](https://docs.docker.com/machine/img/engine.png)
## What’s the difference between Docker Engine and Docker Machine?
- When people say “Docker” they typically mean Docker Engine.
- Docker Machine is a tool for provisioning and managing your Dockerized hosts (hosts with Docker Engine on them).
- Docker Machine has its own command line client docker-machine and the Docker Engine client, docker.
- You can use Machine to install Docker Engine on one or more virtual systems. 
![](https://docs.docker.com/machine/img/machine.png)
## Install Docker Machine
### MAC
```
base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/usr/local/bin/docker-machine &&
  chmod +x /usr/local/bin/docker-machine
```
## Get started with Docker Machine and a local VM
### Create a machine
```
docker-machine create --driver virtualbox default
```
- This command downloads a lightweight Linux distribution (boot2docker) with the Docker daemon installed
- It creates and starts a VirtualBox VM with Docker running.
### Get list of machine
```
docker-machine ls
```
### Connect local Docker Client to Docker Engine
```
docker-machine env default
```
### Install Docker binary
```
brew install docker  docker-compose
```
### Get the host IP address
```
docker-machine ip default
```
### Start and stop machines
```
docker-machine stop default
docker-machine start default
```
## Mac login issue
Make sure to remove following line from `$HOME/.docker/config.json` file
```
"credsStore" : "osxkeychain"
```

