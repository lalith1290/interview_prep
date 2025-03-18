# Docker Interview Questions and Answers

## 1. Commands of Docker

Key Docker commands include:
- `docker run` - Create and start a container
- `docker pull` - Download an image
- `docker build` - Build an image from a Dockerfile
- `docker ps` - List running containers
- `docker images` - List images
- `docker stop/start` - Stop/start containers
- `docker rm/rmi` - Remove containers/images
- `docker exec` - Execute commands in a running container
- `docker logs` - View container logs
- `docker volume` - Manage volumes
- `docker network` - Manage networks
- `docker-compose` - Manage multi-container applications

## 2. What is Docker Hub?

Docker Hub is Docker's official cloud-based registry service that allows you to store, share, and distribute Docker images. It offers both public and private repositories, automated builds, and integration with GitHub/BitBucket. Think of it as "GitHub for Docker images" where you can push your custom images or pull pre-built images created by others.

## 3. Dockerfile: How will you write it?

A Dockerfile is written using a specific syntax with instructions like:

```
FROM ubuntu:20.04
WORKDIR /app
COPY . /app
RUN apt-get update && apt-get install -y python3
EXPOSE 8080
CMD ["python3", "app.py"]
```

It should follow best practices like:
- Using specific base image tags
- Combining RUN commands with && to reduce layers
- Using .dockerignore to exclude unnecessary files
- Ordering instructions from least to most frequently changing

## 4. Difference between Docker image and container

- **Docker Image**: A read-only template with instructions for creating a Docker container. It's like a snapshot or blueprint containing the application code, runtime, libraries, and dependencies.

- **Docker Container**: A runnable instance of an image. It's the actual running process with its own filesystem, networking, and isolated process space created from an image. Think of images as classes and containers as objects instantiated from those classes.

## 5. Where to use RUN and EXEC commands in Docker?

- **RUN**: Used in Dockerfiles during image building to execute commands and create layers. Example: `RUN apt-get update && apt-get install -y nginx`

- **EXEC** (actually `docker exec`): Used on running containers to execute commands inside the container. Example: `docker exec -it container_name bash`

## 6. Docker volume creation and mounting

Creating a volume:
```
docker volume create my_volume
```

Mounting a volume when starting a container:
```
docker run -v my_volume:/app/data image_name
```

Bind mounting a host directory:
```
docker run -v /host/path:/container/path image_name
```

Using the newer syntax:
```
docker run --mount source=my_volume,target=/app/data image_name
```

## 7. Docker log

View logs from a container:
```
docker logs container_name
```

Common options:
- `--follow` or `-f`: Stream logs
- `--tail n`: Show only the last n lines
- `--since`: Show logs since timestamp
- `--timestamps`: Show timestamps

Example:
```
docker logs --follow --tail 100 container_name
```

## 8. Execute a command in Docker container

```
docker exec -it container_name command
```

For example, to get a bash shell:
```
docker exec -it container_name bash
```

Options:
- `-i`: Keep STDIN open
- `-t`: Allocate a pseudo-TTY
- `-e`: Set environment variables
- `-w`: Working directory

## 9. RUN vs CMD in Dockerfile

- **RUN**: Executes commands during image build time, creating new layers in the image. Used for installing packages, creating files, etc.

- **CMD**: Specifies the default command to run when a container starts. It can be overridden by providing command arguments to `docker run`. A Dockerfile should have only one CMD instruction.

## 10. What is Docker containerization?

Docker containerization is a lightweight virtualization technology that allows you to package an application and its dependencies into a standardized unit (container) for software development and deployment. Containers share the host OS kernel but run in isolated processes with their own filesystem, networking, and resources. This provides consistency across different environments, efficient resource usage, and fast deployment.

## 11. Difference between CMD and ENTRYPOINT in Dockerfile

- **CMD**: Specifies default commands and parameters that can be overridden from the command line when running the container.

- **ENTRYPOINT**: Configures the container to run as an executable. Command-line arguments to `docker run` are appended to the ENTRYPOINT.

Best practice is often to use them together:
```
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```

## 12. What is Docker?

Docker is an open-source platform that automates the deployment, scaling, and management of applications using containerization technology. It allows you to package an application with all its dependencies into a standardized unit (container) that can run consistently on any environment. Docker provides isolation, scalability, and portability while being more lightweight than traditional virtual machines.

## 13. What is a Dockerfile?

A Dockerfile is a text document containing a series of instructions that Docker uses to automatically build an image. It defines everything needed to run an application: the base OS, application code, runtime, dependencies, environment variables, ports, and startup commands. Dockerfiles allow for consistent, reproducible builds and follow a specific syntax using instructions like FROM, RUN, COPY, EXPOSE, CMD, etc.

## 14. Difference between CMD and RUN in Dockerfile

