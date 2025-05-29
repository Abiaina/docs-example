# Docker Disambiguated: The Essential Guide

## What is Docker?

Docker is a platform that packages your application and its dependencies into isolated environments called containers. Think of it as a lightweight, portable virtual machine that runs consistently across different computers.

### Why Use Docker?

- **Consistency**: Works the same way on every computer
- **Isolation**: Applications don't interfere with each other
- **Portability**: Easy to share and run applications
- **Simplicity**: Simplifies development and deployment
- **Scalability**: Easy to run multiple instances

## The Docker Ecosystem

When you use Docker, you're working with several components that work together:

| Component              | Purpose                | Example                      |
| ---------------------- | ---------------------- | ---------------------------- |
| **Docker CLI**         | Command-line interface | `docker run`, `docker build` |
| **Docker Engine**      | Runs containers        | Background service           |
| **Docker Compose**     | Multi-container setup  | `docker-compose up`          |
| **Docker for Desktop** | GUI for Docker         | Visual container management  |

## Core Concepts

| Term           | Meaning                 | Example                        |
| -------------- | ----------------------- | ------------------------------ |
| **Image**      | Packaged application    | `node:18-alpine`               |
| **Container**  | Running instance        | `docker run my-app`            |
| **Dockerfile** | Build instructions      | `FROM node:18-alpine`          |
| **Volume**     | Persistent storage      | `-v /data:/app/data`           |
| **Network**    | Container communication | `docker network create my-net` |

## Essential Docker Commands

```bash
# Start a container
docker run -p 8080:80 my-app

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container
docker stop my-app

# View container logs
docker logs my-app

# Remove a container
docker rm my-app

# Build an image
docker build -t my-app .

# List images
docker images
```

## Understanding Dockerfile

A Dockerfile is a recipe that tells Docker how to build your application's environment. It's like a set of instructions that creates a consistent environment for your app.

### Basic Dockerfile Example

```dockerfile
# Use an official Node.js runtime as the base image
FROM node:18-alpine

# Set the working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Start the application
CMD ["npm", "start"]
```

### Key Dockerfile Instructions

| Instruction | Purpose                | Example                |
| ----------- | ---------------------- | ---------------------- |
| `FROM`      | Base image to use      | `FROM node:18-alpine`  |
| `WORKDIR`   | Set working directory  | `WORKDIR /app`         |
| `COPY`      | Copy files into image  | `COPY . .`             |
| `RUN`       | Execute commands       | `RUN npm install`      |
| `CMD`       | Default command to run | `CMD ["npm", "start"]` |

## Docker Compose: Managing Multiple Containers

Docker Compose helps you run multiple containers together. Create a `docker-compose.yml` file:

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: example
```

Common commands:

```bash
# Start all services
docker-compose up

# Stop all services
docker-compose down

# View logs
docker-compose logs

# Rebuild services
docker-compose build
```

## Versioning and Tagging

Docker images can have multiple versions, called tags. Tags help you manage different versions of your application.

### Common Tagging Patterns

```bash
# Latest version
docker build -t my-app:latest .

# Version number
docker build -t my-app:1.0.0 .

# Git commit hash
docker build -t my-app:abc123 .

# Environment
docker build -t my-app:dev .
docker build -t my-app:prod .
```

### Best Practices for Tagging

1. **Always use specific versions**

   - Avoid using `latest` in production
   - Use semantic versioning (1.0.0, 1.0.1, etc.)
   - Include environment in tag (dev, staging, prod)

2. **Tag multiple versions**
   ```bash
   # Tag for multiple environments
   docker build -t my-app:1.0.0-dev .
   docker build -t my-app:1.0.0-prod .
   ```

## Container Lifecycle

1. **Create**

   ```bash
   docker build -t my-app .
   ```

2. **Run**

   ```bash
   docker run -d my-app
   ```

3. **Monitor**

   ```bash
   docker ps
   docker logs my-app
   ```

4. **Update**

   ```bash
   docker build -t my-app:new-version .
   docker stop old-container
   docker run -d my-app:new-version
   ```

5. **Cleanup**
   ```bash
   docker stop my-app
   docker rm my-app
   docker rmi my-app
   ```

## Best Practices

1. **Use Official Images**

   - Start with official images from Docker Hub
   - Example: `node:18-alpine` instead of custom Node.js setup

2. **Keep Images Small**

   - Use minimal base images (like `alpine`)
   - Remove unnecessary files
   - Combine commands to reduce layers

3. **Security**

   - Don't run as root
   - Use specific version tags
   - Keep images updated

4. **Environment Variables**
   ```bash
   docker run -e DB_HOST=localhost my-app
   ```

## Quick Reference

### Common Issues

- Port conflicts: Change the host port
- Permission issues: Check file ownership
- Container not starting: Check logs with `docker logs`

### Useful Flags and Commands

#### Container Management

```bash
# Run a container
docker run -d -p 8080:80 --name my-app my-image
docker run -it --rm my-image bash  # Interactive mode

# Execute commands in a running container
docker exec -it my-app bash        # Get a shell
docker exec my-app ls /app         # Run a single command

# View container logs
docker logs my-app                 # View all logs
docker logs -f my-app              # Follow logs in real-time
docker logs --tail 100 my-app      # View last 100 lines
```

#### Common Flags

- `-d`: Run in background (detached mode)
- `-p`: Map ports (host:container)
- `-v`: Mount volumes (host:container)
- `-e`: Set environment variables
- `--name`: Assign a name to the container
- `-it`: Interactive mode with terminal
- `--rm`: Remove container when it stops
- `--restart`: Set restart policy (always, unless-stopped)

#### Examples

```bash
# Run with volume and environment
docker run -d \
  -v $(pwd):/app \
  -e DB_HOST=localhost \
  -p 3000:3000 \
  --name my-app \
  my-image

# Run with restart policy
docker run -d \
  --restart unless-stopped \
  --name my-app \
  my-image

# Execute with specific user
docker exec -u 1000 my-app bash
```

### Development Workflow

1. Write your application
2. Create a Dockerfile
3. Build the image
4. Run the container
5. Test your application

---

Remember: Docker is a tool to make your development and deployment process more consistent and reliable. Start simple and add complexity as needed.
