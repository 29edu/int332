# Virtualization

Virtualization is the process of creating a virtual version of something, like a computer, server, storage, or network. Instead of using separate physical hardware for everything, you can run multiple virtual machines on a single physical machine. Each virtual machine acts like its own separate computer.

---

## Types of Virtualization

**Hardware Virtualization**
Running multiple virtual machines on one physical machine. Each VM has its own OS and runs independently.
Example: Running Windows and Linux at the same time on one laptop using VMware.

**OS-level Virtualization**
Instead of full virtual machines, the OS runs isolated containers that share the same kernel.
Example: Docker containers running on a Linux host.

**Storage Virtualization**
Combining multiple physical storage devices into one virtual storage unit.
Example: RAID systems or cloud storage like AWS S3.

**Network Virtualization**
Creating virtual networks on top of physical network hardware.
Example: VLANs or virtual private networks (VPN).

**Desktop Virtualization**
Running a desktop environment remotely or virtually.
Example: Amazon WorkSpaces, Citrix.

---

## Examples in Real Life

- **VMware, VirtualBox** — let you run a full OS inside another OS.
- **Docker** — runs lightweight containers instead of full virtual machines.
- **AWS EC2** — gives you virtual servers in the cloud.
- **Hyper-V** — Microsoft's built-in virtualization tool on Windows.
