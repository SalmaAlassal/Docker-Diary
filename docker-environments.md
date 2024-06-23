# Docker Environments

Remember why we use Docker? To create isolated environments for our applications. We can create different environments for development, testing, and production. This way, we can ensure that our application works the same way in all environments.

Docker Compose is an excellent tool for optimizing the process of creating development, testing, staging, and production environments. It allows you to define your application's services in a `docker-compose.yml` file and run them with a single command.

## Development Environment
    
When working on a project, you might want to have a development environment that is different from the production environment. For example, you might want to use hot reload to see the changes you make to the code immediately. You can define a development environment in a `docker-compose.dev.yml` file and run it with the `docker-compose -f docker-compose.dev.yml up -d` command.

> Notice that by default, Docker Compose looks for a `docker-compose.yml` file in the current directory. If you use a different file name, you need to specify it with the `-f` flag.

Here's an example of a `docker-compose.dev.yml` file for a Node.js application:

```yaml
services:
  express-node-app:
    container_name: express-node-app-container
    build: .
    ports:
      - '4000:4000'
    volumes:
      - .:/app
      - /app/node_modules
    environment:
        NODE_ENV: development
    env_file:
      - .env
```

## Production Environment

When deploying your application to production, you might want to use a different configuration. For example, you might want to use a different database or set environment variables for security reasons. You can define a production environment in a `docker-compose.prod.yml` file and run it with the `docker-compose -f docker-compose.prod.yml up -d` command.

Here's an example of a `docker-compose.prod.yml` file for a Node.js application:

```yaml
services:
  express-node-app:
    container_name: express-node-app-container
    build: .
    ports:
      - '4000:4000'
    environment:
        NODE_ENV: production
    env_file:
      - .env
```

> We use simple examples here, but in real-world applications, you might have more services and configurations.

## Common Configuration

You can notice that the their is common configuration between the two files above, and only the environment variables and the volumes are different. So, you can define a `docker-compose.yml` file with the common configuration and add the specific configuration for each environment in separate files.


**docker-compose.yml**

```yaml
services:
  express-node-app:
    container_name: express-node-app-container
    build: .
    ports:
      - '4000:4000'
    env_file:
      - .env
```

**docker-compose.dev.yml**

```yaml
services:
  express-node-app:
    volumes:
      - .:/app
      - /app/node_modules
    environment:
        NODE_ENV: development
```

**docker-compose.prod.yml**

```yaml
services:
  express-node-app:
    environment:
        NODE_ENV: production
```

Now you can run the development environment with `docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d` and the production environment with `docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d`.

To close all the services, you can use `docker-compose -f docker-compose.yml -f docker-compose.dev.yml down` or `docker-compose -f docker-compose.yml -f docker-compose.prod.yml down`.

This way, you can have different environments for your application without duplicating the common configuration.

--------------------------------------------

## Notes

- Don't forget to add yml files to your `.gitignore` file to avoid committing sensitive information like environment variables.


- You would need to use the `--build` option in `docker-compose up` to ensure that Docker Compose rebuilds the images before starting the services. This is particularly useful in scenarios where your Dockerfile or application code has changed, and you need to make sure that the latest changes are reflected in the running containers.
    - E.g. `docker-compose -f docker-compose.yml -f docker-compose.prod.yml up --build -d`.


### Back to DockerFile

Remember our Dockerfile?

```dockerfile
FROM node

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

ENV PORT=4000

EXPOSE $PORT

CMD ["npm", "run", "start-dev"]
```

In our `docker-compose.yml` file, we used the `build: .` option to build the image from the current directory. So the development and production environments will use the same Dockerfile to build the image.

But `RUN npm install` installs all the dependencies specified in the `package.json` file. This is not ideal for production environments because it installs development dependencies as well. To avoid this, you can create a separate `Dockerfile` for production that only installs production dependencies.

--------------------------------------------

## Dockerfile for each Environment

You can create a `Dockerfile` for each environment to install the dependencies you need. For example, you can create a `Dockerfile.dev` for development and a `Dockerfile.prod` for production.

You need to specify the `Dockerfile` to use in the `docker-compose.dev.yml` and `docker-compose.prod.yml` files.

**docker-compose.yml**

```yaml
services:
  express-node-app:
    container_name: express-node-app-container
    ports:
      - '4000:4000'
    env_file:
      - .env
```

**docker-compose.dev.yml**

```yaml
services:
  express-node-app:
    build: ./Dockerfile.dev
    volumes:
      - .:/app
      - /app/node_modules
    environment:
        NODE_ENV: development
```

