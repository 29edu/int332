# Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. Instead of running separate `docker run` commands for each container, you describe your entire application stack in a single YAML file and manage it all with a few commands.

---

## The Problem Without Docker Compose

Running a three-service application manually requires three separate commands, each with many flags:

```bash
docker run -d --name db -e POSTGRES_PASSWORD=pass -v pgdata:/var/lib/postgresql/data postgres

docker run -d --name backend --link db -e DB_HOST=db -p 5000:5000 mybackend

docker run -d --name frontend --link backend -p 3000:80 mynginx
```

This is error-prone, hard to share with teammates, and impossible to version control cleanly.

---

## With Docker Compose

The same setup in `docker-compose.yml`:

```yaml
version: "3.9"

services:
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: pass
    volumes:
      - pgdata:/var/lib/postgresql/data

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    depends_on:
      - db

  frontend:
    image: nginx
    ports:
      - "3000:80"
    depends_on:
      - backend

volumes:
  pgdata:
```

Start everything with one command:

```bash
docker compose up
```

Stop and remove everything:

```bash
docker compose down
```

---

## Key Commands

**Start services in the background:**

```bash
docker compose up -d
```

**Start and rebuild images:**

```bash
docker compose up --build
```

**Stop services:**

```bash
docker compose stop
```

**Stop and remove containers, networks, and volumes:**

```bash
docker compose down -v
```

**View running services:**

```bash
docker compose ps
```

**View logs:**

```bash
docker compose logs
docker compose logs backend
docker compose logs -f backend
```

**Run a command inside a running service:**

```bash
docker compose exec backend bash
```

**Scale a service to multiple instances:**

```bash
docker compose up --scale backend=3
```

---

## What Docker Compose Manages

- **Containers** — creates and starts all defined services
- **Networks** — creates a shared network so services can reach each other by name
- **Volumes** — creates named volumes for data persistence
- **Environment** — injects environment variables into each container
- **Build** — builds custom images from Dockerfiles before starting

---

## Where Docker Compose is Used

- local development environments
- running integration tests with real services (database, cache, queue)
- CI/CD pipelines where the full stack needs to run temporarily
- simple production deployments for small applications

For large-scale production deployments with many replicas, load balancing, and rolling updates, tools like Kubernetes are used instead.
