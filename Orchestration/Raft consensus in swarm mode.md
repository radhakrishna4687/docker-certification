# Raft consensus Algorithm
http://thesecretlivesofdata.com/raft/
https://raft.github.io/
## Overview
- Raft is a protocol for implementing distributed consensus.
- A node can be in 1 of 3 states
1. The Follower state
2. the Candidate state
3. the Leader state
- All our nodes start in the follower state.
- If followers don't hear from a leader then they can become a candidate.
- The candidate then requests votes from other nodes.
- Nodes will reply with their vote.
- The candidate becomes the leader if it gets votes from a majority of nodes.
- This process is called Leader Election.
- All changes to the system now go through the leader.
- Each change is added as an entry in the node's log.
- This log entry is currently uncommitted so it won't update the node's value.
- To commit the entry the node first replicates it to the follower nodes...
- then the leader waits until a majority of nodes have written the entry.
- The entry is now committed on the leader node and the node state is "5".
- The leader then notifies the followers that the entry is committed.
- The cluster has now come to consensus about the system state.
- This process is called Log Replication.
## Leader Election
-In Raft there are two timeout settings which control elections.
### election timeout 
- The election timeout is the amount of time a follower waits until becoming a candidate. 
- The election timeout is randomized to be between 150ms and 300ms. 
- After the election timeout the follower becomes a candidate and starts a new election term, votes for itself. and sends out Request Vote messages to other nodes. 
- If the receiving node hasn't voted yet in this term then it votes for the candidate and the node resets its election timeout.  
- Once a candidate has a majority of votes it becomes leader.
- The leader begins sending out `Append Entries` messages to its followers.
- These messages are sent in intervals specified by the `heartbeat timeout`.
- Followers then respond to each Append Entries message.
- This election term will continue until a follower stops receiving heartbeats and becomes a candidate.
- If two nodes become candidates at the same time then a split vote can occur.
## Log Replication
- Once we have a leader elected we need to replicate all changes to our system to all nodes.
- This is done by using the same Append Entries message that was used for heartbeats.
- First client sends message to nodes
- The change is appended to the leader's log...
- then the change is sent to the followers on the next heartbeat.
- An entry is committed once a majority of followers acknowledge it...
- and a response is sent to the client.
## Partitions
- Because of our partition we now have two leaders in different terms.
# Raft consensus in swarm mode
- When the Docker Engine runs in swarm mode, `manager nodes` implement the Raft Consensus Algorithm to manage the global cluster state.
- The reason why Docker swarm mode is using a consensus algorithm is to make sure that all the manager nodes that are in charge of managing and scheduling tasks in the cluster, are storing the same consistent state.
- Having the same consistent state across the cluster means that in case of a failure, any Manager node can pick up the tasks and restore the services to a stable state
- Systems using consensus algorithms to replicate logs in a distributed systems do require special care. 
- They ensure that the cluster state stays consistent in the presence of failures by requiring a majority of nodes to agree on values.
- Raft tolerates up to (N-1)/2 failures and requires a majority or quorum of (N/2)+1 members to agree on values proposed to the cluster.
- The implementation of the consensus algorithm in swarm mode means it features the properties inherent to distributed systems:
1. agreement on values in a fault tolerant system. 
2. mutual exclusion through the leader election process
3. cluster membership management
4. globally consistent object sequencing and CAS (compare-and-swap) primitives