# Docker with MongoDB & NodeJS

In this example, we will create a simple Node.js application that connects to a MongoDB database. We will use Docker to run both the Node.js application and the MongoDB database.

## Node.js Application

First, let's create a simple Node.js application that connects to a MongoDB database. Create a new directory for your project and add the following file:

### `index.js`

```javascript
const express = require('express');
const mongoose = require('mongoose');

// initialize app
const port = process.env.PORT || 4000;
const app = express();

// connect to MongoDB
const DB_USER = 'root';
const DB_PASSWORD = 'example';
const DB_PORT = 27017;
const DP_HOST = "172.20.0.2"; // This is the IP address of the MongoDB container
const URI = `mongodb://${DB_USER}:${DB_PASSWORD}@${DP_HOST}:${DB_PORT}`;
mongoose.connect(URI).then(() => {
    console.log('Connected to MongoDB');
}).catch((err) => {
    console.error('Error connecting to MongoDB:', err.message);
});


//
app.get('/', (req, res) => res.send('<h1>Hello World</h1>'));
app.listen(port, () => console.log(`Server is running on port ${port}`));
```

### `package.json`

```json
{
  "name": "node-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "start-dev": "nodemon index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "description": "",
  "dependencies": {
    "express": "^4.17.1"
  },
  "devDependencies": {
    "nodemon": "^2.0.12"
  }
}
```

--------------------------------------------

## Dockerfile

The Dockerfile defines the steps to build an image for the Node.js application. We will use a multi-stage build to create separate images for development and production environments.

```Dockerfile
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

--------------------------------------------

## Docker Compose Files

We will create three Docker Compose files for different environments:
  - `docker-compose.yml`: The base configuration for both development and production environments.
  - `docker-compose.dev.yml`: Configuration for the development environment.
  - `docker-compose.prod.yml`: Configuration for the production environment.

**docker-compose.yml**

In this file, we define two services: `express-node-app` for our Node.js application and `mongo` for the MongoDB database.

> You can always go to docker hub and search for the image you want to use and copy the `docker-compose` file from there.


```yaml
services:
  express-node-app: # Service name
    container_name: express-node-app-container # Container name
    ports:
      - '4000:4000' # Host port:Container port (It will expose the container port 4000 to the host port 4000)
    env_file: # Load environment variables from a file
      - .env
  mongo: # Service name
    image: mongo # Use the official MongoDB image from Docker Hub
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
```

**docker-compose.dev.yml**

```yaml
services:
  express-node-app: # Service name
    build:
        context: . # The path to the Dockerfile
        target: development # The target stage to build
    volumes:
      - .:/app # Mount the current directory to /app in the container
      - /app/node_modules # Prevent the node_modules directory from being overridden
    environment:
        NODE_ENV: development # Set the NODE_ENV environment variable
    command: npm run start-dev # The command to run when the container starts
```

**docker-compose.prod.yml**

```yaml
services:
  express-node-app: # Service name
    build:
        context: . # The path to the Dockerfile
        target: production # The target stage to build
    environment:
        NODE_ENV: production # Set the NODE_ENV environment variable
    command: npm start # The command to run when the container starts
```
--------------------------------------------

## Running the Containers with Docker Compose

To run the containers using Docker Compose, use the following commands:

```bash
cd /path/to/your/project

# For the development environment
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d --build

# For the production environment
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build
```

This will start the Node.js application and the MongoDB database in separate containers. Run `docker container ls` to see the running containers.

You can access the Node.js application at `http://localhost:4000`.

## MongoDB Connection

In the Node.js application, we are connecting to the MongoDB:
  
