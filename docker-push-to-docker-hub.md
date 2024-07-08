# Pushing Docker Images to Docker Hub

When deploying Docker containers in production, it's crucial to follow best practices to ensure a smooth and efficient workflow. **One key practice is to run your pre-built Docker images instead of building them directly in the production environment**. This is where Docker Hub comes in handy. (You can also use other container registries like Amazon ECR, Google Container Registry, etc.)

You first need to build the image in a development environment, test it, and then push it to a container registry. This way, you can pull the image from the registry in the production environment and run it without any issues.

We'll use **Docker Hub** to push our Docker image. Docker Hub is a cloud-based registry service that allows you to store your Docker images. It's free for public repositories, but you need a paid subscription for private repositories.

Here's how you can push your Docker image to Docker Hub:

### Step 1: Log in/Sign up to Docker Hub

If you don't have a Docker Hub account, sign up [here](https://hub.docker.com/). If you already have an account, log in.

### Step 2: Create a Docker Repository

Just like GitHub, Docker Hub uses repositories to store your Docker images. Create a new repository by clicking on the "Create Repository" button on the Docker Hub dashboard.

### Step 3: Tag your Docker Image

Docker tags are just an alias for an image ID.

Before pushing your Docker image to Docker Hub, you need to tag it. The tag should include your **Docker Hub username**, the **repository name**, and **an optional version tag**.

```bash
docker tag <image_name_or_id>:<tag> <docker_username>/<repository_name>:<hub_tag>
```

For example:

```bash
docker tag my_image:latest my_docker_username/my_repository:latest
```

### Step 4: Log in to Docker Hub from the CLI

Log in to Docker Hub from the command line using the following command:

```bash
docker login
```

Enter your Docker Hub credentials when prompted.

### Step 5: Push your Docker Image

Finally, push your Docker image to Docker Hub using the following command:

```bash
docker push <docker_username>/<repository_name>:<hub_tag>
```

For example:

```bash
docker push yourusername/my-app:v1.0
```
> You can list all your Docker images using the `docker image ls` command.

#### Using Docker Compose

You can also use **docker-compose** to push your image to Docker Hub. Just add the `image` key to your `docker-compose.yml` file. For example:

> Ensure the image name is correctly formatted as `my_docker_username/my_repository_name`.

```yaml
services:
  my_service:
    build: .
    image: my_docker_username/my_repository_name:v1.0
```
> Docker will build the image using the `build` key and then give it the name specified in the `image` key.

Then build the image using:

```bash
docker-compose -f docker-compose.yml up build my_service
```

Then run:

```bash
docker-compose -f docker-compose.yml push my_service
```
It will find the image specified in this service and push it to Docker Hub.

That's it! Your Docker image is now available on Docker Hub. You can pull it in any environment and run it without any issues.

### Step 6 Verify the Image on Docker Hub

Go to your Docker Hub account and check the repository you created. You should see your Docker image listed there.

--------------------------------------------

# Pulling Docker Images from Docker Hub

Pulling Docker images from Docker Hub is a straightforward process. You can pull images using the `docker pull` command followed by the image name and tag.

Here's how you can pull a Docker image from Docker Hub:

```bash
docker pull <docker_username>/<repository_name>:<hub_tag>
```

You can also use docker-compose to pull images from Docker Hub. Just specify the image name in your `docker-compose.yml` file under the `image` key.

For example:

```yaml
services:
  my_service:
    image: my_docker_username/my_repository_name
```

Then run your docker-compose file and it will pull the image automatically from Docker Hub.

```bash
docker-compose -f docker-compose.yml up -d
```
--------------------------------------------