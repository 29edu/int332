# YAML Structure

Docker Compose files are written in YAML. Understanding YAML syntax is necessary to write and read Compose files correctly.

---

## What is YAML

YAML stands for YAML Ain't Markup Language. It is a human-readable format used for configuration files. It is designed to be easy to read and write compared to JSON or XML.

---

## Basic Rules

**Indentation uses spaces, not tabs**

YAML is sensitive to indentation. Always use spaces. Mixing tabs and spaces causes errors.

```yaml
services:
  web:            # 2 spaces in
    image: nginx  # 4 spaces in
```

**Key-value pairs use a colon followed by a space**

```yaml
name: my-app
port: 8080
enabled: true
```

**Lists use a dash followed by a space**

```yaml
ports:
  - "80:80"
  - "443:443"

environment:
  - NODE_ENV=production
  - PORT=3000
```

**Nested structure uses indentation**

```yaml
services:
  app:
    image: node:18
    ports:
      - "3000:3000"
```

---

## Data Types

**String**

```yaml
name: my-app
message: "hello world"
path: /usr/src/app
```

Strings usually do not need quotes. Use quotes when the value contains special characters like `:` or `#`.

**Number**

```yaml
replicas: 3
timeout: 30
```

**Boolean**

```yaml
enabled: true
debug: false
```

**Null**

```yaml
value: null
missing:
```

**Multiline string**

Using `|` keeps newlines:

```yaml
command: |
  echo "starting"
  node server.js
```

Using `>` folds newlines into spaces:

```yaml
description: >
  This is a long description
  that spans multiple lines
  but becomes one line.
```

---

## Comments

Comments start with `#` and are ignored by the parser.

```yaml
# This is a comment
services:
  web:
    image: nginx  # using latest nginx
```

---

## Full docker-compose.yml Structure

A complete Compose file has these top-level keys:

```yaml
version: "3.9"       # Compose file format version

services:            # defines each container
  service-name:
    image: ...
    build: ...
    ports: ...
    environment: ...
    volumes: ...
    networks: ...
    depends_on: ...

volumes:             # declares named volumes
  volume-name:

networks:            # declares custom networks
  network-name:

secrets:             # declares secrets
  secret-name:

configs:             # declares config files
  config-name:
```

Each top-level key is optional except `services`, which is required.

---

## Common YAML Mistakes

**Wrong indentation:**

```yaml
# Wrong
services:
app:           # should be indented under services
  image: nginx

# Correct
services:
  app:
    image: nginx
```

**Missing space after colon:**

```yaml
# Wrong
image:nginx

# Correct
image: nginx
```

**Unquoted value with special characters:**

```yaml
# Wrong - colon in value breaks parsing
label: my-app:latest

# Correct
label: "my-app:latest"
```

**Tab instead of spaces:**
Most editors show a tab and spaces as the same width visually, but YAML will reject tabs. Configure your editor to use spaces.
