---
title: "My DevOps with Docker Journey: A Learning Log"
date: "2023-07-01"
author: "Kyle Mapue"
categories:
  - "oc"
  - "cloud-internet"
  - "linux"
---

*Study material from [devopswithdocker.com](https://devopswithdocker.com/).*

Embarking on a journey through the world of DevOps, my focus has been on mastering Docker and its ecosystem. This blog post serves as a quick recap of the key learnings and concepts explored across various exercises and chapters in my study.

## Chapter 1: Docker Fundamentals - Building Blocks of Containerization

Chapter 1 laid the groundwork, focusing on the absolute essentials of Docker. This is where we learned to craft `Dockerfile`s – the blueprints for our containerized applications.

### Dockerfile Basics: Your First Containerized "Hello World"

Let's start with a simple example from `exercise1.10`. This `Dockerfile` creates an image that simply prints "Hello from Docker".

```dockerfile
# chap1/exercise1.10/Dockerfile
FROM alpine
CMD echo "Hello from Docker"
```

*   `FROM alpine`: This instruction specifies the base image for our container. `alpine` is a lightweight Linux distribution, perfect for minimal images.
*   `CMD echo "Hello from Docker"`: This defines the command that will be executed when a container is launched from this image.

To build this image, navigate to the `exercise1.10` directory and run:

```bash
docker build -t my-hello-world .
```

Once built, you can run your first container:

```bash
docker run my-hello-world
```

You should see "Hello from Docker" printed to your console!

### Executing Scripts within Containers: The `improvedcurler` Example

Many real-world applications involve running scripts. The `improvedcurler` exercise demonstrated how to include and execute a shell script inside a Docker container.

First, let's look at the `script.sh` file:

```bash
# chap1/improvedcurler/script.sh
#!/bin/bash

echo "Searching..";
sleep 1;
curl http://$1;
```

And here's its corresponding `Dockerfile`:

```dockerfile
# chap1/improvedcurler/Dockerfile
FROM alpine
COPY script.sh /script.sh
RUN chmod +x /script.sh
ENTRYPOINT ["/script.sh"]
```

*   `COPY script.sh /script.sh`: This instruction copies our `script.sh` file from the host machine into the container's root directory.
*   `RUN chmod +x /script.sh`: We need to make the script executable within the container.
*   `ENTRYPOINT ["/script.sh"]`: This sets the primary command for the container. Any arguments passed to `docker run` will be appended to this entrypoint. For example, `docker run improved-curler example.com` would execute `/script.sh example.com`.

To build and run this example:

```bash
docker build -t improved-curler chap1/improvedcurler
docker run improved-curler example.com
```

This will attempt to `curl` `example.com` from within the container, showcasing how scripts can be integrated and executed.

### Introduction to Multi-Service Applications

While Chapter 1 focused on single-container applications, the `Exercises1.11-1.14` directory, with its `example-backend`, `example-frontend`, and `spring-example-project`, hinted at the complexity of real-world applications. These exercises laid the groundwork for understanding the need to containerize multiple interconnected services, a concept we'll dive deeper into with Docker Compose.

## Chapter 2: Docker Compose - Orchestrating Multi-Container Applications

Chapter 2 was a deep dive into `docker-compose`, the tool for defining and running multi-container Docker applications. This chapter was crucial for understanding how to manage complex service architectures efficiently.

### Defining Services with `docker-compose.yml`: A Simple Web Service

Almost every exercise in this chapter revolved around creating and configuring `docker-compose.yml` files. Let's look at a basic example from `ex2.2` that defines a simple web service:

```yaml
# chap2/ex2.2/docker-compose.yml
version: '3.8'

services:
  message-server:
    image: devopsdockeruh/simple-web-service:alpine
    command: server
    ports:
      - 127.0.0.1:8080:8080
```

*   `version: '3.8'`: Specifies the Docker Compose file format version.
*   `services`: Defines the different services that make up your application. Here, we have `message-server`.
*   `image`: The Docker image to use for this service.
*   `command`: Overrides the default command specified in the image's Dockerfile.
*   `ports`: Maps a port on your host machine (`127.0.0.1:8080`) to a port inside the container (`8080`). This allows you to access the service from your host.

To run this service, navigate to the `ex2.2` directory and execute:

```bash
docker compose up
```

### Service Intercommunication, Networking, and Volumes: A Full-Stack Example

Real-world applications often consist of multiple interconnected services, such as a frontend, backend, and database. `ex2.7` provides a great example of how Docker Compose facilitates this intercommunication:

```yaml
# chap2/ex2.7/docker-compose.yml
version: '3.8'

services:
  frontend:
    image: frontend:latest
    ports:
      - 127.0.0.1:5000:5000

  backend:
    image: backend:latest
    ports:
      - 127.0.0.1:8080:8080
    environment:
      - REDIS_HOST=redis  # Use the Redis service name
      - POSTGRES_HOST=db
    depends_on:
      - redis  # Ensure Redis starts before the backend
      - db

  redis:
    image: redis:8.0-M02-alpine
    restart: unless-stopped

  db:
    image: postgres:13.2-alpine
    restart: unless-stopped
    environment:
      POSTGRES_PASSWORD: postgres
    container_name: db_postgres
    volumes:
      - ./database:/var/lib/postgresql/data
```

Key concepts demonstrated here:

*   **Service Discovery**: Services within the same Docker Compose network can communicate with each other using their service names (e.g., `backend` can reach `redis` and `db` by their names).
*   `environment`: Sets environment variables within the container, crucial for configuring applications (e.g., database connection strings).
*   `depends_on`: Ensures that services start in a specific order. Here, `backend` will only start after `redis` and `db` are running.
*   `volumes`: Mounts a host path (`./database`) into the container (`/var/lib/postgresql/data`). This is essential for persistent data storage, ensuring your database data isn't lost when containers are removed.
*   `restart: unless-stopped`: Configures the container to restart automatically unless explicitly stopped.

To bring up this multi-service application:

```bash
docker compose up
```

### Scaling and Load Balancing

The `scaling-exercise` and `whoamiscale` directories were key to understanding how to scale services horizontally and distribute traffic among them, a core DevOps practice. While `docker-compose` itself has basic scaling capabilities (`docker compose up --scale service_name=N`), more advanced load balancing and scaling often involve external tools or orchestrators like Kubernetes, which build upon these fundamental concepts.

This chapter solidified the understanding of how to define, link, and manage complex multi-container applications, laying the groundwork for robust DevOps practices.

## Chapter 3: Advanced Docker & Deployment Patterns

Chapter 3 moved beyond basic orchestration into more advanced topics, touching upon build processes, CI/CD considerations, and more complex deployment scenarios.

### Automated Builds: The `builder.sh` Script

Automating the Docker image build process is a crucial step towards Continuous Integration (CI). The `builder.sh` script in `ex3.3` provides a great example of how to automate building and pushing Docker images to a registry like Docker Hub.

```bash
# chap3/ex3.3/builder.sh
#!/bin/bash

# Check if the correct number of arguments is passed
if [ "$#" -ne 2 ]; then
  echo "Usage: $0 <github-repo> <dockerhub-repo>"
  echo "Example: $0 mluukkai/express_app mluukkai/testing"
  exit 1
fi

# Parse arguments
GITHUB_REPO=$1
DOCKERHUB_REPO=$2

# Extract the repo name from the GitHub URL
REPO_NAME=$(basename "$GITHUB_REPO")

# Step 1: Clone the GitHub repository
echo "Cloning repository https://github.com/$GITHUB_REPO..."
git clone "https://github.com/$GITHUB_REPO.git" || { echo "Failed to clone the repository"; exit 1; }

# Step 2: Change into the cloned repository directory
cd "$REPO_NAME" || { echo "Failed to access the repository directory"; exit 1; }

# Step 3: Build the Docker image
echo "Building Docker image $DOCKERHUB_REPO..."
docker build -t "$DOCKERHUB_REPO" . || { echo "Failed to build the Docker image"; exit 1; }

# Step 4: Push the Docker image to Docker Hub
echo "Logging into Docker Hub..."
docker login || { echo "Docker login failed"; exit 1; }

echo "Pushing Docker image to Docker Hub..."
docker push "$DOCKERHUB_REPO" || { echo "Failed to push the Docker image"; exit 1; }

# Step 5: Cleanup (optional)
cd ..
echo "Cleaning up..."
rm -rf "$REPO_NAME"

echo "Done! The Docker image is available at Docker Hub: $DOCKERHUB_REPO"
```

This script automates the entire process: cloning a Git repository, building a Docker image from its contents, logging into Docker Hub, pushing the image, and finally cleaning up. This is a fundamental pattern in CI/CD pipelines.

### Scripted Container Execution: Running Tasks within Docker

`ex3.4` demonstrates how to create a Docker image that executes a specific script upon startup. This is useful for running automated tasks, cron jobs, or one-off operations within a controlled environment.

Here's the `Dockerfile` from `ex3.4`:

```dockerfile
# chap3/ex3.4/Dockerfile
FROM docker:latest
RUN apk add --no-cache bash
WORKDIR /app
COPY script.sh /app/script.sh
RUN chmod +x /app/script.sh
ENTRYPOINT ["/app/script.sh"]
```

And the `script.sh` it uses:

```bash
# chap3/ex3.4/script.sh
#!/bin/bash

# Check if the correct number of arguments is passed
if [ "$#" -ne 2 ]; then
  echo "Usage: $0 <github-repo> <dockerhub-repo>"
  echo "Example: $0 mluukkai/express_app mluukkai/testing"
  exit 1
fi

# Parse arguments
GITHUB_REPO=$1
DOCKERHUB_REPO=$2

# Extract the repo name from the GitHub URL
REPO_NAME=$(basename "$GITHUB_REPO")

# Step 1: Clone the GitHub repository
echo "Cloning repository https://github.com/$GITHUB_REPO..."
git clone "https://github.com/$GITHUB_REPO.git" || { echo "Failed to clone the repository"; exit 1; }

# Step 2: Change into the cloned repository directory
cd "$REPO_NAME" || { echo "Failed to access the repository directory"; exit 1; }

# Step 3: Build the Docker image
echo "Building Docker image $DOCKERHUB_REPO..."
docker build -t "$DOCKERHUB_REPO" . || { echo "Failed to build the Docker image"; exit 1; }

# Step 4: Push the Docker image to Docker Hub
echo "Logging into Docker Hub..."
docker login -u "$DOCKER_USER" -p "$DOCKER_PWD" || { echo "Docker login failed"; exit 1; }

echo "Pushing Docker image to Docker Hub..."
docker push "$DOCKERHUB_REPO" || { echo "Failed to push the Docker image"; exit 1; }

# Step 5: Cleanup (optional)
cd ..
echo "Cleaning up..."
rm -rf "$REPO_NAME"

echo "Done! The Docker image is available at Docker Hub: $DOCKERHUB_REPO"
```

This setup allows you to encapsulate a specific task within a Docker image, making it portable and reproducible. You can build this image and run it, passing arguments directly to the `script.sh`.

### Complex Multi-Service Deployments

Exercises like `ex3.5` through `ex3.8-3.9` continued to build on multi-service applications (`example-backend`, `example-frontend`) integrated with `docker-compose.yml` and `nginx.conf`. These likely involved more sophisticated configurations for production-like environments, including environment variables, health checks, and advanced networking. These examples showcase how to combine the concepts from previous chapters to create robust and scalable multi-service architectures.

This journey has provided a solid foundation in containerization with Docker, from single-service deployments to complex, scalable, multi-container applications, laying the groundwork for robust DevOps practices.