# Manage sensitive data with Docker secrets
## How Docker manages secrets
- When you add a secret to the swarm, Docker sends the secret to the swarm manager over a mutual TLS connection. 
- The secret is stored in the Raft log, which is encrypted.
- The entire Raft log is replicated across the other managers, ensuring the same high availability guarantees for secrets as for the rest of the swarm management data.
- When you grant a newly-created or running service access to a secret, the decrypted secret is mounted into the container in an in-memory filesystem.
## Add Secrets
```
$ printf "This is a secret" | docker secret create my_secret_data -
```
## Use Secrets in Compose
```
version: '3.1'

services:
   db:
     image: mysql:latest
     volumes:
       - db_data:/var/lib/mysql
     environment:
       MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_root_password
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD_FILE: /run/secrets/db_password
     secrets:
       - db_root_password
       - db_password

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD_FILE: /run/secrets/db_password
     secrets:
       - db_password


secrets:
   db_password:
     file: db_password.txt
   db_root_password:
     file: db_root_password.txt

volumes:
    db_data:
```