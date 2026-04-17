# Docker

Docker is an open-source platform that lets you build, ship, and run applications inside containers. It packages the application and its dependencies together so it runs the same way on every machine.

---

## Why We Use Docker

Without Docker, an application may work on one system and fail on another because of missing libraries, different settings, or a different environment.

Docker solves this by packaging everything needed to run the app inside a container.

---

## Benefits of Docker

- consistent environment on every machine
- fast startup
- lightweight compared to virtual machines
- easy to share and deploy
- good isolation between applications

---

## Main Docker Components

**Docker Client**
The tool where users type Docker commands.

**Docker Daemon**
The background service that builds images and runs containers.

**Docker Images**
Read-only templates used to create containers.

**Containers**
Running instances of Docker images.

**Docker Registry**
A place where Docker images are stored.

**Docker Hub**
The default public registry used by Docker.

---

## Docker Lifecycle

Docker mainly follows three stages:

- build
- ship
- run

Container states usually move like this:

- created
- running
- paused
- stopped
- removed

---

## Simple Example

```bash
docker pull nginx
docker run -d -p 8080:80 --name mynginx nginx
```

This pulls the Nginx image and starts a web server container.

You can access it in a browser at:

```text
http://localhost:8080
```
