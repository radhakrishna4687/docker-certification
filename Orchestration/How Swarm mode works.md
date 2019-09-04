# How Swarm mode works?
![](https://docs.docker.com/engine/swarm/images/swarm-diagram.png)
## Manager nodes
Manager nodes handle cluster management tasks
- maintaining cluster state
- scheduling services
- serving swarm mode HTTP API endpoints
Using `raft` implementation, the manager maintains a consistent internal state of the entire swarm and all the services running on it.
- When you have multiple managers you can recover from the failure of a manager node without downtime.
- A three-manager swarm tolerates a maximum loss of one manager.
- Docker recommends a maximum of seven manager nodes for a swarm.
## Worker nodes
- Worker nodes are also instances of Docker Engine whose sole purpose is to execute containers.
- Worker nodes don’t participate in the Raft distributed state, make scheduling decisions, or serve the swarm mode HTTP API.
- Worker nodes use `Gossip Network` protocol for networking.
- To prevent the scheduler from placing tasks on a manager node in a multi-node swarm, set the availability for the manager node to Drain
- You can promote a worker node to be a manager by running
```
docker node promote
```
- You can also demote a manager node to a worker node
```
docker node demote
```
# How services work?
To deploy an application image when Docker Engine is in swarm mode, you create a service.
You also define options for the service including:
- the port where the swarm makes the service available outside the swarm
- an overlay network for the service to connect to other services in the swarm
- CPU and memory limits and reservations
- a rolling update policy
- the number of replicas of the image to run in the swarm
## Services, tasks, and containers
- When you deploy the service to the swarm, the swarm manager accepts your service definition as the desired state for the service
- Then it schedules the service on nodes in the swarm as one or more replica tasks. 
- The tasks run independently of each other on nodes in the swarm.
![](https://docs.docker.com/engine/swarm/images/services-diagram.png)
## Tasks and scheduling
- A task is the atomic unit of scheduling within a swarm.
- When you declare a desired service state by creating or updating a service, the orchestrator realizes the desired state by scheduling tasks.
The diagram below shows how swarm mode accepts service create requests and schedules tasks to worker nodes.
![](https://docs.docker.com/engine/swarm/images/service-lifecycle.png)
## Replicated and global services
### Replicated
- you specify the number of identical tasks you want to run
### global services
- A global service is a service that runs one task on every node.
- There is no pre-specified number of tasks.
- Each time you add a node to the swarm, the orchestrator creates a task and the scheduler assigns the task to the new node.
-  Good candidates for global services are monitoring agents, an anti-virus scanners or other types of containers that you want to run on every node in the swarm.
