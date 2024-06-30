# Docker with Nginx, MongoDB & NodeJS

We will continue from the previous [Docker with MongoDB & NodeJS](docker-mongodb-nodejs.md) and add Nginx to the mix.

## Nginx

Nginx is a web server that can also be used as a reverse proxy, load balancer, mail proxy, and HTTP cache. It is free and open-source software.

What we want to do is to use Nginx as a reverse proxy to route requests to our NodeJS server. This way, we can have multiple services running on different ports and have Nginx route the requests to the appropriate service.

## Docker Compose

To add a new service, we need to search for the appropriate image on Docker Hub. For this example, we will use the official Nginx image and integrate it into our `docker-compose.yml` file.

**docker-compose.yml**

```yml
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

  nginx:
      image: nginx:stable-alpine
      ports:
        - "8080:80"
      
volumes:
    mongo-db:
```

> Run the containers and check if the Nginx server is running on `localhost:8080`.

--------------------------------

## Nginx Configuration

We need to configure Nginx to route requests to our NodeJS server. We will create a new file called `default.conf` in the `nginx` directory.

**nginx/default.conf**

```conf
server {
    listen 80; # Listen on port 80 for incoming requests

    location / { # For any incoming requests to the root URL (http://localhost/) apply the following settings
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;

        # Pass the request to the Node.js server running on port 4000
        proxy_pass http://express-node-app:4000; # The name of the Docker container running the Node.js server
        proxy_redirect off;
    }
}
```

The default Nginx configuration file in the container is located at `/etc/nginx/conf.d/default.conf`. We need to replace this file with our custom configuration file.

In the `nginx` service in the `docker-compose.yml` file, we will add a volume to mount our custom configuration file.

**docker-compose.yml**

```yml
...
...
  nginx:
    image: nginx:stable-alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
...
...
```

If you run the containers now, you will notice that the Nginx container is not running. This is because nginx is trying to start before the `express-node-app` service is up and running.We need to add a `depends_on` directive to the `nginx` service to ensure that it starts after the `express-node-app` service.

**docker-compose.yml**

```yml
...
...
  nginx:
    image: nginx:stable-alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - express-node-app
...
...
```

Now, if you run the containers, you should see the Nginx server running on `localhost:8080` and routing requests to the NodeJS server running on `localhost:4000`.

Try accessing `http://localhost:8080` in your browser, and you should see the response from the NodeJS server.

--------------------------------