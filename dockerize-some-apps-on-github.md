# Dockerize some apps on Github

I will pick some random apps from Github and Dockerize them. This will help me gain more experience with Docker and also learn how to Dockerize different types of applications.

- [Calculator App using Flask & JS](#calculator-app-using-flask--js)
- [E-commerce App using NodeJS](#e-commerce-app-using-nodejs)


# [Calculator App using Flask & JS](https://github.com/helloflask/calculator)

I serached for a calculator app using Flask and found this simple calculator app. I will Dockerize this app and run it using Docker. Below are the steps I followed to Dockerize this app.

**requirements.txt** : I added only 2 dependencies to the requirements file that existed in the project.

```txt
markupsafe==2.0.1
itsdangerous==2.0.1
```

**Dockerfile**

```Dockerfile
# Use an official Python runtime as the base image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file into the container
COPY requirements.txt .

# Install the required packages
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code into the container
COPY . .

# Set environment variables
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0

# Expose the port on which the app will run
EXPOSE 5000

# Run the Flask application
CMD ["flask", "run", "--host=0.0.0.0"]
```

> I use `FLASK_RUN_HOST=0.0.0.0` to make the app accessible from outside the container as Flask runs by default on localhost. By default, Flask's development server binds to `127.0.0.1` (localhost), which means it will only be accessible from within the container itself. This is fine if you're running Flask directly on your local machine, but in a Docker container, it would restrict access to the Flask application from outside the container.
Setting `FLASK_RUN_HOST=0.0.0.0` tells Flask to listen on all available network interfaces, not just `127.0.0.1`. This makes the Flask app accessible from outside the container, such as from your host machine or other machines on the network.


**docker-compose.yml**

```yaml
services:
  calculator:
    build: .
    container_name: calculator
    ports:
      - "5000:5000"
    networks:
      - calculator-network

networks:
  calculator-network:
    driver: bridge
```

Run the app using `docker-compose up` and access it at `http://localhost:5000`.

--------------------------------------------

# [E-commerce App using NodeJS](https://github.com/khalidMhd/Node-js-Ecommerce)

I found this E-commerce app using NodeJS. I will Dockerize this app and run it using Docker. Below are the steps I followed to Dockerize this app

**Dockerfile**

```Dockerfile
FROM node:14

WORKDIR /app

COPY package*.json .

RUN npm install

COPY . .

EXPOSE 3000

CMD [ "npm", "start" ]
```

I faced an issue with the MongoDB connection. The app was not able to connect to the MongoDB container. I had to run mongodb in a separate container and run the app locally. Below are the steps I followed to run the app locally.

- `docker run --name mongodb -d -p 27017:27017 mongo`
- `docker exec -it mongodb mongosh`
  - `show dbs`
  - `use Mobile`

I found a problem with the connection string in the app. The connection string was hardcoded to `mongodb://localhost:27017/Mobile`. I changed it to `mongodb://mongodb:27017/Mobile` to connect to the MongoDB container.

```js
// Default MongoDB URI if not provided
const defaultMongoDBUri = "mongodb://localhost:27017/Mobile";
const mongodbConnectionString = process.env.MONGODB_CONNECTION_STRING || defaultMongoDBUri;

mongoose.connect(mongodbConnectionString, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log('MongoDB connected successfully'))
.catch(err => console.error('MongoDB connection error:', err));
```

**docker-compose.yml**

```yaml
services:
  ecommerce:
    build: .
    container_name: ecommerce
    ports:
      - "3000:3000"
    depends_on:
      - mongodb
    environment:
      - MONGODB_CONNECTION_STRING=mongodb://mongodb:27017/Mobile
    networks:
      - ecommerce-network

  mongodb:
    image: mongo
    restart: always
    container_name: mongodb
    ports:
      - "27017:27017"
    networks:
      - ecommerce-network

networks:
  ecommerce-network:
    driver: bridge
```

If you don't want to change the connection string in the app, you can use the  **host network** mode to connect to the MongoDB container. Below is the updated `docker-compose.yml` file.

```yaml
services:
  ecommerce:
    build: .
    container_name: ecommerce
    # ports:
    #   - "3000:3000" # Don't need to expose port since we are using network_mode: host
    depends_on:
      - mongodb
    network_mode: host

  mongodb:
    image: mongo
    restart: always
    container_name: mongodb
    ports:
      - "27017:27017"
    networks:
      - ecommerce-network

networks:
  ecommerce-network:
    driver: bridge
```

Now the app is running successfully. I can access the app at `http://localhost:3000`. But I found that the app is not working as expected, I tried different node versions but still, the app is full of bugs. I will leave this app for now.

> The node versions I tried are `14, 13, 12, 10, 9`.

--------------------------------------------