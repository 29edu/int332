# Docker Architecture

Docker architecture is the way Docker components work together to build, store, and run containers.

---

## Main Components

**Docker Client**
This is the command-line tool the user works with.
Example: `docker run ubuntu`

**Docker Daemon**
This is the background service that does the actual work like building images, creating containers, and managing networks and volumes.

**Docker Images**
These are read-only templates used to create containers.
Example: `ubuntu`, `nginx`

**Containers**
These are running instances of Docker images.
Example: a running Nginx server container

**Docker Registry**
This is where images are stored.
Example: Docker Hub

---

## How It Works

1. You type a command in the Docker Client.
2. The Docker Client sends the request to the Docker Daemon.
3. The Docker Daemon checks for the image locally.
4. If the image is missing, it pulls it from the registry.
5. The Daemon creates and starts the container.

---

## Simple Example

```bash
docker run nginx
```

In this case:

- the Docker Client sends the command
- the Docker Daemon handles it
- the `nginx` image is used
- a container is created and started

---

## Why It Matters

Understanding Docker architecture helps you know:

- who receives the command
- who runs the container
- where images come from
- how Docker components connect with each other
