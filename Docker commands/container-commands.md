# Container Commands

These commands in the PPT are used to create, start, stop, inspect, and manage containers.

---

## Create and Run Containers

```bash
docker run ubuntu
docker run -it ubuntu /bin/bash
docker run -it ubuntu bash
docker run -dit --name myubuntu ubuntu
docker run -d -p 8080:80 --name mynginx nginx
docker run -d -p 8081:80 --name apache-web httpd
```

---

## List Containers

```bash
docker ps
docker ps -aq
```

- `docker ps` shows running containers
- `docker ps -aq` shows all container IDs

---

## Stop and Start Containers

```bash
docker stop <container_id>
docker stop mynginx
docker stop $(docker ps -aq)
docker start mynginx
docker start ubuntu
```

---

## Remove Containers

```bash
docker rm <container_id>
docker rm -f mynginx
docker rm $(docker ps -aq)
```

---

## Execute Commands Inside a Running Container

```bash
docker exec -it <container_id> bash
```

Opens a shell inside a running container.

---

## View Logs

```bash
docker logs <container_id>
```

Shows the output logs of a container.

---

## Inspect Container Details

```bash
docker inspect <container_id>
```

Displays detailed container information in JSON format.

---

## Monitor Resource Usage

```bash
docker stats
```

Shows live CPU and memory usage of running containers.

---

## View Processes Running in a Container

```bash
docker top <container_id>
```

Lists processes running inside the container.
