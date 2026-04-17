# Docker Daemon

The Docker Daemon is the background service that does all the heavy lifting in Docker. When you run a Docker command, it is actually the daemon doing the work behind the scenes.

---

## What It Does

- Listens for requests from the Docker Client.
- Builds, runs, and manages containers.
- Pulls and pushes images to registries.
- Manages networks and volumes.

---

## How It Works

When you type a command like `docker run ubuntu`, here is what happens:

1. The Docker Client sends that request to the Docker Daemon.
2. The Daemon checks if the image exists locally.
3. If not, it pulls it from Docker Hub.
4. It then creates and starts the container.

The client and daemon talk to each other using a REST API over a Unix socket or network.

---

## Docker Client vs Docker Daemon

- **Docker Client** — the tool you type commands into. It just sends instructions.
- **Docker Daemon** — the one that actually carries out those instructions.

They can run on the same machine or on different machines. When you connect to a remote Docker server, your local client is talking to a daemon running elsewhere.

---

## Checking the Daemon

To check if the daemon is running:

```bash
docker info
```

To start the Docker service:

```bash
sudo systemctl start docker
```

To enable it on startup:

```bash
sudo systemctl enable docker
```
