# Access control model
With Docker `Universal Control Plane`, you get to control who can create and edit container resources in your swarm, like services, images, networks, and volumes. You can grant and manage permissions to enforce fine-grained access control as needed.
# Grant access to swarm resources
If you’re a UCP administrator, you can create grants to control how users and organizations access swarm resources.
- A grant is made up of a subject, a role, and a resource collection. 
- A grant defines who (subject) has how much access (role) to a set of resources (collection). 
![](https://docs.docker.com/datacenter/ucp/2.2/guides/images/ucp-grant-model.svg)
# Subjects
- User
- Organization
- Team
# Roles
# Resource collections
Swarm resources that can be placed in to a collection include:
- Physical or virtual nodes
- Containers
- Services
- Networks
- Volumes
- Secrets
- Application configs
# Collection architecture
# Role composition
# Grant composition
![](https://docs.docker.com/datacenter/ucp/2.2/guides/images/access-control-grant-composition.png)
# Access architecture
The resulting access architecture defined by these grants is depicted below.
![](https://docs.docker.com/datacenter/ucp/2.2/guides/images/access-control-collection-architecture.png)
