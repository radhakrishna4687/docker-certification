# Bridge Networks
## Networking Definition
- A bridge network is a `Link Layer` device, which forwards traffic between network segments. 
- A bridge can be a hardware device or a software device running within a host machine's kernel.
## In Context of Docker
- A bridge network uses software bridge which allows containers connected to the same bridge network to communicate, while providing isolation form containers which are not connected to their bridge network.
- The `Docker Bridge Driver`, automatically install rules in the host machine so that containers on different bridge network cannot communicate directly with each other.
## Rules
- Bridge networks apply to containers running on the same `Docker daemon` host.
- For communication among containers running on different `Docker daemon` host, you can either manage `routing at the OS level`, or you can use `Overlay` network.
- When you start Docker, a default `bridge` network is created automatically and newly started containers connect to it unless otherwise specified. 
- You can also create user defined bridge network. 
- User defined bridge networks are superior to the default bridge network.
## Differences between user-defined bridges and the default bridge
- User-defined bridges provide better isolation and interoperability between containerized applications.
- Containers connected to the same user-defined bridge network automatically expose all ports to each other, and no ports to the outside world.
- User-defined bridges provide automatic DNS resolution between containers.
- Containers can be attached and detached from user-defined networks on the fly
- Each user-defined network creates a configurable bridge
- Linked containers on the default bridge network share environment variables
## Manage a user-defined bridge
### Create `bridge` network
```
docker network create <name of network>
```
Following options can be configured as well
- subnet
- ip address range
- gateway
### Remove network
```
docker network rm <name of network>
```
### Underlying Design
- When you create or remove a user-defined bridge or connect or disconnect a container from a user-defined bridge, Docker uses tools specific to the operating system to manage the underlying network infrastructure (such as adding or removing bridge devices or configuring iptables rules on Linux).

### Connect a container to a user-defined bridge
When we create new container, we specify one or more `--network` flags. 
```
docker create --name my-nginx \
  --network my-net \
  --publish 8080:80 \
  nginx:latest
```
To connect running container to an existing user defined bridge, use `docker network connect` command.
```
docker network connect my-net my-nginx
```
### Disconnect a container from a user-defined bridge
```
docker network disconnect my-net my-nginx
```
## Use IPv6
- If you need IPv6 support for Docker containers, you need to enable the option on the Docker daemon and reload its configuration, before creating any IPv6 networks or assigning containers IPv6 addresses.
## Enable forwarding from Docker containers to the outside world
- By default, traffic from containers connected to the default bridge network is not forwarded to the outside world.
- To enable forwarding, you need to change two settings. 
- These are not Docker commands and they affect the Docker host’s kernel.

1. Configure the Linux kernel to allow IP forwarding
```
sysctl net.ipv4.conf.all.forwarding=1
```
2. Change the policy for the iptables FORWARD policy from DROP to ACCEPT
```
sudo iptables -P FORWARD ACCEPT
```
Note: These settings do not persist across a reboot, so you may need to add them to a start-up script.
## Connect a container to the default bridge network
- If you do not specify a network using the --network flag, and you do specify a network driver, your container is connected to the default bridge network by default. 
## Configure the default bridge network
- To configure the default bridge network, you specify options in daemon.json
- Restart Docker for the changes to take effect.



