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

--------------------------------------------

## Bind Mounts

Commands for using bind mounts in Docker containers.

### Mounting Directories

- `docker run -v <host-absolute-path>:<container-path> <image-name>`: Mounts a directory from the host machine to a directory inside the container. Any changes made to the files in the host directory will be reflected in the container directory, and vice versa.
    > The `-v` flag is used to specify the volume mount.
    > The host path must be an absolute path.an

- `docker run -v <host-absolute-path>:<container-path>:ro <image-name>`: Mounts a directory as read-only in the container. This means that you can only read from the container, not write to it.
    > The `:ro` flag makes the mount read-only.

--------------------------------------------

## Docker Compose

- `docker-compose --version`: Checks the version of Docker Compose installed on the system.

- `docker-compose --help`: Shows the help message for Docker Compose commands.

- `docker-compose up`: Builds, (re)creates, starts, and attaches to containers for a service.
    > This command is used to start the services defined in the `docker-compose.yml` file.

- `docker-compose up --build`: Builds the images before starting the services.
    > This is useful when you have made changes to the Dockerfile or application code.

- `docker-compose up -d`: Starts the services in detached mode.
    > The containers run in the background, and you can use the terminal for other commands.

- `docker-compose down`: Stops and removes the containers created by `docker-compose up`.


- `docker-compose -f <file-name> up`: You can use this command to start services defined in a file other than `docker-compose.yml`.

- `docker-compose -f <file-name> down`: Stops and removes the containers created by a specific Docker Compose file.

- `docker-compose -f <file-name> -f <another-file-name> up`: You can use this command to start services defined in multiple files.

- `docker-compose -f <file-name> up --build`: Rebuilds the images before starting the services.
    > This is useful when you have made changes to the Dockerfile or application code.

- `docker-compose -f <file-name> down -v`: Stops containers and removes volumes defined in the Docker Compose file.
    > The `-v` flag is used to remove volumes.

--------------------------------------------

## Docker Environment Variables

- `docker run -e <variable-name>=<value> <image-name>`: Passes environment variables to a Docker container at runtime.
    > The `-e` flag is used to pass environment variables. You can use `--env` as an alternative.
    > You can pass multiple environment variables by specifying them multiple times.
    > The best practice for naming environment variables is to use uppercase letters with underscores.

- `docker run --env-file <file-path> <image-name>`: Passes environment variables to a Docker container using an environment file.
    > The environment file is a text file that contains key-value pairs of environment variables.

- `docker exec -it <container-name> printenv`: Displays all the environment variables set in a running Docker container.

--------------------------------------------

## Container Inspections

- `docker inspect <container-id>/<container-name>`: Shows detailed information about a Docker container in JSON format.


- `docker inspect network <network-id>/<network-name>`: Shows detailed information about a Docker network in JSON format.

--------------------------------------------

## Docker Networking

- `docker network ls`: Lists all the Docker networks on the system.

--------------------------------------------

## Docker Volumes

- `docker volume ls`: Lists all the Docker volumes on the system.
- `docker volume prune`: Removes anonymous local volumes not used by at least one container.
--------------------------------------------

## Bridge Networks

- `docker network create <network-name>`: Creates a new bridge network in Docker.

- `docker network create <network-name> --subnet=<io-address>/<number> --gateway=<ip-address>`: Creates a new bridge network with a custom subnet and gateway.

- `docker network rm <network-name>`: Removes a Docker network.

--------------------------------------------