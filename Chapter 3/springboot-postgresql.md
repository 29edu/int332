# Use Case: Java Spring Boot + PostgreSQL

A Java Spring Boot REST API backed by a PostgreSQL database. Spring Boot connects to PostgreSQL using Spring Data JPA and Hibernate. The Spring Boot service is built from a custom Dockerfile while PostgreSQL uses the official image.

---

## How It Works

```
Client → [ Spring Boot :8080 ] → [ PostgreSQL :5432 ]
```

- PostgreSQL starts and passes a health check
- Spring Boot starts only after PostgreSQL is confirmed healthy
- Hibernate auto-creates tables based on JPA entity classes
- PostgreSQL data is stored in a named volume

---

## Project Structure

```
project/
  app/
    Dockerfile
    pom.xml
    src/
      main/
        java/com/example/demo/
          DemoApplication.java
          Item.java
          ItemRepository.java
          ItemController.java
        resources/
          application.properties
  docker-compose.yml
  .env
```

---

## app/Dockerfile

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

This uses a multi-stage build:
- the first stage (`build`) compiles the code using Maven
- the second stage (`runtime`) uses a lightweight JRE image and only copies the final JAR
- the result is a smaller image that does not include Maven or source code

---

## app/src/main/resources/application.properties

```properties
spring.datasource.url=${SPRING_DATASOURCE_URL}
spring.datasource.username=${SPRING_DATASOURCE_USERNAME}
spring.datasource.password=${SPRING_DATASOURCE_PASSWORD}
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
```

Environment variables are injected by Docker Compose at runtime, so the same JAR works in any environment.

---

## app/src/main/java/com/example/demo/Item.java

```java
package com.example.demo;

import jakarta.persistence.*;

@Entity
@Table(name = "items")
public class Item {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    public Item() {}
    public Item(String name) { this.name = name; }

    public Long getId() { return id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

---

## app/src/main/java/com/example/demo/ItemController.java

```java
package com.example.demo;

import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/items")
public class ItemController {

    private final ItemRepository repo;

    public ItemController(ItemRepository repo) {
        this.repo = repo;
    }

    @GetMapping
    public List<Item> getAll() {
        return repo.findAll();
    }

    @PostMapping
    public Item create(@RequestBody Item item) {
        return repo.save(item);
    }
}
```

---

## docker-compose.yml

```yaml
version: "3.9"

services:
  app:
    build: ./app
    container_name: spring-app
    restart: on-failure
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/appdb
      SPRING_DATASOURCE_USERNAME: appuser
      SPRING_DATASOURCE_PASSWORD: apppass
    depends_on:
      db:
        condition: service_healthy
    networks:
      - spring-net

  db:
    image: postgres:15
    container_name: postgres-db
    restart: always
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - spring-net

volumes:
  pgdata:

networks:
  spring-net:
```

---

## .env (optional)

```env
DB_USER=appuser
DB_PASSWORD=apppass
DB_NAME=appdb
```

Then reference in Compose:

```yaml
environment:
  POSTGRES_DB: ${DB_NAME}
  POSTGRES_USER: ${DB_USER}
  POSTGRES_PASSWORD: ${DB_PASSWORD}
```

---

## Running It

Build and start:

```bash
docker compose up --build
```

Spring Boot may take 20-30 seconds to start while it initializes and connects to PostgreSQL. Watch the logs:

```bash
docker compose logs -f app
```

When you see a line like:

```
Started DemoApplication in 3.5 seconds
```

the app is ready.

Test the API:

```bash
# Get all items
curl http://localhost:8080/items

# Create an item
curl -X POST http://localhost:8080/items \
  -H "Content-Type: application/json" \
  -d '{"name": "test item"}'
```

---

## How the JDBC URL Works

`jdbc:postgresql://db:5432/appdb`

- `db` is the service name in the Compose file — Docker DNS resolves it to the PostgreSQL container
- `5432` is the default PostgreSQL port
- `appdb` is the database name created by `POSTGRES_DB`

---

## Adding pgAdmin (Optional UI)

pgAdmin is a web-based PostgreSQL admin interface:

```yaml
  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin
    restart: always
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    depends_on:
      - db
    networks:
      - spring-net
```

Access at `http://localhost:5050`. Use `db` as the hostname when adding the PostgreSQL server in pgAdmin.
