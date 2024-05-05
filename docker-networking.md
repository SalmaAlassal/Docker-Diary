# Docker Networking

# Container Port vs Host Port

- App inside Docker container runs in an isolated docker network.
- To access the app from outside the container, we need to map the container port to the host port which is called port binding.

![Container Port vs Host Port](imgs/container-port-vs-host-port.png)

## Port Binding

- Port binding is the process of mapping a container port to a host port.
- It allows you to expose a container port to the outside world so that other machines on the network can access the application running inside the container.

- It's a standard practice to use the same port number for both the container port and the host port. For example, if your application is running on port 80 inside the container, you can map it to port 80 on the host machine.

> Note that only 1 service can run on a port at a time. If you try to run a service on a port that is already in use, you will get an error.
