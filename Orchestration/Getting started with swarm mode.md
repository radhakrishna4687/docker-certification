# Getting started with swarm mode
## Set Up
### Create hosts
```
docker-machine create --driver virtualbox manager1
docker-machine create --driver virtualbox worker1
docker-machine create --driver virtualbox worker2
docker-machine create --driver virtualbox worker3
```
### Get IP Address
```
manager1    192.168.99.101
worker1     192.168.99.102
worker2     192.168.99.103
worker3     192.168.99.104
```
## Create a swarm
### Initialize swarm cluster on master
```
docker-machine ssh manager1
docker swarm init --advertise-addr 192.168.99.101
```
### Add nodes to the swarm
```
docker-machine ssh worker1
 docker swarm join --token SWMTKN-1-28pe9ntutgkd90fjce6i8rlhwx5bi3zx5n4iq6x0o6kx5adn3g-dmff9pldrw6v0s8kgditlkjv5 192.168.99.101:2377
```
### retrieve the join command for a worker
```
docker swarm join-token worker
```
### Drain node
```
docker node update --availability drain worker3
```
### Activate node
```
$ docker node update --availability active worker3
```
### Deploy a service to the swarm
On `manager1`
```
docker service create --replicas 2 --name helloworld alpine ping docker.com
```
### Inspect a service on the swarm
On `manager1`
```
docker service inspect --pretty helloworld
```
to see which nodes are running the service:
```
docker service ps helloworld
```
### Scale the Service
```
docker service scale helloworld=5
```
### Delete service
```
docker service rm helloworld
```
### Apply rolling update
- Deploy your Redis tag to the swarm and configure the swarm with a 10 second update delay.
```
docker service create \
  --replicas 3 \
  --name redis \
  --update-delay 10s \
  redis:3.0.6
```
- The --update-delay flag configures the time delay between updates to a service task or sets of tasks. 
- By default the scheduler updates 1 task at a time. You can pass the --update-parallelism flag to configure the maximum number of service tasks that the scheduler updates simultaneously.
- Now you can update the container image for redis
```
docker service update --image redis:3.0.7 redis
```
The scheduler applies rolling updates as follows by default:
- Stop the first task.
- Schedule update for the stopped task.
- Start the container for the updated task.
- If the update to a task returns RUNNING, wait for the specified delay period then start the next task.
- If, at any time during the update, a task returns FAILED, pause the update.
### Use swarm mode routing mesh
- Docker Engine swarm mode makes it easy to publish ports for services to make them available to resources outside the swarm. 
- All nodes participate in an ingress routing mesh
- The routing mesh enables each node in the swarm to accept connections on published ports for any service running in the swarm, even if there’s no task running on the node. 
- The routing mesh routes all incoming requests to published ports on available nodes to an active container.
### Publish a port for a service
```
docker service create \
  --name <SERVICE-NAME> \
  --publish published=<PUBLISHED-PORT>,target=<CONTAINER-PORT> \
  <IMAGE>
```
![](https://docs.docker.com/engine/swarm/images/ingress-routing-mesh.png)
## Configure an external load balancer
### Using the routing mesh
![](https://docs.docker.com/engine/swarm/images/ingress-lb.png)
