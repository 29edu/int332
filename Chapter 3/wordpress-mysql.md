# Use Case: WordPress + MySQL

WordPress is a popular content management system that requires a MySQL database. This is one of the most commonly used Docker Compose examples and demonstrates a two-container setup.

---

## How It Works

```
Browser → [ WordPress :8080 ] → [ MySQL :3306 ]
```

- MySQL starts first and creates the `wordpress` database
- WordPress connects to MySQL using the credentials passed through environment variables
- Both services share a private Docker network
- Data is stored in named volumes so it survives container restarts

---

## Project Structure

```
wordpress/
  docker-compose.yml
  secrets/
    db_password.txt
    db_root_password.txt
```

---

## docker-compose.yml

```yaml
version: "3.9"

services:
  wordpress:
    image: wordpress:latest
    container_name: wordpress
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress_data:/var/www/html
    depends_on:
      db:
        condition: service_healthy
    networks:
      - wp-net

  db:
    image: mysql:8.0
    container_name: wordpress-db
    restart: always
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
      MYSQL_ROOT_PASSWORD: rootpass
    volumes:
      - db_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - wp-net

volumes:
  wordpress_data:
  db_data:

networks:
  wp-net:
```

---

## Running It

Start in the background:

```bash
docker compose up -d
```

Check that both containers are running:

```bash
docker compose ps
```

Access WordPress in a browser:

```
http://localhost:8080
```

Complete the WordPress setup wizard (site title, admin username, password, email).

---

## Stopping and Removing

Stop containers without losing data:

```bash
docker compose stop
```

Stop and remove containers but keep volumes (data preserved):

```bash
docker compose down
```

Stop and remove everything including volumes (all data deleted):

```bash
docker compose down -v
```

---

## What Each Part Does

**WordPress service:**
- uses the official `wordpress` image which includes PHP and Apache
- port `8080` on your machine maps to port `80` in the container
- `WORDPRESS_DB_HOST: db` tells WordPress to find the database at hostname `db` — Docker resolves this to the MySQL container's IP
- `wordpress_data` volume stores uploaded files, themes, and plugins

**MySQL service:**
- `MYSQL_DATABASE: wordpress` creates the database on first startup
- `MYSQL_USER` and `MYSQL_PASSWORD` create a user that WordPress uses to connect
- `db_data` volume stores all database files
- the health check ensures WordPress does not attempt to connect before MySQL is ready

---

## Using a .env File for Passwords

Instead of putting passwords directly in the Compose file:

`.env`:

```env
MYSQL_ROOT_PASSWORD=strongrootpass
MYSQL_USER=wpuser
MYSQL_PASSWORD=strongwppass
```

`docker-compose.yml`:

```yaml
environment:
  MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
  MYSQL_USER: ${MYSQL_USER}
  MYSQL_PASSWORD: ${MYSQL_PASSWORD}
```

Add `.env` to `.gitignore` so passwords are not committed to version control.
