# Secrets and Configs

Docker provides two mechanisms — secrets and configs — for passing sensitive or configuration data into containers in a controlled and secure way.

---

## Secrets

Secrets are for sensitive data: passwords, API keys, TLS certificates, private keys. In Docker Swarm mode, secrets are stored encrypted in the cluster. In Docker Compose (non-swarm), secrets are mounted as files from the host filesystem.

**Why use secrets instead of environment variables:**

- environment variables can be leaked through logs, process listings, or child processes
- secrets are mounted as files inside the container, accessible only at `/run/secrets/<secret-name>`
- the secret value never appears in the container's environment

---

### Defining a Secret from a File

Create a file with the secret value:

```
secrets/
  db_password.txt    ← contains: mysecurepassword
```

`docker-compose.yml`:

```yaml
version: "3.9"

services:
  db:
    image: mysql:8
    secrets:
      - db_password
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

Inside the container, the secret is available at `/run/secrets/db_password` as a text file.

Many official Docker images support `_FILE` variants of their environment variables (like `MYSQL_ROOT_PASSWORD_FILE`) that read the value from a file instead of the environment.

---

### Defining a Secret Inline

For development only, you can define the secret value directly in the Compose file:

```yaml
secrets:
  db_password:
    environment: DB_PASSWORD
```

This reads the secret from the host environment variable `DB_PASSWORD`.

---

### Using Multiple Secrets

```yaml
services:
  app:
    build: ./app
    secrets:
      - db_password
      - jwt_secret
      - api_key

secrets:
  db_password:
    file: ./secrets/db_password.txt
  jwt_secret:
    file: ./secrets/jwt_secret.txt
  api_key:
    file: ./secrets/api_key.txt
```

Each secret is mounted at `/run/secrets/<name>` inside the container.

---

## Configs

Configs are for non-sensitive configuration data: config files, nginx configuration, application settings. Unlike secrets, configs are not encrypted, but they allow you to inject configuration files into containers without rebuilding the image.

---

### Injecting a Config File

Local `nginx.conf` file:

```nginx
server {
  listen 80;
  location / {
    proxy_pass http://backend:5000;
  }
}
```

`docker-compose.yml`:

```yaml
version: "3.9"

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    configs:
      - source: nginx_config
        target: /etc/nginx/conf.d/default.conf

configs:
  nginx_config:
    file: ./nginx.conf
```

The local `nginx.conf` is mounted at `/etc/nginx/conf.d/default.conf` inside the container. If you update the config and redeploy, the container gets the new config without rebuilding the image.

---

### Config with Custom Permissions

```yaml
configs:
  app_config:
    source: app_config
    target: /app/config.json
    mode: 0440
```

`mode` sets the file permission using octal notation.

---

## Secrets vs Configs vs Environment Variables

| | Environment Variables | Secrets | Configs |
|---|---|---|---|
| Use for | General config | Sensitive data | Config files |
| Stored as | Key-value in container env | File at `/run/secrets/` | File at target path |
| Encrypted | No | Yes (Swarm) | No |
| Visible in `docker inspect` | Yes | No | No |
| Best for | URLs, ports, feature flags | Passwords, keys, certs | nginx.conf, app configs |
