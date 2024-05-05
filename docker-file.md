# DockerFile

## Structure

- `FROM` - The base image for the new image

- `COPY` - Copies files from the host machine to the container
    - E.g. `COPY . /app` will copy all the files from the current directory on the host machine to the `/app` directory inside the container

- `WORKDIR` - Sets the working directory for all the following commands(`RUN`, `COPY`, etc.)
    - E.g. `WORKDIR /app` will set the working directory to `/app` inside the container

- `RUN` - Will execute any commands in a shell inside the container
    - E.g. `RUN npm install` will install all the dependencies for a Node.js application inside the container

- `CMD` - Specifies the command to run when the container starts
    - There can only be one `CMD` instruction in a Dockerfile

- > While `RUN` is executed in the container, `COPY` is executed on the host machine.

### Base Image `FROM`

- Dockerfiles start from a parent image or **base image**. The base image can be any image available on Docker Hub or a custom image created by you.

- It's a Docker image that your image is based on. It's like the foundation of a building. You can build on top of it, but you can't change it.

- You choose a base image based on the requirements of your application. For example, if you are building a Python application, you might choose the `python` image as your base image.

- Each image consists of multiple image layers. E.g. you can have a base image with an OS, then add a layer with Python, then add a layer with your application code.
    - This makes Docker so efficient. If you have multiple images that share the same base image, Docker only needs to store the base image once.

### Docker Container File System `COPY`

- Docker containers have their own file system that is isolated from the host machine's file system. This means that any changes made inside the container do not affect the host machine.

- The structure of a Docker container file system is similar to a Linux file system. It has directories like `/bin`, `/etc`, `/usr`, etc.

------------------------------------------------------------

## Building an Image

- To build an image from a Dockerfile, you use the `docker build` command.

- The `docker build` command takes the following syntax:
    ```bash
    docker build -t <your-image-name>:<tag> <path-to-your-dockerfile>
    ```