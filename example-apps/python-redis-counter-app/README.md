# Test the app with Compose
# Build 
- Start the app with docker-compose up. This builds the web app image, pulls the Redis image if you don’t already have it, and creates two containers.
```
docker-compose up -d
```
# Bind mount
Edit `docker-compose.yml` in your project directory to add a bind mount for the web service:
```
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
  redis:
    image: "redis:alpine"
```
- The new volumes key mounts the project directory (current directory) on the host to /code inside the container, allowing you to modify the code on the fly, without having to rebuild the image. 
- The environment key sets the FLASK_ENV environment variable, which tells flask run to run in development mode and reload the code on change.

# Stop containers
```
docker-compose stop
docker-compose stop --volumes
```
