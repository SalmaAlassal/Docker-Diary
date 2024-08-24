# Dockerize some apps on Github

I will pick some random apps from Github and Dockerize them. This will help me gain more experience with Docker and also learn how to Dockerize different types of applications.


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