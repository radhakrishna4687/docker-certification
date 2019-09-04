# Overlay networks
- The `overlay` network driver creates a distributed network among multiple `Docker` daemon host. This network sits on top of the host specific network, allowing containers connected to it to communicate securely.
- Docker transparently handles routing of each packet to and from the correct Docker daemon host and the correct destination container.
- When you initialize a swarm or join a Docker host to an existing swarm, two new networks are created on that Docker host:
1. An overlay network called `ingress`, which handles control and data traffic related to swarm `services`. When you create a swarm service and do not connect it to a user-defined overlay network, it connects to the `ingress` network by default.
2. A bridge network called `docker_gwbridge`, which connects the individual Docker daemon to the other daemons participating in the swarm.
- Services or containers can only communicate across networks they are each connected to.
- Although you can connect both `swarm services` and `standalone containers` to an overlay network, the default behaviors and configuration concerns are different.
## Operations for all overlay networks
### Create an overlay network
#### Prerequisites
- Firewall rules for Docker daemons using overlay networks
You need the following ports open to traffic to and from each Docker host participating on an overlay network:
1. TCP port 2377 for cluster management communications
2. TCP and UDP port 7946 for communication among nodes
3. UDP port 4789 for overlay network traffic
- Before you can create an overlay network, you need to either initialize your Docker daemon as a swarm manager using `docker swarm init` or join it to an existing swarm using `docker swarm join`.
- Either of these creates the default `ingress` overlay network which is used by `swarm services` by default.
- You need to do this even if you never plan to use swarm services. 
- Afterward, you can create additional user-defined overlay networks.

To create an overlay network for use with swarm services, use a command like the following:
```
docker network create -d overlay my-overlay
```
To create an overlay network which can be used by swarm services or standalone containers to communicate with other standalone containers running on other Docker daemons, add the --attachable flag:
```
docker network create -d overlay --attachable my-attachable-overlay
```
### Encrypt traffic on an overlay network
- All swarm service management traffic is encrypted by default, using the AES algorithm in `GCM mode`. 
- Manager nodes in the swarm rotate the key used to encrypt gossip data every 12 hours.
- To encrypt application data as well, add --opt encrypted when creating the overlay network. 
- This enables `IPSEC encryption` at the level of the `vxlan`.
- This encryption imposes a `non-negligible` performance penalty, so you should test this option before using it in production.
- When you enable overlay encryption, Docker creates `IPSEC tunnels` between all the nodes where tasks are scheduled for services attached to the overlay network.
- These tunnels also use the AES algorithm in `GCM mode` and manager nodes automatically rotate the keys every 12 hours.
- Overlay network encryption is not supported on Windows.
### SWARM MODE OVERLAY NETWORKS AND STANDALONE CONTAINERS
You can use the overlay network feature with both `--opt encrypted --attachable` and attach unmanaged containers to that network:
```
docker network create --opt encrypted --driver overlay --attachable my-attachable-multi-host-network
```
### Customize the default ingress network
- Most users never need to configure the ingress network, but Docker 17.05 and higher allow you to do so. 
- Customizing the `ingress` network involves removing and recreating it. 
```
docker network create \
  --driver overlay \
  --ingress \
  --subnet=10.11.0.0/16 \
  --gateway=10.11.0.2 \
  --opt com.docker.network.driver.mtu=1200 \
  my-ingress
```
Note: You can name your ingress network something other than ingress, but you can only have one. An attempt to create a second one fails.
### Customize the docker_gwbridge interface
- The docker_gwbridge is a virtual bridge that connects the overlay networks  (including the ingress network) to an individual Docker daemon’s physical network.
- Docker creates it automatically when you initialize a swarm or join a Docker host to a swarm, but it is not a Docker device. 
- It exists in the kernel of the Docker host.
- If you need to customize its settings, you must do so before joining the Docker host to the swarm, or after temporarily removing the host from the swarm.
```
docker network create \
--subnet 10.11.0.0/16 \
--opt com.docker.network.bridge.name=docker_gwbridge \
--opt com.docker.network.bridge.enable_icc=false \
--opt com.docker.network.bridge.enable_ip_masquerade=true \
docker_gwbridge
```
## Operations for swarm services
### Publish ports on an overlay network
- Swarm services connected to the same overlay network effectively expose all ports to each other.
### Bypass the routing mesh for a swarm service
- By default, swarm services which publish ports do so using the routing mesh.
- When you connect to a published port on any swarm node (whether it is running a given service or not), you are redirected to a worker which is running that service, transparently. 
- Effectively, Docker acts as a load balancer for your swarm services. 
- Services using the routing mesh are running in virtual IP (VIP) mode. 
- Even a service running on each node (by means of the --mode global flag) uses the routing mesh.
- When using the routing mesh, there is no guarantee about which Docker node services client requests.
- To bypass the routing mesh, you can start a service using `DNS Round Robin (DNSRR)` mode, by setting the `--endpoint-mode` flag to `dnsrr`.
- You must run your own load balancer in front of the service.
- A DNS query for the service name on the Docker host returns a list of IP addresses for the nodes running the service.
### Separate control and data traffic
- By default, control traffic relating to swarm management and traffic to and from your applications runs over the same network, though the swarm control traffic is encrypted. 
- You can configure Docker to use separate network interfaces for handling the two different types of traffic. 
- When you initialize or join the swarm, specify `--advertise-addr` and `--datapath-addr` separately.
## Operations for standalone containers on overlay networks
### Attach a standalone container to an overlay network
- The `ingress` network is created without the `--attachable` flag, which means that only swarm services can use it, and not standalone containers. 
- You can connect standalone containers to user-defined `overlay` networks which are created with the `--attachable` flag.
## Container discovery
- For most situations, you should connect to the service name, which is load-balanced and handled by all containers (“tasks”) backing the service. 
- To get a list of all tasks backing the service, do a DNS lookup for tasks.<service-name>


