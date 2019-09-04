# Manage swarm security with public key infrastructure (PKI)
- The swarm mode public key infrastructure (PKI) system built into Docker makes it simple to securely deploy a container orchestration system.
- The nodes in a swarm use mutual Transport Layer Security (TLS) to authenticate, authorize, and encrypt the communications with other nodes in the swarm.
- When you create a swarm by running `docker swarm init`, Docker designates itself as a manager node. 
- By default, the manager node generates a new root Certificate Authority (CA) along with a key pair, which are used to secure communications with other nodes that join the swarm. 
-  If you prefer, you can specify your own externally-generated root CA, using the --external-ca flag of the docker swarm init command.
- The manager node also generates two tokens to use when you join additional nodes to the swarm: one worker token and one manager token. 
- Each token includes the digest of the root CA’s certificate and a randomly generated secret.
- When a node joins the swarm, the joining node uses the digest to validate the root CA certificate from the remote manager. 
- The remote manager uses the secret to ensure the joining node is an approved node.
- Each time a new node joins the swarm, the manager issues a certificate to the node.
- The certificate contains a randomly generated node ID to identify the node under the certificate common name (CN) and the role under the organizational unit (OU). 
- The node ID serves as the cryptographically secure node identity for the lifetime of the node in the current swarm.

The diagram below illustrates how manager nodes and worker nodes encrypt communications using a minimum of TLS 1.2.
![](https://docs.docker.com/engine/swarm/images/tls.png)
- By default, each node in the swarm renews its certificate every three months
-  You can configure this interval by running the 
```
docker swarm update --cert-expiry <TIME PERIOD>
```
- The minimum rotation value is 1 hour. 
# Rotating the CA certificate
- In the event that a cluster CA key or a manager node is compromised, you can rotate the swarm root CA so that none of the nodes trust certificates signed by the old root CA anymore.
- generate a new CA certificate and key
```
docker swarm ca --rotate
```
Following happens on rotating the CA certificate
- Docker generates a cross-signed certificate. This means that a version of the new root CA certificate is signed with the old root CA certificate. This cross-signed certificate is used as an intermediate certificate for all new node certificates. This ensures that nodes that still trust the old root CA can still validate a certificate signed by the new CA.
- In Docker 17.06 and higher, Docker also tells all nodes to immediately renew their TLS certificates. This process may take several minutes, depending on the number of nodes in the swarm.
- After every node in the swarm has a new TLS certificate signed by the new CA, Docker forgets about the old CA certificate and key material, and tells all the nodes to trust the new CA certificate only.
- This also causes a change in the swarm’s join tokens. The previous join tokens are no longer valid.



