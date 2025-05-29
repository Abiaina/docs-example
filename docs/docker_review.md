# Docker for Beginners: CLI, Compose, and Docker for Desktop Explained

If you've worked with containerized applications, you've probably used Docker without thinking too much about how it works. This guide will help you understand the components you interact with daily and how to use them effectively in your development workflow.

Docker is a container platform that packages your application, dependencies, and configuration into portable, isolated environments called containers. Containers run consistently across development, staging, and production — making them ideal for modern software workflows.

---

## Quick Start: Common Docker Commands

Here are the most common Docker commands you'll use in your daily development:

```bash
# Start a container
docker run -p 8080:80 my-app

# View running containers
docker ps

# View logs
docker logs my-app

# Stop a container
docker stop my-app

# Start all services with Docker Compose
docker-compose up

# Stop all services
docker-compose down
```

---

## What Does "Docker" Actually Mean?

When you run `docker` commands or use Docker in your development environment, you're actually interacting with several components working together:

- **Docker CLI** – The commands you type in your terminal (`docker run`, `docker build`, etc.)
- **Docker Engine** – The background service that makes your containers work
- **Docker Compose** – The tool that helps you run multiple containers together
- **Docker for Desktop** – The application that makes Docker easy to use on your computer

---

## Why Use Containers?

You might be wondering why your team uses containers. Here's why they're valuable:

- Match production on local machines (no more "works on my machine" problems)
- Portability across dev, test, and prod (same environment everywhere)
- Declarative environments (env vars, versions, config)
- Isolated and fast to spin up (no conflicts between projects)
- Supports cloud-native architectures and CI/CD workflows

---

## Docker for Desktop: Easiest Way to Start

**Docker for Desktop** is what most developers use to run Docker on their computers. It's like having a complete Docker environment in a single application. Here's what it gives you:

- A simple way to run Docker commands
- A visual interface to see your containers and images
- Built-in support for running multiple containers together
- Easy setup for common development tasks

**What's happening behind the scenes:**

- Docker runs your containers in a lightweight Linux environment
- Your computer's files can be shared with containers
- Each container gets its own network space
- You can access containers using their service names

**Common issues you might run into:**

