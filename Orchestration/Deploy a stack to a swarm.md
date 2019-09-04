# Deploy a stack to a swarm
- When running Docker Engine in swarm mode, you can use `docker stack deploy` to deploy a complete application stack to the swarm.
- The `deploy` command accepts a stack description in the form of docker-compose file.
## Set up a Docker registry
- Because a swarm consists of multiple Docker Engines, a registry is required to distribute images to all of them. 
- You can use the Docker Hub or maintain your own.
- Here’s how to create a throwaway registry, which you can discard afterward.
1. Start the registry as a service on your swarm:
```
docker service create --name registry --publish published=5000,target=5000 registry:2
```
2. Check that it’s working 
```
curl http://localhost:5000/v2/
```
## Deploy the stack to the swarm
### Create stack 
```
docker stack deploy --compose-file docker-compose.yml python-redis-counter-app
```
### bring your Docker Engine out of swarm mode
```
docker swarm leave --force
```
### Scale service 
```
docker service scale SERVICE=REPLICAS [SERVICE=REPLICAS...]
docker service scale python-redis-counter-app_redis=2
```