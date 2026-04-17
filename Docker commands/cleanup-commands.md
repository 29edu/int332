# Cleanup Commands

These commands in the PPT are used to clean up containers, images, networks, and volumes.

---

## Remove One Container

```bash
docker rm <container_id>
docker rm -f mynginx
```

Deletes a container. `-f` force removes a running container.

---

## Remove All Containers

```bash
docker stop $(docker ps -aq)
docker rm $(docker ps -aq)
```

Stops and removes all containers.

---

## Remove Images

```bash
docker rmi ubuntu
docker rmi nginx
docker rmi $(docker images -q)
```

Deletes one image or all images.

---

## System Cleanup

```bash
docker system prune -a
docker system prune -a --volumes
```

- `docker system prune -a` removes unused containers, networks, and images
- `docker system prune -a --volumes` also removes unused volumes