**docker-compose.prod.yml**

```yaml
services:
  express-node-app:
    build: ./Dockerfile.prod
    environment:
        NODE_ENV: production
```

--------------------------------------------

## Using single Dockerfile for multiple Environments

In this docker file we have two problems:

1. We are installing all the dependencies in both environments.
2. We only need to run `npm run start-dev` in the development environment and `npm start` in the production environment.


```dockerfile
FROM node

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

ENV PORT=4000

EXPOSE $PORT

CMD ["npm", "run", "start-dev"]
```

- To solve the first problem, we can use `ARG` to pass the environment to the Dockerfile and conditionally install the dependencies.
- To solve the second problem, we can override the `CMD` instruction in the `docker-compose.dev.yml` and `docker-compose.prod.yml` files.

```dockerfile
FROM node

WORKDIR /app

COPY package.json .

ARG NODE_ENV

RUN if [ "$NODE_ENV" = "production" ]; \
    then npm install --only=production; \
    else npm install; \
    fi

COPY . .

ENV PORT=4000

EXPOSE $PORT

CMD ["npm", "run", "start-dev"]
```

**docker-compose.yml**

```yaml
services:
  express-node-app:
    container_name: express-node-app-container
    ports:
      - '4000:4000'
    env_file:
      - .env
```

**docker-compose.dev.yml**

```yaml
services:
  express-node-app:
    build:
        context: . # The path to the Dockerfile
        args:
            - NODE_ENV=development
    volumes:
      - .:/app
      - /app/node_modules
    environment:
        NODE_ENV: development
    command: npm run start-dev # Override the CMD instruction
```

**docker-compose.prod.yml**

```yaml
services:
  express-node-app:
    build:
        context: . # The path to the Dockerfile
        args:
            - NODE_ENV=production
    environment:
        NODE_ENV: production
    command: npm start # Override the CMD instruction
```

**Run the development environment**

```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d --build
docker exec -it express-node-app-container bash
cd node_modules
ls -d nod* # You should see nodemon in the list
```

**Run the production environment**

```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build
docker exec -it express-node-app-container bash
cd node_modules
ls -d nod* # You should not see nodemon in the list
```

-------------------------------------------

# Docker Multistage Builds

Multistage builds are a feature of Docker that allows you to create multiple stages in a single Dockerfile. Each stage can have its own set of instructions and build context, which means that the resulting image can be optimized for size and performance. In a multi-stage build, each stage produces an intermediate image that is used as the build context for the next stage. The final stage produces the image that will be used to run the application.

The key advantage of using multi-stage builds is that it allows developers to reduce the size of the final image. By breaking down the build process into smaller stages, it becomes easier to remove unnecessary files and dependencies that are not needed in the final image. This can significantly reduce the size of the image, which can lead to faster deployment times and lower storage costs.

### Backing to the Example

We will create two stages: one for development and one for production. The development stage will include all the instructions needed to run the application in a development environment, while the production stage will include only the instructions needed to run the application in a production environment.

```dockerfile
FROM node AS development

WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
ENV PORT=4000
EXPOSE $PORT
CMD ["npm", "run", "start-dev"]

FROM node AS production

WORKDIR /app
COPY package.json .
RUN npm install --only=production
COPY . .
ENV PORT=4000
EXPOSE $PORT
CMD ["npm", "start"]
```

When there are common instructions between the stages, you can define a base stage and use it as the build context for the other stages. This can help reduce duplication and make the Dockerfile easier to maintain.

```dockerfile
FROM node AS base
# Common instructions

FROM base AS development
# Development-specific instructions

FROM base AS production
# Production-specific instructions
```

And to specify the target stage to build, you can use the `target` option in the `docker-compose.yml` file.


**docker-compose.yml**

```yaml
services:
  express-node-app:
    container_name: express-node-app-container
    ports:
      - '4000:4000'
    env_file:
      - .env
```

**docker-compose.dev.yml**

```yaml
services:
  express-node-app:
    build:
        context: . # The path to the Dockerfile
        target: development # The target stage to build
    volumes:
      - .:/app
      - /app/node_modules
    environment:
        NODE_ENV: development
    command: npm run start-dev # Override the CMD instruction
```

**docker-compose.prod.yml**

```yaml
services:
  express-node-app:
    build:
        context: . # The path to the Dockerfile
        target: production # The target stage to build
    environment:
        NODE_ENV: production
    command: npm start # Override the CMD instruction
```
--------------------------------------------