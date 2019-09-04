# Lock your swarm to protect its encryption key
- In Docker 1.13 and higher, the Raft logs used by swarm managers are encrypted on disk by default.
- When Docker restarts, both the TLS key used to encrypt communication among swarm nodes, and the key used to encrypt and decrypt Raft logs on disk, are loaded into each manager node’s memory.
- Docker 1.13 introduces the ability to protect the mutual TLS encryption key and the key used to encrypt and decrypt Raft logs at rest, by allowing you to take ownership of these keys and to require manual unlocking of your managers. This feature is called autolock.
# Initialize a swarm with autolocking enabled
```
docker swarm init --autolock
```
# Unlock a swarm
```
docker swarm unlock
```
# Enable or disable autolock on an existing swarm
```
docker swarm update --autolock=true
docker swarm update --autolock=false
```
# Unlock a swarm
```
docker swarm unlock
```
# View the current unlock key for a running swarm
- If the key has not been rotated since the node left the swarm, and you have a quorum of functional manager nodes in the swarm, you can view the current unlock key using `docker swarm unlock-key` without any arguments.
- If the key was rotated after the swarm node became unavailable and you do not have a record of the previous key, you may need to force the manager to leave the swarm and join it back to the swarm as a new manager.
# Rotate the unlock key
```
docker swarm unlock-key --rotate
```
