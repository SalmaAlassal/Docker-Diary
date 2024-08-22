# My Solution to the [Traffic Light Docker Challenge](https://github.com/hayk96/trafficlight-docker-challenge#vertical_traffic_light-traffic-light-docker-challenge)


## 1️⃣ Task1: Writing Dockerfiles and building Docker images

### 1.1 Dockerfile

- `~/trafficlight-docker-challenge $ vim Dockerfile`
- `cp Dockerfile green/ && cp Dockerfile red/ && cp Dockerfile yellow/`

This dockerfile will be placed in each of the three web app directories.
```
|
|-- green
|   |-- Dockerfile
|   |-- ...
|
|-- red
|   |-- Dockerfile
|   |-- ...
|
|-- yellow
|   |-- Dockerfile
```

```Dockerfile
# Use Node.js 17.0 with Alpine 3.14 as the base image
FROM node:17.0-alpine3.14

# Set the working directory inside the container to /app
WORKDIR /app

# Copy the package files to the container
COPY package*.json .

# Install npm version 7.17.0 via Alpine package manager (apk)
RUN apk add npm=7.17.0-r0

# Install project dependencies defined in package.json
RUN npm install

# Create a directory for view templates and static assets
RUN mkdir /app/views/

# Copy the main application file to the working directory
COPY app.js .

# Copy the view template and favicon to the views directory
COPY index.pug favicon.ico /app/views/

# Open port 80 for incoming traffic
EXPOSE 80

# Start the application using Node.js
CMD ["node", "app.js"]
```

### 1.2 Building Docker Images

- `cd green/ && docker build -t trafficlight/green:v1.0 .`
- `cd red/ && docker build -t trafficlight/red:v1.0 .`
- `cd yellow/ && docker build -t trafficlight/yellow:v1.0 .`

----------------

## 2️⃣ Task2: Running web apps behind Nginx reverse proxy


### 2.1 Create Docker Network

- `docker network create --subnet=172.20.0.0/16 --gateway=172.20.0.1 traffic-light`

### 2.2 Run Containers for Each Web App

```bash
docker run -d --name red-app --network traffic-light trafficlight/red:v1.0
docker run -d --name yellow-app --network traffic-light trafficlight/yellow:v1.0
docker run -d --name green-app --network traffic-light trafficlight/green:v1.0
```

### 2.3 Setup Nginx as a Reverse Proxy

- `mkdir -p nginx/conf.d/`
- `vim nginx/conf.d/trafficlight.conf`

```nginx
server {
    listen 3000;
    location / {
        proxy_pass http://red-app:80;
    }
}

server {
    listen 4000;
    location / {
        proxy_pass http://yellow-app:80;
    }
}

server {
    listen 5000;
    location / {
        proxy_pass http://green-app:80;
    }
}
```

- `docker pull nginx:1.21`
- `docker run -d --name nginx-proxy --network traffic-light -p 3000:3000 -p 4000:4000 -p 5000:5000 -v $(pwd)/nginx/conf.d:/etc/nginx/conf.d nginx:1.21`
- Visit `http://localhost:3000`, `http://localhost:4000`, `http://localhost:5000` to see the web apps.

----------------

## 3️⃣ Task3: Running web apps with Nginx load balancing

### 3.1 && 3.2 Create docker-compose.yml

- `vim docker-compose.yml`

```yaml
services:
  red-app:
    image: trafficlight/red:v1.0
    networks:
      - traffic-light

  yellow-app:
    image: trafficlight/yellow:v1.0
    networks:
      - traffic-light

  green-app:
    image: trafficlight/green:v1.0
    networks:
      - traffic-light

  nginx-load-balancer:
    image: nginx:1.21
    ports:
      - "80:80"
    volumes:
      - ./nginx/conf.d/:/etc/nginx/conf.d/
      - /var/log/nginx/:/var/log/nginx/
    networks:
      - traffic-light

networks:
  traffic-light:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1
```

### 3.3 && 3.4 Nginx Load Balancer Configuration

- `vim nginx/conf.d/trafficlight.conf`

```nginx
upstream backend {
    server red-app:80;
    server yellow-app:80;
    server green-app:80;
}

server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

```

<!-- - `htpasswd -c nginx/conf.d/.htpasswd user1` -->
- `docker-compose up -d`

------------------------------