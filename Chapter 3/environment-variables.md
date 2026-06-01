# Environment Variables

Environment variables pass configuration values into containers at runtime. This keeps sensitive data and settings out of the image and makes the same image reusable across different environments.

---

## Why Use Environment Variables

Without environment variables, you would hardcode values like database passwords and hostnames directly in your application code or Dockerfile. This means:

- the same image cannot be used in development and production
- secrets end up in the codebase or image history
- changing a config value requires rebuilding the image

Environment variables solve this by injecting values at the time the container starts.

---

## Inline in docker-compose.yml

Pass variables directly in the service definition using a list:

```yaml
services:
  db:
    image: mysql:8
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=mydb
      - MYSQL_USER=appuser
      - MYSQL_PASSWORD=apppass
```

Or using a map (key: value) format:

```yaml
services:
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: mydb
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppass
```

Both formats work the same way. The map format is cleaner when you have many variables.

---

## Using a .env File

Create a `.env` file in the same directory as your `docker-compose.yml`:

```env
DB_PASSWORD=rootpass
DB_NAME=mydb
DB_USER=appuser
APP_PORT=3000
```

Reference these in `docker-compose.yml` using `${VARIABLE_NAME}`:

```yaml
services:
  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
    ports:
      - "${APP_PORT}:3000"
```

Docker Compose automatically reads `.env` from the same directory. You can also specify a different file:

```bash
docker compose --env-file .env.production up
```

---

## Using env_file in a Service

Load all variables from a file directly into the container:

```yaml
services:
  app:
    image: node:18
    env_file:
      - .env
      - .env.local
```

The difference from `.env`:
- `.env` is used by Compose itself for variable substitution in `docker-compose.yml`
- `env_file` loads variables directly into the container's environment

Both can be used together.

---

## Passing a Variable from the Host

If you omit the value, Compose passes the variable's value from the host shell:

```yaml
services:
  app:
    environment:
      - NODE_ENV
```

If `NODE_ENV=production` is set in your shell, the container gets `NODE_ENV=production`.

---

## Variable Precedence (Highest to Lowest)

1. Variables set with `docker compose run -e KEY=val`
2. Variables declared in the `environment` block of `docker-compose.yml`
3. Variables in the `env_file` file specified in the service
4. Variables in the `.env` file in the project directory
5. Variables set in the Dockerfile with `ENV`

---

## Security Notes

- Never put production passwords in `docker-compose.yml` if the file is committed to version control
- Add `.env` to your `.gitignore` file
- For production secrets, use Docker Secrets or a secrets manager (AWS Secrets Manager, HashiCorp Vault)
- Keep a `.env.example` file in the repo with all variable names but no real values, so teammates know what to fill in
