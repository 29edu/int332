# Microservices

Microservices is an architectural style where an application is built as a collection of small, independent services. Each service handles one specific business function and runs in its own process.

---

## What is a Microservice

A microservice is a small, focused unit that does one thing well. It:

- handles one specific business function
- runs independently in its own process or container
- communicates with other services over HTTP, REST APIs, or message queues
- can be deployed, updated, and scaled on its own

Example: An e-commerce application broken into microservices might look like this:

```
[ User Service ]  →  handles login, registration, profiles
[ Product Service ]  →  handles product listings and search
[ Cart Service ]  →  manages shopping cart
[ Payment Service ]  →  processes payments
[ Notification Service ]  →  sends emails and SMS
```

Each of these is a separate service, deployed independently, possibly written in different languages.

---

## Need for Microservices

As applications grow, they become harder to manage as a single unit. The need for microservices comes from real problems:

**Growing complexity**
Large applications with many features become difficult to understand, change, and maintain as one codebase.

**Team scaling**
Multiple teams working on one codebase leads to conflicts. Microservices let different teams own and work on separate services without interfering with each other.

**Slow deployment cycles**
In a monolith, a small change to one feature requires redeploying the entire application. With microservices, you only redeploy the changed service.

**Uneven scaling needs**
Different parts of an app get different amounts of traffic. Microservices let you scale only the parts that need more resources.

**Mixed technology requirements**
Some features work better with certain tools. Microservices allow each service to choose the right language, framework, or database for its job.

**Fault isolation**
In a monolith, a bug in one feature can crash the whole application. With microservices, a failure is contained to one service.

---

## How Services Communicate

Services in a microservices system talk to each other in two main ways:

**Synchronous (HTTP / REST)**
One service makes a request and waits for a response.

```
Cart Service  →  HTTP request  →  Product Service
              ←  response      ←
```

**Asynchronous (Message Queue)**
One service sends a message and does not wait. The other service picks it up when ready.

```
Order Service  →  message  →  [ Queue ]  →  Notification Service
```

Common tools: RabbitMQ, Apache Kafka, Redis.

---

## Simple Example

A user places an order on an e-commerce site:

1. The frontend calls the **Order Service** via the API Gateway
2. The Order Service calls the **Product Service** to check stock
3. The Order Service calls the **Payment Service** to charge the user
4. The Order Service publishes an event to the message queue
5. The **Notification Service** picks up the event and sends a confirmation email

Each step is handled by a separate service. If the Notification Service is down, the order still completes — only the email is delayed.
