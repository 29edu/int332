# Image Registry

An image registry is a storage place where container images are saved, managed, and shared.

It helps teams build an image once and use the same image in development, testing, and production.

---

## Why It Is Important

Without a registry:

- every system may build the image separately
- different environments may have different versions
- deployment becomes slower and inconsistent

With a registry:

- images are stored in one place
- teams can pull the same version everywhere
- deployment becomes faster and more reliable

---

## Types of Image Registry

**Public Registry**
Anyone can pull images from it.
Example: Docker Hub

**Private Registry**
Only authorized users can access it.
Examples: AWS ECR, Azure Container Registry, GitHub Container Registry

---

## Image Naming Format

Images usually follow this format:

```text
<registry>/<namespace>/<image>:<tag>
```

Example:

```text
docker.io/manpreet/webapp:v1
```

Here:

- `docker.io` is the registry
- `manpreet` is the namespace
- `webapp` is the image name
- `v1` is the tag

---

## Simple Example

```bash
docker pull ubuntu
```

This command pulls the `ubuntu` image from a registry, usually Docker Hub.

---

## In CI/CD

In DevOps pipelines, the registry acts like a bridge:

- CI builds the image
- CI pushes it to the registry
- CD pulls the image from the registry
- the application is deployed
