# Build vs Image Fields

When defining a service in Docker Compose, you specify where the container image comes from using either `image`, `build`, or both.

---

## Using `image`

The `image` field tells Docker Compose to pull a pre-built image from Docker Hub or another registry.

```yaml
services:
  db:
    image: postgres:15

  cache:
    image: redis:7-alpine

  web:
    image: nginx:latest
```

**When to use `image`:**
- you are using a standard, official image without modifications
- the image already exists on Docker Hub or your private registry
- you do not need to change the image before running it

**Image name formats:**

```yaml
image: postgres          # latest tag from Docker Hub
image: postgres:15       # specific version
image: postgres:15-alpine  # specific variant
image: myregistry.io/myapp:v2.1  # private registry
```

---

## Using `build`

The `build` field tells Docker Compose to build an image from a local Dockerfile before starting the container.

**Simple form — Dockerfile in the current directory:**

```yaml
services:
  app:
    build: .
```

**Specifying context and Dockerfile path:**

```yaml
services:
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
```

- `context` is the directory Docker uses as the build context (files available to `COPY` in the Dockerfile)
- `dockerfile` is the path to the Dockerfile, relative to the context

**Using a different Dockerfile for development:**

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
```

**Passing build arguments:**

```yaml
services:
  app:
    build:
      context: .
      args:
        NODE_VERSION: 18
        APP_ENV: production
```

The Dockerfile can receive these with `ARG`:

```dockerfile
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}
```

---

## Using Both `build` and `image`

When both are specified, Compose builds the image and tags it with the name given in `image`.

```yaml
services:
  app:
    build: ./app
    image: myapp:latest
```

This is useful when you want to:
- build the image locally and also push it to a registry
- tag the built image with a specific version
- cache the built image under a known name

To push after building:

```bash
docker compose build
docker compose push
```

---

## Forcing a Rebuild

By default, `docker compose up` does not rebuild if the image already exists. To force a rebuild:

```bash
docker compose up --build
```

To only build without starting:

```bash
docker compose build
docker compose build app     # build only the app service
```

---

## Summary

| Scenario | Use |
|---|---|
| Using an official image (nginx, postgres, redis) | `image` |
| Running your own application code | `build` |
| Building and tagging with a specific name | both `build` and `image` |
| Different Dockerfiles for dev and prod | `build` with `dockerfile` field |
| Passing values into the Dockerfile | `build` with `args` |
