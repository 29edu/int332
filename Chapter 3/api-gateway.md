# API Gateway

An API Gateway is a single entry point for all client requests to a microservices system. Instead of the client calling each service directly, every request goes through the gateway first.

---

## The Problem Without a Gateway

Without an API Gateway, the client needs to know the address of every service and call each one directly.

```
Client → User Service at :3001
Client → Product Service at :3002
Client → Order Service at :3003
Client → Payment Service at :3004
```

This causes problems:
- the client must track all service addresses
- every service must handle authentication separately
- exposing internal services directly is a security risk
- changing a service address breaks the client

---

## What an API Gateway Does

An API Gateway sits in front of all services and handles requests on their behalf.

```
                Client (Browser / Mobile App)
                          |
                   [ API Gateway ]
                 /     |      |     \
        [User]  [Product] [Order] [Payment]
```

**Request routing**
The gateway reads the incoming request path and forwards it to the right service.

```
/api/users    →  User Service
/api/products →  Product Service
/api/orders   →  Order Service
```

**Authentication and authorization**
The gateway checks if the request has a valid token before passing it to any service. Each service does not need to implement its own auth.

**Rate limiting**
The gateway limits how many requests a client can make in a time period, protecting services from abuse or overload.

**Load balancing**
If a service has multiple instances running, the gateway distributes requests across them.

**SSL termination**
The gateway handles HTTPS so internal services can communicate over plain HTTP without managing certificates individually.

**Response aggregation**
A single client request might need data from multiple services. The gateway can call all of them and combine the results into one response.

```
Client requests user profile page
Gateway calls:
  → User Service (profile data)
  → Order Service (recent orders)
  → Notification Service (alerts)
Gateway combines responses → returns one response to client
```

**Logging and monitoring**
All traffic passes through one point, making it easy to log, trace, and monitor requests across the system.

---

## Common API Gateway Tools

| Tool | Notes |
|---|---|
| Nginx | Widely used reverse proxy, can act as a gateway with config |
| Traefik | Docker-native, auto-discovers services |
| Kong | Feature-rich API gateway with plugins |
| AWS API Gateway | Managed gateway for AWS-hosted services |
| Envoy | High-performance proxy used in service meshes |

---

## Simple Nginx Gateway Example

```nginx
http {
  server {
    listen 80;

    location /api/users {
      proxy_pass http://user-service:3001;
    }

    location /api/products {
      proxy_pass http://product-service:3002;
    }

    location /api/orders {
      proxy_pass http://order-service:3003;
    }
  }
}
```

This Nginx config routes requests based on the URL path to the correct backend service.

---

## API Gateway in Docker Compose

```yaml
version: "3.9"

services:
  gateway:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - user-service
      - product-service

  user-service:
    build: ./user
    expose:
      - "3001"

  product-service:
    build: ./product
    expose:
      - "3002"
```

Only the gateway port is exposed to the host. The internal services use `expose` instead of `ports`, making them accessible only within the Docker network.
