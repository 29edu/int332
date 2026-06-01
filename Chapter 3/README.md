# Chapter 3: Microservices with Docker Compose

---

## Microservices Architecture

- [Microservices](microservices.md) — what microservices are, why they exist, how services communicate
- [Monolithic vs Microservices](monolithic-vs-microservices.md) — comparison, diagrams, when to use each
- [Advantages of Microservices](advantages-of-microservices.md) — scalability, isolation, agility, fault tolerance
- [API Gateway](api-gateway.md) — what it does, routing, auth, rate limiting, Nginx example

---

## Docker Compose

- [Docker Compose](docker-compose.md) — what it is, key commands, where it is used
- [YAML Structure](yaml-structure.md) — YAML syntax, data types, common mistakes
- [Writing docker-compose.yml](docker-compose-yml.md) — version, services, volumes, networks in detail
- [Environment Variables](environment-variables.md) — inline, .env file, env_file, variable precedence
- [Secrets and Configs](secrets-and-configs.md) — secrets for passwords, configs for config files
- [Build vs Image](build-vs-image.md) — when to use each, build args, using both together
- [Service Dependency Ordering](service-dependency.md) — depends_on, healthcheck, conditions

---

## Use Case Deployments

- [WordPress + MySQL](wordpress-mysql.md) — full Compose setup with health checks and volumes
- [Node.js + MongoDB](nodejs-mongodb.md) — custom Dockerfile, REST API, Mongo Express
- [Java Spring Boot + PostgreSQL](springboot-postgresql.md) — multi-stage build, JPA, pgAdmin
