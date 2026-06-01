# Service Dependency Ordering

When starting multiple services, some must be ready before others. For example, a backend should not try to connect to a database before the database is running.

---

## depends_on

The `depends_on` key defines which services must start before the current service.

```yaml
services:
  app:
    build: ./app
    depends_on:
      - db

  db:
    image: postgres:15
```

With this config, Docker Compose starts `db` first and then starts `app`.

**Multiple dependencies:**

```yaml
services:
  app:
    build: ./app
    depends_on:
      - db
      - cache

  db:
    image: postgres:15

  cache:
    image: redis:7
```

Compose starts `db` and `cache` before starting `app`.

---

## The Problem with Basic depends_on

`depends_on` only waits for the container to start, not for the service inside it to be ready. A PostgreSQL container may be running but the database process may still be initializing. If `app` connects immediately, it will fail.

```
db container starts  →  postgres still initializing  →  app tries to connect  →  connection refused
```

---

## depends_on with condition

To wait until a service is actually ready, use `depends_on` with a `condition` and add a `healthcheck` to the dependency.

```yaml
services:
  app:
    build: ./app
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

`app` will not start until `db` passes its health check.

---

## healthcheck

The `healthcheck` block defines how Docker checks whether a service is ready.

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U postgres"]
  interval: 10s
  timeout: 5s
  retries: 5
  start_period: 30s
```

| Field | Meaning |
|---|---|
| `test` | Command to run. Exit 0 = healthy, non-zero = unhealthy |
| `interval` | How often to run the check |
| `timeout` | Max time to wait for the check to complete |
| `retries` | How many consecutive failures before marking unhealthy |
| `start_period` | Grace period before starting health checks (for slow startup) |

**Examples for common services:**

PostgreSQL:
```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U postgres"]
  interval: 10s
  retries: 5
```

MySQL:
```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 10s
  timeout: 5s
  retries: 5
```

Redis:
```yaml
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
  interval: 10s
  retries: 3
```

MongoDB:
```yaml
healthcheck:
  test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
  interval: 10s
  retries: 5
```

---

## Available Conditions

| Condition | Meaning |
|---|---|
| `service_started` | Default. Waits for the container to start |
| `service_healthy` | Waits for the health check to pass |
| `service_completed_successfully` | Waits for the container to exit with code 0 |

`service_completed_successfully` is useful for one-time setup containers like database migration runners:

```yaml
services:
  migrate:
    build: ./migrations
    command: npm run migrate
    depends_on:
      db:
        condition: service_healthy

  app:
    build: ./app
    depends_on:
      migrate:
        condition: service_completed_successfully
      db:
        condition: service_healthy
```

Here, `app` only starts after migrations have finished successfully.

---

## Startup Order Summary

```
db starts
  → healthcheck passes
    → migrate runs
      → migrate exits with code 0
        → app starts
```

This ensures a clean and predictable startup sequence even when services take varying amounts of time to become ready.
