# Manage Docker as a non-root user
- The Docker daemon binds to a Unix socket instead of a TCP port.
- By default that Unix socket is owned by the user root and other users can only access it using sudo. 
- The Docker daemon always runs as the root user.
- If you don’t want to preface the docker command with sudo, create a Unix group called docker and add users to it. 
- When the Docker daemon starts, it creates a Unix socket accessible by members of the docker group.
- The docker group grants privileges equivalent to the root user. For details on how this impacts security in your system, see Docker Daemon Attack Surface.

To create the docker group and add your user:
1. Create the docker group.
```
sudo groupadd docker
```
2. Add your user to the docker group.
```
sudo usermod -aG docker $USER
```
3. Log out and log back in so that your group membership is re-evaluated.

Note1: If testing on a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.
Note2: On a desktop Linux environment such as X Windows, log out of your session completely and then log back in.
Note3: On Linux, you can also run the following command to activate the changes to groups:
```
 newgrp docker
```
4. Verify that you can run docker commands without sudo.
```
docker run hello-world
```
To fix issue related to permission if earlier `docker run`
```
$ sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
$ sudo chmod g+rwx "$HOME/.docker" -R
```
# Configure Docker to start on boot
Most current Linux distributions (RHEL, CentOS, Fedora, Ubuntu 16.04 and higher) use `systemd` to manage which services start when the system boots. Ubuntu 14.10 and below use upstart.
## systemd
```
sudo systemctl enable docker
```
To disable this behavior, use disable instead.
```
$ sudo systemctl disable docker
```
## upstart
Docker is automatically configured to start on boot using upstart. To disable this behavior, use the following command:
```
$ echo manual | sudo tee /etc/init/docker.override
```
## chkconfig
```
$ sudo chkconfig docker on
```
# Use a different storage engine
The default storage engine and the list of supported storage engines depend on your host’s Linux distribution and available kernel drivers.
# Configure default logging driver
- Docker provides the capability to collect and view log data from all containers running on a host via a series of logging drivers. 
- The default logging driver, json-file, writes log data to JSON-formatted files on the host filesystem. 
- Over time, these log files expand in size, leading to potential exhaustion of disk resources. 
- To alleviate such issues, either configure an alternative logging driver such as `Splunk` or `Syslog`, or set up log rotation for the default driver. 
- If you configure an alternative logging driver, see Use docker logs to read container logs for remote logging drivers.
# Configure where the Docker daemon listens for connections
- By default, the Docker daemon listens for connections on a UNIX socket to accept requests from local clients. 
- It is possible to allow Docker to accept requests from remote hosts by configuring it to listen on an IP address and port as well as the UNIX socket. 
# Configuring remote access with systemd unit file
1. Use the command `sudo systemctl edit docker.service` to open an override file for `docker.service` in a text editor.
2. Add or modify the following lines, substituting your own values.
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375
```
3. Save the file.
4. Reload the systemctl configuration.
```
sudo systemctl daemon-reload
```
5. Restart Docker.
```
sudo systemctl restart docker.service
```
6. Check to see whether the change was honored by reviewing the output of netstat to confirm dockerd is listening on the configured port.
```
sudo netstat -lntp | grep dockerd
```
# Configuring remote access with daemon.json
1. Set the hosts array in the /etc/docker/daemon.json to connect to the UNIX socket and an IP address, as follows:
```
{
"hosts": ["unix:///var/run/docker.sock", "tcp://127.0.0.1:2375"]
}
```
2. Restart Docker.
```
sudo systemctl restart docker
```
3. Check to see whether the change was honored by reviewing the output of netstat to confirm dockerd is listening on the configured port.
```
sudo netstat -lntp | grep dockerd
```
# Troubleshooting
## Kernel compatibility
Docker cannot run correctly if your kernel is older than version 3.10 or if it is missing some modules. To check kernel compatibility, you can download and run the check-config.sh script.
```
$ curl https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh > check-config.sh

$ bash ./check-config.sh
```
## Cannot connect to the Docker daemon
If you see an error such as the following, your Docker client may be configured to connect to a Docker daemon on a different host, and that host may not be reachable.
```
Cannot connect to the Docker daemon. Is 'docker daemon' running on this host?
```
To see which host your client is configured to connect to, check the value of the `DOCKER_HOST` variable in your environment.
```
env | grep DOCKER_HOST
```
- If this command returns a value, the Docker client is set to connect to a Docker daemon running on that host. 
- If it is unset, the Docker client is set to connect to the Docker daemon running on the local host. 
- If it is set in error, use the following command to unset it:
```
unset DOCKER_HOST
```
> If DOCKER_HOST is set as intended, verify that the Docker daemon is running on the remote host and that a firewall or network outage is not preventing you from connecting.

## IP forwarding problems
If you manually configure your network using systemd-network with systemd version 219 or higher, Docker containers may not be able to access your network. 
## Specify DNS servers for Docker
- The default location of the configuration file is `/etc/docker/daemon.json`. 
- You can change the location of the configuration file using the `--config-file` daemon flag. 
- The documentation below assumes the configuration file is located at `/etc/docker/daemon.json`.
Add a dns key with one or more IP addresses as values. If the file has existing contents, you only need to add or edit the dns line.
```
{
	"dns": ["8.8.8.8", "8.8.4.4"]
}
```









