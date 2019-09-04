# Administer and maintain a swarm of Docker Engines
- When you run a swarm of Docker Engines, manager nodes are the key components for managing the swarm and storing the swarm state.
## Operate manager nodes in a swarm
- Swarm manager nodes use the Raft Consensus Algorithm to manage the swarm state. 
- There is no limit on the number of manager nodes.
- The decision about how many manager nodes to implement is a trade-off between performance and fault-tolerance. 
- Raft requires a majority of managers, also called the quorum, to agree on proposed updates to the swarm, such as node additions or removals.
## Maintain the quorum of managers
- If the swarm loses the quorum of managers, the swarm cannot perform management tasks.
- If your swarm has multiple managers, always have more than two.
- To maintain quorum, a majority of managers must be available. 
- An odd number of managers is recommended, because the next even number does not make the quorum easier to keep. 
## Configure the manager to advertise on a static IP address
- you should use a fixed IP address for the advertise address to prevent the swarm from becoming unstable on machine reboot.
- Dynamic IP addresses are OK for worker nodes.
## Add manager nodes for fault tolerance
- You should maintain an odd number of managers in the swarm to support manager node failures.
## Distribute manager nodes
- In addition to maintaining an odd number of manager nodes, pay attention to datacenter topology when placing managers. 
- For optimal fault-tolerance, distribute manager nodes across a minimum of 3 availability-zones to support failures of an entire set of machines or common maintenance scenarios.
## Run manager-only nodes
- By default manager nodes also act as a worker nodes. 
- because manager nodes use the Raft consensus algorithm to replicate data in a consistent way, they are sensitive to resource starvation. You should isolate managers in your swarm from processes that might block swarm operations like swarm heartbeat or leader elections.
- To avoid interference with manager node operation, you can drain manager nodes to make them unavailable as worker nodes:
```
docker node update --availability drain <NODE>
```
## Add worker nodes for load balancing
- Add nodes to the swarm to balance your swarm’s load. 
- Replicated service tasks are distributed across the swarm as evenly as possible over time, as long as the worker nodes are matched to the requirements of the services.
## Monitor swarm health
- You can monitor the health of manager nodes by querying the docker nodes API in JSON format through the /nodes HTTP endpoint.
```
docker node inspect manager1 --format "{{ .ManagerStatus.Reachability }}"
reachable
```
To query the status of the node as a worker that accept task:
```
docker node inspect manager1 --format "{{ .Status.State }}"
ready
```
## Troubleshoot a manager node
- You should never restart a manager node by copying the raft directory from another node. The data directory is unique to a node ID. A node can only use a node ID once to join the swarm. The node ID space should be globally unique.

To cleanly re-join a manager node to a cluster:
- To demote the node to a worker, run `docker node demote <NODE>`.
- To remove the node from the swarm, run `docker node rm <NODE>`.
- Re-join the node to the swarm with a fresh state using `docker swarm join`.

## Forcibly remove a node
- In most cases, you should shut down a node before removing it from a swarm with the `docker node rm ` command.
```
docker node rm node9
docker node rm --force node9
```
## Back up the swarm
- Docker manager nodes store the swarm state and manager logs in the `/var/lib/docker/swarm/` directory. 
- In 1.13 and higher, this data includes the keys used to encrypt the Raft logs.
- Without these keys, you cannot restore the swarm.
## Recover from disaster
### Restore from a backup
```
docker swarm init --force-new-cluster
```
### Recover from losing the quorum
```
docker swarm init --force-new-cluster --advertise-addr node01:2377

```
### Force the swarm to rebalance
