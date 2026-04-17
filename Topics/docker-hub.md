# Docker Hub

Docker Hub is the default public registry for Docker images. It is a central place where images are stored and shared.

When people use Docker, Docker Hub is usually the first place they pull images from.

---

## What Docker Hub Provides

- public repositories
- private repositories
- official images
- image sharing across teams

---

## Official Images

Docker Hub provides official images that are maintained and trusted by Docker or software vendors.

Examples:

- `ubuntu`
- `nginx`
- `python`
- `httpd`

---

## Simple Examples

Pull Ubuntu from Docker Hub:

```bash
docker pull ubuntu
```

Pull Nginx from Docker Hub:

```bash
docker pull nginx
```

Run Nginx after pulling:

```bash
docker run -d -p 8080:80 --name mynginx nginx
```

---

## Why It Is Useful

Docker Hub is useful because:

- images are easy to find
- teams can reuse common images
- sharing becomes simple
- deployment becomes faster

---

## Real-Life Example

If a developer wants to test a web server quickly, they can pull the `nginx` image from Docker Hub and run it in a few seconds.
