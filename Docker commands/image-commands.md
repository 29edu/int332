# Image Commands

These commands in the PPT are related to managing Docker images.

---

## Pull Images

```bash
docker pull ubuntu
docker pull nginx
docker pull httpd
```

Downloads images from Docker Hub.

---

## View Local Images

```bash
docker images
```

Lists all images stored locally.

---

## Remove an Image

```bash
docker rmi ubuntu
docker rmi nginx
docker rmi $(docker images -q)
```

Deletes one image or all local images.

Note: `$(docker images -q)` returns only image IDs.
