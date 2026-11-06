# Glossary

Key terms used in this book and in the Docker world.

---

## B

**Base image**
The image named in a Dockerfile's `FROM` line, which later instructions build upon.

**Bind mount**
Storage that maps a specific host directory into a container; the container sees the real host files. Good for development.

**Bridge network**
The default Docker network type, a private network on a single host. A *user-defined* bridge also gives name-based DNS between containers.

**BuildKit**
Docker's modern build engine (default since Engine 23.0): parallel stages, better caching, smarter context handling.

---

## C

**cgroups (control groups)**
A Linux kernel feature that limits and accounts for a process group's resources (CPU, memory, I/O, process count). How Docker enforces `--memory` and `--cpus`.

**containerd**
The high-level container runtime Docker uses to manage image storage and container lifecycle; it calls `runc` to actually create containers.

**Container**
A running (or stopped) instance of an image, with its own writable layer, isolated by namespaces and limited by cgroups.

**Copy-on-write (CoW)**
Writing a file that lives in a read-only layer first copies it into the container's writable layer; the original layer is unchanged.

---

## D

**Daemon (dockerd)**
The background service that builds images and runs containers. The `docker` CLI sends it commands over an API.

**Digest**
A content hash (`sha256:…`) that identifies exact image bytes. Unlike a tag, a digest never moves.

**Distroless**
A minimal base image containing only an app and its runtime, no shell or package manager, reducing size and attack surface.

**Dockerfile**
A text recipe of instructions that `docker build` executes to produce an image.

**.dockerignore**
A file listing paths to exclude from the build context (like `.gitignore`).

---

## E

**ENTRYPOINT**
The fixed executable a container runs at start; `CMD` supplies its default arguments.

**EXPOSE**
A Dockerfile instruction that *documents* a listening port. It does not publish the port; only `-p` does.

---

## I

**Image**
A read-only template (a packaged, layered filesystem plus metadata) from which containers are created.

**Image reference**
An image's name: `registry/namespace/repo:tag` (e.g. `docker.io/library/nginx:latest`).

---

## L

**Layer**
One read-only slice of an image, created by a filesystem-changing build instruction. Images are stacks of layers; layers are shared and cached.

**LXC (Linux Containers)**
Early userspace tooling (c. 2008) that combined namespaces and cgroups. Docker originally ran on LXC before moving to libcontainer/runc.

**libcontainer / runc**
Docker's own container library (libcontainer), later donated and standardized as **runc**, the OCI reference runtime that sets up namespaces and cgroups.

**libvirt**
A toolkit and API for managing *machine* virtualization (KVM/QEMU and others), the VM world, contrasted with container process isolation.

---

## M

**Multi-stage build**
A Dockerfile with multiple `FROM` stages where the final image copies only the built artifact from an earlier build stage, leaving the toolchain behind.

---

## N

**Namespace**
A Linux kernel feature that gives a process its own isolated view of a resource. Container-relevant kinds: PID, NET, MNT, UTS, IPC, USER (and cgroup).

---

## O

**OCI (Open Container Initiative)**
The standards body defining the image and runtime formats, so images and runtimes work across tools (`runc` is the reference runtime).

**OverlayFS / overlay2**
The union filesystem (and Docker storage driver) that stacks read-only image layers under a writable layer and presents a merged view.

---

## P

**Publish (a port)**
Map a container port to a host port with `-p HOST:CONTAINER`, so clients outside Docker can reach the container.

**Prune**
Commands that delete unused Docker objects (`docker system prune`, `docker image prune`, …) to reclaim space.

---

## R

**Registry**
A server that stores and distributes images. Docker Hub is the default public registry; others include GHCR and cloud registries.

**Repository**
A named collection of related images (all the tags of `nginx`, say) in a registry.

---

## S

**Storage driver / snapshotter**
The component that assembles image layers into a container filesystem (`overlay2`, or a containerd snapshotter on recent Docker).

**Shim**
A small process that stays as a container's parent after `runc` exits, so the container survives daemon restarts.

---

## T

**Tag**
A human-readable label on an image within a repository (`nginx:1.27`). `latest` is just the default tag, not necessarily newest.

**tmpfs mount**
A mount held in RAM that disappears when the container stops; used for secrets or scratch data.

---

## U

**Union filesystem**
A filesystem that overlays multiple directories (layers) into one view, the basis of Docker's layered images.

---

## V

**Volume**
Docker-managed storage that persists beyond a container's life, the recommended way to keep application data.

---

## W

**Writable layer**
The thin, per-container layer added on top of an image's read-only layers; it holds everything the container writes and is discarded when the container is removed.
