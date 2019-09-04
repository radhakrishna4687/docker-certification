# Docker Compose
- Compose is a tool for defining and running multi-container Docker applications.
- With Compose, you use a YAML file to configure your application’s services.
- Then, with a single command, you create and start all the services from your configuration. 

Using Compose is basically a three-step process:
1. Define your app’s environment with a Dockerfile so it can be reproduced anywhere.
2. Define the services that make up your app in docker-compose.yml so they can be run together in an isolated environment.
3. Run `docker-compose up` and Compose starts and runs your entire app.

# Features
- Multiple isolated environments on a single host
- Preserve volume data when containers are created
- Only recreate containers that have changed
- Variables and moving a composition between environments

# Sample docker-compose.yaml
```
version: '3'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```
# Install Compose for Docker Machine Manager
- ssh into manager node
- Run following command
```
curl -L "https://github.com/docker/compose/releases/download/1.10.0/docker-compose-$(uname -s)-$(uname -m)" -o /tmp/docker-compose
chmod +x /tmp/docker-compose
sudo cp /tmp/docker-compose /usr/local/bin/docker-compose
```
- Check installation of Docker Compose
```
docker-compose version
```

# Build and run your app with Compose
```
docker-compose up
```




