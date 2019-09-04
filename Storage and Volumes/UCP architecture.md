# UCP architecture
Universal Control Plane is a containerized application that runs on Docker Enterprise Edition, extending its functionality to simplify the deployment, configuration, and monitoring of your applications at scale.

UCP also secures Docker with role-based access control so that only authorized users can make changes and deploy applications to your Docker cluster.
![](https://docs.docker.com/ee/ucp/images/ucp-architecture-1.svg)

Once Universal Control Plane (UCP) instance is deployed, developers and IT operations no longer interact with Docker Engine directly, but interact with UCP instead. Since UCP exposes the standard Docker API, this is all done transparently, so that you can use the tools you already know and love, like the Docker CLI client and Docker Compose.
# Under the hood
Docker UCP leverages the clustering and orchestration functionality provided by Docker.
![](https://docs.docker.com/ee/ucp/images/ucp-architecture-2.svg)


