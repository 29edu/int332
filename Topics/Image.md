# Docker Image

A Docker image is a read-only template used to create containers. Think of it like a recipe, it has everything your app needs to run: the OS, dependencies, and your code. When you run an image, it becomes a container.

---

## Image Layers

A Docker image is not just one big file. It is built up in layers, where each layer represents one instruction from the Dockerfile.

For example, this Dockerfile:

```dockerfile
FROM ubuntu
RUN apt-get update
COPY . /app
CMD ["echo", "Hello"]
```

Creates these layers:

- **Layer 1** — the base Ubuntu OS (from `FROM ubuntu`)
- **Layer 2** — the result of running `apt-get update`
- **Layer 3** — your app files copied in (from `COPY . /app`)
- **Layer 4** — the command to run (from `CMD`)

Each layer only stores what changed from the layer before it. So layers are small and efficient.

**Layers are cached.** If you rebuild the image and nothing changed in a layer, Docker reuses the cached version and skips rebuilding it. This makes builds much faster.

**Layers are shared.** If two images use the same base layer like `ubuntu`, they share it on disk instead of storing two copies. This saves space.

When you run a container, Docker adds one extra writable layer on top of all the read-only image layers. Everything you do inside the container goes into that writable layer. When the container is deleted, that layer is gone.

---

## Image Registry

An image registry is a place where Docker images are stored and shared. You can push your images there and pull them from anywhere.

- **Docker Hub** — the default public registry.
- **Private registries** — like AWS ECR, GitHub Container Registry, or your own self-hosted one.

When you do `docker pull ubuntu`, Docker fetches that image from Docker Hub's registry.

---

## Create an Image

Make a file called `Dockerfile` in your project folder:

```dockerfile
FROM ubuntu
RUN apt-get update
COPY . /app
CMD ["echo", "Hello from container"]
```

---

## Build the Image

```bash
docker build -t your-image-name .
```

The `-t` flag gives your image a name. The `.` means use the current folder.

---

## Pull an Image

```bash
docker pull ubuntu
```

Replace `ubuntu` with whatever image you need.

---

## Push to Docker Hub

Login first:

```bash
docker login
```

Tag your image with your Docker Hub username:

```bash
docker tag your-image-name yourusername/your-image-name
```

Push it:

```bash
docker push yourusername/your-image-name
```
