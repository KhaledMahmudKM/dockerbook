# Appendix A: Command Reference

A grouped quick-reference to the Docker commands used in this book. Each table collects commands of similar purpose. Placeholders: `IMAGE` an image reference, `CT` a container (name or ID), `VOL` a volume, `NET` a network.

> Newer Docker groups many commands under a noun (`docker container ls`, `docker image rm`). The short forms (`docker ps`, `docker rmi`) still work and are shown here where they are common.

---

## Running containers: the key `docker run` flags

```
docker run [FLAGS] IMAGE [COMMAND]
```

| Flag | Meaning |
|---|---|
| `-d`, `--detach` | Run in the background |
| `-it` | Interactive + TTY (a shell you type into) |
| `--name NAME` | Name the container |
| `-p HOST:CT` | Publish a port to the host (`-p 127.0.0.1:8080:80` = host-only) |
| `-e KEY=VAL` | Set an environment variable |
| `--env-file FILE` | Load environment variables from a file |
| `-v VOL:/path` | Mount a volume (or `HOST:/path` for a bind mount) |
| `--mount type=…,source=…,target=…` | Verbose mount form |
| `-w DIR` | Working directory inside the container |
| `--rm` | Remove the container when it exits |
| `--restart POLICY` | `no` · `on-failure` · `always` · `unless-stopped` |
| `--network NET` | Attach to a specific network |
| `--memory 512m` | Cap memory |
| `--cpus 1.5` | Cap CPU |
| `--entrypoint CMD` | Override the image's entrypoint |

---

## Container lifecycle

| Command | Action |
|---|---|
| `docker run IMAGE` | Create **and** start a container |
| `docker create IMAGE` | Create without starting |
| `docker start CT` | Start a stopped container |
| `docker stop CT` | Graceful stop (SIGTERM → SIGKILL) |
| `docker restart CT` | Stop then start |
| `docker pause CT` / `docker unpause CT` | Freeze / resume processes |
| `docker kill CT` | Immediate stop (SIGKILL) |
| `docker rm CT` | Remove a stopped container (`-f` forces) |

---

## Inspecting & interacting with containers

| Command | Shows / does |
|---|---|
| `docker ps` | Running containers |
| `docker ps -a` | All containers, including stopped |
| `docker logs CT` | Container output (`-f` to follow) |
| `docker exec -it CT sh` | Run a command / shell in a running container |
| `docker inspect CT` | Full JSON: config, networks, mounts, state |
| `docker stats` | Live CPU / memory / I/O per container |
| `docker top CT` | Processes running inside |
| `docker cp SRC CT:/path` | Copy files host ↔ container |
| `docker port CT` | Show published ports |
| `docker rename CT NEW` | Rename a container |

---

## Working with images

| Command | Action |
|---|---|
| `docker pull IMAGE` | Download an image from a registry |
| `docker images` / `docker image ls` | List local images |
| `docker build -t NAME:TAG .` | Build an image from a Dockerfile |
| `docker tag SRC NAME:TAG` | Add another name/tag to an image |
| `docker push NAME:TAG` | Upload an image to a registry |
| `docker rmi IMAGE` / `docker image rm IMAGE` | Remove an image |
| `docker history IMAGE` | Show the image's layers and sizes |
| `docker image inspect IMAGE` | Full image metadata |
| `docker search TERM` | Search Docker Hub |
| `docker save IMAGE -o file.tar` | Export an image to a tar archive |
| `docker load -i file.tar` | Import an image from a tar archive |

---

## Registry authentication

| Command | Action |
|---|---|
| `docker login [HOST]` | Authenticate to a registry (use an access token) |
| `docker logout [HOST]` | Remove stored credentials |

---

## Dockerfile instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Base image (first instruction) |
| `WORKDIR` | Set the working directory |
| `COPY` | Copy files from the build context |
| `ADD` | COPY + URL fetch / archive auto-extract |
| `RUN` | Run a command at **build** time (adds a layer) |
| `ENV` | Set an environment variable in the image |
| `ARG` | Build-time variable (`--build-arg`) |
| `EXPOSE` | Document a listening port (does not publish) |
| `CMD` | Default command at container **start** (overridable) |
| `ENTRYPOINT` | Fixed executable; `CMD` becomes its arguments |
| `USER` | Switch to a (non-root) user |
| `VOLUME` | Declare a mount point |
| `LABEL` | Add metadata |
| `HEALTHCHECK` | Command Docker uses to test the app is healthy |

---

## Volumes

| Command | Action |
|---|---|
| `docker volume create VOL` | Create a named volume |
| `docker volume ls` | List volumes |
| `docker volume inspect VOL` | Show a volume's details |
| `docker volume rm VOL` | Remove a volume |
| `docker volume prune` | Remove volumes not used by any container |

Mount forms: `-v VOL:/path` (volume) · `-v "$(pwd)":/path` (bind) · `--tmpfs /path` (memory).

---

## Networks

| Command | Action |
|---|---|
| `docker network create NET` | Create a (user-defined bridge) network |
| `docker network ls` | List networks |
| `docker network inspect NET` | Subnet, connected containers |
| `docker network connect NET CT` | Attach a container |
| `docker network disconnect NET CT` | Detach a container |
| `docker network rm NET` | Remove a network |
| `docker network prune` | Remove unused networks |

---

## Docker Compose

| Command | Action |
|---|---|
| `docker compose up -d` | Create and start the whole stack (detached) |
| `docker compose down` | Stop and remove containers + network |
| `docker compose down -v` | …and remove named volumes (**deletes data**) |
| `docker compose ps` | Services in this project |
| `docker compose logs -f [SVC]` | Follow logs (all services, or one) |
| `docker compose build` | Build services that use `build:` |
| `docker compose exec SVC sh` | Shell into a running service |
| `docker compose restart [SVC]` | Restart services |
| `docker compose pull` | Pull service images |

---

## System & cleanup

| Command | Action |
|---|---|
| `docker version` | Client and daemon versions |
| `docker info` | Daemon details (storage driver, cgroup version, counts) |
| `docker system df` | Disk used by images, containers, volumes, cache |
| `docker system prune` | Remove stopped containers, unused networks, dangling images, build cache |
| `docker system prune -a --volumes` | …plus all unused images **and volumes** (**deletes a lot**) |
| `docker container prune` | Remove stopped containers |
| `docker image prune [-a]` | Remove dangling [or all unused] images |
| `docker volume prune` | Remove unused volumes |
| `docker network prune` | Remove unused networks |
| `docker builder prune` | Remove build cache |
| `docker events` | Stream real-time daemon events |

> The `prune` family **deletes**. The plain `docker system prune` is safe for routine cleanup; `-a --volumes` removes images you will re-pull and volumes whose data you may want. Check `docker system df` and `docker volume ls` first.
