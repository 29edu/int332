# Docker

Docker is an open-source platform that lets you build, ship, and run applications inside containers. It packages your app and everything it needs into a single container so it runs the same way everywhere.

---

## Why We Use Docker

Without Docker, an app might work on your machine but break on someone else's because of different OS versions, missing libraries, or different configurations. Docker solves this by wrapping everything the app needs inside a container. It does not matter what machine you are on, the container runs the same way every time.

---

## Benefits of Using Docker

- **Consistency** — runs the same on any machine, no more "works on my machine" problems.
- **Speed** — containers start in seconds, much faster than virtual machines.
- **Lightweight** — containers share the OS kernel so they use less memory and disk space.
- **Isolation** — each container runs separately, so one app does not affect another.
- **Easy to share** — push your image to Docker Hub and anyone can pull and run it.
- **Scalability** — easy to run multiple containers and scale your app up or down.

---

## Docker Architecture

- **Docker Hub** — the public registry where images are stored and shared.
- **Docker Client** — the CLI tool you use to run Docker commands.
- **Docker Daemon** — the background service that does the actual work of building and running containers.
- **Docker Images** — read-only templates used to create containers.
- **Containers** — running instances of images.
- **Docker Registry** — where images are stored (Docker Hub is the default).

### Docker Lifecycle

- **Docker Stages** — build, ship, run.
- **Container Stages** — created, running, paused, stopped, removed.
- **Network and Ports** — containers communicate through networks and expose ports to the host.
