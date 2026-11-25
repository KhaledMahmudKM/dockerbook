# Troubleshooting and Cleanup

Things go wrong: a container exits immediately, a port is taken, a build fails, the disk fills up. This chapter is a practical field guide: the commands to diagnose common problems and the tools to reclaim space safely.

---

## First moves when something is wrong

Three commands answer most "why isn't this working?" questions:

```bash
docker ps -a           # is the container running, or did it exit? what status/code?
docker logs <name>     # what did it print before it stopped?
docker inspect <name>  # full config: ports, mounts, networks, env, exit code
```

Start here every time. `docker ps -a` shows the **status**: `Exited (1) 10 seconds ago` tells you it crashed; `Exited (0)` means it finished normally. `docker logs` almost always shows the actual error.

---

## A container exits immediately

Very common, and usually one of these:

- **The main process finished.** A container runs only as long as its main process. `docker run ubuntu` exits at once because `bash` has no terminal and nothing to do. Give it work (`docker run -it ubuntu bash`) or run a long-lived process.
- **The command or entrypoint is wrong.** A typo in `CMD`, or a binary that is not in the image. `docker logs` shows the "not found" error.
- **The app crashed on startup.** Misconfiguration, a missing environment variable, a database it cannot reach. `docker logs` shows the stack trace.

Check the exit code in `docker ps -a`, read `docker logs`, and if you need to poke around a *built* image interactively, override the entrypoint:

```bash
docker run -it --entrypoint sh myapp:1.0    # get a shell instead of the app
```

---

## "Port is already allocated"

```
Error: ... Bind for 0.0.0.0:8080 failed: port is already allocated
```

Something already uses that host port, often a container you forgot.

```bash
docker ps                         # find what's publishing 8080
# then either stop it, or publish on a different host port:
docker run -d -p 8081:80 nginx
```

---

## Image won't pull

```
Error response from daemon: pull access denied / manifest unknown
```

- **`manifest unknown`** → the **tag** does not exist. Check the spelling and that the version is real.
- **`pull access denied`** → the image is **private** (or misspelled). `docker login` with authorized credentials.
- **Timeouts** → network or registry reachability; check connectivity and any proxy.

---

## Debugging inside a container

```bash
docker exec -it <name> sh        # or bash, look around a running container
docker exec <name> env           # check environment variables
docker exec <name> cat /etc/hosts
docker logs -f <name>            # stream logs while you reproduce the issue
docker stats                     # live CPU/memory; is it being OOM-killed?
```

If a container is being killed for using too much memory, `docker inspect` shows `OOMKilled: true`. Raise the limit or fix the leak.

---

## Setting resource limits

To stop a container from starving the host (and to reproduce production limits), cap it, the cgroups from Chapter 2 at work:

```bash
docker run --memory=512m --cpus=1.5 myapp:1.0
```

`--memory` caps RAM (the kernel OOM-kills the container if it exceeds it); `--cpus` caps CPU. Useful both as a safety net and to test how an app behaves under constraint.

---

## Reclaiming disk space

Docker accumulates images, stopped containers, unused volumes, networks, and build cache. See what is using space, then prune deliberately:

```bash
docker system df          # overview: images, containers, volumes, build cache
```

| Command | Removes |
|---|---|
| `docker container prune` | Stopped containers |
| `docker image prune` | Dangling (untagged) images |
| `docker image prune -a` | All images not used by a container |
| `docker volume prune` | Volumes not used by any container |
| `docker network prune` | Unused networks |
| `docker builder prune` | Build cache |
| `docker system prune` | Stopped containers + unused networks + dangling images + build cache |
| `docker system prune -a --volumes` | The above **plus** all unused images **and volumes** |

> **Prune deletes, and the big ones delete a lot.** `docker system prune -a --volumes` will remove images you will have to re-pull or rebuild and **volumes whose data you may want**. Run `docker system df` and `docker volume ls` first, and never run the `--volumes` form casually on a machine holding data you care about.

---

## Diagnosing the daemon itself

If `docker` commands hang or fail to connect:

```bash
docker info                       # does the client reach the daemon?
sudo systemctl status docker      # is dockerd running? (Linux)
sudo journalctl -u docker         # daemon logs (Linux)
```

On Docker Desktop, restart Docker Desktop and check its dashboard. "Cannot connect to the Docker daemon" almost always means the daemon (or Desktop) is not running, or you need `sudo`/`docker`-group membership (Chapter 3).

---

## A troubleshooting flow

1. `docker ps -a`: running, or exited with what code?
2. `docker logs <name>`: what did it say?
3. `docker inspect <name>`: ports, mounts, env, `OOMKilled`?
4. `docker exec -it <name> sh`: look inside (if it is running).
5. Reproduce with changes: fix the Dockerfile/command/config and rebuild.
6. `docker system df` → targeted `prune` if space is the problem.

---

## Key Takeaways

- Start every diagnosis with `docker ps -a`, `docker logs`, and `docker inspect`.
- A container that exits immediately usually finished its process, has a wrong command, or crashed. `docker logs` tells you which.
- "Port already allocated" and image-pull errors have quick, specific fixes.
- Cap containers with `--memory` and `--cpus`; check `OOMKilled` when memory is suspect.
- Use `docker system df` then **targeted** `prune` commands. The `-a --volumes` forms delete images and data, so use them knowingly.

## Check your understanding

Choose the best answer for each question, then check your answers in [Answers to Chapter Questions](answers.md#troubleshooting-and-cleanup).

**1. Which commands answer most 'why isn't this working?' questions first?**

- A. docker ps -a, docker logs and docker inspect
- B. docker build and docker push
- C. docker pull and docker tag
- D. docker system prune

**2. What does the status Exited (0) mean?**

- A. The container crashed
- B. The container is paused
- C. The main process finished normally
- D. The image is missing

**3. Why does docker run ubuntu exit immediately?**

- A. The image is broken
- B. Its main process, bash, has no terminal and nothing to do, and a container runs only as long as its main process
- C. Ubuntu is not supported
- D. It needs sudo

**4. A pull fails with 'manifest unknown'. What is the likely cause?**

- A. The image is private
- B. The network is down
- C. Docker Hub is full
- D. The tag does not exist

**5. How can you tell a container was killed for using too much memory?**

- A. docker logs shows OOM
- B. docker inspect shows OOMKilled: true
- C. The image grows
- D. docker ps shows Paused

**6. What does --memory=256m do?**

- A. Reserves disk space
- B. Speeds up the container
- C. Sets the image size
- D. Caps the container's RAM through cgroups, and the kernel kills it if it exceeds the cap

**7. Which command shows what is using Docker's disk space before you prune?**

- A. docker system df
- B. docker ps
- C. docker images -q
- D. docker info

**8. Which prune command can delete volumes whose data you may want?**

- A. docker container prune
- B. docker image prune
- C. docker system prune -a --volumes
- D. docker builder prune