```javascript
// connect to MongoDB
const DB_USER = 'root';
const DB_PASSWORD = 'example';
const DB_PORT = 27017;
const DP_HOST = "172.20.0.2"; // This is the IP address of the MongoDB container
const URI = `mongodb://${DB_USER}:${DB_PASSWORD}@${DP_HOST}:${DB_PORT}`;
mongoose.connect(URI).then(() => {
    console.log('Connected to MongoDB');
}).catch((err) => {
    console.error('Error connecting to MongoDB:', err.message);
});
```

Docker creates a network by default for the services defined in the `docker-compose.yml` file. Each service has its own internal IP address that can be used to communicate between containers.

The Node.js application container can connect to the MongoDB container using the internal IP address. You will find the IP address in the `NetworkSettings` section of the `docker inspect` output.

The IP address `172.20.0.2` is the internal IP address of the MongoDB container. You can find the IP address by running `docker inspect <container_id>`.

The NetworkSettings section will have an entry like this:

```json
"NetworkSettings": {
    "Networks": {
        "nodeapp_default": {
            "IPAMConfig": null,
            "Links": null,
            "Aliases": [
                "nodeapp-mongo-1",
                "mongo"
            ],
            ...
            ...
            "IPAddress": "172.20.0.2",
            ...
            ...
        }
```
### `nodeapp_default`

The `nodeapp_default` is the default network created by Docker Compose for the services defined in the `docker-compose.yml` file, so it has two containers running on it: `nodeapp-mongo-1` and `express-node-app-container`. You can check this by running `docker network inspect nodeapp_default`.

The network name is composed of the project name and the suffix `_default`.

> You can list the networks on your system using the command `docker network ls`

**Do you find yourself wondering if we need to start the container every time we want to connect to the database to obtain the IP address?** It's really bad practice and not a practical solution.

Docker Compose provides a better way to handle this by using service names as hostnames. Service names are the names of the services defined in the `docker-compose.yml` file.

In the `index.js` file, you can replace the IP address with the service name `mongo`:

```javascript
const DP_HOST = "mongo"; // This is the service name of the MongoDB container
```

This way, the Node.js application can connect to the MongoDB container using the service name `mongo` instead of the IP address. Docker Compose will resolve the service name to the internal IP address of the MongoDB container.

--------------------------------------------

## Play with mongoDB

You can access the MongoDB database using the following command:

```bash
docker exec -it nodeapp-mongo-1 bash
```

This will open a bash shell inside the MongoDB container. You can then connect to the MongoDB database using the `mongo` command:

```bash
mongosh -u root -p example
```

> You can execute the `mongosh` directly : `docker exec -it nodeapp-mongo-1 mongosh -u root -p example`

Try running some MongoDB commands to interact with the database:
- `show dbs:` Lists all databases.
- `use testDB:` Switches to (or creates) the testDB database.
- `db.books.insert(...):` Inserts a document into the books collection.
- `db.books.find():` Retrieves all documents from the books collection.

```bash
show dbs
use testDB
db.books.insert({ title: 'Book 1', author: 'Author 1' })
db.books.find()
```

> If you stop the containers and start them again, you will lose the data stored in the MongoDB database. To persist the data, you can use Docker volumes.
--------------------------------------------

## Store Data with Docker Volumes

To persist the data stored in the MongoDB database, you can use Docker volumes. Docker volumes are used to store data outside the container's writable layer.

MongoDB stores its data in the `/data/db` directory inside the container. We can create a Docker volume to store the data on the host machine.

Update the `docker-compose.yml` file to include a volume for the MongoDB service:

```yaml
services:
  express-node-app:
    container_name: express-node-app-container
    ports:
      - '4000:4000'
    env_file:
      - .env
  mongo:
    image: mongo
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    volumes:
      - mongo-db:/data/db # Named volume for MongoDB data. The data to be stored in the /data/db directory inside the container will be stored in the volume named mongo-db.
  
volumes:
  mongo-db: # Volume name
```

What we did here is to create a volume named `mongo-db` that will store the data of the MongoDB database. This volume will be created on the host machine and mounted to the `/data/db` directory inside the MongoDB container. So, the data will be stored on the host machine and will persist even if the containers are stopped or removed.

So try to run the containers again and play with the MongoDB database. You will see that the data persists even if you stop and start the containers.

You can list the volumes on your system using the command `docker volume ls`.

--------------------------------------------

## Stopping the Containers

To stop the containers, use the following command:

```bash
# For the development environment
docker-compose -f docker-compose.yml -f docker-compose.dev.yml down

# For the production environment
docker-compose -f docker-compose.yml -f docker-compose.prod.yml down
```

This will stop and remove the containers created by Docker Compose.

> You can stop the containers and remove the volumes that were created by adding the `-v` flag to the `down` command: `docker-compose -f docker-compose.yml -f docker-compose.dev.yml down -v`.

--------------------------------------------
--------------------------------------------

## Mongo-Express

Mongo Express is a web-based MongoDB admin interface written in Node.js. It provides a simple and easy-to-use interface to interact with MongoDB databases.

We will add the Mongo Express service to our Docker Compose configuration to manage the MongoDB database.

**docker-compose.yml**

```yaml
services:
  express-node-app:
    container_name: express-node-app-container
    ports:
      - '4000:4000'
    env_file:
      - .env
  mongo:
    image: mongo
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    volumes:
      - mongo-db:/data/db

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
      ME_CONFIG_MONGODB_URL: mongodb://root:example@mongo:27017/

volumes:
  mongo-db:
```

In this configuration, we added a new service `mongo-express` that uses the `mongo-express` image from Docker Hub. We exposed the port `8081` on the host machine to access the Mongo Express web interface.

Notice that we set the `ME_CONFIG_MONGODB_URL` environment variable to `mongodb://root:example@mongo:27017/`. This tells Mongo Express to connect to the MongoDB database running in the `mongo` service. The service name `mongo` is resolved to the internal IP address of the MongoDB container.

Try running the containers again with the updated configuration. You will now have 3 services running: the Node.js application, the MongoDB database, and the Mongo Express web interface.

```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d --build
```

To access the Mongo Express web interface, go to `http://localhost:8081` in your browser. You can log in using the username `root` and password `example`.

--------------------------------------------

## Docker Compose Dependencies

Docker Compose does not start services in a specific order by default. However, it allows you to define dependencies between services using the `depends_on` key in the `docker-compose.yml` file.

This means you can specify that one service depends on another service, and Docker Compose will start the services in the correct order.

For example, if the Node.js application depends on the MongoDB database, you can define the dependency as follows:

```yaml
services:
  express-node-app:
    container_name: express-node-app-container
    ports:
      - '4000:4000'
    env_file:
      - .env
    depends_on:
      - mongo
  mongo:
    image: mongo
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    volumes:
      - mongo-db:/data/db

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
      ME_CONFIG_MONGODB_URL: mongodb://root:example@mongo:27017/
    depends_on:
      - mongo

volumes:
  mongo-db:
```
--------------------------------------------