# Basic Docker Commands

These are the basic Docker commands used in the PPT for getting started quickly.

---

## Pull an Image

```bash
docker pull ubuntu
docker pull nginx
docker pull httpd
```

Downloads an image from Docker Hub to your local machine.

---

## List Downloaded Images

```bash
docker images
```

Shows all images available locally.

---

## Run a Container

```bash
docker run ubuntu
docker run -it ubuntu /bin/bash
docker run -it ubuntu bash
docker run -d -p 8080:80 --name mynginx nginx
docker run -d -p 8081:80 --name apache-web httpd
docker run -dit --name myubuntu ubuntu
```

Common flags:

- `-i` keeps standard input open
- `-t` gives an interactive terminal
- `-d` runs the container in detached mode
- `-p host:container` maps ports
- `--name` assigns a custom container name

---

## List Running Containers

```bash
docker ps
```

Shows currently running containers.

---

## Exit a Container Shell

```bash
exit
```

Used after entering an interactive shell inside a container.