- **RUN**: Executes commands during the image build process and creates new layers in the image. Used for installing software, creating directories, etc.
  Example: `RUN apt-get update && apt-get install -y nginx`

- **CMD**: Specifies the default command to run when a container starts. Does not execute during build time. Can be overridden by providing command-line arguments to `docker run`.
  Example: `CMD ["nginx", "-g", "daemon off;"]`

## 15. Difference between CMD and ENTRYPOINT in Dockerfile

Already answered in question 11.

## 16. What is ARG in Dockerfile?

ARG is a Dockerfile instruction that defines a variable that users can pass at build-time to the builder with the `docker build` command using the `--build-arg` flag. Unlike ENV, ARG values are not available to the container at runtime. ARGs are useful for passing values that may change between builds, like version numbers or repository URLs.

Example:
```
ARG VERSION=1.0
RUN echo "Building version: $VERSION"
```

Build with: `docker build --build-arg VERSION=2.0 -t myapp .`

## 17. How to build an image in Docker?

To build a Docker image:

1. Create a Dockerfile in your project directory
2. Use the `docker build` command:
   ```
   docker build -t image_name:tag .
   ```

Options:
- `-t`: Tag the image
- `--no-cache`: Build without using cache
- `--build-arg`: Pass arguments to the build process
- `-f`: Specify a different Dockerfile location

## 18. Docker scenario: Updating an application in a container without disrupting others

Best approach:
1. Build a new image with the updated application
2. Test the new image
3. Stop the old container
4. Start a new container with the updated image, using the same volume mounts and network settings
5. Verify the new container works properly
6. Remove the old container if no longer needed

This follows the immutable infrastructure principle, where containers are replaced rather than modified in-place.

## 19. Docker networks

Docker networks provide isolated communication channels between containers. Main network types:

1. **Bridge**: Default network. Containers can communicate but are isolated from the host.
2. **Host**: Containers share the host's networking namespace (no isolation).
3. **None**: Containers have no external network connectivity.
4. **Overlay**: For multi-host communication in Docker Swarm.
5. **Macvlan**: Assign MAC addresses to containers, making them appear as physical devices.

Commands:
```
docker network create my_network
docker network ls
docker run --network my_network image_name
```

## 20. Docker compose

Docker Compose is a tool for defining and running multi-container Docker applications using a YAML file. It allows you to:
- Define services, networks, and volumes in a single file
- Start all services with a single command
- Manage the entire application lifecycle

Example docker-compose.yml:
```yaml
version: '3'
services:
  web:
    build: ./web
    ports:
      - "80:80"
  database:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data
volumes:
  db-data:
```

Run with: `docker-compose up`

## 21. How to stop a Docker container?

To stop a running container:
```
docker stop container_name_or_id
```

To stop all running containers:
```
docker stop $(docker ps -q)
```

Options:
- `-t`: Specify timeout before forcing kill (default 10s)

For immediate termination:
```
docker kill container_name_or_id
```

## 22. What is Docker?

Already answered in question 12.

## 23. Why use Docker instead of directly providing code to the client?

Benefits of using Docker over providing raw code:

1. **Environment consistency**: "Works on my machine" problems are eliminated
2. **Dependency management**: All dependencies are packaged with the application
3. **Isolation**: Applications run in isolated environments with controlled resources
4. **Portability**: Containers run consistently across different infrastructures
5. **Scalability**: Easy to scale applications horizontally
6. **Versioning**: Container images can be versioned and rolled back
7. **Faster deployment**: Containers start almost instantly compared to VMs
8. **Efficient resource usage**: Containers share the host OS kernel

## 24. Steps to do containerization

1. **Analyze the application**: Identify components, dependencies, and configuration
2. **Create a Dockerfile**: Define the environment and steps to run the application
3. **Build the image**: Use `docker build` to create an image
4. **Test the container**: Run the image and verify it works as expected
5. **Optimize the image**: Reduce size, improve security, and enhance performance
6. **Set up data persistence**: Configure volumes for persistent data
7. **Configure networking**: Set up appropriate network configurations
8. **Implement container orchestration**: Use Docker Compose or Kubernetes for multi-container applications
9. **Set up monitoring and logging**: Configure logging drivers and monitoring tools
10. **Deploy to production**: Push images to a registry and deploy to target environments

## 25. Docker networking

Already answered in question 19.

## 26. Docker file and its use

Already covered in questions 3 and 13.

## 27. Commands for containers (list, start, stop, remove)

- **List containers**:
  ```
  docker ps       # Running containers
  docker ps -a    # All containers
  ```

- **Start containers**:
  ```
  docker start container_name_or_id
  ```

- **Stop containers**:
  ```
  docker stop container_name_or_id
  ```

