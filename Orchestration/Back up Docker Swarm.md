# Back up Docker Swarm
- Docker manager nodes store the swarm state and manager logs in the `/var/lib/docker/swarm/` directory. 
- Swarm raft logs contain crucial information for re-creating Swarm specific resources, including services, secrets, configurations and node cryptographic identity. 
- In 1.13 and higher, this data includes the keys used to encrypt the raft logs. Without these keys, you cannot restore the swarm.
- You must perform a manual backup on each manager node, because logs contain node IP address information and are not transferable to other nodes. 
- If you do not backup the raft logs, you cannot verify workloads or Swarm resource provisioning after restoring the cluster.