- Port conflicts (when something else is using the same port)
- Volume mounting problems (when files aren't shared correctly)
- Memory or CPU limits (when containers need more resources)

---

## Core Docker CLI Concepts

| Term                  | Meaning                                                       |
| --------------------- | ------------------------------------------------------------- |
| **Image**             | Packaged snapshot of an app with its environment              |
| **Container**         | A running instance of an image                                |
| **Dockerfile**        | Instructions for building an image                            |
| **Container Runtime** | Engine that runs and manages containers (e.g., Docker Engine) |

---

## Dockerfile: Building Your Container

A Dockerfile is a text file that contains instructions for building a Docker image. Think of it as a recipe that tells Docker how to create your application's environment.

### Understanding Base Images

A base image is the foundation of your Docker image. It's like choosing an operating system and initial environment for your application. The base image you choose affects:

- The size of your final image
- The available tools and packages
- Security considerations
- Performance characteristics

Here's a table of common base images and when to use them:

| Base Image           | Use Case             | Pros                            | Cons                         | Example Use                        |
| -------------------- | -------------------- | ------------------------------- | ---------------------------- | ---------------------------------- |
| `alpine`             | Minimal applications | Tiny size (~5MB)                | Limited package availability | Static binaries, simple services   |
| `node:18-alpine`     | Node.js applications | Small size, Node.js included    | Limited system tools         | Web applications, APIs             |
| `python:3.11-slim`   | Python applications  | Small size, Python included     | Fewer system packages        | Data processing, web apps          |
| `ubuntu:20.04`       | General purpose      | Full system tools               | Large size (~70MB)           | Complex applications, system tools |
| `maven:3.8-jdk-8`    | Java applications    | Maven + JDK included            | Very large size (~500MB)     | Java applications, builds          |
| `nginx:alpine`       | Web servers          | Small size, Nginx included      | Limited to web serving       | Static websites, reverse proxy     |
| `postgres:14-alpine` | Databases            | Small size, PostgreSQL included | Limited to database          | Database services                  |
| `golang:1.19-alpine` | Go applications      | Small size, Go included         | Limited system tools         | Go services, microservices         |

### Choosing the Right Base Image

When selecting a base image, consider:

1. **Size Requirements**

   - Smaller images deploy faster
   - Less storage needed
   - Better security (smaller attack surface)

2. **Package Availability**

   - Does it have the tools you need?
   - Can you install additional packages?
   - Are the packages up to date?

3. **Security**

   - Is it actively maintained?
   - Are security updates available?
   - Does it follow security best practices?

4. **Performance**
   - Does it meet your application's needs?
   - Are there any known performance issues?
   - Does it support your required features?

### Example Dockerfile with Base Image

```dockerfile
# Use a specific version for stability
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Build the application
RUN npm run build

# Start the application
CMD ["npm", "start"]
```

This example:

- Uses `node:18-alpine` for a small Node.js environment
- Follows best practices for layer caching
- Minimizes the final image size
- Includes only necessary components

### Multi-stage Builds

Multi-stage builds help you create smaller, more secure images by separating the build environment from the runtime environment. This is especially useful for compiled languages or when you need build tools that aren't needed at runtime.

```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Runtime stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/main.js"]
```

Benefits of multi-stage builds:

- Smaller final image size (no build tools included)
- Better security (fewer packages = smaller attack surface)
- Cleaner separation of build and runtime environments
- Faster deployments (smaller images transfer faster)

---

## Port Mapping

Expose your containerized app to your local machine:

```bash
# Build your application
docker build -t my-app .

# Run it with port mapping
docker run -p 8080:80 my-app
```

- `-p 8080:80`: Maps port **8080 on your host** to port **80 inside the container**
- Your app must listen on port `80` in the container
- Access it from `http://localhost:8080`

Remember: The **first** port is your local machine, the **second** is inside the container.

---

## Docker CLI Commands: Build vs Run

Here's the difference between building and running containers:

```bash
# Build: Creates an image from your Dockerfile
docker build -t my-app .

# Run: Starts a container from your image
docker run -p 8080:80 my-app
```

---

## Docker Compose: Multi-Container Setup

When you need to run multiple services together (like a web app with a database), Docker Compose is your friend. It uses a `docker-compose.yml` file to define all your services, volumes, and networks declaratively.

### Benefits

- One config file for everything
- Automatic networking between containers
- Easier port/volume mapping
- Built-in dependency management

### Example

```yaml
version: "3.9"
services:
  app:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      - NODE_ENV=development
      - DB_HOST=db
      - DB_PORT=5432
    env_file:
      - .env
  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

### What This Does

- `app` is exposed on `localhost:8000` and listens on port `8000` in the container
- `db` runs Postgres, listening on internal port `5432`
- Containers communicate via Docker's private network using service names (e.g., `db:5432`)
- You only need to expose ports if you want to access a container from outside Docker
- Environment variables are loaded from both the `environment` section and `.env` file
- The database data is persisted using a named volume
- Health checks ensure the database is ready before the app tries to connect

### Environment Variables Best Practices

1. **Use .env files for sensitive data**

   ```bash
   # .env
   DB_PASSWORD=secret
   API_KEY=your-key
   ```

2. **Use environment section for non-sensitive defaults**

   ```yaml
   environment:
     - NODE_ENV=development
     - LOG_LEVEL=info
   ```

3. **Use env_file for different environments**
   ```yaml
   env_file:
     - .env.development
     - .env.local
   ```

---

## Networking & Volumes

Docker Compose makes networking and storage simple:

- Services can talk to each other using their service names
- Use **named volumes** to keep your data safe (especially for databases)
- Only expose the ports you need to avoid conflicts

---

## Docker Security Best Practices

As a developer working with Docker, here are some practical security considerations to keep in mind:

1. **Container Privileges**

   - Don't run containers as root unless absolutely necessary
   - Use the `--user` flag to run as a non-root user
   - Example: `docker run --user 1000:1000 my-app`

2. **Network Security**

   - Only expose the ports your application needs
   - Use internal networks for services that only need to talk to each other
   - Example: `docker run -p 8080:80 my-app` (only exposes port 8080)

3. **Image Security**

   - Use official images from Docker Hub
   - Keep your base images updated
   - Remove unnecessary packages from your images
   - Example: Use `node:18-alpine` instead of `node:18`

4. **Secrets Management**
   - Never put passwords or API keys in your Dockerfile
   - Use environment variables or Docker secrets
   - Example: `docker run -e DB_PASSWORD=secret my-app`

Remember: These are basic security practices that you can implement in your daily development work. Your team's security experts will handle more advanced security configurations.

---

## Recap

### Key Concepts

- Docker creates reproducible and isolated environments
- Docker Compose is the easiest way to manage multi-service setups
- Map ports carefully: Know what your app and services listen on
- Avoid hardcoded ports or paths that conflict across machines
- Start local → containerize → scale

### Best Practices

- Use multi-stage builds for smaller images
- Implement security scanning
- Use environment variables for configuration
- Follow the principle of least privilege
- Keep base images updated
- Use health checks for service dependencies
- Implement proper logging and monitoring
- Use named volumes for persistent data

### Next Steps

- Learn about Dockerfile patterns
- Explore Docker in CI/CD
- Study container orchestration with Kubernetes
- Implement automated security scanning
- Set up monitoring and logging solutions
- Learn about container networking in depth

### Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Security Best Practices](https://docs.docker.com/engine/security/)
- [Docker Hub](https://hub.docker.com/) for official images

---

## Local Dev Workflow with Docker

Here's a typical flow when using Docker for local development:

1. **Run your app locally first** – Make sure it works outside Docker
2. **Document your environment** – Note OS, env vars, dependencies, versions
3. **Create a Dockerfile** – Use a minimal base image (e.g., `node:18-alpine`, `python:3.11-slim`)
4. **Build and run** – Package your code and define the start command
5. **Add `docker-compose.yml`** – Define services, ports, volumes, and networks
6. **Run with Compose** – Use `docker-compose up` to start everything
7. **Persist data** – Use named volumes for databases
8. **Clean up** – Use `docker-compose down -v` to remove everything

---

## Docker CLI vs. Docker Compose

Here's when to use each tool:

| Feature          | Docker CLI | Docker Compose |
| ---------------- | ---------- | -------------- |
| Single container | Simple     | Overkill       |
| Multi-container  | Manual     | Built-in       |
| Networking       | Manual     | Auto DNS       |
| Startup order    | Manual     | `depends_on`   |
| Volume setup     | Manual     | Declarative    |
| Config reuse     | Limited    | Centralized    |
| Teardown         | Manual     | `down -v`      |
| Prod readiness   | With K8s   | Dev-first      |

---

## Common Docker CLI Commands

Here are the commands you'll use most often:

| Command                       | Description                  |
| ----------------------------- | ---------------------------- |
| `docker ps -a`                | List all containers          |
| `docker volume ls`            | List volumes                 |
| `docker network ls`           | List networks                |
| `docker system prune`         | Clean up unused resources    |
| `docker-compose up`           | Start all services           |
| `docker-compose down`         | Stop and remove all services |
| `docker build --no-cache`     | Rebuild from scratch         |
| `docker exec -it <name> bash` | Shell into container         |
| `docker logs <name>`          | View logs                    |
