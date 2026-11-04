# Installing Docker

How you install Docker depends on your operating system. On Windows and macOS you install **Docker Desktop**, which bundles everything (and quietly runs the Linux VM containers need). On Linux you install **Docker Engine** directly. This chapter covers both and verifies the result.

---

## Windows and macOS: Docker Desktop

Docker Desktop is the supported way to run Docker on Windows and macOS. It packages the Docker Engine, CLI, Compose, and a management GUI, and manages the lightweight Linux VM in the background.

1. Download Docker Desktop from **docs.docker.com** (or the Docker website) for your platform.
   - **Windows** uses the WSL 2 backend; enable WSL 2 first if prompted.
   - **macOS** has separate downloads for Apple Silicon and Intel chips.
2. Run the installer and start Docker Desktop.
3. Open a terminal (PowerShell, or any shell on macOS) and verify (below).

With Docker Desktop you run the same `docker` commands as everyone else; the VM is invisible in day-to-day use.

---

## Linux: Docker Engine

On Linux, install Docker Engine natively. The quickest method is Docker's convenience script, which detects your distribution:

```bash
curl -fsSL https://get.docker.com | sudo sh
```

For production or long-term use, Docker's **apt/yum repositories** are preferred, because upgrades flow through your normal package manager. The official install page lists the exact repository steps per distribution; the convenience script is ideal for labs and getting started.

Start and enable the service:

```bash
sudo systemctl enable --now docker
```

---

## Verify the installation

The traditional first test runs a tiny image that prints a message and exits:

```bash
docker run hello-world
```

If Docker is working, it pulls the `hello-world` image (if not already present) and prints a confirmation beginning "Hello from Docker!". You can also check versions:

```bash
docker version      # client and daemon versions
docker info         # daemon details: storage driver, cgroup version, etc.
```

`docker info` is worth a look now — it reports your **storage driver** (or containerd image store) and **cgroup version**, the very mechanisms from Chapter 2.

---

## Running Docker without sudo (Linux)

By default the Docker daemon runs as root and its socket is root-owned, so Linux commands need `sudo`. To run `docker` as your normal user, add yourself to the `docker` group:

```bash
sudo usermod -aG docker $USER
newgrp docker          # or log out and back in
docker run hello-world # now works without sudo
```

> **Security note.** The `docker` group grants root-equivalent access to the host — a user in it can start a container that mounts the whole filesystem. That is fine on a personal machine; on shared systems, be deliberate, or look into **rootless mode**, which runs the daemon as an unprivileged user.

This book shows `sudo` where Linux commonly needs it. On Docker Desktop you never need `sudo`.

---

## Scaffolding a project (optional)

Modern Docker includes `docker init`, which generates a starter `Dockerfile`, `.dockerignore`, and `compose.yaml` for a project by asking a few questions:

```bash
docker init
```

It is a convenient way to see idiomatic files for your language. We write these by hand in later chapters so you understand each line, but `docker init` is a good accelerator once you do.

---

## Upgrading and uninstalling

- **Docker Desktop** updates itself through its GUI.
- **Docker Engine** upgrades through your package manager (`apt`/`yum`), or by re-running the convenience script.
- To remove Engine on Debian/Ubuntu: `sudo apt remove docker-ce docker-ce-cli containerd.io`.

---

## Key Takeaways

- Windows and macOS use **Docker Desktop** (which runs a hidden Linux VM); Linux uses **Docker Engine** natively.
- Install Engine with the convenience script for labs, or the apt/yum repo for lasting setups.
- Verify with `docker run hello-world`; inspect the setup with `docker version` and `docker info`.
- On Linux, add your user to the `docker` group to skip `sudo` — understanding it grants root-equivalent access.
- `docker init` scaffolds starter Docker files for a project.
