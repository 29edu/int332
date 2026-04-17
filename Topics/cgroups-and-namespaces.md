# cgroups and Namespaces

These are two Linux kernel features that Docker uses under the hood to make containers work. They are the foundation of container isolation.

---

## Namespaces

Namespaces make a container think it is running on its own separate system. Each container gets its own isolated view of the system.

- **PID Namespace** — the container has its own process IDs. Processes inside cannot see processes outside.
- **Network Namespace** — the container gets its own network interfaces, IP address, and ports.
- **Mount Namespace** — the container has its own file system view.
- **UTS Namespace** — the container can have its own hostname.
- **IPC Namespace** — isolates inter-process communication.
- **User Namespace** — maps user IDs inside the container to different IDs on the host.

In simple words: namespaces give each container the feeling that it owns the whole machine.

---

## cgroups (Control Groups)

cgroups control how much resources a container is allowed to use. While namespaces handle isolation, cgroups handle limits.

- **CPU** — limit how much CPU a container can use.
- **Memory** — set a max memory limit for a container.
- **Disk I/O** — control how much read/write a container can do.
- **Network** — limit bandwidth usage.

In simple words: cgroups make sure one container cannot eat up all the resources and starve the others.

---

## How They Work Together

Namespaces isolate what a container can see.
cgroups limit what a container can use.
Together they give you a secure, isolated, and resource-controlled environment, which is exactly what a container is.
