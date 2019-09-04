# Swarm mode key concepts
## What is a swarm?
- The cluster management and orchestration feature embedded in Docker Engine are built using `swarmkit`.
- `Swarmkit` is a separate project, which implements Docker's orchestration layer and is used directly within `Docker`.
- A Swarm consists of multiple Docker hosts which runs in `swarm` mode and act as managers and workers .
- A given Docker host can be a manager, worker or both.
- A task is a running container which is part of a swarm service and managed by a swarm manager, as opposed to standalone container.
- A key difference between standalone containers and swarm services is that only swarm managers can manage a swarm, while standalone containers can be started on any daemon. 
- Docker daemon can participate in swarm as manager, workers or both.
- `Docker compose` can be used to define and run swarm services stack.
## Node
- A node is an instance of the Docker engine participating in the swarm. 
- To deploy application to swarm, we submit a service definition to a manager node. The manager node dispatches units of work called `tasks` to worker node.
- Worker nodes receive and execute tasks dispatched from manager nodes.
- By default manager nodes also run services as worker nodes, but you can configure them to run manager tasks exclusively and be manager-only nodes. 
- An agent runs on each worker node and reports on the tasks assigned to it.
- The worker node notifies the manager node of the current state of its assigned tasks so that the manager can maintain the desired state of each worker.
## Services and tasks
- A service is the definition of the tasks to execute on the manager or worker nodes.
### Replicated services
In the replicated services model, the swarm manager distributes a specific number of replica tasks among the nodes based upon the scale you set in the desired state.
###  Global services
For global services, the swarm runs one task for the service on every available node in the cluster.
### Task
- A task carries a Docker container and the commands to run inside the container. 
- It is the atomic scheduling unit of swarm.
## Load Balancing
- The swarm manager uses `ingress load balancing` to expose the services you want to make available externally to the swarm.
- The swarm manager can automatically assign the service a `PublishedPort` or you can configure a `PublishedPort` for the service. 
- Swarm mode has an internal DNS component that automatically assigns each service in the swarm a DNS entry.
- The swarm manager uses internal load balancing to distribute requests among services within the cluster based upon the DNS name of the service.

