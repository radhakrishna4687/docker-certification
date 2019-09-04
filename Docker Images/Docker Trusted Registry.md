# Docker Trusted Registry overview
- Docker Trusted Registry (DTR) is the enterprise-grade image storage solution from Docker.
- You install it behind your firewall so that you can securely store and manage the Docker images you use in your applications.
# Image and job management
- DTR can be installed on-premises, or on a virtual private cloud. And with it, you can store your Docker images securely, behind your firewall.
- You can use DTR as part of your continuous integration, and continuous delivery processes to build, ship, and run your applications.
- DTR has a web user interface that allows authorized users in your organization to browse Docker images and review repository events. 
- It even allows you to see what Dockerfile lines were used to produce the image and, if security scanning is enabled, to see a list of all of the software installed in your images. 
- Additionally, you can now review and audit jobs on the web interface.
# Availability
DTR is highly available through the use of multiple replicas of all containers and metadata such that if a machine fails, DTR continues to operate and can be repaired.
# Efficiency
# Built-in access control
- DTR uses the same authentication mechanism as Docker Universal Control Plane. 
- Users can be managed manually or synchronized from LDAP or Active Directory. 
- DTR uses Role Based Access Control (RBAC) to allow you to implement fine-grained access control policies for your Docker images.
# Security scanning
- DTR has a built-in security scanner that can be used to discover what versions of software are used in your images. 
- It scans each layer and aggregates the results to give you a complete picture of what you are shipping as a part of your stack. 
- Most importantly, it correlates this information with a vulnerability database that is kept up to date through periodic updates. 
- This gives you unprecedented insight into your exposure to known security threats.
# Image signing
DTR ships with Notary built in so that you can use Docker Content Trust to sign and verify images. For more information about managing Notary data in DTR see the DTR-specific notary documentation.

# DTR architecture
Docker Trusted Registry (DTR) is a containerized application that runs on a Docker Universal Control Plane cluster.
![](https://docs.docker.com/ee/dtr/images/architecture-1.svg)
Once you have DTR deployed, you use your Docker CLI client to login, push, and pull images.

# Under the hood
For high-availability you can deploy multiple DTR replicas, one on each UCP worker node.
![](https://docs.docker.com/ee/dtr/images/architecture-2.svg)
- All DTR replicas run the same set of services and changes to their configuration are automatically propagated to other replicas.
# DTR internal components
When you install DTR on a node, the following containers are started:
## dtr-api-<replica_id>
Executes the DTR business logic. It serves the DTR web application, and API
## dtr-garant-<replica_id>	
Manages DTR authentication
## dtr-jobrunner-<replica_id>	
Runs cleanup jobs in the background
## dtr-nginx-<replica_id>	
Receives http and https requests and proxies them to other DTR components. By default it listens to ports 80 and 443 of the host
## dtr-notary-server-<replica_id>	
Receives, validates, and serves content trust metadata, and is consulted when pushing or pulling to DTR with content trust enabled
## dtr-notary-signer-<replica_id>	
Performs server-side timestamp and snapshot signing for content trust metadata
## dtr-registry-<replica_id>	
Implements the functionality for pulling and pushing Docker images. It also handles how images are stored
## dtr-rethinkdb-<replica_id>	
A database for persisting repository metadata
## dtr-scanningstore-<replica_id>	
Stores security scanning data

All these components are for internal use of DTR. Don’t use them in your applications.

# Networks used by DTR
To allow containers to communicate, when installing DTR the following networks are created:
## dtr-ol type overlay
Allows DTR components running on different nodes to communicate, to replicate DTR data
# Volumes used by DTR
DTR uses these named volumes for persisting data:
## dtr-ca-<replica_id>	
Root key material for the DTR root CA that issues certificates
## dtr-notary-<replica_id>	
Certificate and keys for the Notary components
## dtr-postgres-<replica_id>	
Vulnerability scans data
## dtr-registry-<replica_id>	
Docker images data, if DTR is configured to store images on the local filesystem
## dtr-rethink-<replica_id>	
Repository metadata
## dtr-nfs-registry-<replica_id>	
Docker images data, if DTR is configured to store images on NFS

You can customize the volume driver used for these volumes, by creating the volumes before installing DTR. During the installation, DTR checks which volumes don’t exist in the node, and creates them using the default volume driver.
By default, the data for these volumes can be found at /var/lib/docker/volumes/<volume-name>/_data.
# Image storage
By default, Docker Trusted Registry stores images on the filesystem of the node where it is running, but you should configure it to use a centralized storage backend.
![](https://docs.docker.com/ee/dtr/images/architecture-3.svg)

DTR supports these storage backends:
1. NFS
2. Amazon S3
3. Cleversafe
4. Google Cloud Storage
5. OpenStack Swift
6. Microsoft Azure
# How to interact with DTR
DTR has a web UI where you can manage settings and user permissions.
![](https://docs.docker.com/ee/dtr/images/architecture-4.svg)
You can push and pull images using the standard Docker CLI client or other tools that can interact with a Docker registry.
# Docker Trusted Registry system requirements
Docker Trusted Registry can be installed on-premises or on the cloud. Before installing, be sure your infrastructure has these requirements.
## Minimum requirements
- 16GB of RAM for nodes running DTR
- 2 vCPUs for nodes running DTR
- 10GB of free disk space
# Use the CLI to enable pushing to repositories that don’t exist yet
```
curl --user <admin-user>:<password> \
--request POST "<dtr-url>/api/v0/meta/settings" \
--header "accept: application/json" \
--header "content-type: application/json" \
--data "{ \"createRepositoryOnPush\": true}"
```





