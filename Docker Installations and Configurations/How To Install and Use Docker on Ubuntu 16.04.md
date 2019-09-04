# How To Install and Use Docker on Ubuntu 16.04
There are two methods for installing Docker on Ubuntu 16.04. 
1. One method involves installing it on an existing installation of the operating system. 
2. The other involves spinning up a server with a tool called Docker Machine that auto-installs Docker on it.

In this tutorial, you’ll learn how to install and use it on an existing installation of Ubuntu 16.04.
# Uninstall old versions
Older versions of Docker were called `docker`, `docker.io` , or `docker-engine`. If these are installed, uninstall them:
```
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```
> The contents of /var/lib/docker/, including images, containers, volumes, and networks, are preserved. The Docker Engine - Community package is now called docker-ce.

# Prerequisites
To follow this tutorial, you will need the following:
- One Ubuntu 16.04 server set up with a non-root user with sudo privileges and a basic firewall, as explained in the Initial Setup Guide for Ubuntu 16.04
- An account on Docker Hub if you wish to create your own images and push them to Docker Hub
# Step 1 — Installing Docker
The Docker installation package available in the official Ubuntu 16.04 repository may not be the latest version. To get this latest version, install Docker from the official Docker repository. This section shows you how to do just that.

## add the `GPG` key for the official Docker repository to your system
In order to ensure the downloads are valid, add the `GPG` key for the official Docker repository to your system:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
## Add the Docker repository to APT sources:
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
## update the package database with the Docker packages from the newly added repo:
```
sudo apt-get update
```
## Install packages to allow apt to use a repository over HTTPS
```
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

## Install from the Docker repo
Make sure you are about to install from the Docker repo instead of the default Ubuntu 16.04 repo:
```
apt-cache policy docker-ce
```
Output 
```
docker-ce:
  Installed: (none)
  Candidate: 18.06.1~ce~3-0~ubuntu
  Version table:
     18.06.1~ce~3-0~ubuntu 500
        500 https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
```
Notice that docker-ce is not installed, but the candidate for installation is from the Docker repository for Ubuntu 16.04 (xenial).
## Finally, install Docker:
```
sudo apt-get install -y docker-ce
```
Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it’s running:
```
sudo systemctl status docker
```
Installing Docker now gives following
1. The Docker service (daemon) 
2. The docker command line utility, or the Docker client. 

# Step 2 — Executing the Docker Command Without Sudo (Optional)
- By default, running the docker command requires root privileges — that is, you have to prefix the command with sudo. 
- It can also be run by a user in the docker group, which is automatically created during the installation of Docker. 
- If you attempt to run the docker command without prefixing it with sudo or without being in the docker group, you’ll get an output like this:
```
Output
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```
If you want to avoid typing sudo whenever you run the docker command, add your username to the docker group:
```
sudo usermod -aG docker ${USER}
```
To apply the new group membership, you can log out of the server and back in, or you can type the following:
```
su - ${USER}
```


