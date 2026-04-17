# Docker Lifecycle

The Docker lifecycle describes the main stages followed when working with containers and images.

In simple words, Docker usually follows three main steps: build, ship, and run.

---

## Stages of the Docker Lifecycle

**Build**
Create an image for the application.

Example:

```bash
docker build -t myapp .
```

**Ship**
Share the image through a registry like Docker Hub.

Example:

```bash
docker push yourusername/myapp
```

**Run**
Start a container from the image.

Example:

```bash
docker run myapp
```

---

## Container States

A container usually moves through these states:

- `Created`
- `Running`
- `Paused`
- `Stopped`
- `Removed`

---

## Simple Example

If you run:

```bash
docker run -d nginx
```

the container is created first and then moves to the running state.

If you stop it using:

```bash
docker stop <container_id>
```

it moves to the stopped state.

If you delete it using:

```bash
docker rm <container_id>
```

it moves to the removed state.

---

## Why It Is Useful

The Docker lifecycle helps you understand what happens from image creation to container execution and cleanup.
