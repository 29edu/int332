# Container

A container is a lightweight, standalone package that includes everything needed to run a piece of software: the code, runtime, libraries, and settings. Containers run in isolation from each other but share the same OS kernel. They start fast, use fewer resources, and work the same on any machine.

---

## Containerization

Containerization is the process of packaging an application and all its dependencies together into a container so it can run consistently anywhere. Instead of saying "it works on my machine," containerization makes sure it works on every machine, whether that is your laptop, a test server, or production.

---

## Types of Containers

**OS Containers**
These run a full OS environment inside a container and can run multiple processes.
Example: LXC (Linux Containers).

**Application Containers**
These are built to run a single application or service. This is the most common type.
Example: Docker containers.

**Serverless Containers**
Containers that run without you managing any server. You just deploy the container and it runs.
Example: AWS Fargate, Google Cloud Run.

---

## Container Image

A container image is a read-only file that contains everything needed to run a container. When you run an image, it becomes a container.

## Image Registry

A place where container images are stored and shared. You push your image there and pull it on any machine.
Example: Docker Hub.

## Private Image Registry

A registry that is not public. Only your team or organization can access it.
Example: AWS ECR, GitHub Container Registry, Harbor.

## Image Naming and Tags

Images are named like this: `username/image-name`
Tags are used to version images. Example: `ubuntu:20.04` where `20.04` is the tag. If no tag is given, Docker uses `latest` by default.

---

## Container Runtime

A container runtime is the software that actually runs containers. It sits between the OS and the container, handling the low-level work of starting, stopping, and managing containers.

When you run `docker run`, Docker does not do all the work itself. It hands off to a container runtime underneath.

### Types of Container Runtime

**Low-level Runtimes**
These directly interact with the OS kernel to create and run containers using namespaces and cgroups.

- **runc** — the most common low-level runtime. Used by Docker under the hood.
- **crun** — a faster, lighter alternative to runc written in C.

**High-level Runtimes**
These sit on top of low-level runtimes and add features like image management, networking, and an API.

- **containerd** — used by Docker and Kubernetes. Manages the full container lifecycle.
- **CRI-O** — a lightweight runtime built specifically for Kubernetes.

**Full Container Engines**
These are the tools you actually interact with as a user. They wrap everything above.

- **Docker** — the most widely used container engine.
- **Podman** — a daemonless alternative to Docker, runs containers without a background service.

---

## Examples in Real Life

- **Docker** — the most popular tool for creating and running containers.
- **Kubernetes** — manages and scales containers across multiple machines.
- **AWS ECS and Fargate** — run containers in the cloud without managing servers.
