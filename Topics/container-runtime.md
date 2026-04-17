# Container Runtime

A container runtime is the software that actually runs containers on a system. It takes a container image and turns it into a running container.

In simple words, the image is the package and the runtime is what opens and runs that package.

---

## Why It Is Needed

A container image is only a read-only package. To use it, the system needs something that can:

- create an isolated environment
- apply limits on CPU and memory
- start the application process
- stop and manage the container

Without a runtime, a container image cannot run.

---

## Types of Container Runtime

**High-Level Runtime**
These are the tools users usually work with.
Examples: Docker Engine, containerd, CRI-O

They handle:

- pulling images
- networking
- storage
- starting and stopping containers

**Low-Level Runtime**
These directly work with the Linux kernel to run the container process.
Example: `runc`

They use features like:

- namespaces
- cgroups

---

## Simple Example

When you run:

```bash
docker run ubuntu
```

Docker uses a container runtime underneath to create and start the Ubuntu container.

---

## Real-Life Example

Think of a container image like a packed lunch box.
The container runtime is the person who opens it, serves the food, and handles everything needed to use it.
