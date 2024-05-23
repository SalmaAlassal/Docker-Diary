# Docker Commands

Docker commands are used to interact with the Docker daemon via the CLI. Here are some of the most commonly used Docker commands:

## Docker Installation and Setup

Commands related to installing Docker and verifying the installation.


- `docker --version`: Shows the Docker version installed on the system.
- `docker info`: Shows detailed information about the Docker installation.

-------------------

## Docker Images

Commands for managing Docker images.

### Listing Images

- `docker images`: Lists all the Docker images on the local system.
- `docker image ls`: Lists all the Docker images on the local system.


### Pulling Images

- `docker pull <image-name>`: Pulls a Docker image from the Docker Hub.
- `docker pull <image-name>:<tag>`: Pulls a specific version of a Docker image.
    > Using a specific version is the best practice in most cases.


### Building Images

- `docker build -t <image-name> .`: Builds a Docker image from a Dockerfile in the current directory.
    > The `-t` flag is used to tag the image with a name.
    > The `.` at the end of the command specifies the build context (current directory).
    > It pulls the images specified in the Dockerfile if they are not available locally.

### Removing Images

- `docker rmi <image-name>`: Removes a Docker image.
    > You can use the image ID instead of the image name.

-------------------

## Docker Containers

Commands for running and managing Docker containers.

### Listing Containers

- `docker ps`: Lists all the running Docker containers.
- `docker ps -a`: Lists all the Docker containers, including the stopped ones.

### Running Containers

- `docker run <image-name>`: Runs a Docker container from an image.
    > Creates a new container from the specified image. It doesn't reuse an existing container.
    > If the image is not available locally, Docker will pull it from the Docker Hub.

- `docker run <image-name>:<tag>`: Runs a Docker container from a specific version of an image.
    > Docker generates a random name for the container if you don't specify a name.

- `docker run -d <image-name>`: Runs a Docker container in detached mode and prints the container ID.
    > detached mode means the container runs in the background and you can use the terminal for other commands.

- `docker run --name <container-name> <image-name>`: Runs a Docker container with a specific name.

### Containers Logs

- `docker logs <container-id>`: Shows the logs of a Docker container.

### Stopping and Removing Containers

- `docker stop <container-id>`: Stops a running Docker container.

- `docker rm <container-name> -f`: Removes a Docker container.
    > The `-f` flag is used to force the removal of a running container.


### Port Binding

- `docker run -p <host-port>:<container-port> <image-name>`: Maps a host port to a container port.
    > This is called port binding and is used to expose a container port to the outside world.

### Executing Commands in Containers

- `docker exec -it <container-name> <command>`: Executes a command in a running Docker container.
    > The `-it` flag is used to open an interactive terminal in the container.
    - E.g. `docker exec -it my-container bash` will open a bash shell in the `my-container` container.