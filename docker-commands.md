# Docker Commands

Docker commands are used to interact with the Docker daemon via the CLI. Here are some of the most commonly used Docker commands:

- `docker images`: Lists all the Docker images on the local system.
- `docker ps`: Lists all the running Docker containers.
- `docker ps -a`: Lists all the Docker containers, including the stopped ones.
- `docker pull <image-name>`: Pulls a Docker image from the Docker Hub.
- `docker pull <image-name>:<tag>`: Pulls a specific version of a Docker image.
    > Using a specific version is the best practice in most cases.
- `docker run <image-name>`: Runs a Docker container from an image.
    > Creates a new container from the specified image. It doesn't reuse an existing container.
    > If the image is not available locally, Docker will pull it from the Docker Hub.
- `docker run <image-name>:<tag>`: Runs a Docker container from a specific version of an image.
    > Docker generates a random name for the container if you don't specify a name.
- `docker run -d <image-name>`: Runs a Docker container in detached mode and prints the container ID.
    > detached mode means the container runs in the background and you can use the terminal for other commands.

- `docker logs <container-id>`: Shows the logs of a Docker container.
- `docker stop <container-id>`: Stops a running Docker container.
- `docker run -p <host-port>:<container-port> <image-name>`: Maps a host port to a container port.
    > This is called port binding and is used to expose a container port to the outside world.

- `docker run --name <container-name> <image-name>`: Runs a Docker container with a specific name.
