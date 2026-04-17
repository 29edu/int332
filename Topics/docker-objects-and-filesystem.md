# Docker Objects and File System

Docker works with several core objects. Understanding what each one is makes everything else easier to follow.

---

## Docker Objects

**Image**
A read-only template used to create containers. It contains the OS, app code, and dependencies. Built from a Dockerfile.

**Container**
A running instance of an image. It is isolated, has its own file system, and runs its own process. When stopped, the container still exists but is not running. When deleted, it is gone.

**Volume**
A way to persist data outside of a container. If a container is deleted, its data is lost — unless it was stored in a volume. Volumes live on the host machine.

**Network**
Docker containers can communicate with each other through networks. By default, containers on the same network can reach each other by name.

**Dockerfile**
A text file with instructions that tell Docker how to build an image. Each instruction becomes a layer in the image.

---

## Docker File System

Docker uses a layered file system called the **Union File System (UnionFS)**. This is what makes images lightweight and fast.

Every image is made up of read-only layers stacked on top of each other. When you run a container, Docker adds one thin writable layer on top. This is called the **container layer**.

- Read from the bottom layers (image layers).
- Write only to the top layer (container layer).
- When the container is deleted, the writable layer is gone. The image layers stay.

**Copy-on-Write**
If a container needs to modify a file from the image layers, Docker copies that file into the writable layer first, then modifies it. The original in the image layer stays untouched.

---

## Why This Matters

Because layers are shared, if you have ten containers running from the same image, they all share the same image layers on disk. Only the writable layer is unique per container. This saves a huge amount of disk space.
