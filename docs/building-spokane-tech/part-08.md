# **Building Spokane Tech: Part 8**

Welcome to part 8 of the "Building Spokane Tech" series! In this article, we'll discuss adding Docker and Docker Compose for running components of our service in containers.

Containerization has become an essential tool for modern web development, and Docker is at the forefront of this revolution. When developing a Django-based web application like ours, using Docker ensures consistency across development and deployed environments. By leveraging Docker Compose, we can efficiently manage multiple services required by our application.


## **Docker Compose**
Docker Compose is a tool that allows you to define and manage multi-container Docker applications using a simple YAML file (docker-compose.yaml). It enables developers to run interconnected services, such as a web application, database, and message broker, with a single command. The  Docker Compose basic concepts include:

***Key Docker Compose Configuration Options***

- **version:** Defines the Compose file format version. In our case, we use "3.9", which is one of the latest stable versions.

- **services:** Lists all the containers that make up the application. Each service runs in its own container.

***Service Configuration Keys***

- **image:** Specifies the Docker image to use for the container. If the image is not found locally, Docker will pull it from a registry like Docker Hub.

- **build:** Defines how to build the image from a Dockerfile. It usually includes:
    - context: The directory containing the Dockerfile.
    - dockerfile: The path to the specific Dockerfile used to build the image.

- **container_name:** Gives a custom name to the container instead of a randomly generated one.

- **command:** Overrides the default command specified in the Dockerfile, allowing you to run specific commands when the container starts.

- **env_file:** Loads environment variables from an external .env file.

- **ports:** Maps ports between the container and the host.

- **depends_on:** Specifies service dependencies. A container will not start until its dependencies are up and running.

***Volumes***

Volumes store persistent data outside the container filesystem, ensuring data is not lost when containers are restarted or removed.


## **Our Services**
Let's review the components in our system, each of these will be a service in our docker-compose.yaml file.

- Django (Web Application) – The core application running on Gunicorn or the Django development server
- PostgreSQL (Database) – Stores application data
- Redis (Message Broker) – Used by Celery for task queuing
- Celery Worker – Executes asynchronous tasks
- Celery Beat – Handles scheduled tasks
- Celery Flower – Provides a web UI for monitoring Celery tasks


## **Our docker-compose.yaml file**

```
version: '3.9'

services:
  django:
    image: spokanetech-django:latest
    container_name: django
    env_file:
      - .env.compose
    build:
      context: ../..
      dockerfile: src/docker/Dockerfile
    command: ./entrypoint.sh
    ports:
      - "8080:8000"
    depends_on:
      - db
      - redis

  db:
    image: postgres:17
    container_name: postgres_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    env_file:
      - .env.compose

  redis:
    image: redis:7.2-alpine
    container_name: redis
    restart: unless-stopped
    ports:
      - "6379:6379"

  worker:
    image: spokanetech-django:latest
    container_name: worker
    env_file:
      - .env.compose
    build:
      context: ../..
      dockerfile: src/docker/Dockerfile
    command: celery -A core worker -l info
    depends_on:
      - redis
      - db

  beat:
    image: spokanetech-django:latest
    container_name: beat
    env_file:
      - .env.compose
    build:
      context: ../..
      dockerfile: src/docker/Dockerfile
    command: celery -A core beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
    depends_on:
      - redis
      - db

  flower:
    image: spokanetech-django:latest
    container_name: flower
    env_file:
      - .env.compose
    command: ["celery", "-A", "core", "--config=flowerconfig.py", "flower"]
    ports:
      - "5555:5555"
    depends_on:
      - redis
      - db

volumes:
  postgres_data:
  static_volume:
```


## **Running the Application**
Docker Compose provides several commands to manage services. Here are the basics:

***Building containers***

To build the containers run: ```docker-compose build```

This builds images for the services defined in docker-compose.yaml using the specified Dockerfile. If an image already exists, it will only rebuild if changes are detected.

***Starting containers***

To start the containers run: ```docker-compose up```

This starts all services defined in docker-compose.yaml. It also automatically builds missing images if they are not found.

To run the containers in detached mode use: ```docker-compose up -d```

This runs containers in the background and allows applications to run persistently.

***Stopping containers***

To stop the containers use: ```docker-compose down```

This stops and removes all containers, networks, and volumes (if specified); it does not remove built images.


***Rebuild and restart containers***

To build the container when running, use: ```docker-compose up --build```

This rebuilds images before starting containers and ensures the latest changes in the Dockerfile are applied.


## *Accessing Services*

All of our components are available on localhost on various their applicable ports: 

- Django App: http://localhost:8080
- Celery Flower UI: http://localhost:5555
- PostgreSQL: Connect via localhost:5432
- Redis: Available on localhost:6379
