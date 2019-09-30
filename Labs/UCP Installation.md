# Install Docker Enterprize engine
https://docs.docker.com/install/linux/docker-ee/ubuntu/
# Install UCP
```
# Pull the latest version of UCP
docker image pull docker/ucp:3.2.1

# Install UCP
docker container run --rm -it --name ucp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  docker/ucp:3.2.1 install \
  --host-address 192.168.99.101 \
  --interactive \
  --force-minimums \
  --swarm-port 2777
```
# Get free trial license 
https://hub.docker.com/editions/enterprise/docker-ee-trial/trial
# Install Docker Trusted Registry
Use ssh to log in to the host where you already installed UCP, and run:
```
docker container run -it --rm \
  docker/dtr:2.7.2 install \
  --ucp-node <node-hostname> \
  --ucp-insecure-tls
```