- **Remove containers**:
  ```
  docker rm container_name_or_id    # Remove stopped container
  docker rm -f container_name_or_id # Force remove running container
  ```

## 28. How to run a Docker container

```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

Common options:
- `-d`: Run in detached mode (background)
- `-p host_port:container_port`: Port mapping
- `-v host_path:container_path`: Volume mounting
- `-e KEY=VALUE`: Set environment variables
- `--name`: Assign a name to the container
- `--network`: Connect to a network
- `--restart`: Restart policy
- `--rm`: Remove container when it exits

Example:
```
docker run -d --name my_nginx -p 8080:80 -v /local/path:/usr/share/nginx/html nginx
```

## 29. Docker commands for creating containers and volume mapping

Creating a container with volume mapping:
```
docker run -v /host/path:/container/path image_name
```

Named volumes:
```
docker volume create my_volume
docker run -v my_volume:/container/path image_name
```

Newer syntax with --mount:
```
docker run --mount type=bind,source=/host/path,target=/container/path image_name
docker run --mount type=volume,source=my_volume,target=/container/path image_name
```

## 30. Deployment file for Docker

In the context of Kubernetes, a deployment file for Docker would be a YAML manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

For simpler deployments, Docker Compose files (docker-compose.yml) are used.

## 31. Write a Dockerfile for nginx

```dockerfile
# Use official nginx image as base
FROM nginx:1.21-alpine

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Copy static website files
COPY ./html /usr/share/nginx/html

# Create a non-root user to run nginx
RUN adduser -D -H -u 1000 -s /sbin/nologin www-data && \
    chown -R www-data:www-data /usr/share/nginx/html /var/cache/nginx

# Set working directory
WORKDIR /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Switch to non-root user
USER www-data

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

# Keep container running
CMD ["nginx", "-g", "daemon off;"]
```

## 32. What is the logging driver in Docker?

The Docker logging driver is a component that determines how container logs are processed and stored. By default, Docker uses the `json-file` driver, which stores logs as JSON files on the host.

Available logging drivers include:
- `json-file`: Default driver, stores logs as JSON files
- `syslog`: Sends logs to syslog
- `journald`: Sends logs to journald
- `fluentd`: Sends logs to fluentd
- `awslogs`: Sends logs to Amazon CloudWatch
- `splunk`: Sends logs to Splunk
- `gelf`: Sends logs in GELF format to Graylog or Logstash

Configure logging drivers:
```
# System-wide in daemon.json
{
  "log-driver": "syslog"
}

# Per container
docker run --log-driver=syslog image_name
```

## 33. Docker-compose file to build an image

```yaml
version: '3.8'

services:
  web:
    build:
      context: ./web
      dockerfile: Dockerfile
      args:
        - VERSION=1.0
    ports:
      - "80:80"
    environment:
      - NODE_ENV=production
    volumes:
      - app_data:/usr/src/app/data
    networks:
      - app_network
    depends_on:
      - db

  db:
    image: postgres:13
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=secretpassword
      - POSTGRES_DB=myapp
    networks:
      - app_network

volumes:
  app_data:
  db_data:

networks:
  app_network:
    driver: bridge
```

## 34. Docker up, Docker run, Docker start in detail

- **Docker run**:
  - Creates and starts a new container from an image
  - Can specify configuration options (ports, volumes, etc.)
  - Will pull the image if not available locally
  - Example: `docker run -d -p 80:80 nginx`

- **Docker start**:
  - Starts an existing stopped container
  - Preserves container settings and data
  - Cannot modify container configuration
  - Example: `docker start container_name`

- **Docker-compose up**:
  - Starts all services defined in a docker-compose.yml file
  - Creates networks and volumes if needed
  - Builds or pulls images if necessary
  - Can be run with `-d` to detach
  - Example: `docker-compose up -d`

## 35. List Docker commands

Core Docker commands:
- **Container management**: `run`, `start`, `stop`, `restart`, `kill`, `rm`, `ps`, `exec`, `attach`, `logs`
- **Image management**: `pull`, `push`, `build`, `images`, `rmi`, `tag`, `history`
- **Volume management**: `volume create`, `volume ls`, `volume rm`, `volume inspect`
- **Network management**: `network create`, `network ls`, `network rm`, `network inspect`
- **System**: `info`, `version`, `events`, `stats`, `system prune`
- **Compose**: `docker-compose up`, `docker-compose down`, `docker-compose ps`

## 36. What is meant by Dockerfile?

A Dockerfile is a text document containing a series of instructions that Docker uses to automatically build an image. It specifies the base image, dependencies, application code, configuration, and commands needed to create a reproducible environment for running the application. Dockerfiles follow a specific syntax with instructions like FROM, RUN, COPY, CMD, etc., each creating a layer in the resulting image.
