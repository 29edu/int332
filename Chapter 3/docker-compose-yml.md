# Writing docker-compose.yml

The `docker-compose.yml` file defines the full application stack — every service, its configuration, volumes, and networks. This file is placed in the root of your project.

---

## version

Specifies the Compose file format version. Version `3.x` is the standard for modern Docker.

```yaml
version: "3.9"
```

In recent Docker Compose versions (v2+), the `version` field is optional, but it is still widely included for clarity and compatibility with older setups.

---

## services

Defines all the containers in your application. Each entry under `services` is one container.

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"

  app:
    build: ./app
    ports:
      - "3000:3000"
```

Each service has its own name, which is also its hostname on the Docker network. Other services can reach it using this name.

**Common service fields:**

| Field | Purpose |
|---|---|
| `image` | Pull a ready-made image |
| `build` | Build from a local Dockerfile |
| `ports` | Map host port to container port |
| `environment` | Set environment variables |
| `volumes` | Mount files or directories |
| `networks` | Connect to specific networks |
| `depends_on` | Define startup order |
| `restart` | Set restart policy |
| `container_name` | Give the container a custom name |
| `command` | Override the default start command |
| `expose` | Expose a port only within Docker network |

**restart policies:**

```yaml
restart: always           # always restart on failure or reboot
restart: on-failure       # only restart on non-zero exit
restart: unless-stopped   # restart unless manually stopped
restart: "no"             # never restart (default)
```

---

## volumes

Volumes store data outside the container so it is not lost when the container is removed or recreated.

**Declare a named volume:**

```yaml
volumes:
  dbdata:
  appfiles:
```

**Use it in a service:**

```yaml
services:
  db:
    image: postgres
    volumes:
      - dbdata:/var/lib/postgresql/data
```

**Bind mount (map a local folder):**

```yaml
services:
  app:
    image: node:18
    volumes:
      - ./src:/app/src
```

The left side is the host path, the right side is the path inside the container.

**Read-only mount:**

```yaml
volumes:
  - ./config:/app/config:ro
```

`:ro` makes the mount read-only inside the container.

---

## networks

By default, all services in a Compose file are placed on one auto-created network and can reach each other. Custom networks let you control which services can communicate.

**Declare custom networks:**

```yaml
networks:
  frontend:
  backend:
```

**Assign services to networks:**

```yaml
services:
  nginx:
    image: nginx
    networks:
      - frontend

  api:
    build: ./api
    networks:
      - frontend
      - backend

  db:
    image: postgres
    networks:
      - backend
```

In this setup:
- `nginx` can reach `api` via the `frontend` network
- `api` can reach `db` via the `backend` network
- `nginx` cannot reach `db` directly — they share no network

This is a common security pattern: the database is not exposed to the public-facing layer.

---

## Complete Example

```yaml
version: "3.9"

services:
  frontend:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - frontend
    depends_on:
      - backend

  backend:
    build: ./backend
    container_name: api
    restart: always
    ports:
      - "5000:5000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
      - DB_PORT=5432
    networks:
      - frontend
      - backend
    depends_on:
      - db

  db:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - backend

volumes:
  pgdata:

networks:
  frontend:
  backend:
```
