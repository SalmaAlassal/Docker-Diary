# Docker Networking

When you run applications in Docker containers, these containers need to communicate with each other, with the host machine, and sometimes with external systems. Docker Networking provides the tools and mechanisms to make this possible.

Docker Networking is to connect the docker container to each other and outside world so they can communicate with each other also they can talk to Docker Host. You can connect docker containers to non-Docker workloads such as VMs, physical servers, and other cloud workloads.

# Container Port vs Host Port

- App inside Docker container runs in an isolated docker network.
- To access the app from outside the container, we need to map the container port to the host port which is called port binding.

![Container Port vs Host Port](imgs/container-port-vs-host-port.png)

## Port Binding

- Port binding is the process of mapping a container port to a host port.
- It allows you to expose a container port to the outside world so that other machines on the network can access the application running inside the container.

- It's a standard practice to use the same port number for both the container port and the host port. For example, if your application is running on port 80 inside the container, you can map it to port 80 on the host machine.

> Note that only 1 service can run on a port at a time. If you try to run a service on a port that is already in use, you will get an error.

----------------

## Docker Networking Types

Docker networks configure communications between neighboring containers and external services. Containers must be connected to a Docker network to receive any network connectivity. The communication routes available to the container depend on the network connections it has.

Let's take a look at the different types of Docker networks:

### None Network Driver

- The none network driver does not attach containers to any network. Containers do not access the external network or communicate with other containers. You can use it when you want to disable the networking on a container.

### Bridge Network Driver (Default)

- **This is the default.** Whenever you start Docker, a bridge network gets created and all newly started containers will connect automatically to the default bridge network.

- You can use this whenever you want your containers running in isolation to connect and communicate with each other. Since containers run in isolation, the bridge network solves the port conflict problem. Containers running in the same bridge network can communicate with each other, and Docker uses **iptables** on the host machine to prevent access outside of the bridge.

- Containers on the same bridge network can communicate with each other using IP addresses. Every time you run a container, a different IP address gets assigned to it.

- External access to a container is granted by exposing ports to the host machine using **port binding**.

- You can list the bridge networks using the following command: `docker network ls`

### Host Network Driver

- The host network driver allows containers to use the host machine’s network directly, eliminating network isolation between the container and the host. For instance, if a container using the host network is configured to bind to port 80, the application within the container will be accessible on port 80 of the host’s IP address. Essentially, any process within the container that listens on port 80 will be available at `<your_host_ip>:80`.

- You can use the host network if you don’t want to rely on Docker’s networking but instead rely on the host machine networking.

> The host networking driver only works on Linux hosts, and is not supported on Docker Desktop for Mac, Docker Desktop for Windows, or Docker EE for Windows Server.

### Overlay Network Driver (Multi-Host Networking)

- Allows containers running on different Docker hosts (in a Docker Swarm or Kubernetes cluster) to communicate with each other.

### Macvlan Network Driver

- Macvlan allows containers to appear as physical devices on your network. It works by assigning each container in the network a unique MAC address. Docker then routes traffic to the container based on the MAC address.

----------------
